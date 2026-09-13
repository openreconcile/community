# OpenReconcile — Repository Topology

## GitHub organisation

```text
github.com/openreconcile/
```

## Organisation/core repos

### `.github`
Organisation profile, shared community-health defaults, reusable workflow templates.

### `community`
Governance, conventions, disclosure policy, decision/RFC process, project-level documentation.

### `controller-template`

**Authorised after the fixture proves clean.** Seeded from `reconcilebench-fixture`,
not invented up front. When created, it should standardise:
- Kubebuilder/controller-runtime layout;
- Makefile targets;
- CI;
- security scanning;
- release scaffolding;
- Cursor rules;
- RFC/ADR structure;
- Helm layout;
- test layout.

Do not make it a shared runtime framework prematurely.

### `reconcilebench`
Main resilience-testing tool after Gate 1 confirms what the product should be.

### `reconcilebench-fixture`
Ground-truth controller:
- clean control;
- seeded-defect variants.

## Product/control-plane repos

Create/stage deliberately.

### `k8sbricks`
Databricks control plane. First serious production target: Compute/Cluster.

### `kubereserve`
Cloud capacity control plane. Start with one provider.

### OpenEnv-related project

**No org repo.** Contribute `KubernetesProvider` upstream to `meta-pytorch/OpenEnv`.
OpenSandbox already implements pool CRs.

### SmolAgents-related project
Do not lock a repo/API name until the generic workload/runtime RFC is approved.

Current likely shape:
- `MLRuntime`
- `MLWorkload`
- SmolAgents as first provider/runtime definition.

## Repositories not to create yet

Unless evidence justifies them:
- `forge`
- `radar`
- `catalog`
- `drift`
- umbrella `apis` with a frozen shared meta contract
- certification portal
