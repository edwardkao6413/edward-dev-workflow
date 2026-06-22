---
name: plan-inspector
description: >
  Dispatch 3 independent subagents to rigorously inspect a newly generated or updated plan against the
  actual codebase. Use this skill immediately whenever a plan is generated or updated — including
  implementation plans, refactor plans, migration plans, feature plans, or any structured set of
  development steps. Always trigger when the user says "generated a plan", "updated the plan",
  "here is the plan", "review this plan", or "inspect the plan". Especially important for data
  science and bioinformatics projects where data pipeline integrity must be preserved. Each subagent
  operates independently with no memory of prior conversation — only the codebase and the plan.
---

# Plan Inspector

Dispatch **3 independent subagents** to critically inspect the latest plan against the actual
codebase. Each subagent has a single focused role, operates with **no memory of prior conversation**,
and reads only the codebase (scripts, configs, docs) plus the plan text provided to it.

Subagents are dispatched **one at a time** (sequentially), not in parallel.

---

## Triggering

Trigger this skill whenever:
- A new plan has just been generated
- An existing plan has been updated or revised
- The user says "inspect the plan", "review this plan", "check the plan"

---

## UI Change Detection

**Before dispatching any subagent**, scan the plan for UI-related intent. Look for signals such as:

- Changing, updating, or redesigning any user interface component
- Optimising or polishing frontend layout, spacing, or visual hierarchy
- Modifying styles, themes, colours, fonts, or CSS/Tailwind classes
- Improving visual patterns, UX flows, or aesthetic appearance
- Any mention of: `layout`, `design`, `style`, `theme`, `UI`, `UX`, `frontend`, `component`, `responsive`, `animation`, `visual`

If **any** of these signals are present:

> **Ask the user before proceeding:**
>
> _"This plan includes UI changes. Would you like me to spin up a localhost browser preview to showcase examples of the imagined changes before implementation begins? I can generate **3 examples** by default (you can request more or fewer). Just say yes/no or specify a number."_

Wait for the user's response:
- If **yes** (or a number is given): note the count (default = 3) and flag for post-inspection localhost preview generation.
- If **no**: proceed directly to subagent dispatch.

After generating all previews, **do not ask the user to choose**. Instead:

> Review all generated examples yourself, reason about which one best matches the intent expressed in the user's plan or prompt, and **select it autonomously**. Present the chosen option to the user with a one-sentence rationale explaining why it best fits their vision — then proceed with that choice unless the user overrides it.

---

## Pre-Dispatch Setup

Before spawning any subagent, gather:

1. **The plan text** — the full, latest version of the plan (copy verbatim from the conversation or
   from a plan file if one exists on disk).
2. **Codebase snapshot** — identify the key files the subagent should read:
   - All scripts relevant to the plan's scope (e.g. `*.py`, `*.R`, `*.sh`, `*.sql`)
   - Config files (`*.yaml`, `*.json`, `*.toml`, `*.env`)
   - Pipeline definitions and workflow files
   - Relevant documentation or README files
   - Any existing test files

Provide each subagent with:
- The plan text (verbatim)
- A list of key file paths to read from the codebase
- Their specific role (see below)

---

## Subagent Roles

### Subagent 1 — Impact & Risk Analyst

**Dispatch first.**

**Persona**: A cautious senior engineer who has seen many refactors go wrong.

**Task**: Analyse the plan's impact on the codebase and identify risks.

**Instructions to pass verbatim to the subagent**:

