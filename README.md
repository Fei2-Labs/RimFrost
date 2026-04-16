<h1 align="center">⛰ RimFrost</h1>

<p align="center">
  A self-evolving CLI agent that gets smarter the more you use it.
</p>

<p align="center">
  <a href="#-quick-start">Quick Start</a> ·
  <a href="#-core-capabilities">Capabilities</a> ·
  <a href="#-cli-reference">CLI Reference</a> ·
  <a href="./USAGE.md">Full Usage Guide</a> ·
  <a href="./PHILOSOPHY.md">Philosophy</a>
</p>

<p align="center">
  <a href="https://star-history.com/#Fei2-Labs/rimfrost&Date">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=Fei2-Labs/rimfrost&type=Date&theme=dark" />
      <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=Fei2-Labs/rimfrost&type=Date" />
      <img alt="Star History" src="https://api.star-history.com/svg?repos=Fei2-Labs/rimfrost&type=Date" width="600" />
    </picture>
  </a>
</p>

---

## What is RimFrost?

RimFrost is a high-performance Rust CLI agent harness that learns from every session. It writes facts, skills, and working memory to disk — and loads them back on every future run. The agent accumulates project-specific knowledge over time instead of starting from zero.

Built as a Rust rewrite with 9 crates, ~20K lines, and a full tool system for autonomous coding tasks.

---

## ⚡ Quick Start

```bash
git clone https://github.com/Fei2-Labs/rimfrost
cd rimfrost/rust
cargo build --workspace

export ANTHROPIC_API_KEY="sk-ant-..."

./target/debug/rimfrost doctor    # verify setup
./target/debug/rimfrost           # start REPL
./target/debug/rimfrost prompt "summarize this repo"
```

---

## 🧠 Core Capabilities

### Self-Evolving Memory

The agent doesn't just answer — it **learns**.

| Layer | What | Scope |
|---|---|---|
| **Working Checkpoint** | Short-term scratchpad injected every turn — prevents context drift in long tasks | Within a session |
| **Long-Term Memory** | Facts, preferences, environment details the agent writes to disk | All future sessions |
| **Project Memory** | Project-specific patterns and conventions | Per-project, all sessions |
| **Skills** | SOPs the agent writes after learning how to do something | Cross-project |

```
~/.rimfrost/memory/
├── global.md        # User preferences, environment facts
├── insights.md      # Structured summary
└── skills/          # Reusable procedures
    ├── docker-setup.md
    └── rust-ci-fix.md

.rimfrost/memory/
├── project.md       # Project-specific knowledge
└── skills/
    └── deploy-staging.md
```

The agent decides when to remember. You can also tell it: *"remember that I deploy to staging with `make deploy-stg`"*

### Failure Escalation

Built-in retry intelligence — never repeats the same action without new information:

1. **1st failure** → read the error, understand root cause
2. **2nd failure** → probe the environment (check file state, deps, versions)
3. **3rd failure** → switch to a fundamentally different approach or ask the user

### Full Tool System

| Tool | What it does |
|---|---|
| `bash` | Execute shell commands with permission controls |
| `read_file` | Read files with offset/limit |
| `write_file` | Create or overwrite files |
| `edit_file` | Surgical text replacement |
| `glob_search` | Find files by pattern |
| `grep_search` | Regex search across files |
| `WebSearch` | Search the web with cited results |
| `WebFetch` | Fetch and extract content from URLs |
| `WorkingCheckpoint` | Update intra-session working memory |
| `MemoryWrite` | Save learnings to long-term memory |
| `MemoryRead` | Search long-term memory |
| `Agent` | Spawn sub-agent tasks |
| `TodoWrite` | Structured task tracking |
| `NotebookEdit` | Edit Jupyter notebooks |
| `Skill` | Load local skill definitions |
| `ToolSearch` | Discover specialized tools |

### Multi-Provider Support

