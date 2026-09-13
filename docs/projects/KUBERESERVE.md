# KubeReserve — Project Context

## Goal

A Kubernetes control plane for cloud capacity/reservation **lifecycle**
(create, adopt, expire, budget, sweep, release).

## Prior-art triage (2026-09-13)

**Go, last in queue, narrower than originally written.**

Karpenter already *consumes* reserved capacity:

- v1.3 — EC2 On-Demand Capacity Reservations;
- v1.6 — Capacity Blocks for ML;
- v1.10 — interruptible reservations;
- `ReservedCapacity` feature gate, `capacityReservationSelectorTerms` on
  `EC2NodeClass`, `karpenter.sh/capacity-type: reserved`.

Karpenter does **not** create, expire, attach a budget ceiling, or independently
sweep leaked reservations. That lifecycle is the remaining wedge.

Do not advertise “Kubernetes reserved capacity” — that sentence is Karpenter’s.
Advertise “reservation objects with TTL, budget, and a sweeper that works
without Karpenter.”

`no mandatory Karpenter dependency` remains correct: KubeReserve must work with
the default scheduler. Interop with Karpenter (labelling reservations so a
NodeClass can select them) is additive.

## Current direction

Start with **one provider only** (AWS). Do not build AWS/Azure/GCP simultaneously.

## Safety constraints

- safe/dry-run-oriented default posture (`dryRun: true` at install);
- mandatory TTL on claims/reservations;
- budget ceiling enforced at admission;
- independent sweeper for leaked/stale capacity;
- no mandatory Karpenter dependency;
- future multi-cluster support remains additive rather than a v1 API rewrite.

## Architecture discipline

Write the API/lifecycle RFC before deep cloud-SDK implementation.

The controller must explicitly handle:
- reservation creation;
- expiry;
- adoption;
- partial cloud success;
- retries;
- deletion/release;
- budget violations;
- orphan recovery.

## F6

Track Target Onboarding Cost separately for each provider because
provider-specific credentials, lifecycle, and release cadence may justify
separate binaries/repos later.

## Status

Queued last. Reconsider entirely if no second contributor exists when k8sbricks
reaches alpha. Implementation must not outrun the RFC or become a parallel
multi-cloud programme.
