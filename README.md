# Cursor2API

[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-blue.svg)](https://www.typescriptlang.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Node](https://img.shields.io/badge/Node-18+-green.svg)](https://nodejs.org/)

> ⚠️ As of 2026-04-01, Cursor Docs page only provides `gemini-3-flash`. This project may have limited functionality.

A proxy service that converts Cursor Web Docs free AI API into **Anthropic Messages API** and **OpenAI Chat Completions API**, enabling use with **Claude Code** and **Cursor IDE**.

> 📖 中文文档: [README_CN.md](README_CN.md)

## How It Works

```
┌─────────────┐     ┌──────────────┐     ┌──────────────┐
│ Claude Code  │────▶│              │────▶│              │
│ (Anthropic)  │     │  cursor2api  │     │  Cursor API  │
│              │◀────│  (proxy)      │◀────│  /api/chat   │
└─────────────┘     └──────────────┘     └──────────────┘
       ▲                    ▲
       │                    │
┌──────┴──────┐     ┌──────┴──────┐
│  Cursor IDE  │     │ OpenAI Compat│
│(/v1/responses│     │(/v1/chat/   │
│ + Agent mode)│     │ completions)│
└─────────────┘     └─────────────┘
```

## Core Features

- **Anthropic Messages API** — Full `/v1/messages` streaming/non-streaming compatibility for Claude Code
- **OpenAI Chat Completions API** — `/v1/chat/completions` for ChatBox, LobeChat and other clients
- **Cursor IDE Agent Mode** — `/v1/responses` endpoint with flat tool format and incremental streaming
- **Full-link Log Viewer** — Web UI for real-time request/response/tool call inspection with dark/light theme
- **API Token Auth** — Bearer token / x-api-key dual mode with multi-token support
- **Thinking Support** — Client-driven Anthropic `thinking` block + OpenAI `reasoning_content`
- **response_format Support** — `json_object` / `json_schema` with auto markdown stripping
- **Tool Parameter Auto-fix** — Field name mapping, smart quote replacement, fuzzy matching
- **Vision Fallback** — Built-in CPU OCR for image text extraction (zero config), or external vision API
- **Full Tool Support** — No tool whitelist, all MCP tools and custom extensions supported
- **Multi-layer Refusal Defense** — 50+ regex patterns (CN/EN), auto-retry + cognitive reframing bypass
- **Identity Protection** — Probe interception + refusal retry + response sanitization
- **Truncation Continuation** — Seamless resume of truncated long Write/Edit calls
- **Progressive History Compression** — Smart message type recognition, tool call summarization
- **Schema Compression** — Compacts full JSON Schema (~135k chars) to compact type signatures (~15k chars)
- **Chrome TLS Fingerprint** — Simulates real browser request headers
- **SSE Streaming** — Real-time response with 128-byte incremental tool call chunks

## Quick Start

### 1. Install

```bash
git clone https://github.com/xuweizhengo/cursor-free-api.git
cd cursor-free-api
npm install
```

### 2. Configure

```bash
cp config.yaml.example config.yaml
```

Key options:

| Option | Description | Default |
|--------|-------------|---------|
| `port` | Server port | `3010` |
| `auth_tokens` | API auth tokens (recommended for public deployment) | Allow all |
| `cursor_model` | Model to use | `google/gemini-3-flash` |
| `max_history_tokens` | History token limit | `120000` |
| `vision.enabled` | Enable vision interception | `true` |
| `vision.mode` | Vision mode: `ocr` / `api` | `ocr` |
| `compression.enabled` | History compression | `true` |
| `compression.level` | Compression level 1-3 | `2` |
| `proxy` | Global proxy (optional) | none |

### 3. Run

```bash
# Development
npm run dev

# Production
npm run build && npm start
```

### 4. Use with Claude Code

```bash
export ANTHROPIC_BASE_URL=http://localhost:3010
claude
```

With auth:

```bash
export ANTHROPIC_BASE_URL=http://localhost:3010
export ANTHROPIC_API_KEY=sk-your-secret-token-1
claude
```

### 5. Use with Cursor IDE

Set in Cursor IDE settings:
```
OPENAI_BASE_URL=https://your-domain.example.com/v1
```
Select a Claude model name (check `/v1/models` for available models).

> ⚠️ Cursor Pro subscription may be required for custom Base URL.

## Log Viewer

Visit `http://localhost:3010/logs` after starting the service.

Features:
- Real-time SSE log streaming
- Request list with user question as title
- Global search + time filtering
- Status filtering (success/degraded/failed/processing/blocked)
- Detail panel for full request/response inspection
- Degradation reason display
- Phase timing visualization
- Dark/light theme toggle
- SQLite persistence (recommended) or JSONL file logging

## Project Structure

```
cursor2api/
├── src/
│   ├── index.ts            # Entry + Express server + routes + auth middleware
│   ├── config.ts           # Configuration management
│   ├── types.ts            # Type definitions
│   ├── constants.ts        # Global constants
│   ├── cursor-client.ts    # Cursor API client + Chrome TLS fingerprint
│   ├── converter.ts        # Protocol conversion + prompt injection + context cleaning
│   ├── handler.ts          # Anthropic API handler + identity protection + refusal defense
│   ├── openai-handler.ts   # OpenAI / Cursor IDE compatible handler
│   ├── log-viewer.ts       # Full-link log Web UI
│   ├── logger.ts           # Log collection + SSE push
│   ├── proxy-agent.ts      # Proxy support (global + Vision independent)
│   └── tool-fixer.ts       # Tool parameter auto-fix
├── public/                 # Log viewer static files
├── test/                   # Test suites
├── config.yaml.example     # Config template
├── package.json
└── tsconfig.json
```

## Environment Variables

All config options can be overridden via environment variables (higher priority than `config.yaml`). See `config.yaml.example` for details.

## Technical Architecture

### Cognitive Reframing Strategy

The core insight: instead of fighting Cursor's locked-down "Documentation Assistant" identity, we **embrace it**. The model is told it's writing API documentation examples, tricking it into outputting tool call JSON blocks that the proxy layer then converts into real tool calls.

### Multi-layer Refusal Defense

| Layer | Location | Strategy |
|-------|----------|----------|
| L1: Context Cleaning | `converter.ts` | Remove refusal text from conversation history |
| L2: XML Separation | `converter.ts` | Separate Claude Code system reminders from user requests |
| L3: Output Interception | `handler.ts` | 50+ regex patterns to intercept refusals in real-time |
| L4: Response Sanitization | `handler.ts` | Post-process all output to replace Cursor identity references |

## Disclaimer

This project is for educational and research purposes only. It is not affiliated with Cursor or Anysphere. Users should comply with Cursor's Terms of Service and assume all risks. The author is not responsible for any account bans or consequences.



## Related Projects

- [aws-auto-register](https://github.com/xuweizhengo/aws-auto-register) — Automated AWS Builder ID account registration with fingerprint randomization
- [fingerprint-toolkit](https://github.com/xuweizhengo/fingerprint-toolkit) — Browser fingerprint randomization toolkit for web automation
- [llm-api-purity](https://github.com/xuweizhengo/llm-api-purity) — OpenAI and Claude API purity checker
- [skills-hub](https://github.com/xuweizhengo/skills-hub) — AI Skills & MCP marketplace

## License

[MIT](LICENSE)
