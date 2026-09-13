# ADR-0001 — Engine-agnostic ReconcileBench architecture

**Status:** Accepted
**Date:** 2026-09-13
**Deciders:** Gate 1 (`docs/context/GATE1.md`)
**Supersedes:** implicit “build a mid-reconcile fault injector” thesis in Plan v2 and the
original `PROJECT-CONTEXT.md` §2.

## Context

The original ReconcileBench thesis was that no maintained tool injects faults *during*
reconciliation and measures convergence. That was false:

- Sieve (OSDI ’22) already does deterministically-timed mid-reconcile injection.
- operator-chaos (Open Data Hub / Red Hat, 2026) is a maintained Go framework with
  twenty injection types and four integration modes.
- Acto already does push-button operator-correctness testing.

Building a fifth engine would violate upstream-first and lose F6 (Target Onboarding Cost)
before a single target was onboarded.

What those tools still do not do:

- treat the Kubernetes status contract as an oracle (Sieve and Acto mask
  `observedGeneration` and `conditions`);
- derive the target / knowledge model from controller source and CRDs;
- emit machine-actionable evidence an agent can consume in a detect-fix-verify loop;
- sit above more than one perturbation engine.

## Decision

ReconcileBench **does not implement a fault-injection engine**.

It owns four layers above pluggable engines:

1. **Scenario spec** — what to perturb, against which target, with which expected
   invariants.
2. **Status-contract oracle** — `observedGeneration`, condition semantics,
   `metadata.generation`, semantic convergence (Anvil/ESR-shaped).
3. **`reconcilebench derive`** — generate the target model from source and CRD schemas.
4. **Evidence + MCP** — structured findings, agent-driven fix and re-verify.

First-class engines: **operator-chaos**, **envtest**, **chainsaw**. Acto is an optional
adapter. Chaos Mesh / Litmus remain infrastructure primitives an engine may already use;
we do not wrap them directly.

## Consequences

- No `pkg/inject`, no API-server proxy of our own, no patched Kubernetes.
- Engine adapters are allowed to be thin. A missing adapter is a gap, not a reason to
  reimplement the engine.
- Knowledge models produced by `derive` must be exportable as operator-chaos
  `knowledge.yaml` so we are a producer for their ecosystem, not a fork.
- Claims of novelty are restricted to the four owned layers. Docs and comments that
  reassert “we inject faults during reconciliation” are defects.
- Complementary outreach to operator-chaos maintainers is required (Phase 0.75), not
  optional courtesy.

## Alternatives considered

| Alternative | Why not |
|---|---|
| Revive Sieve | Requires a patched Kubernetes and ~1.18-era toolchain. Maintenance cost killed it once. |
| Contribute only to Acto | Acto’s architecture is state-centric and Python. Status-contract oracles and an agent loop do not fit as a small PR. |
| Contribute only to operator-chaos | Their knowledge models are hand-written YAML; their verdicts are not agent-actionable. Complementary layers are large enough to own. |
| Build our own injector anyway | Violates upstream-first; loses F6 immediately. |

## Follow-up

- `docs/projects/RECONCILEBENCH.md` is the living product brief.
- Scenario file schema is specified when Phase 3 lands, not in this ADR.
