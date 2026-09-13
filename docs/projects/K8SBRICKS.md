# k8sbricks — Project Context

## Goal

A production-grade Kubernetes control plane for Databricks, designed around
Kubernetes reconciliation rather than Terraform reconciliation.

## Prior-art triage (2026-09-13)

**Go — with a live incumbent, not an empty market.**

| Project | Status | Implication |
|---|---|---|
| `Azure/azure-databricks-operator` | Archived, last push 2021-06-04 | Dead Kubernetes-native attempt |
| `mach-kernel/databricks-kube-operator` | Archived 2025-06 | Dead kube-rs attempt |
| `glalanne/provider-databricks` | **Live.** Upjet/Crossplane, marketplace `lalanne/provider-databricks` **v2.5.0 (2026-08-31)** | Terraform-derived managed resources. This is the incumbent. |
| `upbound/provider-databricks` | 404 — path was wrong | Do not cite |
| `upbound/provider-azure-databricks` | Live | Azure *workspace* infra, not workspace-level compute/jobs |

k8sbricks is justified only as Kubernetes-native reconciliation + OIDC-first auth
+ encoded Databricks domain knowledge. It is not justified as “nobody else does
Databricks on Kubernetes.”

## Current stage

Do not start with the full Databricks surface.

### First family: Compute / Cluster

Reasons:
- genuine asynchronous lifecycle;
- meaningful state transitions for ReconcileBench;
- manageable first scope;
- exposes important external-system semantics early.

### Second family: Serving

Adds richer rollout/lifecycle semantics once the generator and testing approach are proven.

## Architecture direction

- Go + Kubebuilder/controller-runtime.
- Investigate code generation from `databricks-sdk-go` public service packages.
- Evaluate reuse/adaptation of ACK code-generator concepts before writing a generator from scratch.
- Keep generated structure separate from handwritten operational hooks.
- Adoption of pre-existing resources is important.
- Late initialization is important where Databricks applies server-side defaults.
- OIDC/workload identity is preferred over PAT-first design.

## Domain knowledge to preserve

- configurations referencing an existing cluster may not honour the same
  environment/image/Spark configuration semantics as new cluster execution;
  unsafe assumptions should be gated/warned explicitly;
- git source should favour commit pinning for reproducibility over
  branch-tracking defaults.

## Critical blocker before public CRDs

**Permanent project/API identity must be settled.**

Do not ship a public API group based on a provisional name/domain.

Donatability means the API identity should not be unnecessarily tied to the
OpenReconcile umbrella.

**Locked 2026-09-13** (see `K8SBRICKS-API-IDENTITY.md`):

- Cosmetic name: `k8sbricks`
- API group family: `compute.k8sbricks.io` (later `serving.k8sbricks.io`, …)
- **Not** `*.openreconcile.io`
- Register `k8sbricks.io` as an ops follow-up; the group is locked regardless so
  `kubebuilder init` is unblocked for Compute spike code only. Do not publish
  a v1 channel until the domain is registered.

## ReconcileBench role

k8sbricks is real production software, not merely a demo target.

It is also the first serious reference implementation of OpenReconcile controller
conventions and a valuable ReconcileBench subject.
