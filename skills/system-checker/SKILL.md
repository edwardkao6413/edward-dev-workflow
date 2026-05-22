---
name: system-checker
description: >
  Automatically dispatch an independent subagent to validate the codebase after any conversation
  that changes features or functions — excluding comment-only edits. Use this skill whenever
  code has been modified, a new feature was added, a bug was fixed, or a refactor was performed.
  If a plugin/superpower is active, triggers after it completes; otherwise triggers immediately after the conversation ends. Also trigger when the user says things like
  "check the system", "make sure nothing broke", "run the system check", "validate the changes",
  or "test my code". Applies universally: data analytics pipelines, bioinformatics workflows,
  biomechanics tools, web apps, APIs, CLIs, notebooks, and any other codebase type.
---

# System Checker

An autonomous post-change validation agent. After any meaningful code change, dispatch an
independent subagent that knows only the latest conversation — it reads what changed, tests
the affected feature, traces all linked features, and iterates until everything passes.

---

## When to Activate

Activate **after** the primary work is done. This means:

- Any file edit that changes logic, structure, or behaviour (not just comments/docstrings)
- New feature added or existing feature removed
- Bug fix applied
- Refactor or dependency change

**Superpower/plugin is optional.** If one is active, wait for it to finish first, then activate.
If no superpower is in use, activate immediately once the conversation's code changes are written.

**Do NOT activate** for:
- Comment-only edits (`#`, `"""`, `//`, `/* */`)
- Whitespace / formatting changes only
- Documentation updates with no code change
- Config changes that have no runtime effect

---

## Subagent Briefing

Dispatch the subagent with **only the latest conversation** as context. The subagent follows
this protocol exactly:

```
You are a system-checker subagent. Your only context is the conversation excerpt provided.
Follow the System Checker Protocol below without deviation.
```

Pass the subagent:
1. The latest user↔assistant exchange (enough to understand the intended change)
2. The relevant file paths that were modified
3. The project type (data pipeline / web app / CLI / notebook / etc.) if determinable

---

## System Checker Protocol

### Step 0 — Change Detection

Diff the codebase against the pre-conversation state (or the last known clean state).

- If **no substantive change** → exit silently. Log: `[system-checker] No code changes detected.`
- If **comment-only change** → exit silently. Log: `[system-checker] Comment-only change. Skipped.`
- If **substantive change detected** → proceed to Step 1.

---

### Step 1 — Understand the Intended Change

Read the latest conversation to answer:

- **What feature/function was added, modified, or removed?** (call this the *Indicated Target*)
- **What was the expected behaviour before vs after?**
- **Are there explicit acceptance criteria or test cases mentioned by the user?**

Record the Indicated Target(s) clearly before moving on.

---

### Step 2 — Test the Indicated Target

Run tests against the Indicated Target:

- Use existing test files if present (`pytest`, `unittest`, `jest`, `cargo test`, etc.)
- If no tests exist, write minimal smoke tests that exercise the changed behaviour
- For data pipelines: run the pipeline on a minimal sample and validate output shape/values
- For notebooks: execute affected cells and assert on outputs

**Pass condition**: the Indicated Target behaves as described in the latest conversation.

**If FAIL**: diagnose, fix, re-test. Repeat until pass. Do not proceed to Step 3 until Step 2 passes.

---

### Step 3 — Map Linked Features/Functions

Identify everything connected to the Indicated Target:

- Functions/methods that **call** the Indicated Target
- Functions/methods **called by** the Indicated Target
- Modules that **import** the Indicated Target
- Shared state, config values, or data schemas the Indicated Target **reads or writes**
- Downstream pipeline stages or API endpoints that depend on it

Record the *Linked Set* — an ordered list of linked features/functions.

---

### Step 4 — 1-vs-1 Linked Testing

For each item in the Linked Set, test it **in isolation** against the Indicated Target:

