# OpenReconcile — Current State

**Status:** Gate 1 closed / fixture implementation
**Date:** 2026-09-13

This is the latest consolidated project state. It overrides conflicting historical wording in `sources/`.

## Identity

- Umbrella/community: **OpenReconcile**
- GitHub org: `github.com/openreconcile`
- Primary domain: `openreconcile.org`
- Developer domain: `openreconcile.dev`

The umbrella name is considered locked for practical execution.

## Current positioning

OpenReconcile is an open-source, vendor-neutral community focused on improving the resilience and correctness of Kubernetes controllers.

The hero user is an **MLE** who already has an ML tool and wants it productionised on Kubernetes without becoming a controller-runtime author.

ReconcileBench is the first technical wedge. After Gate 1 it is:

> a scenario library, status-contract oracle, and agent loop over existing perturbation engines.

Do not position it as a new chaos category. Do not position OpenReconcile as a certification authority.

Public line until the oracle catches the eight fixture defects: *Capabilities are declared. Resilience is demonstrated.*

After L3 has been replayed by someone other than the author: *Build a Kubernetes operator in natural language.* k8sbricks is the AI/ML proof point.

AI/ML is the first market. The pipeline is general-purpose Kubernetes.

## Immediate goal

Implement Gate 1’s surviving product:

1. `reconcilebench-fixture` — clean Widget + eight defect branches, consumable by operator-chaos, envtest, chainsaw.
2. Portable agent context + Skills (`new-operator`, `contribute`).
3. ReconcileBench core (scenario + adapters), oracle, `derive`, evidence, MCP.
4. `controller-template` seeded from the fixture.
5. `openreconcile new` CLI.
6. Operators one at a time: k8sbricks Compute, upstream OpenEnv `KubernetesProvider`, KubeReserve.

## After the fixture

Collect later evidence as a by-product of real work:

- F2: trust/maintainer response;
- F3a: convention interest from a simple published conventions document;
- F4: coupling/neutrality evidence (watch, already decided via donatability);
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

`reconcilebench derive` is the lever. If every target still needs a hand-written knowledge model, reframe as an author SDK.

## Current project directions

### ReconcileBench

Highest priority. Engine-agnostic. See `GATE1.md` and ADR-0001. Black-box first.

### k8sbricks

First serious production controller after the fixture.

Current staging:
1. Compute/Cluster
2. Serving

Do not implement the full Databricks surface first.

Before a public CRD is released, settle the permanent project name/domain/API-group identity.

The Crossplane incumbent `glalanne/provider-databricks` is live. Differentiate on Kubernetes-native reconciliation, OIDC, and encoded domain knowledge.

### KubeReserve

Reservation *lifecycle* control plane (create, TTL, budget, sweeper, adoption, orphans).
Karpenter already *consumes* ODCRs. Start with one provider. RFC before cloud SDK.

### OpenEnv

Upstream `KubernetesProvider` contribution to `meta-pytorch/OpenEnv`. No org repo.

### SmolAgents-related control plane

Still deferred. Generic `MLRuntime` + `MLWorkload` if it ever starts.

## Deliberately deferred

Do not build yet:
- hosted NL demo (gated on k8sbricks + oracle);
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
