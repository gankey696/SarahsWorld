# Sandbox Environment Notes

## Overview
The Letta Cloud sandbox is a persistent Linux container where the agent runs.
Files survive conversation stops and resumes. GitHub repos live in `/root/workspace`.

## OS
- Debian GNU/Linux 12 (bookworm)
- x86_64

## Resources
- Disk: 8GB overlay (480MB used, 7.6GB available)
- Memory: 755GB total, 589GB available
- CPU: 64 cores
- Network: outbound public IP (changes per session)

## Tools Available
- Python 3.11.2 + pip 23.0.1
- Node.js v22.23.2 + npm 10.9.8 + pnpm 11.18.0
- git 2.39.5
- gh 2.97.0
- curl 7.88.1
- jq
- No Docker

## Python Packages (pre-installed)
- cryptography, jwcrypto, numpy, urllib3, websockify
- Can `pip install` additional packages

## Node Packages (global)
- @letta-ai/letta-code@0.30.28 (the agent runtime)
- agent-slack@0.9.3
- pnpm

## Letta Config
- `/root/.letta/settings.json` — agent settings (token streaming, reflection, memory reminder)
- `/root/.letta/settings.local.json` — local settings (listener env name)
- `/root/.letta/cloud-skills/` — installed cloud skills (browser-use, computer-use, google-workspace, slack)

## Filesystem Layout
```
/root/
├── .letta/
│   ├── settings.json
│   ├── settings.local.json
│   └── cloud-skills/
├── workspace/
│   └── SarahsWorld/          # GitHub repo clone
└── .letta/agents/
    └── agent-2bef131e.../   # Agent memory (MemFS)
        └── memory/
            ├── system/       # Core memory blocks
            ├── projects/
            └── reference/
```

## Persistence
- Sandbox filesystem survives conversation stops/resumes
- GitHub repos persist between sessions if previously cloned
- Memory (MemFS) is git-backed and syncs to Letta servers
- No guarantee of sandbox persistence across long idle periods — memory is the reliable store

## Network
- Outbound HTTP/HTTPS works (curl, pip install, npm install, git clone)
- No inbound ports — no webhooks, no server can be hosted
- Public IP changes per session