```
You are a senior engineer doing an independent code review. You have NO memory of any prior
conversation. You will be given:
  (A) A plan (a set of proposed changes or steps)
  (B) Paths to codebase files you should read

Your job:

1. READ the codebase files listed. Understand what each script/module does.

2. TRACE the plan's changes through the codebase:
   - Which files, functions, classes, or modules will be affected?
   - Which data pipeline stages, ETL steps, or processing functions are touched?
   - Are there downstream consumers of any modified data or APIs?
   - Are there shared utilities, config values, or constants that will change?

3. IDENTIFY risks:
   - Any change that could silently break data integrity or produce wrong results
   - Any change that removes or renames a function/variable used elsewhere
   - Any schema change, column rename, or file format change that affects downstream steps
   - Any side effects on tests, CI, or deployment
   - Anything in the plan that contradicts how the code currently works

4. For data science / bioinformatics projects specifically:
   - Flag any step that modifies input/output file paths, column names, or data types
   - Flag any change to filtering logic, normalisation steps, or aggregation logic
   - Flag any change that could affect reproducibility (seeds, environment, versioning)

OUTPUT FORMAT:
## Affected Areas
(List each file/function/module that will be touched, with a one-line description of how)

## Risk Register
(Numbered list of risks, each with: Risk | Severity [High/Med/Low] | Reason)

## Data Pipeline Concerns
(Any specific concerns about data integrity, pipeline order, or reproducibility — or "None identified")

## Contradictions with Current Code
(Things in the plan that don't match reality in the codebase — or "None identified")
```

---

### Subagent 2 — Ambiguity Detective

**Dispatch second, after Subagent 1 finishes.**

**Persona**: A meticulous technical writer who demands precision.

**Task**: Find every ambiguous, vague, or undefined term in the plan and surface them as questions
for the user.

**Instructions to pass verbatim to the subagent**:

```
You are a technical writer doing an independent review. You have NO memory of any prior
conversation. You will be given:
  (A) A plan (a set of proposed changes or steps)
  (B) Paths to codebase files you can read for context

Your job:

1. READ the plan carefully, sentence by sentence.

2. IDENTIFY ambiguous or underspecified terms:
   - Vague verbs: "refactor", "clean up", "improve", "handle", "fix" — what exactly?
   - Undefined acronyms or project-specific jargon not explained in the plan
   - Relative terms without anchors: "later", "eventually", "as needed", "if necessary"
   - Unclear scope: "update the pipeline" — which pipeline? which steps?
   - Missing specifics: "add validation" — what validation? what criteria? what happens on failure?
   - Implicit assumptions: things the plan assumes but never states (e.g. assumes a file exists,
     assumes a column name, assumes a certain Python version)

3. READ the codebase files to check if ambiguous terms are clarified there — if they are, note it
   but still flag the ambiguity in the plan itself so it can be made explicit.

4. DO NOT make assumptions. If something is unclear, flag it.

OUTPUT FORMAT:
## Ambiguous Terms & Questions for the User
(Numbered list. For each item: Quote the exact phrase from the plan → Your question)

Example:
1. "clean up the data loader" → What specific changes are needed? (e.g. remove dead code,
   rename variables, split into smaller functions, add type hints?)

## Implicit Assumptions
(Things the plan assumes without stating — numbered list, or "None identified")
```

---

### Subagent 3 — Plan Improver

**Dispatch third, after Subagent 2 finishes.**

**Persona**: A pragmatic tech lead who wants the plan to actually work.

**Task**: Synthesise the findings from Subagents 1 and 2 (which you will be given) and produce an
improved, revised version of the plan.

**Instructions to pass verbatim to the subagent**:

