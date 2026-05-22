# Dev-Manager Checklist

Use this checklist at each stage gate before advancing `state.json`.

---

## Planning → Developing

- [ ] Plan saved to `.dev-manager/plans/`
- [ ] plan-inspector has reviewed and approved (`plan_approved: true`)
- [ ] Pre-implementation evals run (or explicitly skipped with reason logged)
- [ ] `state.json → workflow.stage` updated to `DEVELOPING`

---

## Developing → Review

- [ ] All planned changes implemented
- [ ] karapathy-guideline has reviewed code/docs (`karapathy-guideline.approved: true`)
- [ ] New evals written for any new behaviour
- [ ] Post-implementation evals passing (`post_impl_evals_passed: true`)
- [ ] `state.json → workflow.stage` updated to `REVIEW`

---

## Review → Finalized

- [ ] system-checker has run end-to-end validation (`system-checker.approved: true`)
- [ ] Final eval suite passing (`final_evals_passed: true`)
- [ ] Eval results saved to `.claude/results/iteration-N/`
- [ ] `state.json → workflow.stage` updated to `FINALIZED`
- [ ] `gates.task_closed` set to `true`
- [ ] Audit trail entry added

---

## Per-Project Customisation

Add project-specific checklist items below this line.
This section is safe to edit without breaking the universal workflow above.

<!-- Example for biomechanics:
- [ ] .c3d / .trc files validated against expected marker set
- [ ] Sampling rate confirmed in metadata
-->

<!-- Example for bioinformatics:
- [ ] FASTQ quality scores checked
- [ ] Reference genome version logged in state.json → project.notes
-->
