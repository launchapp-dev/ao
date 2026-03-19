# AO — Autonomous Agent Orchestrator

AO is a workflow-driven agent orchestrator for software delivery. It breaks complex tasks into multi-phase pipelines, dispatches them to specialized AI agents, and manages quality gates — all from a single YAML config.

## Quick Install (macOS)

```bash
curl -fsSL https://raw.githubusercontent.com/launchapp-dev/ao/main/install.sh | bash
```

### Options

```bash
# Install a specific version
AO_VERSION=v0.0.11 curl -fsSL https://raw.githubusercontent.com/launchapp-dev/ao/main/install.sh | bash

# Install to a custom directory
AO_INSTALL_DIR=/usr/local/bin curl -fsSL https://raw.githubusercontent.com/launchapp-dev/ao/main/install.sh | bash
```

## What Gets Installed

| Binary | Purpose |
|--------|---------|
| `ao` | Main CLI — tasks, workflows, daemon, queue, MCP server |
| `agent-runner` | IPC daemon managing LLM CLI processes |
| `llm-cli-wrapper` | Abstraction layer over AI CLI tools |

Binaries are installed to `~/.local/bin` by default.

## Prerequisites

AO orchestrates AI coding agents. You need at least one LLM CLI installed:

```bash
# Claude (recommended)
npm install -g @anthropic-ai/claude-code

# Codex
npm install -g @openai/codex

# Gemini
npm install -g @google/gemini-cli
```

## Getting Started

```bash
# Navigate to a git repository
cd your-project

# Check prerequisites
ao doctor

# Initialize project
ao setup

# Create and run a task
ao task create --title "Fix the login bug" --type bugfix --priority high
ao workflow run --task-id TASK-001

# Start the autonomous daemon
ao daemon start --autonomous
```

## What AO Does

AO defines autonomous software delivery pipelines in YAML:

- **Agent profiles** — bind models, tools, MCP servers, and system prompts to named agents
- **Phases** — reusable execution units (agent, command, or manual modes)
- **Workflows** — ordered phase compositions with skip conditions and post-success hooks
- **Schedules** — cron-based autonomous execution
- **Decision contracts** — typed verdicts (advance/rework/skip/fail) with confidence thresholds
- **Model routing** — assign different LLMs by task type and complexity
- **Fallback chains** — automatic model failover (e.g., GLM → MiniMax → Claude)

## Supported Platforms

| Platform | Architecture | Status |
|----------|-------------|--------|
| macOS | Apple Silicon (M1+) | Supported |
| macOS | Intel x86_64 | Supported |
| Linux | x86_64 | Supported |
| Windows | x86_64 | Supported |

## Updating

Run the install script again to update to the latest version:

```bash
curl -fsSL https://raw.githubusercontent.com/launchapp-dev/ao/main/install.sh | bash
```

## Uninstall

```bash
rm -f ~/.local/bin/ao ~/.local/bin/agent-runner ~/.local/bin/llm-cli-wrapper
```

## License

Proprietary. All rights reserved.
