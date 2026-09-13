# OpenReconcile — Full Context Export for AI/Cursor

This is a consolidated export of the OpenReconcile discussion. It is intentionally comprehensive but distinguishes current decisions from historical ideas.

## 1. Why OpenReconcile exists

The project began from a concern that AI/ML open-source projects often lack strong Kubernetes production paths. Domain experts may understand models, training, inference, evaluation, or agents while not being experts in Go, controller-runtime, CRD lifecycle, finalizers, status semantics, upgrades, failure recovery, RBAC, or Kubernetes reconciliation.

The initial idea was an AI-native operator community/factory.

That framing changed.

The current principle is **operator-or-not**: the correct Kubernetes architecture may be an existing ecosystem integration, a Job, Helm, a provider, a CRD/controller, or no new Kubernetes abstraction.

## 2. What became the core insight

AI-generated code is increasingly cheap.

The difficult part is:
- choosing the correct architecture;
- preserving Kubernetes semantics;
- handling partial failure;
- proving recovery under real reconciliation failure.

That moved the differentiation toward **ReconcileBench**.

## 3. ReconcileBench

ReconcileBench is the proposed adversarial controller-testing tool/methodology.

It targets failures during reconciliation rather than only checking successful deployment or steady-state assertions.

Important defect examples:
- controller restart during create;
- external child create succeeds but status write fails;
- duplicate ownership/creation;
- stale-cache behaviour;
- finalizer deadlock;
- retry storm on terminal error;
- concurrent reconcile conflict;
- scale change during reconcile.

Initial claim: **defect discovery, not certification**.

## 4. Convergence correction

The early definition used literal write quiescence.

That was rejected because valid controllers resync and update observational fields.

Current direction: **semantic convergence**.

A target converges when, within its bounded window:
- declared invariants hold;
- observed/status state corresponds to current desired state;
- no unresolved actionable error remains;
- additional reconciliation causes no semantically meaningful desired-state mutation.

Per-target ignore lists may exclude fields such as:
- `resourceVersion`;
- `managedFields`;
- observation timestamps;
- heartbeats;
- other justified non-semantic fields.

## 5. Invariants

Current vocabulary:
- resource invariants;
- state invariants;
- safety invariants;
- progress/liveness invariants;
- idempotency invariants;
- recovery invariants;
- deletion invariants.

Safety invariants are especially valuable because they represent states that must never occur.

## 6. Fixture ground truth

A major improvement was the decision to create a fixture controller with:
- clean control;
- seeded-defect variants.

Why:
- defect variants measure sensitivity;
- clean control measures specificity/false positives.

Without a clean control, ReconcileBench cannot distinguish genuine findings from a broken oracle.

## 7. Fault timing

Black-box mode ships first.

Random failure injection is possible but only probabilistically hits certain reconciliation windows.

A high-value design avenue is **API-server-boundary interception**:
- observe/delay/fail selected API calls;
- inject conflicts/throttling;
- allow create then fail status update;
- trigger controller interruption around an observed call.

This can provide much of grey-box timing precision without controller instrumentation.

Grey-box and fully instrumented modes remain future possibilities.

## 8. Falsification, not confirmation

The project deliberately introduced a strategy-falsification process.

Outcome labels:
- VALIDATED;
- WEAKENED;
- REFRAMED;
- KILLED.

REFRAMED/KILLED are successful outcomes when they save future investment.

### F1 — prior art

Try to detect seeded defects using existing tools first.

Do not judge only by number detected. Judge:
- coverage quantity;
- significance/distinctness of missed defect classes.

ReconcileBench may become:
- a distinct tool;
- a narrower hard-failure-class tool;
- a scenario library + convergence oracle over existing tools;
- or nothing.

### F2 — trust

Use real findings against real controllers.

Observe maintainer behaviour.

One fix proves initial value, not ecosystem trust.

### F3 — convention interest/conformance

Do not build a large conformance system first.

Publish a lightweight `CONVENTIONS.md` alongside real work.

If maintainers value/adopt conventions, a later conformance suite may be justified.

If they only want the tests, the standards layer may disappear.

### F4 — coupling

Test whether one umbrella should both:
- ship assurance tooling;
- ship production controllers.

Potential conflict: an org publishing resilience findings on controllers while shipping competing controllers can appear biased.

### F5a — technical scope

Is ReconcileBench truly AI/ML-specific?

Likely possibility: no, the core is general Kubernetes controller resilience.

### F5b — market wedge

Even if general, is AI/ML the best first market?

