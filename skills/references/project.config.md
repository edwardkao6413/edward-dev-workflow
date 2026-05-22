# project.config.md — Project Configuration Template

This file is copied into a new project by `init-project` and placed at the project root.
Fill in every section before starting any work. All fields marked `(fill in)` are required.

---

## 1. Project Identity

- **Project name:** (fill in — e.g. "Gait Analysis Pipeline v2")
- **Project type:** (fill in — biomechanics | bioinformatics | data-science | data-engineering | web | script | other)
- **Domain:** (fill in — e.g. clinical biomechanics | genomics | sports science | leave blank if not applicable)
- **One-line description:** (fill in — what does this project produce or do?)
- **Primary output:** (fill in — e.g. processed .csv files | a Python package | a report)

---

## 2. Language & Framework

- **Primary language:** (fill in — e.g. Python 3.11)
- **Key libraries / frameworks:** (fill in — e.g. NumPy, Pandas, ezc3d, Biopython)
- **Environment / runtime:** (fill in — e.g. conda env `biomech` | virtual env `.venv`)
- **Package manager:** (fill in — e.g. pip + requirements.txt | conda | poetry)

---

## 3. Coding Standards

- **Style guide:** (fill in — e.g. PEP 8 strictly enforced)
- **Reproducibility rules:** (fill in — e.g. all random seeds must be set and logged; no hardcoded paths)
- **Naming conventions:** (fill in — e.g. snake_case for functions, UPPER_SNAKE for constants)
- **Docstring format:** (fill in — e.g. NumPy-style docstrings on all public functions)
- **Testing framework:** (fill in — e.g. pytest; minimum 80% coverage on processing modules)

---

## 4. Files & Data — Never Modify

List every file, folder, or pattern that must never be edited by Claude without explicit user approval:

- (fill in — e.g. `data/raw/**` — raw input files are immutable)
- (fill in — e.g. `references/` — reference databases must not be altered)
- (fill in — e.g. any `.c3d` or `.trc` source file)
- (fill in — e.g. `config/production.yaml`)

---

## 5. Domain-Specific Rules

Delete the sections that don't apply. Fill in the ones that do.

### Biomechanics
- Always log the sampling rate and capture system in metadata
- Always record the marker set name and version used
- Never interpolate gaps > (fill in) frames without flagging to user
- Coordinate system convention: (fill in — e.g. right-hand, Y-up)

### Bioinformatics
- Always record the reference genome version (e.g. GRCh38) in metadata
- Always log the tool versions used for alignment/variant calling
- Never modify raw FASTQ or BAM files
- Pipeline must be reproducible: log all parameters to a sidecar JSON

### General Data Science
- All processed outputs must include a provenance header (input file, date, script version)
- No results may be reported without confidence intervals or equivalent uncertainty measure
- Intermediate outputs must be saved to `outputs/intermediate/` not overwritten in place

---

## 6. Active Agents

By default all three dev-manager subordinates are active. Disable any that don't apply:

- [x] plan-inspector
- [x] karapathy-guideline
- [x] system-checker

Additional project-specific agents (if any):
- (fill in — e.g. `.agents/biomech-validator/SKILL.md`)

---

## 7. Evals

- **Eval suite location:** `.claude/evals/evals.json`
- **Results location:** `.claude/results/`
- **Minimum passing threshold:** (fill in — e.g. 100% | 90%)
- **Eval scope:** (fill in — e.g. validate marker trajectory ranges; check output column names match schema)

---

## 8. Folder Reference

```
project-root/
├── project.config.md                ← this file (project identity + rules)
├── .agents/
│   ├── dev-manager/                 ← always active
│   │   ├── SKILL.md
│   │   ├── plan-inspector/SKILL.md
│   │   ├── karapathy-guideline/SKILL.md
│   │   └── system-checker/SKILL.md
│   └── superpower/                  ← Superpower plugin fallback agents
├── .claude/
│   ├── evals/
│   │   └── evals.json
│   └── results/
├── .dev-manager/
│   ├── state.json                   ← shared workflow bridge
│   ├── workflow.md                  ← dispatch logic reference
│   ├── checklist.md                 ← stage gate checklist
│   └── plans/                      ← plan handoff directory
└── (your project files)
```
