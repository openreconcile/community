# OpenReconcile — Decision Register

This file separates **locked**, **current direction**, **provisional**, and **deferred** decisions.

When a decision changes, update it here in the same commit as the work that changed it.

## Decision log

| Date | Decision |
|---|---|
| 2026-09-13 | **Operators are the examples.** KubeReserve is regenerated greenfield (`new-operator` / `openreconcile new` from `KUBERESERVE-RFC.md`). k8sbricks is retrofitted (`onboard` / `openreconcile onboard` against the handwritten spike). Synthetic MLflow ModelRegistry example rejected. Slogan wording is *Build a Kubernetes operator in natural language*; k8sbricks is the AI/ML proof point. See `process/LAUNCH.md`. |
| 2026-09-13 | **Hold operator-chaos outreach.** Do not comment on `opendatahub-io/operator-chaos#6`. Do not file further issues until `derive` attaches a real `knowledge.yaml`. |
| 2026-09-13 | **Launch bar.** Announce only after L3 (KubeReserve greenfield example) and L4 (k8sbricks retrofit example). OpenEnv optional. See `process/LAUNCH.md`. |
| 2026-09-13 | **Gate 1 closed.** Outcomes C+D. ReconcileBench is an engine-agnostic oracle-and-agent layer. Recorded in `GATE1.md` and `adr/ADR-0001-engine-agnostic-architecture.md`. |
| 2026-09-13 | Adopt **operator-chaos** as a first-class engine. Do not build a fault injector. |
| 2026-09-13 | OpenEnv work is an **upstream `KubernetesProvider`** to `meta-pytorch/OpenEnv`. No env-pool org repo. OpenSandbox already implements pool CRs. |
| 2026-09-13 | Databricks incumbent path corrected to `glalanne/provider-databricks` (live, v2.5.0). Space is not vacant. |
| 2026-09-13 | KubeReserve remaining wedge is reservation *lifecycle*, not consumption. Karpenter already consumes ODCRs / Capacity Blocks. |
| 2026-09-13 | Hero user is an MLE. NL front door: Skills, then CLI, then hosted demo. Slogan gated on oracle + eight fixture defects. |
| 2026-09-13 | **`controller-template` re-authorised after the fixture**, seeded from it. Earlier deferral (guesses before two builds) is replaced by “seed from proven fixture”. |
| 2026-09-13 | Prior-art claim corrected. "Nothing injects faults during reconciliation" is false; Sieve and Acto occupy the space. Recorded in `PRIOR_ART.md`, and `PROJECT-CONTEXT.md` §2 rewritten. |
| 2026-09-13 | **F0 prior-art execution spike** inserted ahead of the fixture. Two days. Adds a new Gate-1 outcome: contribute upstream rather than build. |
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
- Upstream-first — including assurance tooling (adopt engines; do not fork them).
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

### ReconcileBench (Gate 1)
- **No owned fault-injection engine.** See ADR-0001.
- First-class engines: operator-chaos, envtest, chainsaw. Acto optional.
- Owns scenario spec, status-contract oracle, `derive`, evidence, MCP.
- fixture has both clean and seeded-defect variants;
- measure false positives as well as detection;
- semantic convergence replaces literal write quiescence;
- black-box mode first;
- evidence records exact tested matrix/as-of basis;
- F1 uses coverage quantity + defect-class significance;
- Target Onboarding Cost is a first-class metric; Acto's push-button onboarding is the
  benchmark to beat; adopting engines is how we stay under that bar.

### Scope
- Pipeline is general Kubernetes (F5a).
- First market and Skills/marketing are AI/ML (F5b).
- Hero user: MLE productionising an existing ML tool.

## Current direction, not immutable

- ReconcileBench is the first technical wedge, at Gate 1 scope.
- k8sbricks Compute is the first serious production controller after the fixture.
- Serving follows Compute.
- KubeReserve starts with one cloud provider; RFC before SDK; last in the operator queue.
- OpenEnv: upstream `KubernetesProvider`, not an org operator.
- SmolAgents use case should likely be implemented through a generic workload/runtime control plane (still deferred).
- publish lightweight `CONVENTIONS.md` early enough to observe external interest.
- `controller-template` after the fixture proves clean.

## Provisional / requires evidence

- F2 trust / maintainer response;
- F3 convention interest;
- exact scenario file schema (lands with Phase 3);
- exact MCP tool surface (lands with Phase 6);
- k8sbricks permanent API-domain identity;
- ACK code-generator reuse/adaptation;
- whether KubeReserve’s lifecycle wedge is wide enough after the RFC;
- compute budget ceiling.

## Deferred

- hosted NL demo (after k8sbricks Compute);
- Forge;
- Radar;
- Catalog;
- Drift;
- certification;
- reputation/community portal;
- broad conformance framework;
- frozen umbrella meta API;
- full multi-cloud KubeReserve;
- full Databricks surface at first release;
- SmolAgents / MLWorkload operator.

## Rejected / explicitly avoid

- a new mid-reconcile fault-injection engine;
- reviving Sieve as a product;
- an OpenReconcile environment-pool operator (OpenSandbox + upstream OpenEnv);
- generic operator factory;
- one operator per AI/ML project by default;
- AI-generated infrastructure marketplace;
- advertising *Build a Kubernetes operator in natural language* (or the older AI/ML-specific wording) before L3 has been replayed by someone other than the author;
- a throwaway MLflow ModelRegistry pipeline example (the operators are the examples);
- commenting on `opendatahub-io/operator-chaos#6` or filing further issues until `derive` attaches a real `knowledge.yaml`;
- duplicate KServe/KubeRay/Kubeflow/Crossplane-style control planes without a real gap;
- certification claims before ecosystem standing exists.
