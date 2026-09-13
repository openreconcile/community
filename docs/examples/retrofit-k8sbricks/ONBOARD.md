# Onboard k8sbricks

Derived from `config/crd/bases`. This command did **not** restructure
the repo or change the API group.

## Next

1. Snapshot a live or envtest object (`kubectl get … -o yaml`).
2. `reconcilebench verify --object <snapshot>`
3. Apply a minimal fix for each finding. Re-verify.
4. Do not claim the operator is resilient.

See the `onboard` Skill. Use `new-operator` only for a new repo.
