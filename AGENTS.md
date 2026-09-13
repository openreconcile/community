# AGENTS.md — OpenReconcile Engineering Instructions

You are an engineering agent working on OpenReconcile.

## Read first

Before making architectural or production-code changes, read:

1. `docs/PROJECT-CONTEXT.md` — accumulated background, locked decisions, rejected approaches
2. `docs/context/GATE1.md` — product-shape decision
3. `docs/adr/ADR-0001-engine-agnostic-architecture.md`
4. `docs/ROADMAP.md` — sequenced work and what is currently active
5. `docs/context/CURRENT_STATE.md`
6. `docs/context/DECISIONS.md`
7. `docs/context/PRIOR_ART.md` — **required before any claim about differentiation**
8. the relevant file under `docs/projects/`
9. applicable files under `.cursor/rules/` and `.cursor/skills/` (or `skills/`)

## Source-of-truth precedence

When documents conflict:

1. `AGENTS.md`
2. `docs/PROJECT-CONTEXT.md`
3. `docs/context/GATE1.md` and `docs/adr/*`
4. `docs/ROADMAP.md`
5. `docs/context/*`
6. `docs/projects/*`
7. `docs/process/*`
8. `sources/*` — **historical only.** Never treat as current specification.

## Mission

OpenReconcile is an open-source, vendor-neutral community focused on improving the
resilience and correctness of Kubernetes controllers.

ReconcileBench is the first technical wedge: a **scenario, status-contract oracle, and
agent loop over existing engines**. Defect discovery, not certification. Not a new
fault-injection engine.

Hero user: an MLE productionising an existing ML tool. The pipeline is general
Kubernetes. Marketing and Skills are AI/ML-first.

Positioning line: *Capabilities are declared. Resilience is demonstrated.*

Do not advertise *Build a Kubernetes operator in natural language* until the
KubeReserve greenfield example (L3) has been replayed by someone other than the
author. k8sbricks is the AI/ML proof point, not the slogan.

Do not write code, documentation, or comments that assert “we inject faults during
reconciliation” as a novelty claim. operator-chaos and Sieve already do that.

Do not describe anything as verified, certified, production-ready, or resilient —
resilience is a property to demonstrate, never to assert.

## Non-negotiable principles

- Do not assume every project needs an operator.
- Prefer existing Kubernetes/control-plane ecosystems when they already solve the problem.
- Prefer upstream contribution to unnecessary forks. Adopt operator-chaos as an engine.
  Contribute OpenEnv `KubernetesProvider` upstream. Do not revive Sieve.
- Production controllers use Go + Kubebuilder/controller-runtime unless an approved RFC says
  otherwise. This constrains controller runtimes, not test harnesses.
- Controllers never write to `spec`; they write observed state to `status`.
- Use standard `metav1.Condition`.
- Reconciliation must be idempotent.
- Every external-resource create path must account for partial success: external create
  succeeds, local/status persistence fails.
- Every finalizer must have a safe termination path.
- Same-namespace references are the default security boundary.
- Distinguish terminal errors from recoverable errors.
- Never log secrets, credentials, tokens, or raw sensitive payloads.
- Do not freeze shared Go API structs until repeated real implementations prove the common
  shape.
- Do not publish public CRDs under a provisional API identity.
- Keep project APIs donatable: no `openreconcile` in user-facing API groups
  (fixture domain excepted).

## Current development gate

Gate 1 is closed. Build the surviving product in roadmap order:

1. Fixture (clean + eight defects)
2. Skills / portable context
3. ReconcileBench core → oracle → derive → evidence/MCP
4. controller-template, then CLI
5. Operators one at a time

## Blocked work

- **Do not run `kubebuilder init` for k8sbricks and do not commit any public
  `groupversion_info.go` for it** until permanent API identity is written in
  `docs/projects/K8SBRICKS.md`.
- The fixture's throwaway domain `fixture.openreconcile.org` is the one sanctioned exception.
- Do not create an OpenReconcile environment-pool repo.
- Do not implement a ReconcileBench fault injector (`pkg/inject`, API-server proxy,
  patched Kubernetes).
- Do not start KubeReserve cloud-SDK work before its lifecycle RFC.
- Do not ship the hosted NL demo or the NL slogan before Phase 4 passes the fixture.

## AI-agent independence

Implementation generation and adversarial test generation must be independently reviewed.

Do not ask one coding context to write a feature and then accept its own tests as sufficient
proof.

**Do not write the fixture's seeded defects as textbook implementations of the branch
name.** Defects must look like plausible mistakes. Prefer a human or a separate
adversarial pass.

## Change discipline

Before changing a public API, security boundary, reconciliation semantic, or cross-project
convention:

1. identify the existing decision;
2. explain whether the change is compatible;
3. write or update an RFC or ADR when required;
4. add migration notes if users could be affected.

When a decision changes, update `docs/context/DECISIONS.md` in the same commit.

## Testing expectations

Every controller feature should have, where applicable:

- normal-path test;
- idempotency test;
- terminal-error test;
- recoverable-error test;
- failure/recovery test;
- deletion/finalizer test;
- partial-success test for external systems.

ReconcileBench results must record exact target version, Kubernetes version, scenario,
repetitions, and validity/as-of basis.

## Do not silently broaden scope

Do not build these unless later evidence explicitly justifies them:
Forge, Radar, Catalog, Drift, certification, broad conformance machinery, a frozen shared
meta API, one bespoke operator for every AI framework, an env-pool operator, a new
fault-injection engine.

Historical ambition is not current authorization.

## If uncertain

Prefer a small RFC and an explicit question over silently inventing project policy.
