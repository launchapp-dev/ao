<p align="center">
  <img src="https://img.shields.io/github/v/release/launchapp-dev/ao?style=flat-square&color=18181b&labelColor=18181b" alt="Release" />
  <img src="https://img.shields.io/badge/rust-100%25-18181b?style=flat-square&labelColor=18181b" alt="Rust" />
  <img src="https://img.shields.io/badge/platforms-macOS%20%7C%20Linux%20%7C%20Windows-18181b?style=flat-square&labelColor=18181b" alt="Platforms" />
  <img src="https://img.shields.io/badge/license-proprietary-18181b?style=flat-square&labelColor=18181b" alt="License" />
</p>

<h1 align="center">AO</h1>
<p align="center"><strong>The autonomous agent orchestrator that ships code while you sleep.</strong></p>

<p align="center">
Define your entire engineering team as YAML — agents, workflows, schedules, quality gates.<br/>
AO dispatches tasks to AI agents, reviews their work, and merges the results.<br/>
One config file. Zero human bottlenecks.
</p>

---

## Install

```bash
curl -fsSL https://raw.githubusercontent.com/launchapp-dev/ao/main/install.sh | bash
```

That's it. Installs `ao`, `agent-runner`, and `llm-cli-wrapper` to `~/.local/bin`.

<details>
<summary>Options</summary>

```bash
# Specific version
AO_VERSION=v0.0.11 curl -fsSL https://raw.githubusercontent.com/launchapp-dev/ao/main/install.sh | bash

# Custom install directory
AO_INSTALL_DIR=/usr/local/bin curl -fsSL https://raw.githubusercontent.com/launchapp-dev/ao/main/install.sh | bash
```

</details>

## What is AO?

AO is a **16-crate Rust workspace** that turns a YAML file into an autonomous software delivery pipeline.

You define agents, wire them into phases, compose phases into workflows, and schedule everything with cron — in a single `custom.yaml`. AO's daemon picks up tasks, dispatches them to AI agents running in isolated git worktrees, and manages the full lifecycle: triage → refine → implement → review → push → PR → merge.

```
  Task Queue          Daemon             Worktrees           GitHub
  ┌────────┐       ┌──────────┐       ┌───────────┐       ┌────────┐
  │ TASK-1 │──────▶│ Dispatch │──────▶│ Claude    │──────▶│  PR #1 │
  │ TASK-2 │──────▶│ Schedule │──────▶│ Codex     │──────▶│  PR #2 │
  │ TASK-3 │──────▶│ Route    │──────▶│ Gemini    │──────▶│  PR #3 │
  └────────┘       └──────────┘       └───────────┘       └────────┘
                        │                                      │
                   Phase Verdicts                         Auto-Merge
                   (advance/rework)                       via PR Reviewer
```

### Everything in one YAML

```yaml
agents:
  default:
    model: claude-sonnet-4-6
    tool: claude
    mcp_servers: ["ao", "context7"]

  work-planner:
    system_prompt: |
      You are the work planner. Scan tasks, check dependencies,
      and enqueue ready work for the daemon.
    model: claude-sonnet-4-6
    tool: claude

phases:
  implementation:
    mode: agent
    agent: default
    directive: "Implement production-quality code for this task."
    decision_contract:
      min_confidence: 0.7
      max_risk: medium

  code-review:
    mode: agent
    directive: "Review for defects. Verdict: advance or rework."
    decision_contract:
      required_evidence: [code_review_clean]

workflows:
  - id: standard
    phases:
      - requirements
      - implementation
      - push-branch
      - create-pr
    post_success:
      merge: { strategy: squash, auto_merge: true }

schedules:
  - id: work-planner
    cron: "*/5 * * * *"
    workflow_ref: work-planner
```

## Core Concepts

### Agents
Named profiles that bind a model, CLI tool, MCP servers, and system prompt. Mix Claude, Codex, Gemini, GLM, MiniMax — route by task type and complexity.

### Phases
Reusable execution units in three modes:
- **`agent`** — AI agent with decision contracts and typed verdicts
- **`command`** — shell commands (`cargo build`, `git push`, `gh pr create`)
- **`manual`** — human-in-the-loop approval gates

### Workflows
Ordered phase compositions. A workflow like `standard` chains `requirements → implementation → push → PR`. Add `post_success` hooks for auto-merge and worktree cleanup.

### Decision Contracts
Every agent phase returns a typed verdict: **advance**, **rework**, **skip**, or **fail**. Rework loops send the reviewer's feedback back to the implementer with context. Configurable `max_rework_attempts` prevents infinite loops.

### Schedules
Cron expressions that run workflows autonomously. The work-planner scans every 5 minutes, the PR reviewer merges every 5 minutes, PO agents scan their domains every few hours.

### Model Routing
Route tasks to different models based on type and complexity:

| Complexity | Type | Model | Why |
|---|---|---|---|
| low | bugfix | GLM-5-Turbo | Cheapest |
| medium | feature | Claude Sonnet | Reliable |
| medium | UI/UX | Gemini 3.1 Pro | Vision + design |
| high | architecture | Claude Opus | Maximum quality |
| critical | any | Claude Opus | No compromises |

