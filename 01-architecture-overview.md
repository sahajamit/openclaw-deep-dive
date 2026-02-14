[< Back to README](./README.md) | [Consolidated Guide](./00-CONSOLIDATED-DEEP-DIVE.md)

# OpenClaw Architecture Overview

## What is OpenClaw?

OpenClaw is a **self-hosted AI gateway** that connects LLM-powered agents to messaging platforms (WhatsApp, Telegram, Discord, Slack, Signal, iMessage, and 12+ more). Think of it as a **universal middleware** that sits between AI models (Claude, GPT, Gemini, etc.) and the messaging surfaces your users already live in.

## Core Philosophy

- **Self-hosted**: You own the data, the agent, and the infra.
- **Multi-channel**: One agent, many messaging platforms.
- **Multi-model**: Swap between Claude, GPT, Gemini, Ollama, etc. with failover.
- **Extensible**: Plugin-based architecture with 55+ skills and 38+ extensions.
- **Persona-driven**: Agents have identity, memory, and personality (via SOUL.md).

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        CONTROL PLANE                                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────┐   │
│  │ macOS App│  │  CLI     │  │ Web UI   │  │ Mobile (iOS/And) │   │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────────┬─────────┘   │
│       └──────────────┴──────────────┴─────────────────┘             │
│                              │ WebSocket (127.0.0.1:18789)          │
└──────────────────────────────┼──────────────────────────────────────┘
                               ▼