```
You are a tech lead doing an independent plan revision. You have NO memory of any prior
conversation. You will be given:
  (A) The original plan
  (B) Paths to codebase files you can read
  (C) A Risk Register from a prior reviewer (Subagent 1 output)
  (D) An Ambiguity Report from a prior reviewer (Subagent 2 output)

Your job:

1. READ the codebase files listed to understand the current state of the code.

2. READ the Risk Register and Ambiguity Report carefully.

3. REVISE the plan:
   - Resolve or explicitly flag each identified risk (add mitigation steps, reorder steps,
     add rollback notes, add test checkpoints)
   - Replace every ambiguous term with a precise description where you can infer the intent
     from the codebase; mark with [NEEDS USER INPUT] where you cannot resolve it
   - Add explicit checkpoints for data validation in any step touching the data pipeline
   - Ensure steps are ordered safely (no step depends on an output that hasn't been produced yet)
   - Add any missing steps that are implied but not stated
   - Remove or flag any step that contradicts the current codebase

4. DO NOT hallucinate features or functionality not present in the codebase.
   DO NOT change the intent of the plan — only make it clearer and safer.

OUTPUT FORMAT:
## Summary of Changes Made to Plan
(Brief bullet list of what you changed and why)

## Revised Plan
(The full, updated plan — clearly formatted, with [NEEDS USER INPUT] markers where ambiguity
could not be resolved)

## Unresolved Issues
(Anything that requires the user to make a decision before the plan can be finalised)
```

---

## Dispatch Procedure

For each subagent, follow this sequence:

1. **Collect inputs**: plan text + relevant file paths + subagent-specific instructions above.
2. **Spawn the subagent** via the Anthropic API (model: `claude-sonnet-4-20250514`).
   - Pass the plan, file contents (or file paths if the subagent has filesystem access), and
     the verbatim instructions as the user message.
   - Set `system` to: `"You are an independent code reviewer. You have no memory of any prior
     conversation. Read only what is provided to you."`
3. **Wait for completion** before dispatching the next subagent.
4. **Collect the output** and pass it as context to the next subagent where required
   (Subagent 3 receives outputs from Subagents 1 and 2).

---

## Output to User

After all 3 subagents complete, present findings to the user in this order:

```
## 🔍 Plan Inspection Report

### Subagent 1 — Impact & Risk Analysis
[paste Subagent 1 output]

---

### Subagent 2 — Ambiguity Report
[paste Subagent 2 output]

---

### Subagent 3 — Revised Plan
[paste Subagent 3 output]

---

### Next Steps
- Resolve all [NEEDS USER INPUT] items before proceeding
- Address High-severity risks before implementation begins
- Confirm the revised plan replaces the original

### UI Preview (if applicable)
If the user confirmed localhost examples during the UI Change Detection step, generate the
agreed number of browser-rendered previews (default = 3) via a localhost dev server.
Each preview should visually realise one interpretation of the planned UI changes so the
user can pick a direction before any code is committed.
```

---

## Special Guidance for Data Science / Bioinformatics Projects

When the plan touches data processing pipelines, each subagent must pay extra attention to:

- **Column name changes** — any rename propagates through all downstream scripts
- **File path changes** — input/output paths must stay consistent across pipeline stages
- **Filter / threshold changes** — affects all downstream analyses and results
- **Data type changes** — silent coercions can corrupt numerical results
- **Aggregation logic** — changing group-by keys or aggregation functions changes results
- **Reproducibility** — random seeds, package versions, environment specs
- **Order of operations** — some pipeline steps are not commutative; reordering has consequences

---

## Notes

- Subagents are stateless: give them everything they need in their prompt.
- If the codebase is large, prioritise the files most relevant to the plan's scope.
- The goal is not to block progress but to surface what the user needs to know before coding starts.
- The revised plan from Subagent 3 is a **proposal** — the user makes the final call.

---

## Handoff — codex-plan-inspector

After presenting the full Plan Inspection Report to the user, return control to dev-manager
with this explicit handoff:

```
[plan-inspector] Complete. Returning control to dev-manager.
Revised plan ready for external review.
```

Dev-manager will then dispatch **codex-plan-inspector** as the next step, unless:
- The user has said `"not dispatch"`, `"skip codex"`, `"no codex"`, or equivalent
- Codex CLI is not found on PATH

If either skip condition is detected, dev-manager logs the skip and proceeds directly
to the `plan_approved` gate decision.

**Do not proceed to the plan approval gate yourself** — that decision belongs to dev-manager
after codex-plan-inspector (or its skip) completes.
