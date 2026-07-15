---
name: init-project
description: >
  Use when starting a brand new project with the edward-workflow plugin.
  Triggers on: "init project", "set up project", "start new project",
  "scaffold project", "new pipeline", or any first-session request where
  .dev-manager/state.json does not yet exist in the project root.
  Creates governance files and asks the minimum questions needed to configure
  them. Run once per project before planning or implementation begins.
compatibility: Codex, Claude Code, Cowork. Requires write access to project root.
---

# Init Project

You are the `init-project` skill. Scaffold the edward-workflow governance
structure into a new project root, then hand control to `dev-manager`.

If `.dev-manager/state.json` already exists, stop and say:

```text
Project already initialised. Use dev-manager to continue.
```

---

## 1. Pre-flight

1. Check whether `.dev-manager/state.json` exists in the project root.
   - Exists: stop, do not overwrite, and report to the user.
   - Missing: proceed.
2. Confirm the project root path with the user if it is ambiguous.

---

## 2. Ask the User

Ask these questions in a single message:

```text
To initialise your project I need a few details:

1. Project name
2. Project type (biomechanics | bioinformatics | data-science | data-engineering | web | script | other)
3. Domain (e.g. clinical biomechanics | genomics | sports science | blank if not applicable)
4. One-line description
5. Primary language and key libraries
6. Environment / package manager
7. Test command, or blank if none yet
8. Files / folders the agent must never modify
9. Enable codebase-orchestrator? (yes / no)
10. Preferred implementation mode? (codex / superpowers)
```

Wait for all answers before proceeding.

---

## 3. Scaffold Files

Create only the following files/directories.

### 3.1 `.dev-manager/state.json`

Use this schema. Fill `project` from the user's answers. Leave runtime fields at
their defaults; `dev-manager` manages them.

```json
{
  "_schema_version": "1.1",
  "project": {
    "name": "<answer 1>",
    "type": "<answer 2>",
    "domain": "<answer 3>",
    "description": "<answer 4>",
    "file_patterns": ["<answer 8 split into array, empty array if none>"],
    "active_agents": ["plan-inspector", "karpathy-guidelines", "system-checker"],
    "active_plan": "",
    "notes": ""
  },
  "workflow": {
    "stage": "PLANNING",
    "path": null,
    "handshake_required": true,
    "superpower_available": null,
    "implementation_mode": "<answer 10 or empty>",
    "last_updated": "",
    "session_id": "",
    "stage_entered_at": "",
    "stall_threshold_hours": 24,
    "rollback_checkpoint": {
      "stage": "",
      "saved_at": "",
      "gates_snapshot": {},
      "agents_snapshot": {},
      "description": ""
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
    "plan-inspector": { "status": "pending", "last_run": "", "run_count": 0, "approved": false, "blocked_since": "", "blocked_reason": "", "notes": "" },
    "data-manager": { "status": "pending", "last_run": "", "run_count": 0, "approved": false, "blocked_since": "", "blocked_reason": "", "notes": "Auto-activates on data domain projects. Silent skip otherwise." },
    "karpathy-guidelines": { "status": "pending", "last_run": "", "run_count": 0, "approved": false, "blocked_since": "", "blocked_reason": "", "notes": "" },
    "system-checker": { "status": "pending", "last_run": "", "run_count": 0, "approved": false, "blocked_since": "", "blocked_reason": "", "notes": "" },
    "codebase-orchestrator": { "status": "pending", "last_run": "", "run_count": 0, "approved": false, "blocked_since": "", "blocked_reason": "", "notes": "Add 'codebase-orchestrator' to project.active_agents to enable." }
  },
  "evals": {
    "suite_path": ".dev-manager/evals/evals.json",
    "results_path": ".dev-manager/results/",
    "last_run": "",
    "iteration": 0,
    "passing": null,
    "total": null,
    "skipped": false,
    "skip_reason": ""
  },
  "audit_trail": []
}
```

If the user enabled `codebase-orchestrator`, add it to
`project.active_agents`.

### 3.2 `.dev-manager/workflow.md`

Copy verbatim from `skills/references/workflow.md`.

### 3.3 `.dev-manager/checklist.md`

Copy verbatim from `skills/references/checklist.md`.

### 3.4 `project.config.md`

Copy from `skills/references/project.config.md` and fill it using the user's
answers.

### 3.5 Optional host guidance

Create `AGENTS.md` only when the target host/project uses it and the user wants
persistent repository instructions.

### 3.6 Directories

Create:

```text
.dev-manager/plans/
.dev-manager/evals/
.dev-manager/results/
```

---

## 4. Confirm to the User

After all files are created, report:

```text
Project initialised: <project name>

Files created:
  .dev-manager/state.json
  .dev-manager/workflow.md
  .dev-manager/checklist.md
  .dev-manager/plans/
  .dev-manager/evals/
  .dev-manager/results/
  project.config.md

Active agents: <list from state.json>
Data domain: <detected type>
```

Then announce:

```text
Handing control to dev-manager. Ready to begin PLANNING stage.
```

---

## 5. Never Do

- Never overwrite an existing `state.json`
- Never modify installed plugin files or host configuration without explicit user approval
- Never start PLANNING, DEVELOPING, or REVIEW work directly
- Never assume project type; ask the user
- Never create files beyond those listed above
