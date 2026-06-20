---
name: codex-plan-inspector
description: >
  External Codex CLI plan inspector — dispatched automatically by dev-manager immediately
  after plan-inspector completes, as a second-opinion pass using the user's locally installed
  Codex CLI. This agent is OPTIONAL and dispatched by default. Skip it when: the user says
  "not dispatch", "skip codex", "no codex", or "skip codex-plan-inspector"; or when Codex
  CLI is not found on PATH. Never trigger this skill directly — it is always invoked by
  dev-manager as a post-plan-inspector step in the PLANNING stage.
---

# Codex Plan Inspector

An external, CLI-driven second-opinion agent that runs **after** `plan-inspector` finishes.
It submits the revised plan (Subagent 3 output from `plan-inspector`) to the user's locally
installed Codex CLI and surfaces any additional issues or confirmations before the plan is
approved.

---

## Position in Workflow

```
plan-inspector (Subagents 1→2→3 complete)
        │
        ▼
dev-manager dispatches: codex-plan-inspector   ← YOU ARE HERE
        │
        ├── Skip condition met? → log + skip → return to dev-manager
        │
        ▼
Codex CLI invoked with revised plan
        │
        ▼
Output parsed and presented to user
        │
        ▼
dev-manager: plan_approved gate decision
```

---

## Pre-Dispatch Checks (run in order, stop at first skip condition)

### Check 1 — User opt-out

Scan the current conversation for any of the following phrases (case-insensitive):

- `"not dispatch"`
- `"skip codex"`
- `"no codex"`
- `"skip codex-plan-inspector"`
- `"skip external inspector"`

**If found →** log and skip:
```
[codex-plan-inspector] Skipped — user opted out. Returning control to dev-manager.
```

### Check 2 — Codex CLI availability

Run:
```bash
which codex 2>/dev/null || codex --version 2>/dev/null
```

If the command fails or returns nothing, Codex CLI is not installed.

**If not found →** log and skip:
```
[codex-plan-inspector] Skipped — Codex CLI not found on PATH.
Install via: npm install -g @openai/codex  (or your IDE's Codex extension)
Returning control to dev-manager.
```

**If found →** proceed to Invocation.

---

## Invocation

### Input

Use the **Revised Plan** produced by `plan-inspector` Subagent 3 as the input.
If no revised plan exists (plan-inspector was skipped or produced no output), fall back
to the latest raw plan from `.dev-manager/plans/`.

### Prompt to Codex CLI

Construct the prompt as follows:

```
You are an independent plan reviewer. You have no memory of prior conversation.

Below is a development plan that has already been reviewed for risks and ambiguities
by an internal team. Your job is to provide an independent second opinion.

Review the plan and answer:
1. Are there any risks or issues the prior reviewers MISSED?
2. Are there any steps that are logically inconsistent or in the wrong order?
3. Are there any missing prerequisites or setup steps?
4. Is the overall approach sound? If not, what would you change?
5. Final verdict: APPROVE (safe to proceed) | REVISE (specific changes needed) | REJECT (fundamental problems)

Plan:
---
{revised_plan_text}
---

Respond in this format:

## Codex Review

### Additional Risks / Missed Issues
(numbered list, or "None identified")

### Ordering / Consistency Issues
(numbered list, or "None identified")

### Missing Prerequisites
(numbered list, or "None identified")

### Overall Assessment
(1–3 sentences on the approach)

### Verdict
APPROVE | REVISE | REJECT — (one-line reason)
```

### CLI invocation

```bash
echo "{escaped_prompt}" | codex --no-interactive
```

Or, if Codex CLI supports a `--prompt` flag:

```bash
codex --no-interactive --prompt "{escaped_prompt}"
```

Capture stdout. Timeout: **120 seconds**. If it exceeds the timeout:
```
[codex-plan-inspector] Timeout after 120s. Skipping external review.
Returning control to dev-manager.
```

---

## Output Parsing

After Codex CLI returns, parse the `### Verdict` line:

| Verdict | Action |
|---------|--------|
| `APPROVE` | Present full output to user. Recommend proceeding. |
| `REVISE`  | Present full output to user. List specific revision items. Recommend returning to `writing-plans`. |
| `REJECT`  | Present full output to user. Escalate to user — do not auto-reject. User decides whether to revise or override. |

---

## Output to User

Present results in this block immediately after `plan-inspector`'s report:

```
---

## 🤖 Codex CLI Plan Review (External Inspector)

{full Codex CLI output}

---

### Codex Verdict: {APPROVE | REVISE | REJECT}

{One of the following:}

[APPROVE]
Codex confirms the plan is sound. Proceeding to plan approval gate.

[REVISE]
Codex identified issues that should be addressed before proceeding:
  • {item 1}
  • {item 2}
Recommendation: return to writing-plans to resolve these, then re-run plan-inspector + codex-plan-inspector.

[REJECT]
Codex flagged fundamental problems. Review the output above.
This is an external opinion — you may override it and proceed if you disagree.
Please confirm: proceed anyway, or revise the plan?
```

---

## Handoff Back to dev-manager

After presenting output, return control explicitly:

```
[codex-plan-inspector] Complete. Returning control to dev-manager.
Verdict: {APPROVE | REVISE | REJECT | SKIPPED}
```

Dev-manager then decides the `plan_approved` gate:
- `APPROVE` or `SKIPPED` → dev-manager may set `plan_approved = true` (pending any remaining user confirmation)
- `REVISE` → dev-manager routes back to `writing-plans`
- `REJECT` → dev-manager awaits explicit user instruction before setting any gate

---

## Audit Trail Entry

Append to `state.json → audit_trail`:

```json
{
  "timestamp": "ISO-8601",
  "agent": "codex-plan-inspector",
  "action": "External Codex CLI plan review — Verdict: {APPROVE|REVISE|REJECT|SKIPPED}",
  "stage_before": "PLANNING",
  "stage_after": "PLANNING",
  "evals": "n/a"
}
```

This agent does **not** change `workflow.stage`.

---

## Behaviour Rules

| Rule | Detail |
|------|--------|
| **Optional by default** | Dispatched automatically unless a skip condition is met |
| **Never blocks permanently** | A REJECT verdict escalates to the user, never auto-halts the workflow |
| **Stateless** | Receives only the revised plan text — no conversation history |
| **No code changes** | This agent reads and reviews only — it never edits files |
| **Sequential** | Always runs after plan-inspector; never in parallel with it |
| **Idempotency** | If `state.json → agents.codex-plan-inspector.approved == true` and `last_run != ""`, offer to skip or re-run rather than auto-dispatching again |

---

## state.json Agent Entry

Dev-manager must maintain this entry:

```json
"codex-plan-inspector": {
  "approved": false,
  "last_run": "",
  "run_count": 0,
  "verdict": "",
  "skipped": false,
  "skip_reason": "",
  "blocked_since": ""
}
```

`verdict` stores the last Codex verdict string (`APPROVE`, `REVISE`, `REJECT`, or `SKIPPED`).
