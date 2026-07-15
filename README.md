# edward-dev-workflow

An AI-driven development workflow plugin for both Codex and Claude Code. It
orchestrates the development lifecycle through planning, implementation, review,
and finalization gates.

This plugin is a workflow layer. It relies on
[Superpowers](https://github.com/obra/superpowers) for the general-purpose
process skills: brainstorming, writing plans, subagent-driven development,
systematic debugging, TDD, code review, and completion verification.

For the detailed pipeline, see
[`skills/references/workflow.md`](skills/references/workflow.md).

---

## What It Does

- Orchestrates stages: `PLANNING -> DEVELOPING -> REVIEW -> FINALIZED`
- Enforces quality gates before implementation and finalization
- Routes data-heavy projects to specialist data agents
- Uses Superpowers for parallel/subagent execution where the host supports it
- Keeps project workflow state in `.dev-manager/state.json`

---

## Prerequisite: Superpowers

Install Superpowers before using this plugin.

### Codex

Install Superpowers from Codex:

```text
/plugins
```

Search for `Superpowers`, install it, then start a new Codex task.

For Superpowers subagent workflows in Codex, enable multi-agent support in
`~/.codex/config.toml`:

```toml
[features]
multi_agent = true
```

If a `[features]` block already exists, add `multi_agent = true` inside that
existing block rather than creating a second `[features]` block.

### Claude Code

Install Superpowers using the instructions from the
[Superpowers repository](https://github.com/obra/superpowers). This plugin's
Claude workflow assumes those skills are available.

---

## Installing This Plugin

### Codex

This repository includes Codex packaging at:

```text
.codex-plugin/plugin.json
```

For local development, place or clone this repository where your Codex personal
plugin marketplace can reference it, then install it through Codex's plugin
browser. A repo-local `.agents/` marketplace is not required for the plugin
itself; it is only an optional distribution mechanism.

### Claude Code

This repository keeps Claude packaging at:

```text
.claude-plugin/plugin.json
.claude-plugin/marketplace.json
```

Install from Claude Code:

```text
/plugin marketplace add edwardkao6413/edward-dev-workflow
/plugin install edward-dev-workflow@edward-dev-workflow
```

For a local clone:

```text
/plugin marketplace add /path/to/edward-workflow
/plugin install edward-dev-workflow@edward-dev-workflow
```

---

## Getting Started

Once installed, start any new project with:

```text
edward-dev-workflow:init-project
```

In Claude Code this may be available as a slash command:

```text
/edward-dev-workflow:init-project
```

This scaffolds `.dev-manager/`, `project.config.md`, state, checklist, and
workflow reference files, then hands control to `dev-manager`.

---

## Skills

### Orchestration

| Skill | Purpose |
|---|---|
| `dev-manager` | Central orchestrator for stages, gates, and specialist dispatch |
| `init-project` | Scaffolds governance files into a project |
| `plan-inspector` | Validates implementation plans before development begins |
| `system-checker` | Runs end-to-end validation after implementation |

### Edward Workflow Specialists

| Skill | Purpose |
|---|---|
| `data-manager` | Routes data projects to the right specialist |
| `data-engineer` | Data pipeline and ETL validation |
| `data-analyst` | Analytics and reporting specialist |
| `data-scientist` | ML/statistical modeling specialist |
| `database-optimizer` | Database performance specialist |
| `codebase-orchestrator` | Cross-file structural validation |
| `ui-designer` | Frontend/UI implementation support |
| `karpathy-guidelines` | Code quality review principles |

### Superpowers Skills Used By This Plugin

These are provided by the separate Superpowers plugin:

| Superpowers skill | Purpose |
|---|---|
| `brainstorming` | Turns ideas into specs |
| `writing-plans` | Generates implementation plans |
| `subagent-driven-development` | Dispatches fresh subagents per task |
| `dispatching-parallel-agents` | Runs independent tasks concurrently |
| `executing-plans` | Inline execution path for simpler plans |
| `systematic-debugging` | Root cause investigation before fixes |
| `test-driven-development` | Red-green-refactor discipline |
| `requesting-code-review` | Structured code-review request |
| `receiving-code-review` | Processes review feedback |
| `verification-before-completion` | Evidence-based completion check |
| `finishing-a-development-branch` | Branch integration and handoff guidance |

---

## Author

Edward Kao
