# OpenReconcile — Decision Register

This file separates **locked**, **current direction**, **provisional**, and **deferred** decisions.

When a decision changes, update it here in the same commit as the work that changed it.

## Decision log

| Date | Decision |
|---|---|
| 2026-09-13 | Prior-art claim corrected. "Nothing injects faults during reconciliation" is false; Sieve and Acto occupy the space. Recorded in `PRIOR_ART.md`, and `PROJECT-CONTEXT.md` §2 rewritten. |
| 2026-09-13 | **F0 prior-art execution spike** inserted ahead of the fixture. Two days. Adds a new Gate-1 outcome: contribute upstream rather than build. |
| 2026-09-13 | **`controller-template` deferred.** Conflict between `ROADMAP.md` (later) and `REPO_TOPOLOGY.md` / `ORG_BOOTSTRAP_CHECKLIST.md` (first batch) resolved in favour of later — a template written before two real builds encodes guesses. |
| 2026-09-13 | Eighth fixture defect branch added: `defect/unobserved-event`, matching Sieve's unobserved-state pattern. The original seven had no equivalent. |
| 2026-09-13 | Canonical context consolidated into `openreconcile/community`. Two parallel doc sets merged; historical originals preserved verbatim under `sources/`. |


## Locked / strong decisions

### Project identity
- Umbrella name: OpenReconcile.
- `openreconcile.org` is the primary community domain.
- `openreconcile.dev` is owned for developer/docs use.
- GitHub org: `openreconcile`.

### Product philosophy
- Operator-or-not.
- Upstream-first.
- Reuse existing control planes.
- AI-assisted, human-governed.
- Adversarial review must be independent of implementation review.
- Defect discovery before certification.
- Donatability matters.

### Controller engineering
- Go + Kubebuilder/controller-runtime for production controllers.
- Standard `metav1.Condition`.
- Controllers do not mutate `spec`.
- Same-namespace references by default.
- Terminal vs recoverable errors must be distinguishable.
- Reconciliation must be idempotent.
- Finalizers require safe termination paths.

### ReconcileBench
- fixture has both clean and seeded-defect variants;
- measure false positives as well as detection;
- semantic convergence replaces literal write quiescence;
- black-box mode first;
- evidence records exact tested matrix/as-of basis;
- investigate API-server-boundary interception;
- F1 uses coverage quantity + defect-class significance;
- F1 must include Sieve and Acto in the candidate tool set;
- Target Onboarding Cost is a first-class metric, and Acto's push-button onboarding is the
  benchmark to beat.

## Current direction, not immutable

- ReconcileBench is the first technical wedge.
- AI/ML is likely the first market, not necessarily a technical boundary.
- k8sbricks Compute is the first serious production controller after Gate 1.
- Serving follows Compute.
- KubeReserve starts with one cloud provider.
- OpenEnv use case should likely be implemented through a generic environment-pool control plane.
- SmolAgents use case should likely be implemented through a generic workload/runtime control plane.
- publish lightweight `CONVENTIONS.md` early enough to observe external interest.

## Provisional / requires evidence

- **whether to build ReconcileBench at all, or contribute to Acto / revive Sieve** — F0;
- exact ReconcileBench implementation architecture;
- exact fault-injection mechanism;
- exact v0.1 scenario list;
- whether ReconcileBench is a standalone tool, orchestration/scenario layer, or author SDK;
- AI/ML market wedge;
- formal conformance layer;
- organisational coupling between assurance tooling and production controllers;
- shared reusable Go structs;
- project/API names for EnvPool and MLWorkload;
- k8sbricks permanent API-domain identity;
- ACK code-generator reuse/adaptation.

## Deferred

- `controller-template`;
- Forge;
- Radar;
- Catalog;
- Drift;
- certification;
- reputation/community portal;
- broad conformance framework;
- frozen umbrella meta API;
- full multi-cloud KubeReserve;
- full Databricks surface at first release.

## Rejected / explicitly avoid

- generic operator factory;
- one operator per AI/ML project by default;
- AI-generated infrastructure marketplace;
- duplicate KServe/KubeRay/Kubeflow/Crossplane-style control planes without a real gap;
- certification claims before ecosystem standing exists.
