# RFC — KubeReserve reservation lifecycle (v1alpha1)

**Status:** Draft, required before cloud-SDK work
**Provider:** AWS only
**Date:** 2026-09-13

## Problem

Karpenter already *consumes* ODCRs, Capacity Blocks, and interruptible
reservations. Teams still lack a Kubernetes object that **creates, TTLs,
budgets, adopts, and sweeps** those reservations without depending on
Karpenter.

## API

Group: `capacity.kubereserve.io`

`Reservation` (namespaced):

| Field | Rule |
|---|---|
| `spec.provider` | `aws` only in v1alpha1 |
| `spec.dryRun` | default true at install |
| `spec.ttl` | **required** |
| `spec.budgetUSD` | admission rejects if org ceiling exceeded |
| `spec.instanceType` / `availabilityZone` | required for create |
| `spec.adoptID` | optional; adopt an existing reservation |

Status: `observedGeneration`, `externalID`, `expiresAt`, `Ready` condition.

## Status and dry-run behaviour (required for greenfield regeneration)

These rules were proven on the handwritten stub and must be in the RFC so
regeneration does not consult that controller:

- `spec.ttl` missing or `<= 0` → `Ready=False`, reason `TTLRequired`. Do not
  create. Set `status.observedGeneration`.
- `spec.dryRun` defaults to `true`. While true, the controller **never** calls
  a cloud SDK.
- `spec.dryRun: false` is **not implemented** until a second reviewer signs the
  AWS path. The controller refuses to spend (`Ready=False`, reason
  `LifecycleOnly` or equivalent) and still makes no cloud call.
- `status.expiresAt` is derived once from `metadata.creationTimestamp + spec.ttl`.
  Do not rewrite it on later reconciles.
- While dry-run (or refusing spend), `Ready` stays `False` with reason
  `LifecycleOnly`. Message must state that Karpenter consumes reservations and
  we only manage lifecycle.
- Requeue before `expiresAt` so expiry can be observed without a busy loop.
- Persist `externalID` before reporting Ready once the AWS path is unlocked
  (partial-success path). Not required while dry-run creates nothing.

## Controller duties

1. Create (or adopt) the cloud reservation — only after dry-run is lifted.
2. Persist `externalID` before reporting Ready (partial-success path).
3. Expire / delete on TTL.
4. Independent sweeper lists cloud reservations tagged with the cluster UID
   and deletes orphans.
5. Budget webhook sums active reservations in the namespace/org.

## Non-goals

- Scheduling pods onto reserved capacity (Karpenter / default scheduler).
- Multi-cloud in v1.
- Replacing Karpenter.

## Unlock

This RFC is enough to scaffold types. Cloud SDK calls stay behind
`spec.dryRun: true` until a second reviewer signs the AWS path.
