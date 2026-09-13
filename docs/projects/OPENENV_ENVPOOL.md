# OpenEnv / Environment Pooling — Project Context

## User intent

OpenEnv is one of the first desired OpenReconcile project areas.

## Current architecture recommendation

Do **not** automatically create a bespoke `OpenEnv Operator`.

The stronger current abstraction is a generic environment-pool control plane.

### Control-plane boundary

The CR boundary is the **pool**, not the episode/rollout lease.

High-frequency RL rollouts may check out/release environments thousands of times. Representing each lease as a CR would put inappropriate high-frequency data-plane work into etcd/Kubernetes APIs.

### Likely API shape

- `EnvironmentClass`
- `EnvironmentPool`

### Provider model

Protocol-specific behaviour sits behind a small provider interface.

OpenEnv is the first provider.

High-frequency lease/check-out behaviour belongs in an in-memory/data-plane broker.

## Before implementation

- validate current OpenEnv upstream direction;
- avoid competing with upstream architecture;
- write an RFC;
- settle permanent project/repo/API name only after the abstraction is approved.

## What must not happen

Do not create one Kubernetes CR per episode simply because CRDs are available.
