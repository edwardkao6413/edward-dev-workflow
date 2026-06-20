# Dev-Manager Workflow

This is the **authoritative reference** for the entire agent orchestration system.

- Read by: dev-manager, all Superpower agents (via their DEV-MANAGER-GATE sections), Claude (fallback mode)
- Referenced by: `CLAUDE.md` Sections 1–4, all SKILL.md governance sections
- Project-agnostic — do not modify this file per project
- Project-specific notes belong in `state.json → project.notes`

---

## 1. Stages

| Stage | Meaning | Entry condition |
|---|---|---|
| `PLANNING` | Spec and plan being written; no code touched | Default on project start |
| `DEVELOPING` | Code/scripts/docs being created or edited | `gates.plan_approved = true` |
| `REVIEW` | Implementation declared done; agents sign off | `gates.implementation_complete = true` |
| `FINALIZED` | All gates passed; task closed | All review gates passed |

**No agent may advance the stage without dev-manager updating `state.json` first.**

---

## 2. Complete Trigger Chain

```
User request arrives
        │
        ▼
CLAUDE.md: trigger phrase detected → re-read state.json
        │
        ├── "plan", "design", "feature", "build", "create"
        │         ▼
        │    ╔══════════════════════════════════╗
        │    ║         PLANNING STAGE           ║
        │    ╚══════════════════════════════════╝
        │         │
        │         ▼
        │    brainstorming
        │    (clarify intent → propose approaches → write spec)
        │    saves spec → .dev-manager/plans/
        │    updates state.json: stage = PLANNING
        │         │
        │         ▼
        │    writing-plans
        │    (reads spec → writes full plan with exact code,
        │     test commands, commit steps)
        │    saves plan → .dev-manager/plans/
        │    updates state.json: plan_approved = false
        │         │
        │         ▼
        │    dev-manager dispatches: plan-inspector
        │         │
        │         ▼
        │    dev-manager dispatches: codex-plan-inspector (optional)
        │    ┌──────────────────────────────────────┐
        │    │  Skip conditions (any one = skip):    │
        │    │    • user said "not dispatch",        │
        │    │      "skip codex", or "no codex"      │
        │    │    • codex CLI not on PATH            │
        │    │  If skipped → log + continue          │
        │    │                                       │
        │    │  Codex verdict?                       │
        │    │    APPROVE / SKIPPED                  │
        │    │      → proceed to plan approval gate  │
        │    │    REVISE → writing-plans revises     │
        │    │             → plan-inspector reruns   │
        │    │             → codex-plan-inspector    │
        │    │    REJECT → escalate to user;         │
        │    │             user decides: override    │
        │    │             or revise                 │
        │    └──────────────────────────────────────┘
        │    ┌──────────────────────────────────────┐
        │    │  Plan approved?                       │
        │    │    NO  → writing-plans revises        │
        │    │          back to plan-inspector       │
        │    │    YES → state.json:                  │
        │    │          plan_approved = true         │
        │    └──────────────────────────────────────┘
        │         │
        │         ▼
        │    dev-manager checks: data domain?
        │    (project.type / project.domain / plan keywords)
        │    ┌──────────────────────────────────────┐
        │    │  Data domain detected?                │
        │    │    NO  → skip data-manager silently   │
        │    │    YES → dispatch data-manager        │
        │    │          selects sub-agents:          │
        │    │          ├─ data-engineer?            │
        │    │          ├─ data-analyst?             │
        │    │          ├─ data-scientist?           │
        │    │          └─ database-optimizer?       │
        │    │          plan fits data domain?       │
        │    │            NO  → revisions → back to  │
        │    │                  writing-plans        │
        │    │            YES → approved             │
        │    └──────────────────────────────────────┘
        │         │
        │         ▼
        │    ╔══════════════════════════════════╗
        │    ║        DEVELOPING STAGE          ║
        │    ╚══════════════════════════════════╝
        │         │
        │         ▼
        │    dev-manager dispatches: subagent-driven-development
        │         │
        │         │  ┌─── Per-task loop ───────────────────────────────┐
        │         │  │                                                  │
        │         │  │  Are tasks independent?                          │
        │         │  │    YES → dispatching-parallel-agents             │
        │         │  │           runs tasks concurrently                │
        │         │  │    NO  → implementer subagent (serial)           │
        │         │  │                    │                             │
        │         │  │                    ▼                             │
        │         │  │  Something breaks / unexpected output?           │
        │         │  │    YES → ┌── systematic-debugging ──┐           │
        │         │  │          │  diagnose root cause      │           │
        │         │  │          │  fix found?               │           │
        │         │  │          │    YES → resume task ◄───-┘           │
        │         │  │          │    NO  → escalate to user             │
        │         │  │          └───────────────────────────            │
        │         │  │    NO  → continue                                │
        │         │  │                    │                             │
        │         │  │                    ▼                             │
        │         │  │  spec-reviewer subagent                          │
        │         │  │  (confirms code matches spec)                    │
        │         │  │         │                                        │
        │         │  │         ▼                                        │
        │         │  │  code-quality-reviewer subagent                  │
        │         │  │  (karapathy-style quality check per task)        │
        │         │  │         │                                        │
        │         │  │         ▼                                        │
        │         │  │  task complete → next task                       │
        │         │  └──────────────────────────────────────────────────┘
        │         │
        │         │  All tasks done?
        │         │    YES → state.json: implementation_complete = true
        │         │
        │         ▼
        │    ╔══════════════════════════════════╗
        │    ║          REVIEW STAGE            ║
        │    ╚══════════════════════════════════╝
        │         │
        │         ▼
        │    dev-manager dispatches: karapathy-guideline
        │    ┌──────────────────────────────────────┐
        │    │  Issues found?                        │
        │    │    YES → implementer fixes            │
        │    │          back to karapathy-guideline  │
        │    │    NO  → approved                     │
        │    └──────────────────────────────────────┘
        │         │
        │         ▼
        │    dev-manager dispatches: codebase-orchestrator
        │    (optional — only if in state.json → project.active_agents)
        │    ┌──────────────────────────────────────┐
        │    │  Enabled?                             │
        │    │    NO  → skip, log, continue          │
        │    │    YES → map repo → propose fixes     │
        │    │          await user approval          │
        │    │          execute approved actions     │
        │    │          critical issues found?       │
        │    │            YES → offer rollback or    │
        │    │                  fix-forward          │
        │    │            NO  → approved             │
        │    └──────────────────────────────────────┘
        │         │
        │         ▼
        │    dev-manager dispatches: requesting-code-review
        │    (generates structured review request for user)
        │         │
        │         ▼
        │    [User reviews and responds]
        │         │
        │         ▼
        │    dev-manager dispatches: receiving-code-review
        │    (processes user feedback → fixes applied if needed)
        │         │
        │         ▼
        │    dev-manager dispatches: system-checker
        │    ┌──────────────────────────────────────┐
        │    │  Issues found?                        │
        │    │    YES → implementer fixes            │
        │    │          back to system-checker       │
        │    │    NO  → approved                     │
        │    └──────────────────────────────────────┘
        │         │
        │         ▼
        │    dev-manager dispatches: verification-before-completion
        │    ┌──────────────────────────────────────┐
        │    │  All verification commands pass?      │
        │    │    YES → state.json:                  │
        │    │          final_evals_passed = true    │
        │    │          task_closed = true           │
        │    │          stage = FINALIZED            │
        │    │    NO  → back to implementer → fix   │
        │    └──────────────────────────────────────┘
        │
        ├── "bug", "error", "failing", "not working",
        │   "unexpected", "broken", "exception", "traceback"
        │         ▼
        │    ╔══════════════════════════════════╗
        │    ║    FLOATING: systematic-debugging ║
        │    ╚══════════════════════════════════╝
        │    (fires in ANY stage — does not change stage)
        │    diagnoses root cause → applies fix
        │    returns control to dev-manager at same stage
        │
        └── "create skill", "write skill",
            "improve skill", "update skill"
                  ▼
            ╔══════════════════════════════════╗
            ║    FLOATING: writing-skills      ║
            ╚══════════════════════════════════╝
            (fires in ANY stage — does not change stage)
            creates or improves a SKILL.md file
            returns control to dev-manager at same stage
```

