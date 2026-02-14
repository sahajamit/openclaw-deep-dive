[< Back to README](./README.md) | [Prev: Memory & Sessions](./07-memory-and-sessions.md) | [Consolidated Guide](./00-CONSOLIDATED-DEEP-DIVE.md)

# Plugin & Extension System

## Architecture Overview

OpenClaw has a layered extensibility system with **plugins** (internal hook system), **extensions** (channel + feature packages), and **skills** (agent capabilities).

```
┌─────────────────────────────────────────────────┐
│              Extensibility Layers                │
│                                                   │
│  Skills (55+)                                    │
│  └── Agent-facing capabilities                   │
│      Loaded on-demand via SKILL.md               │
│                                                   │
│  Extensions (38)                                 │
│  └── Channel adapters + feature modules          │
│      Loaded at gateway startup                   │
│                                                   │
│  Plugins (internal hooks)                        │
│  └── Lifecycle event interception                │
│      40+ hook types for deep customization       │
│                                                   │
│  Gateway Hooks (command hooks)                   │
│  └── Shell scripts triggered by events           │
│      Pre/post command execution                  │
└─────────────────────────────────────────────────┘
```

## Plugin System

### Hook Types (40+)

The plugin system exposes hooks at every stage of the agent and gateway lifecycle:

**Agent Lifecycle Hooks**
| Hook | Phase | Purpose |
|------|-------|---------|
| `before_agent_start` | Pre-run | Inject context, override prompts |
| `agent_end` | Post-run | Inspect results, metadata |
| `before_tool_call` | Pre-tool | Intercept, modify, block tools |
| `after_tool_call` | Post-tool | Transform results |
| `tool_result_persist` | Pre-persist | Sanitize before writing |
| `before_compaction` | Pre-compact | Observe, annotate |
| `after_compaction` | Post-compact | React to compacted context |

**Message Lifecycle Hooks**
| Hook | Phase | Purpose |
|------|-------|---------|
| `message_received` | Inbound | Pre-process, filter |
| `message_sending` | Pre-send | Modify outgoing |
| `message_sent` | Post-send | Log, trigger followups |

**Session Lifecycle Hooks**
| Hook | Phase | Purpose |
|------|-------|---------|
| `session_start` | Begin | Initialize state |
| `session_end` | End | Cleanup, persist |

**Gateway Lifecycle Hooks**
| Hook | Phase | Purpose |
|------|-------|---------|
| `gateway_start` | Boot | Initialize services |
| `gateway_stop` | Shutdown | Cleanup |

### Gateway Hooks (Command Hooks)

Event-driven shell scripts for command interception:

```
agent:bootstrap    →  Runs while building bootstrap files
/new, /reset       →  Session reset commands
/stop              →  Agent stop command
```

### Bundled Hook: bootstrap-extra-files

```json5
{
  hooks: {
    "bootstrap-extra-files": {
      patterns: ["context/*.md", "docs/api-reference.md"]
    }
  }
}
```

Loads additional files into the system prompt via glob patterns.

## Extensions

Extensions are **self-contained packages** that add channels or features:

```
extensions/
├── Channel Extensions
│   ├── discord/        # Discord adapter
│   ├── slack/          # Slack adapter
│   ├── telegram/       # Telegram adapter
│   ├── signal/         # Signal adapter
│   ├── imessage/       # iMessage adapter
│   ├── msteams/        # Microsoft Teams
│   ├── googlechat/     # Google Chat
│   ├── matrix/         # Matrix/Element
│   ├── mattermost/     # Mattermost
│   ├── line/           # LINE
│   ├── twitch/         # Twitch chat
│   ├── irc/            # IRC
│   ├── nostr/          # Nostr
│   ├── tlon/           # Tlon/Urbit
│   ├── zalo/           # Zalo
│   └── bluebubbles/    # BlueBubbles
│
├── Feature Extensions
│   ├── memory-core/       # Memory system core
│   ├── memory-lancedb/    # LanceDB vector store
│   ├── talk-voice/        # Voice messaging
│   ├── llm-task/          # LLM task execution
│   ├── device-pair/       # Device pairing
│   ├── voice-call/        # Voice calling
│   ├── lobster/           # Lobster framework
│   ├── open-prose/        # Prose workflow
│   └── thread-ownership/  # Thread management
│
├── Auth Extensions
│   ├── google-antigravity-auth/
│   ├── google-gemini-cli-auth/
│   ├── minimax-portal-auth/
│   ├── qwen-portal-auth/
│   └── copilot-proxy/     # GitHub Copilot proxy
│
└── Diagnostics
    └── diagnostics-otel/   # OpenTelemetry
```

### Extension Loading

Extensions are loaded at gateway startup based on configuration:

```json5
{
  plugins: {
    slots: {
      memory: "memory-core",     // or "memory-lancedb", or "none"
      diagnostics: "diagnostics-otel"
    },
    autoEnable: true  // Auto-enable matching extensions
  }
}
```

### Plugin SDK

OpenClaw provides a **Plugin SDK** for building custom extensions:

```
src/plugin-sdk/
└── index.ts        # Public API for extension developers
```

Published as `dist/plugin-sdk/index.js` with TypeScript declarations.

## Channel Adapter Pattern

Each channel adapter follows a common pattern:

```typescript
// Simplified channel adapter structure
interface ChannelAdapter {
  // Lifecycle
  start(): Promise<void>;
  stop(): Promise<void>;

  // Inbound
  onMessage(handler: MessageHandler): void;

  // Outbound
  send(target: string, message: OutboundMessage): Promise<void>;

  // Status
  getStatus(): ChannelStatus;
}
```

### Channel Libraries

| Channel | Library | Version |
|---------|---------|---------|
| WhatsApp | Baileys | v7.0.0-rc.9 |
| Telegram | grammY | v1.40.0 |
| Slack | Bolt | v4.6.0 |
| Discord | discord.js | Latest |
| Signal | signal-cli | System binary |
| iMessage | BlueBubbles | API |
