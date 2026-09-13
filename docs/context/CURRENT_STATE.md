# OpenReconcile — Current State

**Status:** implementation bootstrap / pre-Gate-1  
**Date:** September 2026

This is the latest consolidated project state. It overrides conflicting historical wording in `sources/`.

## Identity

- Umbrella/community: **OpenReconcile**
- GitHub org: `github.com/openreconcile`
- Primary domain: `openreconcile.org`
- Developer domain: `openreconcile.dev`

The umbrella name is considered locked for practical execution.

## Current positioning

OpenReconcile is an open-source, vendor-neutral community focused on improving the resilience and correctness of Kubernetes controllers.

ReconcileBench is the first technical wedge.

The initial claim is:

> find controller defects ordinary tests miss by disrupting reconciliation and measuring whether the system returns to correct state.

**That claim is not yet differentiated and must not be published as though it were.** Sieve
(OSDI '22) already does deterministically-timed mid-reconcile fault injection and Acto
already does automated operator-correctness testing; see `PRIOR_ART.md`. The narrower
defensible position — a maintained, low-onboarding-cost productisation of a proven research
approach, strongest at the external-API boundary — is what F0 and F1 exist to test.

Do not position OpenReconcile as a certification authority at this stage.

AI/ML is the first intended market/application area, but the core tool may be general-purpose. F5 explicitly tests that question.

## Immediate goal

Do not build the full historical roadmap.

Complete Gate 1 cheaply:

1. run the **F0 prior-art execution spike** against Sieve and Acto — two days, runs first;
2. build `reconcilebench-fixture`, designed to be consumable by every candidate harness;
3. maintain a clean control and seeded-defect variants;
4. run F1 prior-art challenge, with Sieve and Acto in the candidate set;
5. run F5a technical-scope challenge;
6. run F5b market-wedge challenge;
7. decide what ReconcileBench actually is.

Possible Gate-1 outcomes include:
- distinct controller-resilience testing capability;
- narrower hard-failure-class tool;
- scenario library + convergence oracle over existing tools;
- general Kubernetes resilience tool, with AI/ML as first market;
- upstream contribution to Acto or revival of Sieve rather than a third tool;
- no sufficiently differentiated product.

## After Gate 1

Build only the surviving product thesis.

Do not stop building to create months of additional falsification machinery.

Collect later evidence as a by-product of real work:

- F2: trust/maintainer response;
- F3a: convention interest from a simple published conventions document;
- F4: coupling/neutrality evidence;
- F6: Target Onboarding Cost telemetry.

Only build F3b (formal conformance suite/framework) if F3a shows genuine demand.

## F6 principle

Every target onboarding records Target Onboarding Cost.

At minimum:
- engineer hours to first reliable test;
- target-specific configuration;
- target-specific code;
- reusable vs bespoke scenarios;
- false-positive investigation;
- compute cost;
- maintenance events.

If black-box external onboarding is consistently expensive but author-side integration is cheap, ReconcileBench may be reframed as an SDK/framework for controller authors. That outcome is acceptable and may collapse the independent-assurance/conformance story.

## Current project directions

### ReconcileBench

Highest priority. Black-box first. Investigate API-server-boundary interception — noting
that Sieve already interposes at the API server for stale-state testing, so this is partly
solved prior art rather than an open field.

### k8sbricks

First serious production controller after the fixture/Gate-1 work.

Current staging:
1. Compute/Cluster
2. Serving

Do not implement the full Databricks surface first.

Before a public CRD is released, settle the permanent project name/domain/API-group identity.

### KubeReserve

A dedicated cloud-capacity control plane remains a credible project direction.

Start with one provider only.

Safety semantics from current design:
- dry-run-oriented safe starting posture;
- mandatory TTL on claims/reservations;
- admission-time budget ceiling;
- independent sweeper;
- no required Karpenter dependency.

Exact public API requires an RFC.

### OpenEnv-related control plane

Do not assume a bespoke `OpenEnv Operator` is the final abstraction.

Current architecture recommendation:
- control-plane boundary is the environment pool, not individual episodes;
- `EnvironmentClass` + `EnvironmentPool`;
- high-frequency lease/check-out data plane should not create CRs per episode;
- OpenEnv is the first provider/protocol.

Treat naming (`envpool` or other) as provisional until RFC/upstream validation.

### SmolAgents-related control plane

Do not assume a bespoke `SmolAgent Operator`.

Current architecture recommendation:
- generic `MLRuntime` + `MLWorkload`;
- SmolAgents is the first runtime/provider;
- future frameworks should be declarative contributions where possible rather than separate Go operators.

Treat naming as provisional until RFC.

## Deliberately deferred

Do not build yet:
- Forge;
- Radar;
- Catalog;
- Drift;
- certification;
- broad community reputation/portal;
- frozen shared meta API;
- large conformance suite;
- one operator per AI/ML project.

## Tooling

Production controllers: Go + Kubebuilder/controller-runtime.

Python may be used for clients/tooling where appropriate, but not as the default control-loop runtime.

## Distribution

ArtifactHub is optional/deferred, not permanently rejected.

Public releases should eventually provide:
- OCI Helm chart;
- container image;
- SBOM;
- signature;
- provenance/verification instructions.
