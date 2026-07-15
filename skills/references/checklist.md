# Dev-Manager Checklist

Use this checklist at each stage gate before advancing `state.json`.

---

## Planning -> Developing

- [ ] Plan saved to `.dev-manager/plans/`
- [ ] `plan-inspector` has reviewed and approved (`plan_approved: true`)
- [ ] Pre-implementation evals run, or explicitly skipped with reason logged
- [ ] `state.json -> workflow.stage` updated to `DEVELOPING`

---

## Developing -> Review

- [ ] All planned changes implemented
- [ ] `karpathy-guidelines` has reviewed code/docs
- [ ] New evals written for any new behavior
- [ ] Post-implementation evals passing (`post_impl_evals_passed: true`)
- [ ] `state.json -> workflow.stage` updated to `REVIEW`

---

## Review -> Finalized

- [ ] `system-checker` has run end-to-end validation
- [ ] Final eval suite passing (`final_evals_passed: true`)
- [ ] Eval results saved to `.dev-manager/results/iteration-N/`
- [ ] `state.json -> workflow.stage` updated to `FINALIZED`
- [ ] `gates.task_closed` set to `true`
- [ ] Audit trail entry added

---

## Per-Project Customization

Add project-specific checklist items below this line.
