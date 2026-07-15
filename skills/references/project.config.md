# project.config.md - Project Configuration Template

This file is copied into a new project by `init-project` and placed at the
project root. Fill in every section before starting work.

---

## 1. Project Identity

- **Project name:** (fill in; e.g. "Gait Analysis Pipeline v2")
- **Project type:** (fill in; biomechanics | bioinformatics | data-science | data-engineering | web | script | other)
- **Domain:** (fill in; e.g. clinical biomechanics | genomics | sports science | leave blank if not applicable)
- **One-line description:** (fill in)
- **Primary output:** (fill in; e.g. processed CSV files | a Python package | a report)

---

## 2. Language & Framework

- **Primary language:** (fill in; e.g. Python 3.11)
- **Key libraries / frameworks:** (fill in; e.g. NumPy, Pandas, ezc3d, Biopython)
- **Environment / runtime:** (fill in; e.g. conda env `biomech` | virtual env `.venv`)
- **Package manager:** (fill in; e.g. pip + requirements.txt | conda | poetry)

---

## 3. Coding Standards

- **Style guide:** (fill in)
- **Reproducibility rules:** (fill in)
- **Naming conventions:** (fill in)
- **Docstring format:** (fill in)
- **Testing framework:** (fill in)

---

## 4. Files & Data - Never Modify

List every file, folder, or pattern that must never be edited by the agent
without explicit user approval:

- (fill in; e.g. `data/raw/**` - raw input files are immutable)
- (fill in; e.g. `references/` - reference databases must not be altered)
- (fill in; e.g. any `.c3d` or `.trc` source file)
- (fill in; e.g. `config/production.yaml`)

---

## 5. Domain-Specific Rules

Delete the sections that do not apply. Fill in the ones that do.

### Biomechanics

- Always log the sampling rate and capture system in metadata
- Always record the marker set name and version used
- Never interpolate gaps above the chosen threshold without flagging to user
- Coordinate system convention: (fill in)

### Bioinformatics

- Always record the reference genome version in metadata
- Always log the tool versions used for alignment/variant calling
- Never modify raw FASTQ or BAM files
- Pipeline must be reproducible: log all parameters to a sidecar JSON

### General Data Science

- All processed outputs must include provenance information
- No results may be reported without uncertainty information where applicable
- Intermediate outputs must be saved to `outputs/intermediate/` and not overwritten in place

---

## 6. Active Agents

By default all dev-manager subordinates are active. Disable any that do not apply:

- [x] plan-inspector
- [x] codex-plan-inspector - Claude-to-Codex-CLI helper; skip inside Codex
- [x] karpathy-guidelines
- [x] system-checker

Additional project-specific validators, if any:

- (fill in; e.g. `biomech-validator`, `privacy-reviewer`, or "none")

---

## 7. Evals

- **Eval suite location:** `.dev-manager/evals/evals.json`
- **Results location:** `.dev-manager/results/`
- **Minimum passing threshold:** (fill in; e.g. 100% | 90%)
- **Eval scope:** (fill in)

---

## 8. Folder Reference

```text
project-root/
├── project.config.md
├── .dev-manager/
│   ├── state.json
│   ├── workflow.md
│   ├── checklist.md
│   ├── plans/
│   ├── evals/
│   │   └── evals.json
│   └── results/
└── (your project files)
```
