# k8sbricks — Project Context

## Goal

A production-grade Kubernetes control plane for Databricks, designed around Kubernetes reconciliation rather than Terraform reconciliation.

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

Examples discussed:
- configurations referencing an existing cluster may not honour the same environment/image/Spark configuration semantics as new cluster execution; unsafe assumptions should be gated/warned explicitly;
- git source should favour commit pinning for reproducibility over branch-tracking defaults.

## Critical blocker before public CRDs

Permanent project/API identity must be settled.

Do not ship a public API group based on a provisional name/domain.

Donatability means the API identity should not be unnecessarily tied to the OpenReconcile umbrella.

## ReconcileBench role

k8sbricks should be real production software, not merely a demo target.

It is also the first serious reference implementation of OpenReconcile controller conventions and a valuable ReconcileBench subject.
