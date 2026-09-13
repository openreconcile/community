# SmolAgents / Generic ML Workload Runner — Project Context

## User intent

SmolAgents is one of the first desired OpenReconcile project areas.

## Current architecture recommendation

Do **not** default to a bespoke `SmolAgent Operator`.

The stronger current abstraction is a generic workload/runtime control plane.

### Likely APIs

- `MLRuntime` — cluster-scoped runtime/provider definition
- `MLWorkload` — namespaced workload request

SmolAgents becomes the first runtime/provider.

Future frameworks should be able to participate via declarative runtime definitions or small provider integrations instead of requiring a new Go operator for every framework.

## Zero-Docker/developer path

A developer-friendly path may use:
- curated base images;
- git-sync/commit pinning;
- cached dependencies.

Production should promote workloads to baked/pinned images rather than rely on mutable `pip install` at pod startup.

## Before implementation

- write an RFC for workload/runtime boundaries;
- decide how runtime providers are represented;
- define security/isolation expectations;
- avoid importing ML framework libraries into the controller process;
- settle permanent repo/API naming after architecture approval.
