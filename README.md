# edward-dev-workflow

An AI-driven development workflow plugin for Claude Code. It orchestrates the full development lifecycle through a multi-stage gate system — from brainstorming and planning, through implementation and review, to finalization — using specialized agents that enforce quality at every transition.

---

## What It Does

- **Orchestrates every stage** of development: PLANNING → DEVELOPING → REVIEW → FINALIZED
- **Enforces quality gates** — no code before a plan, no merge before review passes
- **Routes to domain specialists** for data engineering, analytics, data science, and database work
- **Supports parallel agent dispatch** for independent tasks
- **Provides discipline skills** for TDD, debugging, code review, and git worktrees

---

## Installation

### Option A — Via Claude Code (recommended)

Run these two commands inside Claude Code:

```
/plugin marketplace add edwardkao6413/edward-dev-workflow
/plugin install edward-dev-workflow@edward-dev-workflow
```

The first command registers this repo as a plugin source. The second installs the plugin.

### Option B — Local (Git Clone)

1. Clone this repo:
   ```
   git clone https://github.com/edwardkao6413/edward-dev-workflow.git
   ```
2. In Claude Code, run:
   ```
   /plugin marketplace add /path/to/edward-workflow
   /plugin install edward-dev-workflow@edward-dev-workflow
   ```

---

## Getting Started

Once installed, start any new project with:

```
/edward-dev-workflow:init-project
```

This scaffolds the full governance structure (`.dev-manager/`, `project.config.md`, `state.json`) into your project and hands control to `dev-manager` to begin work.

---

## Skills

### Orchestration
| Skill | Purpose |
|-------|---------|
| `dev-manager` | Central orchestrator — governs all stage transitions and agent dispatch |
| `init-project` | Scaffolds governance files into a new project |
| `plan-inspector` | Validates implementation plans before development begins |
| `system-checker` | End-to-end system validation after implementation |

### Planning
| Skill | Purpose |
|-------|---------|
| `brainstorming` | Turns ideas into fully-formed specs through collaborative dialogue |
| `writing-plans` | Generates detailed implementation plans from approved specs |

### Implementation
| Skill | Purpose |
|-------|---------|
| `subagent-driven-development` | Dispatches fresh subagents per task with two-stage review |
| `dispatching-parallel-agents` | Runs independent tasks concurrently |
| `executing-plans` | Inline execution path for simpler plans |
| `using-git-worktrees` | Isolates feature work via git worktrees |

### Review & Quality
| Skill | Purpose |
|-------|---------|
| `karapathy-guideline` | Per-file code quality review |
| `requesting-code-review` | Generates structured review requests |
| `receiving-code-review` | Handles review feedback with technical rigor |
| `verification-before-completion` | Runs verification before claiming work is done |
| `finishing-a-development-branch` | Guides branch integration decisions |

### Debugging & Testing
| Skill | Purpose |
|-------|---------|
| `systematic-debugging` | Root cause investigation before any fix |
| `test-driven-development` | Enforces RED-GREEN-REFACTOR discipline |

### Domain Specialists
| Skill | Purpose |
|-------|---------|
| `data-manager` | Routes data projects to the correct specialist |
| `data-engineer` | Data pipeline and ETL validation |
| `data-analyst` | Analytics and reporting domain specialist |
| `data-scientist` | ML/model domain specialist |
| `database-optimizer` | Database performance specialist |
| `codebase-admin` | Structural validation for large codebases |

### Meta
| Skill | Purpose |
|-------|---------|
| `writing-skills` | TDD-based skill authoring and testing |
| `using-superpowers` | Master skill — when and how to invoke all others |

---

## Author

Edward Kao
