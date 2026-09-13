# Gate 1 — Decision Record

**Date:** 2026-09-13
**Status:** Closed for product shape. F2 and F3 remain open pending external evidence.

This is the written Gate 1 answer required by `ROADMAP.md` and `DECISIONS.md`.
It records what was falsified, what survived, and what we will build.

## Verdict

**C + D, not A, not E, not F-alone.**

ReconcileBench is a **scenario library, status-contract oracle, and agent loop over
existing perturbation engines**, general-purpose for Kubernetes controllers, launched
into AI/ML as the first market.

It is not a new fault-injection category. It is not an upstream-only contribution.
It does not revive Sieve.

| Outcome | Decision |
|---|---|
| A. Distinct testing category | **Rejected.** Sieve (OSDI '22) and operator-chaos already inject faults during reconciliation. |
| B. Narrower tool for a few hard classes | **Absorbed into C.** The hard classes we own are status-contract and external-boundary oracles, not a standalone engine. |
| C. Scenario library + convergence oracle over existing tools | **Accepted.** Primary product shape. |
| D. General controller-testing tool, AI/ML as first market | **Accepted.** F5a REFRAMED, F5b VALIDATED. |
| E. Nothing differentiated remains | **Rejected.** Status-contract oracles, source-derived target models, and machine-actionable evidence are unclaimed. |
| F. Upstream-only (Acto or Sieve) | **Rejected as the only move.** Complementary contribution to operator-chaos is in scope; owning the oracle layer is not donated. |

## Workstream verdicts

### F0 — Prior-art spike

**Closed on source-level analysis plus a 2026-09-13 re-search.** Execution of Sieve/Acto
against a live operator is still useful telemetry (F6 hours) but is no longer load-bearing
for the build-vs-contribute decision.

Findings that closed F0:

1. Sieve is dormant (last commit 2024-09-26), pins Kubernetes ~1.18–1.23, and requires a
   patched kind node image. Revival means maintaining a Kubernetes fork.
2. Acto is alive and has the best black-box onboarding cost. It does not time faults to a
   reconcile phase and it masks `observedGeneration` / `generation`.
3. **operator-chaos** (`opendatahub-io/operator-chaos`, Red Hat / Open Data Hub) is the
   current maintained engine: 20 injection types, four modes (CLI, chaostransport,
   ChaosClient, ActionInterceptor), knowledge models, structured verdicts. It occupies
   the “inject faults during reconciliation” slot we originally claimed.
4. Both Sieve and Acto **mask** `status.observedGeneration` and `status.conditions`.
   `defect/missing-observed-generation` is invisible to both by configuration.
5. Neither Sieve nor Acto models an external (non-Kubernetes) API boundary.

### F1 — Prior-art challenge

**Partially falsified. Reframed.**

Generic mid-reconcile fault injection is **REFRAMED**: Sieve did it first; operator-chaos
does it now. Building a fifth engine is rejected.

What remains **VALIDATED** as unclaimed:

- a status-contract oracle (`observedGeneration`, condition semantics, generation handling);
- source-derived target models (`reconcilebench derive`) instead of hand-written knowledge YAML;
- machine-actionable evidence an agent can consume in a detect-fix-verify loop;
- an engine-agnostic scenario layer (operator-chaos, envtest, chainsaw, Acto as backends).

F1 execution against the fixture (coverage count per engine) still happens in Phase 1–4.
That is measurement, not a second product decision.

### F2 — Trust

**Open.** An unknown org’s findings are not trusted until a third-party maintainer acts
on one. The public survey and operator-chaos outreach exist to start that ladder. Do not
publish third-party defects before `DISCLOSURE.md`.

### F3 — Conformance interest

**Open.** Publish lightweight conventions beside first findings. Do not build a
conformance suite until someone other than us uses the conventions.

### F4 — Coupling

**Resolved by donatability, not by splitting the org.** Assurance and operators may share
the OpenReconcile umbrella. Public API groups must not contain `openreconcile`. Projects
must be transferable without a user API migration. Perceived conflict of interest is
watched, not used as a reason to delay the oracle.

### F5a — Technical scope

**REFRAMED.** Nothing in the scenario, invariant, or convergence model is AI/ML-specific.
Sieve and Acto were evaluated on Cassandra, RabbitMQ, TiDB, ZooKeeper. The pipeline is
general Kubernetes.

### F5b — First market

**VALIDATED.** The hero user is an MLE who already has an ML tool and wants it
productionised on Kubernetes. Skills, CLI examples, and marketing are AI/ML-specific.
The slogan *Build a Kubernetes operator in natural language* is gated on the
KubeReserve greenfield example being replayed by someone other than the author.
k8sbricks is the AI/ML proof point.

### F6 — Economics

**Provisionally VALIDATED by adopting engines.** Building our own fault injector would
have made Target Onboarding Cost worse than Acto before we started. Adopting
operator-chaos / envtest / chainsaw collapses that cost. `reconcilebench derive` is the
remaining F6 lever: if every target still needs a hand-written knowledge model, the
generic-tool thesis fails and we reframe as an author SDK.

Measure TOC on fixture → k8sbricks → two third-party targets. Acto’s deployment-script
onboarding remains the bar.

## What we will build

See `adr/ADR-0001-engine-agnostic-architecture.md` and `ROADMAP.md`.

1. `reconcilebench-fixture` — clean Widget + eight defect branches.
2. Portable agent context + Skills (`new-operator`, `contribute`; later `verify`, `disclose`).
3. ReconcileBench core — scenario spec + engine adapters. No owned fault injector.
4. Status-contract oracle.
5. `reconcilebench derive`.
6. Evidence format + MCP server.
7. `controller-template` seeded from the proven fixture.
8. `openreconcile new` CLI, then a hosted NL demo after k8sbricks exists.

Operators, one at a time, behind the §10 entry gate:

1. k8sbricks Compute — after permanent API identity.
2. OpenEnv `KubernetesProvider` — **upstream** to `meta-pytorch/OpenEnv`, no new org repo.
3. KubeReserve — last; RFC first; one provider; Karpenter already consumes reservations.

## What we will not claim

- “Nothing injects faults during reconciliation.”
- Certification, verification, or resilience as an asserted property.
- “Build a Kubernetes operator in natural language” before L3 has been replayed.
- That k8sbricks has an empty market. `glalanne/provider-databricks` is live (v2.5.0,
  2026-08-31). The gap is Kubernetes-native reconciliation vs Terraform-derived
  Crossplane resources, plus encoded Databricks domain knowledge.
- That environment pooling is unclaimed. OpenSandbox already ships `Pool` / `BatchSandbox`.
- That KubeReserve is unclaimed for *consuming* reservations. Karpenter does that.
  The remaining question is *creating, expiring, budgeting, and sweeping* them.

## Sign-off

Recorded 2026-09-13 in the community repository. Subsequent product work implements this
shape; it does not reopen outcome A or a new engine.