| Provider | Auth | Models |
|---|---|---|
| Anthropic | `ANTHROPIC_API_KEY` | Claude Opus, Sonnet, Haiku |
| xAI | `XAI_API_KEY` | Grok 3, Grok 3 Mini |
| OpenAI-compatible | `OPENAI_API_KEY` + `OPENAI_BASE_URL` | GPT-4, Llama, any |
| DashScope | `DASHSCOPE_API_KEY` | Qwen series |
| Ollama | `OPENAI_BASE_URL=http://localhost:11434/v1` | Any local model |
| OpenRouter | `OPENAI_API_KEY` + `OPENAI_BASE_URL` | 200+ models |

### Session Persistence

Every REPL session is auto-saved to `.rimfrost/sessions/`. Resume any session:

```bash
rimfrost --resume latest
rimfrost --resume latest /status /diff
```

### Permission System

Three modes controlling what the agent can do:

- `read-only` — can only read files and search
- `workspace-write` — can modify files in the workspace
- `danger-full-access` — full shell access (default)

### MCP Server Support

Connect external tools via the Model Context Protocol:

```bash
rimfrost mcp                    # list connected servers
rimfrost mcp show my-server     # inspect a server
```

---

## 🖥 CLI Reference

### Top-Level Commands

```bash
rimfrost                                    # Interactive REPL
rimfrost prompt "do something"              # One-shot task
rimfrost "explain this code"                # Shorthand one-shot
rimfrost doctor                             # Health check
rimfrost status                             # Workspace status
rimfrost sandbox                            # Container detection
rimfrost agents                             # List agent definitions
rimfrost mcp                                # MCP server status
rimfrost skills                             # Skill inventory
rimfrost system-prompt                      # Print assembled system prompt
rimfrost init                               # Initialize workspace
```

### Options

```bash
rimfrost --model sonnet                     # Pick model (opus/sonnet/haiku)
rimfrost --output-format json prompt "..."  # JSON output for scripting
rimfrost --permission-mode read-only        # Restrict permissions
rimfrost --allowedTools read,glob "..."     # Whitelist specific tools
rimfrost --resume latest                    # Resume last session
rimfrost --compact "summarize Cargo.toml"   # Compact output for piping
```

### REPL Slash Commands

| Command | Action |
|---|---|
| `/help` | Show all commands |
| `/status` | Session state, model, cost |
| `/model` | Current model info |
| `/cost` | Token usage and cost estimate |
| `/config` | View/edit settings |
| `/memory` | Inspect long-term memory |
| `/session` | Session management |
| `/compact` | Toggle compact mode |
| `/diff` | Show git diff |
| `/commit` | Commit changes |
| `/export` | Export conversation |
| `/doctor` | Health check |
| `/skills` | List/install skills |
| `/agents` | List agent definitions |
| `/mcp` | MCP server status |
| `/hooks` | Lifecycle hooks |

---

## 🏗 Architecture

```
rust/crates/
├── api/                 # Provider clients, streaming, auth
├── commands/            # Slash command registry
├── runtime/             # Session, config, permissions, prompts, memory
├── tools/               # Built-in tools (bash, files, web, memory, agents)
├── plugins/             # Plugin system
├── rusty-claude-cli/    # Main CLI binary
├── telemetry/           # Usage tracking
├── mock-anthropic-service/  # Deterministic mock for testing
└── compat-harness/      # Upstream manifest extraction
```

- **~20K lines** of Rust across **9 crates**
- Binary: `rimfrost`
- Default model: `claude-opus-4-6`

---

## 📖 Documentation

- [USAGE.md](./USAGE.md) — Full usage guide with examples
- [PHILOSOPHY.md](./PHILOSOPHY.md) — Why this project exists
- [PARITY.md](./PARITY.md) — Rust port status
- [ROADMAP.md](./ROADMAP.md) — Active roadmap
- [docs/container.md](./docs/container.md) — Container workflows
- [docs/MODEL_COMPATIBILITY.md](./docs/MODEL_COMPATIBILITY.md) — Model-specific handling

---

## 📄 License

MIT. See repository root.

This project is **not affiliated with or endorsed by Anthropic**.
