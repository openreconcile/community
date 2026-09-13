# OpenReconcile — Project History and Evolution

This file explains how the current direction emerged so an agent does not mistake old decisions for current policy.

## Stage 1 — Operator-community concept

The original idea focused on an AI-native community/factory that could help AI/ML domain experts create Kubernetes operators without requiring every contributor to be a Go/controller-runtime expert.

The valuable insight that survived was:
- domain experts know operational intent;
- agents can accelerate implementation;
- Kubernetes experts still need to govern architecture and correctness.

The "generate one operator per project" framing did **not** survive.

## Stage 2 — Operator-or-not

The project broadened from operator generation to a decision system:

```text
Discover
→ Understand
→ Architect
→ integrate/build only where needed
→ Prove
→ maintain
```

An explicit "do not build an operator" verdict became a valid outcome.

This created concepts such as Radar, ProjectOperationalProfile, Architect, Forge, Proof, Catalog, and Drift.

Those concepts remain historical roadmap ideas but most are deferred.

## Stage 3 — Proof becomes the differentiator

The insight shifted from AI-generated code to operational correctness.

The strongest question became:

> Does the controller converge correctly when failures happen during reconciliation?

This produced the ReconcileBench concept:
- adversarial fault injection;
- invariant checking;
- convergence analysis;
- reproducible findings.

The claim was narrowed from "certification" to "defect discovery."

## Stage 4 — Project Plan v2

Plan v2 defined two coupled tracks:
- assurance: ReconcileBench;
- production controllers/control planes: k8sbricks, KubeReserve, environment pooling, workload runner.

It introduced strong API/convention ideas, donatability, responsible disclosure, and a decision gate.

Some v2 decisions were later corrected:
- "no writes" convergence → semantic convergence;
- permanently frozen shared meta Go types → defer extraction until repeated implementations prove the common shape;
- permanent rejection of ArtifactHub → optional/deferred;
- broad Track-B scope → stage projects rather than launch all at once.

## Stage 5 — Strategy falsification

A review identified that execution had become stronger than the strategic validation.

A falsification framework was created with workstreams around:
- F1 prior art;
- F2 trust;
- F3 conformance;
- F4 coupling/conflict of interest;
- F5 scope;
- later F6 economic/maintenance viability.

Important refinements:
- F1 verdict uses defect coverage and significance, not just a numeric count;
- F5 split into technical scope and market wedge;
- F6 introduced Target Onboarding Cost;
- conformance was split into cheap convention-interest observation vs expensive formal suite;
- pre-build falsification is limited to Gate 1;
- post-Gate-1 falsification is collected while building, not as a separate months-long research program.

## Stage 6 — Naming and assets

The umbrella name OpenReconcile was locked for execution.

Acquired:
- `openreconcile.org`
- `openreconcile.dev`
- `github.com/openreconcile`

The umbrella identity must remain separate from individual project API identities where donatability requires it.

## Stage 7 — Prior-art correction

September 2026. During context consolidation the prior-art list was found to cover only
industry tooling. The academic Kubernetes-controller-testing literature was missing, and it
is exactly where the differentiation claim was weakest:

- **Sieve** (OSDI '22) already performs deterministically-timed mid-reconcile fault
  injection against unmodified controller logic, and its intermediate-state,
  unobserved-state and stale-state patterns map onto three of the planned defect branches.
  Dormant since September 2024.
- **Acto** already performs automated state-centric operator-correctness testing, needs only
  a deployment script to onboard a target, and has found 50–80+ real bugs.
- **Anvil** formally verifies controller liveness as Eventually Stable Reconciliation.

Consequences: the claim "nothing injects faults during reconciliation" was retired; an
**F0 execution spike** was inserted ahead of the fixture; F1's candidate tool set now
includes Sieve and Acto; an eighth defect branch was added for the unobserved-state class;
and Gate 1 gained an outcome — contribute upstream rather than build a third tool.

The lesson generalises: the project's falsification framework was sound, but it was pointed
at the wrong comparison set. Check the comparison set before trusting the verdict.

## Current stage

Move from planning into repository bootstrap and Gate-1 implementation.

The immediate code/research combination is:
- **F0 prior-art execution spike** (first, two days);
- ReconcileBench fixture;
- F1;
- F5a/F5b;
- Gate 1.

The project should become narrower as evidence accumulates, not broader merely because more ideas exist.
