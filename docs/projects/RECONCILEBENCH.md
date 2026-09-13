# ReconcileBench — Project Context

## Goal

Find Kubernetes controller defects that ordinary tests miss by running existing
perturbation engines against a target and asking a **status-contract oracle** whether
the controller semantically converged.

Defect discovery, not certification. Not a new chaos engine.

## Architecture (Gate 1 / ADR-0001)

ReconcileBench owns:

- scenario specification;
- status-contract oracle (`observedGeneration`, conditions, generation, semantic
  convergence defined against Anvil/ESR);
- `reconcilebench derive` — target models from source and CRDs, exportable as
  operator-chaos `knowledge.yaml`;
- machine-actionable evidence and an MCP server.

It does **not** own a fault injector. Engines:

| Engine | Role |
|---|---|
| operator-chaos | first-class live-cluster / transport / interceptor engine |
| envtest | first-class in-process controller tests |
| chainsaw | first-class declarative cluster assertions |
| Acto | optional adapter |

## Prior art — read before anything else

`../context/PRIOR_ART.md` and `../context/GATE1.md`.

Sieve already did timed mid-reconcile injection (dormant). operator-chaos already
does operator-semantic faults with knowledge models and verdicts (maintained).
Acto already does push-button correctness testing and masks status fields.

The original "nothing does this" framing is retired.

## Ground-truth fixture

`reconcilebench-fixture` must provide:
- clean control;
- seeded defects.

Defect classes:

- duplicate child/external creation after restart around create/status boundary;
- finalizer deadlock;
- missing `observedGeneration`;
- external create succeeds but status write fails;
- stale-cache race;
- retry storm on terminal error;
- ownership/concurrent-reconcile conflict;
- unobserved event — controller misses an event and never converges.

The fixture must be consumable by operator-chaos, envtest, and chainsaw.

**Write seeded defects by hand.** A bug generated from a description of the bug is
the textbook version.

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

## Status-contract oracle

Primary differentiator. Must detect, at minimum:

- `status.observedGeneration` never updated, or not equal to `metadata.generation`
  after a successful reconcile of that generation;
- missing or contradictory `metav1.Condition` (`Ready`, `Ready` true with an
  actionable error, stale `observedGeneration` on the condition);
- spec/status split-brain after a partial external write.

Sieve and Acto mask these fields. If our oracle also ignores them, we have no product.

## F1 measurement (post-Gate-1)

Gate 1 already reframed the product. Fixture-era F1 measures coverage per engine
against the eight defects. Report detected / not detected / detected only with
substantial target-specific code. Record Target Onboarding Cost.

## F6

Record Target Onboarding Cost for every new subject.

`derive` is the economic lever. A REFRAMED outcome toward an operator-author SDK
is acceptable if derive fails to collapse per-target YAML.

## Agent / MLE surface

- Skills: `verify` (Phase 6), used with `new-operator` / `contribute`.
- MCP server: detect → evidence → fix → re-verify.
- CLI `openreconcile new` after `controller-template` exists.
- Hosted NL demo after k8sbricks Compute exists.
- Do not advertise the NL slogan before the oracle passes the fixture.

## Avoid

- claiming certification;
- building a generic chaos engine;
- reimplementing fault primitives that operator-chaos, envtest, or chainsaw provide;
- reporting third-party findings publicly before disclosure/remediation policy;
- asserting “we inject faults during reconciliation” as a novelty claim.
