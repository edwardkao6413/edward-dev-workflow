---
name: verification-before-completion
description: Use when about to claim work is complete, fixed, or passing, before committing or creating PRs - requires running verification commands and confirming output before making any success claims; evidence before assertions always
---

# Verification Before Completion

## Overview

Claiming work is complete without verification is dishonesty, not efficiency.

**Core principle:** Evidence before claims, always.

**Violating the letter of this rule is violating the spirit of this rule.**

## The Iron Law

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

If you haven't run the verification command in this message, you cannot claim it passes.

## The Gate Function

```
BEFORE claiming any status or expressing satisfaction:

1. IDENTIFY: What command proves this claim?
2. RUN: Execute the FULL command (fresh, complete)
3. READ: Full output, check exit code, count failures
4. VERIFY: Does output confirm the claim?
   - If NO: State actual status with evidence
   - If YES: State claim WITH evidence
5. ONLY THEN: Make the claim

Skip any step = lying, not verifying
```

## Common Failures

| Claim | Requires | Not Sufficient |
|-------|----------|----------------|
| Tests pass | Test command output: 0 failures | Previous run, "should pass" |
| Linter clean | Linter output: 0 errors | Partial check, extrapolation |
| Build succeeds | Build command: exit 0 | Linter passing, logs look good |
| Bug fixed | Test original symptom: passes | Code changed, assumed fixed |
| Regression test works | Red-green cycle verified | Test passes once |
| Agent completed | VCS diff shows changes | Agent reports "success" |
| Requirements met | Line-by-line checklist | Tests passing |

## Red Flags - STOP

- Using "should", "probably", "seems to"
- Expressing satisfaction before verification ("Great!", "Perfect!", "Done!", etc.)
- About to commit/push/PR without verification
- Trusting agent success reports
- Relying on partial verification
- Thinking "just this once"
- Tired and wanting work over
- **ANY wording implying success without having run verification**

## Rationalization Prevention

| Excuse | Reality |
|--------|---------|
| "Should work now" | RUN the verification |
| "I'm confident" | Confidence ≠ evidence |
| "Just this once" | No exceptions |
| "Linter passed" | Linter ≠ compiler |
| "Agent said success" | Verify independently |
| "I'm tired" | Exhaustion ≠ excuse |
| "Partial check is enough" | Partial proves nothing |
| "Different words so rule doesn't apply" | Spirit over letter |

## Key Patterns

**Tests:**
```
✅ [Run test command] [See: 34/34 pass] "All tests pass"
❌ "Should pass now" / "Looks correct"
```

**Regression tests (TDD Red-Green):**
```
✅ Write → Run (pass) → Revert fix → Run (MUST FAIL) → Restore → Run (pass)
❌ "I've written a regression test" (without red-green verification)
```

**Build:**
```
✅ [Run build] [See: exit 0] "Build passes"
❌ "Linter passed" (linter doesn't check compilation)
```

**Requirements:**
```
✅ Re-read plan → Create checklist → Verify each → Report gaps or completion
❌ "Tests pass, phase complete"
```

**Agent delegation:**
```
✅ Agent reports success → Check VCS diff → Verify changes → Report actual state
❌ Trust agent report
```

## Why This Matters

From 24 failure memories:
- your human partner said "I don't believe you" - trust broken
- Undefined functions shipped - would crash
- Missing requirements shipped - incomplete features
- Time wasted on false completion → redirect → rework
- Violates: "Honesty is a core value. If you lie, you'll be replaced."

## When To Apply

**ALWAYS before:**
- ANY variation of success/completion claims
- ANY expression of satisfaction
- ANY positive statement about work state
- Committing, PR creation, task completion
- Moving to next task
- Delegating to agents

**Rule applies to:**
- Exact phrases
- Paraphrases and synonyms
- Implications of success
- ANY communication suggesting completion/correctness

## The Bottom Line

**No shortcuts for verification.**

Run the command. Read the output. THEN claim the result.

This is non-negotiable.

---

## Dev-Manager Governance

<DEV-MANAGER-GATE>
verification-before-completion is the FINAL gate in the REVIEW stage.
It runs AFTER system-checker, as the last step before dev-manager closes the task.
It does NOT replace system-checker — it confirms the claim that system-checker approved.
</DEV-MANAGER-GATE>

### When verification passes

After all verification commands confirm success, update `state.json` and signal dev-manager to finalize:

```json
{
  "workflow": {
    "stage": "FINALIZED",
    "last_updated": "<ISO-8601 timestamp>"
  },
  "gates": {
    "final_evals_passed": true,
    "task_closed": true
  },
  "audit_trail": [
    {
      "timestamp": "<ISO-8601>",
      "agent": "verification-before-completion",
      "action": "All verification commands passed. Evidence confirmed. Signalling dev-manager to close task.",
      "stage_before": "REVIEW",
      "stage_after": "FINALIZED",
      "evals": "<N passing / M total>"
    }
  ]
}
```

Then announce:

> "Verification complete — all commands passed with evidence. **dev-manager** is closing the task. `state.json → stage: FINALIZED`."

### When verification fails

Do NOT update `state.json` to FINALIZED. Instead update the audit trail with the failure:

```json
{
  "audit_trail": [
    {
      "timestamp": "<ISO-8601>",
      "agent": "verification-before-completion",
      "action": "Verification FAILED. Blocking task closure. Returning to dev-manager for triage.",
      "stage_before": "REVIEW",
      "stage_after": "REVIEW",
      "evals": "<N passing / M total — FAILURES PRESENT>"
    }
  ]
}
```

Then signal dev-manager to re-triage (it will decide whether to re-run karapathy-guideline or escalate to the user).

---

## Dev-Manager Governance

<DEV-MANAGER-GATE>
verification-before-completion is the FINAL gate in the REVIEW stage.
It runs AFTER system-checker, as the last step before dev-manager closes the task.
It does NOT replace system-checker — it confirms the claim that system-checker approved.
</DEV-MANAGER-GATE>

### When verification passes

After all verification commands confirm success, update `state.json` and signal dev-manager to finalize:

```json
{
  "workflow": {
    "stage": "FINALIZED",
    "last_updated": "<ISO-8601 timestamp>"
  },
  "gates": {
    "final_evals_passed": true,
    "task_closed": true
  },
  "audit_trail": [
    {
      "timestamp": "<ISO-8601>",
      "agent": "verification-before-completion",
      "action": "All verification commands passed. Evidence confirmed. Signalling dev-manager to close task.",
      "stage_before": "REVIEW",
      "stage_after": "FINALIZED",
      "evals": "<N passing / M total>"
    }
  ]
}
```

Then announce:

> "Verification complete — all commands passed with evidence. **dev-manager** is closing the task. `state.json → stage: FINALIZED`."

### When verification fails

Do NOT update `state.json` to FINALIZED. Instead log the failure in the audit trail:

```json
{
  "audit_trail": [
    {
      "timestamp": "<ISO-8601>",
      "agent": "verification-before-completion",
      "action": "Verification FAILED. Blocking task closure. Returning to dev-manager for triage.",
      "stage_before": "REVIEW",
      "stage_after": "REVIEW",
      "evals": "<N passing / M total — FAILURES PRESENT>"
    }
  ]
}
```

Then signal dev-manager to re-triage (it will decide whether to re-run karapathy-guideline or escalate to the user).
