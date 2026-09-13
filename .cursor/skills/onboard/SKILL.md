---
name: onboard
description: >-
  Point ReconcileBench at an existing Kubernetes controller repository.
  Use when a maintainer wants to join OpenReconcile or retrofit an operator
  they already have. Do not use this to scaffold a new operator.
---

# onboard

You are onboarding an **existing** controller repo. You do not own its layout.

## Do

1. Confirm the path is a controller repo (`PROJECT` or `config/crd/bases`).
2. Run `openreconcile onboard <path>` (or `reconcilebench derive --name <op> --crds <path>/config/crd/bases`).
3. Read the written `knowledge.yaml` and `ONBOARD.md`.
4. Obtain an object snapshot (envtest or live YAML).
5. `reconcilebench verify --object <snapshot>`.
6. Apply a **minimal** fix for each finding. Re-verify.
7. Record target version, Kubernetes version, scenario, as-of date.

## Refuse

- Restructuring the host repo (new layout, monorepo moves, rewrite).
- Changing the API group or donating it to `*.openreconcile.io`.
- Replacing the controller runtime (no Python/Kopf, no Crossplane conversion).
- Claiming the operator is resilient, verified, or certified.
- Scaffolding a new operator — use `new-operator` instead.

If `findings` is empty: the oracle did not find a status-contract defect
**on this object**. That is not a general claim.