```
For each linked_item in Linked Set:
    result = test(indicated_target ↔ linked_item)
    if result == PASS:
        continue
    else:
        → Step 4a: Root-cause analysis
        → Step 4b: Fix
        → Step 4c: Re-test (indicated_target ↔ linked_item)
        repeat until PASS
```

#### Step 4a — Root-Cause Analysis
Trace why the Indicated Target causes failure in the linked item:
- Changed interface / signature?
- Changed return type or data shape?
- Changed side-effect or shared state?
- Timing / ordering issue?

#### Step 4b — Fix
Apply the minimal fix:
- Prefer fixing the integration point over rewriting either component
- If the Indicated Target itself must change, re-run Step 2 before continuing

#### Step 4c — Re-Test
Confirm both the Indicated Target (Step 2 condition) AND the linked item now pass.

---

### Step 5 — Full Sweep (if any Step 4 fix was applied)

If **any** fix was made in Step 4, run a full sweep:

```
Test: Indicated Target ↔ ALL items in Linked Set simultaneously
```

- Use integration tests if available; otherwise run the full pipeline / application flow
- If ALL pass → proceed to Step 6
- If ANY fail → identify which linked item(s) still fail, return to Step 4 for those items, then repeat Step 5

Cap retries at **5 full sweeps**. If errors persist after 5 sweeps, surface a detailed failure
report to the user instead of continuing silently.

---

### Step 6 — Finish & Report

Emit a concise report to the user:

```
✅ System Check Complete
─────────────────────────────
Indicated Target : <name>
Linked Features  : <count> checked
Status           : All pass  /  N fixed  /  N still failing (see below)

[If fixes were applied]
Fixed:
  • <linked_item> — <one-line description of fix>

[If failures remain after retry cap]
Still failing:
  • <linked_item> — <error summary>
  → Recommended action: <suggestion>
```

---

## Project-Type Guidance

### Data Analytics / Biomechanics / Bioinformatics Pipelines
- Smoke test with a small representative sample (not the full dataset)
- Validate output column names, dtypes, and value ranges
- For statistical functions: assert on known-good reference outputs
- Treat notebook cells as functions — test affected cells in order

### Web App / API
- Test affected routes/endpoints with representative payloads
- Check auth middleware is not broken by the change
- Verify database schema compatibility if models changed

### CLI Tools
- Run the CLI with `--help` and at least one representative subcommand
- Assert on exit codes and stdout/stderr shape

### Shared Libraries / Utilities
- Run the full existing test suite, not just the changed module
- Pay special attention to public API surface

---

## Behaviour Rules

| Rule | Detail |
|------|--------|
| **Independence** | The subagent uses only the latest conversation + file diffs as context — no prior session memory |
| **Superpower sequencing** | If a plugin/superpower is active, wait for it to finish before starting. If none is active, start immediately after code changes are written. |
| **Minimal footprint** | Write test code to a temp location; do not commit test files unless user asks |
| **No silent failure** | If the retry cap is hit, always surface the failure report |
| **Fix scope** | Only fix code that is broken *because of* the Indicated Target change — do not refactor unrelated code |
| **Comment immunity** | Never flag comment-only changes as substantive |

---

## Quick Reference: Decision Tree

```
Conversation ends
      │
      ▼
Superpower/plugin active? ──Yes──► wait for it to finish
      │ No (or finished)
      ▼
Code changed (non-comment)?──No──► exit silently
      │ Yes
      ▼
Step 1: Identify Indicated Target
      │
      ▼
Step 2: Test Indicated Target ──FAIL──► fix → retry
      │ PASS
      ▼
Step 3: Map Linked Set
      │
      ▼
Step 4: 1-vs-1 tests ──FAIL──► root-cause → fix → retest
      │ all PASS
      ▼
Any Step 4 fixes? ──No──► Step 6 Report
      │ Yes
      ▼
Step 5: Full sweep ──FAIL──► back to Step 4
      │ PASS (or retry cap hit)
      ▼
Step 6: Report
```