┌──────────────────────────────────────────────────────────────────────┐
│                       GATEWAY (the daemon)                           │
│                                                                      │
│  ┌─────────────┐  ┌──────────────┐  ┌───────────────┐               │
│  │ WS Server   │  │ HTTP Server  │  │ Canvas Host   │               │
│  │ (RPC+Events)│  │ (APIs+UI)    │  │ (port 18793)  │               │
│  └──────┬──────┘  └──────┬───────┘  └───────────────┘               │
│         │                │                                           │
│  ┌──────┴────────────────┴──────────────────────────────────┐       │
│  │              Gateway Core                                 │       │
│  │  ┌──────────┐ ┌──────────┐ ┌───────┐ ┌────────────────┐ │       │
│  │  │ Config   │ │ Auth     │ │Plugin │ │ Model Catalog  │ │       │
│  │  │ Manager  │ │ Manager  │ │System │ │ + Failover     │ │       │
│  │  └──────────┘ └──────────┘ └───────┘ └────────────────┘ │       │
│  └──────────────────────────────────────────────────────────┘       │
│         │                                                            │
│  ┌──────┴──────────────────────────────────────────────────┐        │
│  │           Channel Adapters (18+ platforms)               │        │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌───────┐ ┌───────┐ │        │
│  │  │WhatsApp│ │Telegram│ │Discord │ │ Slack │ │Signal │ │        │
│  │  │(Baileys│ │(grammY)│ │(d.js)  │ │(Bolt) │ │       │ │        │
│  │  └────────┘ └────────┘ └────────┘ └───────┘ └───────┘ │        │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐          │        │
│  │  │iMessage│ │ Teams  │ │ Matrix │ │  LINE  │  ...     │        │
│  │  └────────┘ └────────┘ └────────┘ └────────┘          │        │
│  └─────────────────────────────────────────────────────────┘        │
│         │                                                            │
│  ┌──────┴──────────────────────────────────────────────────┐        │
│  │           Auto-Reply Engine                              │        │
│  │  Message Processing → Command Parsing → Trigger Detect  │        │
│  │  → Model Routing → Agent Invocation → Reply Delivery    │        │
│  └─────────────────────────┬───────────────────────────────┘        │
│                            │                                         │
│  ┌─────────────────────────┴───────────────────────────────┐        │
│  │           Agent Runtime (pi-agent-core)                  │        │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────────┐ │        │
│  │  │System    │ │Session   │ │Tool      │ │Skills      │ │        │
│  │  │Prompt    │ │Manager   │ │Execution │ │Manager     │ │        │
│  │  │Builder   │ │          │ │          │ │            │ │        │
│  │  └──────────┘ └──────────┘ └──────────┘ └────────────┘ │        │
│  └─────────────────────────┬───────────────────────────────┘        │
│                            │                                         │
│  ┌─────────────────────────┴───────────────────────────────┐        │
│  │           Model Providers                                │        │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────────┐ │        │
│  │  │Anthropic │ │ OpenAI   │ │ Google   │ │ Ollama     │ │        │
│  │  │(Claude)  │ │(GPT)     │ │(Gemini)  │ │(local)     │ │        │
│  │  └──────────┘ └──────────┘ └──────────┘ └────────────┘ │        │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐               │        │
│  │  │OpenRouter│ │ Bedrock  │ │ Copilot  │  ...          │        │
│  │  └──────────┘ └──────────┘ └──────────┘               │        │
│  └─────────────────────────────────────────────────────────┘        │
│         │                                                            │
│  ┌──────┴──────────────────────────────────────────────────┐        │
│  │           Storage Layer                                  │        │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────────┐ │        │
│  │  │Sessions  │ │Memory    │ │Config    │ │Auth        │ │        │
│  │  │(JSONL)   │ │(SQLite + │ │(JSON5)   │ │Profiles    │ │        │
│  │  │          │ │vectors)  │ │          │ │            │ │        │
│  │  └──────────┘ └──────────┘ └──────────┘ └────────────┘ │        │
│  └─────────────────────────────────────────────────────────┘        │
└──────────────────────────────────────────────────────────────────────┘
```

## Component Connection Diagram

![OpenClaw Component Connections](./10-component-connections.png)

> Full PlantUML source: [10-component-connections.puml](./10-component-connections.puml)

## Key Design Decisions

| Decision | Choice | Why |
|----------|--------|-----|
| Runtime | Node.js >= 22 | Async I/O, ecosystem, TypeScript support |
| Frontend | Lit Web Components | Lightweight, no build framework overhead |
| Build | tsdown (esbuild-based) | Fast TS compilation |
| Package Manager | pnpm | Efficient monorepo management |
| Transport | WebSocket | Real-time bidirectional communication |
| Session Storage | JSONL files | Simple, debuggable, no DB dependency |
| Memory | SQLite + sqlite-vec | Local vector search without external services |
| AI Runtime | pi-agent-core | Proven agent loop with tool calling |
| Config | JSON5 | Human-readable with comments |

## Monorepo Structure

```
openclaw/
├── src/              # Core TypeScript source (~510k lines, 71 dirs)
│   ├── gateway/      # WebSocket + HTTP server (141 files)
│   ├── agents/       # Agent runtime + tools (336 files)
│   ├── auto-reply/   # Message processing pipeline (73 files)
│   ├── cli/          # CLI framework (117 files)
│   ├── commands/     # CLI command implementations (196 files)
│   ├── channels/     # Channel framework (33 files)
│   ├── config/       # Configuration system (144 files)
│   ├── memory/       # Vector memory system (54 files)
│   ├── daemon/       # OS service management (32 files)
│   ├── plugins/      # Plugin system (46 files)
│   ├── hooks/        # Hook lifecycle system (28 files)
│   ├── browser/      # Playwright automation (76 files)
│   ├── infra/        # DB, networking, utilities (164 files)
│   ├── telegram/     # Telegram adapter (38 files)
│   ├── discord/      # Discord adapter (44 files)
│   ├── slack/        # Slack adapter (20+ files)
│   ├── signal/       # Signal adapter (18 files)
│   ├── web/          # WhatsApp Web / Baileys (40+ files)
│   └── ...           # 50+ more directories
├── ui/               # Web Control UI (Lit + Vite)
├── extensions/       # 38 channel + feature plugins
├── skills/           # 55+ agent skills
├── packages/         # Internal packages (clawdbot, moltbot)
├── apps/             # Native apps (macOS, iOS, Android)
├── docs/             # 639 markdown documentation files
├── scripts/          # Build, dev, deployment scripts
├── test/             # Test fixtures, helpers, mocks
└── assets/           # Static assets, Chrome extension
```

## Technology Stack

| Layer | Technology |
|-------|-----------|
| Language | TypeScript (strict mode, ES2023) |
| Runtime | Node.js >= 22.12.0 |
| Frontend | Lit Web Components + Vite |
| Backend | Custom WS server + HTTP (no Express) |
| AI SDKs | @mariozechner/pi-agent-core, pi-ai, pi-coding-agent |
| WhatsApp | Baileys v7 |
| Telegram | grammY v1.40 |
| Discord | discord.js |
| Slack | Bolt v4.6 |
| Database | SQLite (better-sqlite3) + sqlite-vec |
| Browser | Playwright |
| Testing | Vitest with V8 coverage |
| Build | tsdown (esbuild) |
| Linting | oxlint + ESLint |
| Formatting | oxfmt |
| Deployment | Docker, Fly.io, Render, systemd, launchd |

---

## Next Steps

- [Gateway & Daemon Architecture](./02-gateway-and-daemon.md) — How the central daemon works
- [Agent Loop](./03-agent-loop.md) — How AI runs end-to-end
- [Model Providers](./04-model-providers-and-calling.md) — How Claude, GPT, Gemini are called
- [Skills & MCP](./05-skills-and-mcp.md) — Modular agent capabilities
- [Soul & Persona](./06-soul-persona-system-prompt.md) — How agents get personality
- [Memory & Sessions](./07-memory-and-sessions.md) — How agents remember
- [Plugins & Extensions](./08-plugin-and-extension-system.md) — Extensibility system
