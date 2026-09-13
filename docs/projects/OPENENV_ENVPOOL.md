# OpenEnv — Project Context

## Decision (2026-09-13)

**Do not create an OpenReconcile environment-pool operator or repo.**

Contribute a `KubernetesProvider` upstream to
[`meta-pytorch/OpenEnv`](https://github.com/meta-pytorch/OpenEnv)
(docs still list it as 🚧 planned; the class is a placeholder).

## Prior-art triage

### OpenSandbox already shipped the pool design

[opensandbox-group/opensandbox](https://github.com/opensandbox-group/opensandbox)
implements `Pool` and `BatchSandbox` custom resources:

- pre-warmed pod buffers;
- `PoolMin` / `PoolMax` capacity;
- automatic allocation and deallocation;
- batch delivery aimed at high-throughput agentic-RL;
- pause/resume via rootfs snapshots.

`PoolReconciler.scalePool` already computes `desiredSchedulableCnt` against
buffer counts and evicts redundant pods.

The earlier OpenReconcile design call — “the CR boundary is the pool, never the
episode” — is therefore implemented. Rebuilding it would violate upstream-first
and Operator-or-Not.

### OpenEnv core is the right upstream

- Runtime providers already include Docker, Swarm, UV, Daytona, Azure Container
  Apps, Modal.
- `KubernetesProvider` is the missing backend.
- Governance is a multi-org technical committee (Meta-PyTorch, Nvidia,
  Microsoft, Hugging Face, Modal, Prime Intellect, and others).
- Issue #441 documents third parties already wiring OpenEnv onto their own
  Kubernetes orchestrators.

A provider that starts a pod/service, returns a `base_url`, and implements
`wait_for_ready` / `stop` is a fraction of a new operator and is the advertised
gap.

## What must not happen

- Do not create one Kubernetes CR per episode.
- Do not create `EnvironmentClass` / `EnvironmentPool` under this org.
- Do not compete with OpenSandbox on pooling.

## Delivery

Draft contribution (workspace, not an org repo):
`contrib/openenv-kubernetes-provider/` next to this community tree.
