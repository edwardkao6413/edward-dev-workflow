# Dev-Manager Workflow

This file is copied into `.dev-manager/workflow.md` by `init-project`.

Read by: `dev-manager`, this plugin's skills, and Superpowers skills when
installed. Project-specific notes belong in `.dev-manager/state.json ->
project.notes`.

---

## 1. Stages

| Stage | Meaning | Required gate |
|---|---|---|
| `PLANNING` | Spec and implementation plan are being created or reviewed | none |
| `DEVELOPING` | Code/scripts/docs are being created or edited | `gates.plan_approved = true` |
| `REVIEW` | Implementation is complete; reviewers and system checks run | `gates.implementation_complete = true` |
| `FINALIZED` | All review gates passed and the task is closed | `gates.final_evals_passed = true` and `gates.task_closed = true` |

Only `dev-manager` may advance `workflow.stage`.

---

## 2. Core Flow

```text
PLANNING
  - superpowers:brainstorming, if requirements are unclear
  - superpowers:writing-plans
  - plan-inspector
  - codex-plan-inspector, Claude-to-Codex-CLI helper only
  - gemini-plan-inspector, optional
  - data-manager, only for data-domain projects

DEVELOPING
  - implementation mode selection
  - codex-development, only from Claude Code when implementation_mode = "codex"
  - superpowers:subagent-driven-development, if implementation_mode = "superpowers" or running in Codex

REVIEW
  - karpathy-guidelines
  - codebase-orchestrator, optional
  - superpowers:requesting-code-review
  - superpowers:receiving-code-review, if user feedback exists
  - system-checker
  - codex-system-checker, Claude-to-Codex-CLI helper only
  - superpowers:verification-before-completion

FINALIZED
  - task closed
```

---

## 3. Path Selection

### Path A - Full Workflow

Use when the task is non-trivial, requirements are unclear, or implementation
should not start before an approved plan.

1. Brainstorm/spec if needed.
2. Write plan.
3. Run plan gates.
4. Implement.
5. Review and verify.
6. Finalize.

### Path B - Small Fix

Use when the user requests a narrow, low-risk change and no plan is needed.

1. Implement.
2. Run `karpathy-guidelines`.
3. Run `system-checker`.
4. Run `superpowers:verification-before-completion`.

### Path C - Direct Agent Call

Use when the user explicitly names an agent.

1. Confirm current stage from `.dev-manager/state.json`.
2. Run the requested agent.
3. Report remaining gates and return control to `dev-manager`.

---

## 4. Host Rules for Codex Helpers

The `codex-*` skills are Claude-side helpers that call Codex CLI:

- `codex-plan-inspector`
- `codex-development`
- `codex-system-checker`

Use them only when running under Claude Code and Codex CLI delegation is
intended.

When running inside Codex, do not invoke these helpers recursively. Use native
Codex behavior and Superpowers instead:

- plan review: current Codex task or Superpowers review flow
- implementation: current Codex task or `superpowers:subagent-driven-development`
- system review: current Codex task plus `system-checker` and Superpowers verification

---

## 5. Implementation Mode

At the DEVELOPING stage entry point, ask for implementation mode unless
`workflow.implementation_mode` is already set.

```text
[1] Codex development
    Claude-side helper that delegates to Codex CLI.
    Do not use recursively from inside Codex.

[2] Superpowers development
    Use superpowers:subagent-driven-development.
```

Default on no answer:

- Codex host: `superpowers`
- Claude Code with Codex CLI delegation requested: `codex`
- otherwise: `superpowers`

Allowed values:

- `codex`
- `superpowers`
- `claude` as a backward-compatible alias for `superpowers`

---

## 6. Superpowers Dependency

This plugin expects Superpowers to be installed for:

- `superpowers:brainstorming`
- `superpowers:writing-plans`
- `superpowers:subagent-driven-development`
- `superpowers:dispatching-parallel-agents`
- `superpowers:executing-plans`
- `superpowers:systematic-debugging`
- `superpowers:requesting-code-review`
- `superpowers:receiving-code-review`
- `superpowers:verification-before-completion`
- `superpowers:finishing-a-development-branch`

When Superpowers is unavailable, do not assume `.agents/` fallback files exist.
Surface the missing dependency and ask the user to install Superpowers or choose
a narrow inline fallback.

In Codex, Superpowers subagent workflows require multi-agent support:

```toml
[features]
multi_agent = true
```

If `[features]` already exists in `~/.codex/config.toml`, add
`multi_agent = true` inside the existing block.

---

## 7. Data Domain Delegation

Run `data-manager` when any of these are true:

1. `state.json -> project.type` is one of:
   - `biomechanics`
   - `bioinformatics`
   - `data-science`
   - `data-engineering`
   - `machine-learning`
   - `analytics`
2. `state.json -> project.domain` contains data-related terms.
3. The active plan mentions data pipeline keywords such as `pipeline`, `ETL`,
   `dataset`, `dataframe`, `SQL`, `schema`, `raw data`, `preprocessing`,
   `feature engineering`, `time series`, or `signal processing`.

`data-manager` is a planning-stage domain validator. It does not advance stages.

---

## 8. Floating Agents

Floating agents may run at any stage without changing the stage:

| Agent | Trigger |
|---|---|
| `superpowers:systematic-debugging` | Bug, error, failing test, or unexpected output |
| `superpowers:writing-skills` | Creating or improving any `SKILL.md` file |

Floating agents must append an audit entry with `stage_before == stage_after`.

---

## 9. Gate Rules

| Transition | Requires |
|---|---|
| `PLANNING -> DEVELOPING` | `gates.plan_approved = true` |
| `DEVELOPING -> REVIEW` | `gates.implementation_complete = true` |
| `REVIEW -> FINALIZED` | `gates.final_evals_passed = true` and `gates.task_closed = true` |

Never advance a stage without updating `.dev-manager/state.json` first.

---

## 10. Evals

Evals live in `.dev-manager/evals/evals.json`.
Results live in `.dev-manager/results/iteration-N/`.

| Situation | Eval action |
|---|---|
| New feature added | Add new evals before final review |
| Existing behavior modified | Run evals scoped to affected modules |
| Bug fix | Add or run regression evals for the affected path |
| Cosmetic/UI change | Eval optional, but validation still required |
| Evals missing for changed behavior | Flag to user before proceeding |

If the user says "skip evals", warn once, respect the choice, and log the skip.

---

## 11. Handoff Protocol

When dispatching any agent, provide:

1. Stage - current stage from `.dev-manager/state.json`
2. Context - what was already done
3. Focus - plan, diff, file, or feature under review
4. Eval status - pass/fail count and result path, or `n/a`
5. Blocking condition - what the agent must produce before the next step

---

## 12. Idempotency

Before dispatching any agent, check:

```text
agents.<name>.approved == true
AND
agents.<name>.last_run != ""
```

If true, ask whether to skip or re-run that agent.

`run_count` increments on every dispatch. It is diagnostic, not a gate.

---

## 13. Rollback Checkpoints

Save a rollback checkpoint after each gate passage:

| Gate | Checkpoint |
|---|---|
| `plan_approved = true` | End of PLANNING |
| `implementation_complete = true` | End of DEVELOPING |
| `final_evals_passed = true` | End of REVIEW |

Never roll back silently. Offer rollback only when a later gate fails badly or
the user explicitly requests it.