---

## 3. Agent Registry

| Agent | Category | Stage | Dispatched by | Returns to |
|---|---|---|---|---|
| `brainstorming` | Planning | PLANNING | CLAUDE.md trigger | `writing-plans` |
| `writing-plans` | Planning | PLANNING | `brainstorming` | `dev-manager` |
| `plan-inspector` | Planning | PLANNING | `dev-manager` | `dev-manager` |
| `codex-plan-inspector` | Planning (optional) | PLANNING | `dev-manager` (after `plan-inspector`) | `dev-manager` |
| `data-manager/*` | Planning (domain gate) | PLANNING | `dev-manager` | `dev-manager` |
| `subagent-driven-development` | Implementation | DEVELOPING | `dev-manager` | `dev-manager` |
| `dispatching-parallel-agents` | Implementation | DEVELOPING | `subagent-driven-development` | `subagent-driven-development` |
| `executing-plans` | Implementation | DEVELOPING | `dev-manager` (alt path) | `dev-manager` |
| `systematic-debugging` | **Floating** | ANY | CLAUDE.md trigger or `subagent-driven-development` | caller |
| `karapathy-guideline` | Review | REVIEW | `dev-manager` | `dev-manager` |
| `codebase-admin/codebase-orchestrator` | Review (optional) | REVIEW | `dev-manager` | `dev-manager` |
| `requesting-code-review` | Review | REVIEW | `dev-manager` | `dev-manager` |
| `receiving-code-review` | Review | REVIEW | `dev-manager` | `dev-manager` |
| `system-checker` | Review | REVIEW | `dev-manager` | `dev-manager` |
| `verification-before-completion` | Review (final) | REVIEW | `dev-manager` | closes task |
| `writing-skills` | **Floating** | ANY | CLAUDE.md trigger | caller |

