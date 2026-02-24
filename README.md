# MeshVibe

Composable AI-powered tools that work together. Build your own personal ops ecosystem — each tool is a standalone CLI that plugs into a shared infrastructure of secrets, scheduling, notifications, and discovery.

Built for tinkerers. Every piece works alone, but they're better together.

## Architecture

MeshVibe is conventions, not a framework. Every tool follows the same shape:

```
~/IdeaProjects/ProjectName/   # source code
~/projectname/                 # runtime data, reports, state
```

Shared infrastructure:
- **Vault** — secrets via macOS Keychain
- **Heartbeat** — scheduled task runner
- **Notify** — unified alerting (planned)
- **Registry** — service catalog and discovery (planned)

## Projects

### Infrastructure

| Project | Description | CLI | Status |
|---------|-------------|-----|--------|
| [vault](https://github.com/MeshVibe/vault) | macOS Keychain secret manager | `vault` | Stable |
| [heartbeat](https://github.com/MeshVibe/heartbeat) | Scheduled task runner for Claude Code | `heartbeat` | Stable |
| [registry](https://github.com/MeshVibe/registry) | Service catalog and health checks | `registry` | Planned |
| [notify](https://github.com/MeshVibe/notify) | Unified notification routing | `notify` | Planned |

### Bots

| Project | Description | CLI | Data Dir | Status |
|---------|-------------|-----|----------|--------|
| [newsbot](https://github.com/MeshVibe/newsbot) | Personal news aggregation from RSS + browser history | `newsbot` | `~/newsbot/` | Stable |
| [securitybot](https://github.com/MeshVibe/securitybot) | Autonomous security scanning and threat detection | `securitybot` | `~/securitybot/` | Planned |
| [costbot](https://github.com/MeshVibe/costbot) | API spend monitoring and anomaly detection | `costbot` | `~/costbot/` | Planned |

### Interfaces

| Project | Description | CLI | Status |
|---------|-------------|-----|--------|
| [uterm](https://github.com/MeshVibe/uterm) | Terminal with Claude co-pilot | `uterm` | Stable |
| [voiceclaude](https://github.com/MeshVibe/voiceclaude) | Voice interface for Claude | `voiceclaude` | Stable |
| [remote-companion](https://github.com/MeshVibe/remote-companion) | Mobile app for the ecosystem | — | In Progress |

## Conventions

All projects follow these rules. AI agents building new tools should match them exactly.

### Stack
- TypeScript strict, ESM (`"type": "module"`)
- `tsconfig.json`: target ES2022, module Node16, moduleResolution Node16
- `src/` source, `dist/` output, `tests/` for tests
- npm (not yarn or bun), vitest for testing, commander for CLIs
- Node.js >= 20

### Package structure
```
project-name/
├── manifest.md          # project manifest (see below)
├── package.json         # bin field for CLI, "type": "module"
├── tsconfig.json
├── tsconfig.build.json  # extends tsconfig, excludes tests
├── LICENSE              # MIT
├── README.md
├── src/
│   ├── index.ts         # library exports
│   ├── cli.ts           # CLI entry point (#!/usr/bin/env node)
│   └── templates/
│       └── skill.md.ts  # Claude Code skill installer
└── tests/
    └── *.test.ts
```

### Scripts
```json
{
  "build": "tsc -p tsconfig.build.json",
  "dev": "tsc -p tsconfig.build.json --watch",
  "test": "vitest run",
  "lint": "tsc --noEmit"
}
```

### Secrets
Use Vault. Never hardcode keys. In Heartbeat tasks use the `vault://` prefix:
```yaml
env:
  API_KEY: "vault://my-api-key"
```

### Scheduling
Add a Heartbeat task file in `~/.heartbeat/`:
```yaml
---
schedule: daily at 05:00
timeout: 30m
dir: ~
enabled: true
env:
  ANTHROPIC_API_KEY: "vault://anthropic-api-key"
---

Your prompt here.
```

### Claude Code Skills
Each project installs a skill via `<cli> init` to `~/.claude/skills/<name>/SKILL.md`. Skills use YAML frontmatter with `name` and `description` fields.

## Manifest Format

Every project has a `manifest.md` in its root. This is how the Registry discovers and understands services.

```yaml
---
name: project-name
description: One-line description
cli: command-name
data_dir: ~/datadir
version: 0.1.0
reports:
  - name: Report Name
    path: ~/datadir/report.html
health_check: command-name status
depends_on:
  - vault
notify_on:
  - event: task_complete
    priority: low
  - event: task_failed
    priority: high
---

Longer description of what this project does and how it works.
```

### Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | yes | Project identifier, lowercase, matches CLI name |
| `description` | yes | One-line summary |
| `cli` | yes | CLI command name (after npm link) |
| `data_dir` | no | Runtime data directory |
| `version` | yes | Semver |
| `reports` | no | HTML reports this project generates |
| `health_check` | no | Command to verify the service is healthy |
| `depends_on` | no | Other MeshVibe projects this depends on |
| `notify_on` | no | Events that should trigger notifications |

## Adding a New Bot

1. Create the project following the package structure above
2. Add a `manifest.md` to the project root
3. Build and link: `npm install && npm run build && npm link`
4. Install the skill: `<cli> init`
5. Add a Heartbeat task if it runs on a schedule
6. Push to the MeshVibe org

The Registry will discover it automatically from the manifest.

## Quick Reference

```bash
# Secrets
vault set <key> <value>
vault get <key>
vault list

# Scheduling
heartbeat status
heartbeat list
heartbeat run <task>
heartbeat history

# News
newsbot scan
newsbot status

# Terminal
uterm

# Voice
voiceclaude
```

## License

MIT — all projects.
