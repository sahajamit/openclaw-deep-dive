# OpenClaw

## Core Architecture
### Gateway (Daemon)
- WebSocket Server (port 18789)
  - RPC Methods (46+)
  - Event Broadcasting
  - Device Auth + Pairing
- HTTP Server
  - Web Control UI
  - OpenAI-compatible API
  - Canvas Host (port 18793)
- Config Reloader
- Model Catalog
- Plugin System

### Agent Runtime
- Pi-Agent-Core SDK
  - createAgentSession()
  - streamSimple()
  - Tool execution loop
- System Prompt Builder
  - Bootstrap file injection
  - Skills listing
  - Safety guardrails
- Session Manager
  - JSONL transcripts
  - Session keys
  - Compaction
- Tool System
  - read/write/edit
  - exec (bash)
  - browser (Playwright)
  - messaging tools
  - memory tools

### Auto-Reply Engine
- Message Processing Pipeline
- Command Parsing (/new, /model, /stop)
- Trigger Detection
- Template Engine
- Queue System (collect/steer/followup)

## Model Providers
### Tier 1
- Anthropic (Claude)
  - claude-opus-4-6
  - claude-sonnet-4-5
  - API key / setup-token
- OpenAI (GPT)
  - GPT-4o, GPT-5
  - API key / Codex OAuth
- Google (Gemini)
  - Gemini 2.5 Pro/Flash
  - API key / OAuth

### Tier 2
- Ollama (Local)
- OpenRouter (100+ models)
- AWS Bedrock
- GitHub Copilot
- QianFan, Minimax, Kimi, Z.AI

### Features
- Model Selection + Failover
- Auth Profile Rotation
- Extended Thinking Levels
- Prompt Caching
- Tool Schema Adaptation

## Channels (18+)
### Messaging
- WhatsApp (Baileys)
- Telegram (grammY)
- Discord (discord.js)
- Slack (Bolt)
- Signal (signal-cli)
- iMessage (BlueBubbles)

### Extensions
- Microsoft Teams
- Matrix/Element
- Mattermost
- LINE, Twitch, IRC
- Google Chat
- Nostr, Zalo

## Persona System
### SOUL.md
- Core personality traits
- Values and boundaries
- Tone guidance
- Self-evolving

### IDENTITY.md
- Name, Creature type
- Vibe, Emoji, Avatar

### Bootstrap Files
- AGENTS.md (operating instructions)
- USER.md (user profile)
- TOOLS.md (tool notes)
- MEMORY.md (long-term memory)
- HEARTBEAT.md (heartbeat config)
- BOOTSTRAP.md (first-run ritual)

## Memory System
### Workspace Files
- MEMORY.md (curated, injected every turn)
- memory/YYYY-MM-DD.md (daily logs)

### Vector Search
- SQLite + sqlite-vec
- Hybrid BM25 + Vector
- Embedding Providers
  - OpenAI
  - Gemini
  - Voyage
  - Local GGUF

### QMD Backend (experimental)
- BM25 + Vectors + Reranking
- Local-first (Bun + node-llama-cpp)

### Auto Flush
- Pre-compaction memory save
- Silent agentic turn
- Prevents memory loss

## Skills (55+)
### Categories
- AI & Content (coding-agent, image-gen)
- Productivity (notion, obsidian, trello)
- Communication (github, slack, email)
- Media (spotify, video, camera)
- Smart Home (hue, sonos)
- System (tmux, healthcheck)

### MCP Integration
- mcporter skill
- HTTP + stdio servers
- Standard MCP protocol

## Extensibility
### Plugins
- 40+ hook types
- Agent lifecycle hooks
- Message lifecycle hooks
- Gateway lifecycle hooks

### Extensions (38)
- Channel adapters
- Feature modules
- Auth extensions

### Plugin SDK
- Public API for developers
- TypeScript declarations

## Infrastructure
### Storage
- Sessions (JSONL files)
- Config (JSON5)
- Memory (SQLite + vectors)
- Auth profiles (JSON)

### Deployment
- Docker
- Fly.io
- Render.com
- systemd / launchd / schtasks

### Native Apps
- macOS (Swift)
- iOS (Swift)
- Android

### Web UI
- Lit Web Components
- Vite bundler
- Real-time WebSocket
