---
name: init-project
description: >
  Use when starting a brand new project with the edward-workflow plugin.
  Triggers on: "init project", "set up project", "start new project",
  "scaffold project", "new pipeline", or any first-session request where
  .dev-manager/state.json does not yet exist in the project root.
  Creates all required governance files and asks the user the minimum
  questions needed to configure them. Run once per project, before any
  planning or implementation begins.
compatibility: Claude Code, Cowork. Requires write access to project root.
---

# Init Project

You are the **init-project** skill. Your job is to scaffold the full
edward-workflow governance structure into a new project root, then hand
control to **dev-manager** to begin work.

Run once per project. If `.dev-manager/state.json` already exists, stop
and say: *"Project already initialised. Use dev-manager to continue."* 

---

## 1. Pre-flight Check 

Before doing anything:

1. Check whether `.dev-manager/state.json` exists in the project root.
   - Exists → stop, do not overwrite. Report to user.
   - Missing → proceed.

2. Confirm the project root path with the user if it is ambiguous.

---

## 2. Ask the User (minimum viable questions)

Ask these questions in a single message — do not drip-feed them:

```
To initialise your project I need a few details:

1. Project name  (e.g. "Gait Analysis Pipeline v2")
2. Project type  (biomechanics | bioinformatics | data-science |
                  data-engineering | web | script | other)
3. Domain        (e.g. clinical biomechanics | genomics | sports science |
                  leave blank if not applicable)
4. One-line description  (what does this project produce or do?)
5. Primary language & key libraries
   (e.g. Python 3.11 — NumPy, Pandas, ezc3d)
6. Environment / package manager
   (e.g. conda env `biomech` | pip + requirements.txt | poetry)
7. Test command  (e.g. `pytest tests/ -v` | leave blank if none yet)
8. Files / folders Claude must NEVER modify
   (e.g. data/raw/**, references/, *.c3d — or "none" if not applicable)
9. Enable codebase-orchestrator?  (yes / no — optional deep review agent)
10. Using Codex CLI for implementation?  (yes / no)
```

Wait for all answers before proceeding.

---

## 3. Scaffold Files

Create the following files from the user's answers.
**Do not create files the user did not ask for.**

### 3.1 `.dev-manager/state.json`

Use the schema below exactly. Fill in the `project` block from the user's
answers. Leave all `workflow`, `gates`, `agents`, and `audit_trail` fields
at their defaults — dev-manager manages those at runtime.

```json
{
  "_schema_version": "1.1",
  "project": {
    "name": "<answer 1>",
    "type": "<answer 2>",
    "domain": "<answer 3>",
    "description": "<answer 4>",
    "file_patterns": ["<answer 8 — split into array, empty array if none>"],
    "active_agents": ["plan-inspector", "karapathy-guideline", "system-checker"],
    "active_plan": "",
    "notes": ""
  },
  "workflow": {
    "stage": "PLANNING",
    "path": null,
    "handshake_required": true,
    "superpower_available": null,
    "last_updated": "",
    "session_id": "",
    "stage_entered_at": "",
    "stall_threshold_hours": 24,
    "rollback_checkpoint": {
      "stage": "", "saved_at": "", "gates_snapshot": {},
      "agents_snapshot": {}, "description": ""
    }
  },
  "gates": {
    "plan_approved": false,
    "pre_impl_evals_passed": false,
    "implementation_complete": false,
    "post_impl_evals_passed": false,
    "final_evals_passed": false,
    "task_closed": false
  },
  "agents": {
    "plan-inspector":        { "status": "pending", "last_run": "", "run_count": 0, "approved": false, "blocked_since": "", "blocked_reason": "", "notes": "" },
    "data-manager":          { "status": "pending", "last_run": "", "run_count": 0, "approved": false, "blocked_since": "", "blocked_reason": "", "notes": "Auto-activates on data domain projects. Silent skip otherwise." },
    "karapathy-guideline":   { "status": "pending", "last_run": "", "run_count": 0, "approved": false, "blocked_since": "", "blocked_reason": "", "notes": "" },
    "system-checker":        { "status": "pending", "last_run": "", "run_count": 0, "approved": false, "blocked_since": "", "blocked_reason": "", "notes": "" },
    "codebase-orchestrator": { "status": "pending", "last_run": "", "run_count": 0, "approved": false, "blocked_since": "", "blocked_reason": "", "notes": "Add 'codebase-orchestrator' to project.active_agents to enable." }
  },
  "evals": {
    "suite_path": ".claude/evals/evals.json",
    "results_path": ".claude/results/",
    "last_run": "", "iteration": 0,
    "passing": null, "total": null,
    "skipped": false, "skip_reason": ""
  },
  "audit_trail": []
}
```

If user answered **yes** to codebase-orchestrator (question 9), add
`"codebase-orchestrator"` to `project.active_agents`.

### 3.2 `.dev-manager/workflow.md`

Copy verbatim from the plugin's `references/workflow.md`.
This file is project-agnostic — do not modify it.

### 3.3 `.dev-manager/checklist.md`

Copy verbatim from the plugin's `references/checklist.md`.
The per-project customisation section at the bottom may be pre-filled
with domain-specific examples if `project.type` is `biomechanics` or
`bioinformatics` — uncomment the relevant example block.

### 3.4 `project.config.md`

Copy from the plugin's `references/project.config.md` template.
Fill in all sections using the user's answers:

| Section | Filled from |
|---|---|
| 1. Project Identity | answers 1–4 |
| 2. Language & Framework | answers 5–6 |
| 3. Coding Standards | leave as placeholder — user fills later |
| 4. Files & Data — Never Modify | answer 8 |
| 5. Domain-Specific Rules | uncomment domain block matching answer 2 |
| 6. Active Agents | reflect active_agents from state.json |
| 7. Evals | leave defaults |

### 3.5 `AGENTS.md` (only if answer 10 is "yes")

Copy from the plugin's `references/AGENTS.md` template.
Fill in Section 4 using answers 1, 5–6, 8.

### 3.6 `.dev-manager/plans/` (empty directory)

Create the directory. It will hold plans generated by writing-plans.

### 3.7 `.claude/evals/` and `.claude/results/` (empty directories)

Create both. They hold evals and results managed by dev-manager.

---

## 4. Confirm to the User

After all files are created, report:

```
✅ Project initialised: <project name>

Files created:
  .dev-manager/state.json       ← project state and gate tracker
  .dev-manager/workflow.md      ← agent dispatch reference (read-only)
  .dev-manager/checklist.md     ← stage gate checklist
  .dev-manager/plans/           ← plan handoff directory
  project.config.md             ← project identity and rules
  .claude/evals/                ← eval suite location
  .claude/results/              ← eval results location

Active agents: <list from state.json>
Data domain: <detected type> — data-manager will auto-activate on data tasks

Next step: tell me what you want to build and dev-manager will take it
from there.
```

---

## 5. Hand Off

After confirmation, announce:

> "Handing control to **dev-manager**. Ready to begin PLANNING stage."

Dev-manager takes over from here for all subsequent work.

---

## 6. What Init-Project Never Does

- Never overwrites an existing `state.json`
- Never modifies files inside `.agents/` or the plugin itself
- Never starts PLANNING, DEVELOPING, or any workflow stage
- Never makes assumptions about project type — always asks
- Never creates files beyond those listed in Section 3
