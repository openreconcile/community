# ReconcileBench — Project Context

## Goal

Find Kubernetes controller defects that ordinary tests miss by introducing failures during reconciliation and determining whether the controller semantically converges.

## Initial claim

Defect discovery, not certification.

## Prior art — read before anything else

`../context/PRIOR_ART.md`. **Sieve** already does deterministically-timed mid-reconcile
fault injection (dormant since September 2024) and **Acto** already does automated
operator-correctness testing with a lower onboarding cost than anything planned here. The
original "nothing does this" framing is retired. Whether to build at all is an open question
answered by the F0 spike.

## Ground-truth fixture

`reconcilebench-fixture` must provide:
- clean control;
- seeded defects.

Initial defect classes discussed include:
- duplicate child/external creation after restart around create/status boundary;
- finalizer deadlock;
- missing `observedGeneration`;
- external create succeeds but status write fails;
- stale-cache race;
- retry storm on terminal error;
- ownership/concurrent-reconcile conflict;
- unobserved event — controller misses an event and never converges, i.e. is not truly
  level-triggered (added 2026-09-13 to match Sieve's unobserved-state pattern).

Exact list may change based on F0/F1.

## Convergence

Semantic convergence, not zero writes.

Per-target definitions may include:
- lifecycle transitions;
- invariants;
- expected conditions;
- failure model;
- ignored non-semantic fields.

Invariant classes:
- resource;
- state;
- safety;
- progress/liveness;
- idempotency;
- recovery;
- deletion.

## Fault model

Black-box first.

Investigate API-server-boundary interception to:
- fail selected calls;
- delay calls;
- inject conflicts/throttling;
- create deterministic partial-success windows.

Grey-box/instrumented modes are roadmap concepts, not Milestone-0 requirements.

## F1

Before claiming a new testing category, try to reproduce the value using existing tools.

F1 must capture both:
- coverage count;
- significance of undetected classes.

## F6

Record Target Onboarding Cost for every new subject.

A REFRAMED outcome toward an operator-author SDK is acceptable.

## Avoid

- claiming certification;
- building a generic chaos engine;
- reimplementing fault primitives that mature tools already provide;
- reporting third-party findings publicly before disclosure/remediation policy.
