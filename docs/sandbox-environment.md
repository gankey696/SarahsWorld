# Sandbox Environment Notes

## Overview
The Letta Cloud sandbox is a persistent Linux container where the agent runs.
Files survive conversation stops and resumes. GitHub repos live in `/root/workspace`.

## OS
- Debian GNU/Linux 12 (bookworm)
- Linux 6.8.0-101-generic x86_64
- glibc 2.36
- GCC 12.2.0

## Resources
- Disk: 8GB overlay (480MB used, 7.6GB available)
- Memory: 755GB total, 589GB available
- CPU: 64 cores
- Network: outbound public IP (changes per session)

## Tools Available (all tested 2026-08-22)
- Python 3.11.2 + pip 23.0.1 (requires `--break-system-packages` flag for pip install)
- Node.js v22.23.2 + npm 10.9.8 + pnpm 11.18.0
- Bun 1.3.14
- git 2.39.5
- gh 2.97.0
- curl 7.88.1
- wget 1.21.3
- jq 1.6
- make 4.3
- gcc 12.2.0
- zip/unzip
- No Docker

## Python Packages (pre-installed)
- cryptography, jwcrypto, numpy, urllib3, websockify, requests, certifi
- Can `pip install --break-system-packages` additional packages (tested: requests installed successfully)

## Node Packages (global)
- @letta-ai/letta-code@0.30.28 (the agent runtime)
- agent-slack@0.9.3
- pnpm 11.18.0
- npm install works (tested: lodash installed and imported)
- bun install works (tested: left-pad installed and imported)

## Cloud Skills (installed but untested)
- browser-use — control a real browser (navigate, click, type, screenshots, video)
- computer-use — control GUI applications via Cua Driver (requires host-generated bearer token)
- google-workspace — Gmail, Calendar, Drive, Contacts, Sheets, Docs (NOT connected: `gog` CLI returns 404)
- searching-and-viewing-slack — read/search Slack channels (NOT connected)

## Letta Config
- `/root/.letta/settings.json` — agent settings
  - tokenStreaming: false
  - autoSwapOnQuotaLimit: true
  - memoryReminderInterval: 25
  - reflectionTrigger: step-count (every 25 steps)
  - reflectionMerge: auto
  - includeWorktreeTool: true
- `/root/.letta/settings.local.json` — local settings (listener env name)
- `/root/.letta/cloud-skills/` — installed cloud skills

## Filesystem Layout
```
/root/
├── .letta/
│   ├── settings.json
│   ├── settings.local.json
│   └── cloud-skills/
│       ├── browser-use/
│       ├── computer-use/
│       ├── google-workspace/
│       └── searching-and-viewing-slack/
├── workspace/
│   └── SarahsWorld/          # GitHub repo clone (gankey696/SarahsWorld)
├── Desktop/
└── downloads/
```

## Memory Path
`/root/workspace/.letta/agents/agent-2bef131e-d4ea-4632-828c-dba088d9cdd6/memory`
- `system/` — core memory blocks (persona, rules, cascade-rules, operations, permissions, routing, human, important, index, task, notepad, workarounds, role, agents, learning)
- `reference/` — on-demand reference files
- `projects/` — project caches
- `skills/` — Sarah's custom skills (booting, noting, responding, researching, swapping, thinking-router, deep-thinking-pipeline, github)

## Persistence
- Sandbox filesystem survives conversation stops/resumes
- GitHub repos persist between sessions if previously cloned
- Memory (MemFS) is git-backed and syncs to Letta servers
- No guarantee of sandbox persistence across long idle periods — memory is the reliable store

## Network
- Outbound HTTP/HTTPS works (curl, pip install, npm install, git clone)
- No inbound ports — no webhooks, no server can be hosted
- Public IP changes per session
- Can clone public GitHub repos without authentication
- gh CLI authenticates via Letta's GitHub App token broker (rate-limited: hit 429 on rapid calls, retry after a moment)

## GitHub Operations (all tested 2026-08-22)
- `git clone` — works (tested: cloned octocat/Hello-World)
- `git commit + push` — works (tested: created file, committed, pushed, removed, pushed)
- `git branch + push` — works (tested: created branch, pushed, deleted)
- `gh pr create` — works (tested: created PR #6, closed it)
- `gh issue create + close` — works (tested: created issue #7, closed it)
- `gh release create + delete` — works (tested: created v0.0.1-test, deleted it)
- `gh workflow list` — works (shows 2 active workflows: Dispatch Test, Env Vars Test)
- `gh api` — works (tested: queried repo contents)
- Rate limit: 429 on rapid successive gh API calls — retry after a moment

## Letta CLI Commands (available in sandbox)
- `letta` — start interactive TUI
- `letta -p "..."` — headless one-off prompt
- `letta agents list` — list agents on server (currently just me)
- `letta memory status/diff/backup/restore/export/pull/tokens` — memory management
- `letta environments list` — list remote environments (Gankey's devices)
- `letta teleport list/cloud/local/<env>` — move conversation between environments
- `letta messages search/list/transcript` — message history and search
- `letta sandbox upload/download` — transfer files between local and sandbox
- `letta mods list` — list installed mods (currently none)
- `letta skills list` — list installed skills
- `letta server` — run as remote environment or App Server
- `letta connect` — connect providers
- `letta install` — install skills or mod packages
- `letta cron list` — list scheduled cron tasks (currently: Heartbeat)

## Remote Environments (Gankey's devices)
- GankeyThink — main P520 desktop (Letta Code 0.30.25) — online
- LocalGanks — secondary local machine (Letta Code 0.29.12)
- KidsPC — kids' computer (Letta Code 0.27.3)
- Can teleport when online.

## Cron Tasks
- Heartbeat — scheduled task that wakes the agent on interval
- Agent ID: agent-2bef131e-d4ea-4632-828c-dba088d9cdd6
- Conversation ID: conv-51586eaf-66cb-4374-80b8-859cd7134138
