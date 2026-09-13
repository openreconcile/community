# KubeReserve — Project Context

## Goal

A Kubernetes control plane for cloud capacity/reservation lifecycle.

## Current direction

KubeReserve remains a plausible dedicated operator/control plane.

Start with **one provider only**. Do not build AWS/Azure/GCP simultaneously.

## Safety constraints already discussed

- safe/dry-run-oriented default posture;
- mandatory TTL on claims/reservations;
- budget ceiling enforced at admission;
- independent sweeper for leaked/stale capacity;
- no mandatory Karpenter dependency;
- future multi-cluster support should remain additive rather than force a v1 API rewrite.

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

Track Target Onboarding Cost separately for each provider because provider-specific credentials, lifecycle, and release cadence may justify separate binaries/repos later.

## Status

Project direction is valid, but implementation should not outrun Gate 1 or become a parallel multi-cloud programme.