A healthy combined result may be:
- F5a REFRAMED;
- F5b VALIDATED.

### F6 — economic/maintenance viability

Track Target Onboarding Cost for every target.

If external black-box onboarding is expensive but author integration is cheap, ReconcileBench may become an SDK/framework for controller authors.

That can collapse the independent-assurance/conformance story, and that is acceptable.

## 9. Gate model

### Before Gate 1

Only:
- fixture;
- F1;
- F5a;
- F5b.

Gate 1 asks:
> What exactly remains differentiated enough to build?

### After Gate 1

Build the surviving product.

Collect F2/F3/F4/F6 evidence while building.

Do not turn falsification into months of pre-build process.

Rule:
> No falsification workstream should require an artefact that would not otherwise be useful to the surviving product.

## 10. Umbrella/community

Name: **OpenReconcile**.

Assets acquired:
- `openreconcile.org`;
- `openreconcile.dev`;
- GitHub org `openreconcile`.

The umbrella name should no longer be repeatedly reopened unless new material legal/technical evidence appears.

## 11. Community philosophy

Long-term intent:
- open-source;
- vendor-neutral;
- potentially transferable/donatable;
- real maintainers rather than one-person portfolio branding.

Upstream-first remains important.

A project should be able to move to another org/foundation without forcing users through an API-group migration where avoidable.

## 12. API principles

Strong current principles:
- standard `metav1.Condition`;
- no controller mutation of `spec`;
- same-namespace references by default;
- terminal vs recoverable error classification;
- independent project API identities;
- conversion behaviour before API promotion;
- CRDs are not a high-frequency data plane.

Important correction:
- **do not freeze a shared meta Go module yet**;
- first prove shared shapes across multiple real controllers;
- conventions/policy can be locked earlier because some are expensive to retrofit.

## 13. k8sbricks

Purpose: production Kubernetes control plane for Databricks.

Do not launch the full surface first.

Stage:
1. Compute/Cluster
2. Serving

Compute gives meaningful asynchronous lifecycle semantics with smaller initial scope.

Architecture direction:
- Go/controller-runtime;
- investigate generation from `databricks-sdk-go`;
- evaluate ACK generator reuse/adaptation;
- generated structure + handwritten operational hooks;
- adoption;
- late initialization;
- OIDC/workload identity.

Before public CRDs:
- settle permanent project/domain/API identity.

## 14. KubeReserve

Purpose: cloud capacity/reservation control plane.

Current safety direction:
- safe/dry-run starting posture;
- TTL;
- budget ceiling at admission;
- independent sweeper;
- no hard dependency on Karpenter;
- multi-cluster should be additive later.

Start with one provider.

## 15. OpenEnv

The user wants an OpenEnv-related operator/project.

Current recommendation is **not** to assume a bespoke OpenEnv Operator.

Likely architecture:
- `EnvironmentClass`;
- `EnvironmentPool`;
- OpenEnv provider;
- high-frequency leases outside etcd/CRs.

Validate upstream direction before deep implementation.

## 16. SmolAgents

The user wants a SmolAgents-related operator/project.

Current recommendation is **not** to create one operator per agent framework.

Likely architecture:
- `MLRuntime`;
- `MLWorkload`;
- SmolAgents as first runtime/provider.

Future frameworks should be declarative/configurable where possible.

## 17. Language

Go + Kubebuilder/controller-runtime for production controllers.

Avoid importing ML libraries into the control loop.

Python can be used for client SDKs/tooling.

## 18. Repository model

OpenReconcile is the GitHub umbrella.

Recommended core repos:
- `.github`;
- `community`;
- `controller-template`;
- `reconcilebench`;
- `reconcilebench-fixture`.

Production repos should be independent.

Do not create every historical roadmap repo now.

## 19. Cursor/AI development

The repository is the durable memory.

Use Cursor with:
- builder context;
- separate adversarial reviewer context.

AI-generated implementation must not be considered proven by AI-generated tests from the same context.

## 20. Deferred historical roadmap

Ideas retained only as future possibilities:
- Forge;
- Radar;
- Catalog;
- Drift;
- Operating Knowledge Packs;
- certification;
- broad conformance/community portal.

They are not current build authorization.

## 21. What success means right now

The next success is not "four operators exist."

It is:

1. OpenReconcile org/repo bootstrap is clean.
2. The fixture establishes ground truth.
3. F1 demonstrates what existing tools can/cannot do.
4. F5 clarifies technical scope and market wedge.
5. Gate 1 produces a clear, narrower product definition.
6. The first production controller then provides real-world validation.