## Quick Start

```bash
# 1. Install a coding agent CLI
npm install -g @anthropic-ai/claude-code    # or @openai/codex or @google/gemini-cli

# 2. Initialize your project
cd your-project
ao doctor        # check prerequisites
ao setup         # initialize .ao/ directory

# 3. Create a task and run it
ao task create --title "Add rate limiting to the API" --type feature --priority high
ao workflow run --task-id TASK-001

# 4. Or go fully autonomous
ao daemon start --autonomous
ao queue list                    # watch tasks flow through
ao output tail                   # stream agent output
```

## The Full Agent Team

AO doesn't just run one agent. It runs an **entire product organization**:

```
┌─────────────────────────────────────────────────────────┐
│                    AO Daemon (Rust)                      │
│                                                         │
│  Planners          Builders          Reviewers          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ Work Planner │  │ Claude Eng   │  │ PR Reviewer  │  │
│  │ Reconciler   │  │ Codex Eng    │  │ PO Reviewer  │  │
│  │ Triager      │  │ Gemini Eng   │  │ Code Review  │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│                                                         │
│  Product Owners    Architects        Operations         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ PO: Web      │  │ Rust Arch    │  │ Sys Monitor  │  │
│  │ PO: MCP      │  │ Infra Arch   │  │ Release Mgr  │  │
│  │ PO: Workflow │  │              │  │ Branch Sync  │  │
│  │ PO: CLI      │  │              │  │ Doc Drift    │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
└─────────────────────────────────────────────────────────┘
```

## Claude Code Skills

Install the [AO Skills](https://github.com/launchapp-dev/ao-skills) plugin to get deep AO integration in Claude Code:

```bash
claude mcp add ao-skills -- npx @anthropic-ai/claude-code-mcp
```

Or clone directly:

```bash
git clone https://github.com/launchapp-dev/ao-skills.git ~/ao-skills
claude --plugin-dir ~/ao-skills
```

### Slash Commands

| Command | What it does |
|---|---|
| `/setup-ao` | Initialize AO in your project — config, MCP, first workflow |
| `/getting-started` | Full walkthrough: install, concepts, first task |
| `/workflow-authoring` | Write custom YAML — agents, phases, crons |
| `/pack-authoring` | Build workflow packs for the marketplace |
| `/mcp-setup` | Connect AI tools to AO via `.mcp.json` |
| `/troubleshooting` | Debug daemon crashes, workflow failures, queue issues |

### Auto-Loaded References

These activate automatically when Claude detects you're working with AO:

| Skill | Coverage |
|---|---|
| `configuration` | Project config, daemon config, agent runtime, state layout |
| `task-management` | Full task lifecycle — create, list, update, block/unblock |
| `daemon-operations` | Start, monitor, troubleshoot the daemon |
| `queue-management` | Dispatch queue operations |
| `mcp-tools` | Complete `ao.*` MCP tool reference |
| `workflow-patterns` | Battle-tested patterns from 150+ autonomous PRs |
| `agent-personas` | Product lifecycle agents — PO, architect, auditor |
| `mcp-servers-for-agents` | Connect agents to Context7, GitHub, memory |

## CLI Reference

```
ao task          Create, list, update, prioritize tasks
ao workflow      Run and manage multi-phase workflows
ao daemon        Start/stop the autonomous scheduler
ao queue         Inspect and manage the dispatch queue
ao agent         Control agent runner processes
ao output        Stream and inspect agent output
ao doctor        Health checks and auto-remediation
ao setup         Interactive project initialization
ao requirements  Manage product requirements
ao status        Project overview
ao mcp           Start AO as an MCP server
ao web           Launch the embedded web dashboard
```

## Architecture

```
108k lines of Rust across 16 crates:

orchestrator-cli          30k   CLI commands and dispatch
orchestrator-core         18k   Services, state, config
orchestrator-config       13k   Workflow YAML compilation
workflow-runner-v2         9k   Phase execution engine
agent-runner               6k   LLM CLI process management
llm-cli-wrapper            6k   CLI tool abstraction
orchestrator-web-server    5k   Embedded React dashboard
protocol                   5k   Shared types and model routing
orchestrator-daemon        4k   Daemon scheduler runtime
oai-runner                 3k   OpenAI-compatible API client
orchestrator-providers     3k   Provider abstractions
orchestrator-web-api       3k   GraphQL API layer
+ 4 more crates
```

## Platforms

| Platform | Architecture | Status |
|---|---|---|
| macOS | Apple Silicon (M1+) | Supported |
| macOS | Intel x86_64 | Supported |
| Linux | x86_64 | Supported |
| Windows | x86_64 | Supported |

## Update

```bash
curl -fsSL https://raw.githubusercontent.com/launchapp-dev/ao/main/install.sh | bash
```

## Uninstall

```bash
rm -f ~/.local/bin/ao ~/.local/bin/agent-runner ~/.local/bin/llm-cli-wrapper
rm -rf ~/.ao                    # scoped runtime state
rm -rf .ao                      # project-local state (run from project root)
```

---

<p align="center">
  <strong>Built with Rust. Powered by AI. Ships code autonomously.</strong>
</p>
