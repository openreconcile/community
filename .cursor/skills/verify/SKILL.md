---
name: verify
description: >-
  Run ReconcileBench, interpret machine-actionable evidence, and close the
  detect-fix-verify loop. Use when the user wants to know if a controller
  honours the Kubernetes status contract under failure.
---

# verify

1. Obtain a target snapshot (live object YAML or fixture sample).
2. `reconcilebench verify --scenario <file> --object <file>`
3. Read `findings[].hint`. Apply a **minimal** fix.
4. Re-run verify. Do not expand scope.
5. Record target version, Kubernetes version, scenario, repetitions, as-of date.
6. Never say "resilient", "verified", or "certified".
7. Engines (operator-chaos, envtest, chainsaw, acto) are invoked via
   `reconcilebench run --scenario`. ReconcileBench does not inject faults itself.

If `findings` is empty and `converged` is true: the oracle did not find a
status-contract defect **on this object, in this scenario**. That is not a
general claim.
