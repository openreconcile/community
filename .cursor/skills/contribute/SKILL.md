---
name: contribute
description: >-
  Contribute to an OpenReconcile repository. Use when the user wants to change
  community docs, ReconcileBench, the fixture, controller-template, or a
  Track B operator.
---

# contribute

1. Read `AGENTS.md`, then `docs/PROJECT-CONTEXT.md`, then `docs/context/GATE1.md`.
2. Check `docs/context/DECISIONS.md` before proposing a new engine or a new operator.
3. Prefer small, reviewable diffs. Update `DECISIONS.md` in the same change if a decision moved.
4. Do not implement a fault injector. Do not create an env-pool repo.
5. Do not `kubebuilder init` for k8sbricks until API identity is locked.
6. Tests: happy path, idempotency, terminal vs recoverable, deletion, partial external success.
7. Independent adversarial review — do not accept the builder's own tests as sufficient proof.