---

## 4. Path Selection

```
Has written plan before any edits?
  YES → Path A — full workflow (brainstorming → writing-plans → plan-inspector → developing → review)

  NO  → Path B — small fix, no plan
         skip brainstorming and writing-plans
         skip plan-inspector
         go straight to DEVELOPING
         karapathy-guideline and system-checker still run

User explicitly calls an agent by name → Path C
         confirm stage context
         run that agent
         remind user which agents still remain in the workflow
```

---

## 5. Floating Agent Rules

Floating agents (`systematic-debugging`, `writing-skills`) behave as **interrupts**:

1. They may fire at any stage without breaking the workflow
2. They **do not** update `workflow.stage` in `state.json`
3. They **do not** set or clear any gate flags
4. They append to `audit_trail` with `stage_before == stage_after`
5. When done, they explicitly return control:
   > "Returning control to dev-manager. Current stage: [STAGE]."
6. Dev-manager resumes from exactly where it was

---

## 6. Stage Gate Rules

| Transition | Requires |
|---|---|
| PLANNING → DEVELOPING | `gates.plan_approved = true` |
| DEVELOPING → REVIEW | `gates.implementation_complete = true` |
| REVIEW → FINALIZED | `gates.final_evals_passed = true` AND `gates.task_closed = true` |

**No transition may happen without dev-manager writing the new stage to `state.json` first.**

---

## 7. Trigger Phrases (CLAUDE.md auto re-check)

On any of the following, re-read `state.json` before acting:

**Planning triggers:**
`plan`, `design`, `feature`, `build`, `create`, `write`, `add`, `implement`

**Development triggers:**
`modify`, `update`, `fix`, `refactor`, `edit`, `change`

**Debugging triggers (floating):**
`bug`, `error`, `failing`, `not working`, `unexpected`, `broken`, `exception`, `traceback`

**Skill triggers (floating):**
`create skill`, `write skill`, `improve skill`, `update skill`

---

## 8. Handshake Protocol

`handshake_required: true` in `state.json` means no subagent may begin implementation
until dev-manager has confirmed the gate.

```
Superpower / writing-plans saves plan → .dev-manager/plans/
        │
        ▼
dev-manager reads plan → runs plan-inspector
        │
        ▼
dev-manager runs codex-plan-inspector (optional — skipped if user opted out or CLI absent)
        │
        ▼
plan_approved = true → DEVELOPING unlocked
        │
        ▼
subagent-driven-development begins
        │
        ▼
implementation_complete = true → REVIEW unlocked
        │
        ▼
karapathy-guideline → requesting-code-review → receiving-code-review
→ system-checker → verification-before-completion
        │
        ▼
task_closed = true → FINALIZED
```

---

## 9. Audit Trail Format

Every agent appends to `state.json → audit_trail` on entry and exit:

```json
{
  "timestamp": "ISO-8601",
  "agent": "agent-name",
  "action": "short description of what was done",
  "stage_before": "PLANNING",
  "stage_after": "DEVELOPING",
  "evals": "8/8 passing | skipped | n/a"
}
```

Floating agents always have `stage_before == stage_after`.

---

## 10. Superpower ON vs OFF

| | Superpower ON | Superpower OFF |
|---|---|---|
| Who dispatches agents | Superpower plugin | Claude reads each SKILL.md manually |
| Behaviour | Identical | Identical |
| state.json updates | Same | Same |
| Trigger source | Plugin automates | CLAUDE.md trigger phrases + DEV-MANAGER-GATE sections |
| Speed | Faster | Slightly slower (file reads per agent) |

When Superpower is OFF, Claude reads `.agents/superpower/<agent-name>/SKILL.md`
and follows it as if the plugin had dispatched it. The workflow is identical.

---

## 11. Idempotency — Safe Session Restarts

Before dispatching any agent, dev-manager checks `state.json → agents.<name>`:

```
approved == true  AND  last_run != ""  →  agent already completed this cycle
```

**Behaviour:**
- If already approved → offer to skip or re-run, never silently re-dispatch
- `run_count` increments on every dispatch (diagnostic only, not a gate)
- This makes session restarts safe — no agent runs twice accidentally

**When to reset:**
- User explicitly requests re-run
- Plan changes mid-implementation (plan-inspector must re-run)
- System-checker fails after karapathy-guideline approved (karapathy re-runs too)

---

## 12. Rollback Checkpoint Protocol

### Save points

A checkpoint is saved after each gate passage:

| Gate passed | Checkpoint saved |
|---|---|
| `plan_approved = true` | End of PLANNING |
| `implementation_complete = true` | End of DEVELOPING |
| `final_evals_passed = true` | End of REVIEW |

Stored in `state.json → workflow.rollback_checkpoint` (most recent only).

### Rollback trigger conditions

Offer rollback when:
- system-checker finds issues requiring fundamental rework (not just minor fixes)
- karapathy-guideline identifies architectural problems introduced during implementation
- User explicitly requests "go back to [stage]"

### Rollback procedure

```
User confirms rollback
        │
        ▼
Restore gates from rollback_checkpoint.gates_snapshot
        │
        ▼
Restore agent statuses from rollback_checkpoint.agents_snapshot
        │
        ▼
Set workflow.stage = rollback_checkpoint.stage
Set workflow.stage_entered_at = now
        │
        ▼
Append to audit_trail: rollback event with reason
        │
        ▼
Resume workflow from restored stage
```

**Never rollback silently — always confirm with user first.**

---

## 13. Stall Detection

On every session start, dev-manager checks:

```
hours_since(workflow.stage_entered_at) > workflow.stall_threshold_hours
```

Default threshold: 24 hours. Configurable per project in `state.json`.

**Stall response:**
> "⚠️ Project has been in [STAGE] for [N] hours (threshold: [T]h).
> Continue from current position, or review state first?"

**Blocked agent detection:**
```
agents.<name>.blocked_since != ""  →  agent is stuck
```

Report blocked agents immediately on session start before any dispatch.

### Timestamp fields managed by dev-manager

| Field | Written when |
|---|---|
| `workflow.stage_entered_at` | Every time `workflow.stage` changes |
| `workflow.last_updated` | Every time any field in state.json changes |
| `agents.<name>.last_run` | Every time an agent is dispatched |
| `agents.<name>.blocked_since` | When agent returns BLOCKED; cleared on resolution |
