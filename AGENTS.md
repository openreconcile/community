# AGENTS.md — OpenReconcile Engineering Instructions

You are an engineering agent working on OpenReconcile.

## Read first

Before making architectural or production-code changes, read:

1. `docs/PROJECT-CONTEXT.md` — accumulated background, locked decisions, rejected approaches
2. `docs/ROADMAP.md` — sequenced work and what is currently active
3. `docs/context/CURRENT_STATE.md`
4. `docs/context/DECISIONS.md`
5. `docs/context/PRIOR_ART.md` — **required before any claim about differentiation**
6. `docs/context/FALSIFICATION_EXECUTION.md`
7. the relevant file under `docs/projects/`
8. applicable files under `.cursor/rules/`

## Source-of-truth precedence

When documents conflict:

1. `AGENTS.md`
2. `docs/PROJECT-CONTEXT.md`
3. `docs/ROADMAP.md`
4. `docs/context/*`
5. `docs/projects/*`
6. `docs/process/*`
7. `sources/*` — **historical only.** Contains superseded decisions, preserved verbatim for
   the reasoning behind them. Never treat as current specification.

## Mission

OpenReconcile is an open-source, vendor-neutral community focused on improving the resilience
and correctness of Kubernetes controllers.

ReconcileBench is the first technical wedge: adversarial testing of controller behaviour
during reconciliation. The initial claim is **defect discovery**, not certification.

AI/ML is currently the first market focus, but whether the core tool is technically
AI/ML-specific is deliberately being falsified.

**The project is pre-implementation and its core differentiation claim is unproven.** Sieve
and Acto already occupy much of this space. Do not write code, documentation, or comments
that assert differentiation as established. Do not describe anything as verified, certified,
production-ready, or resilient — resilience is a property to demonstrate, never to assert.

Positioning line: *Capabilities are declared. Resilience is demonstrated.*

## Non-negotiable principles

- Do not assume every project needs an operator.
- Prefer existing Kubernetes/control-plane ecosystems when they already solve the problem.
- Prefer upstream contribution to unnecessary forks. **This applies to the assurance tooling
  itself** — contributing to Acto or reviving Sieve may beat building a third tool.
- Production controllers use Go + Kubebuilder/controller-runtime unless an approved RFC says
  otherwise. This constrains controller runtimes, not test harnesses; Sieve and Acto are
  Python and must be evaluated on merit.
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
- Conventions may be locked earlier when changing them later would create user-facing
  migrations.
- Do not publish public CRDs under a provisional API identity.
- Keep project APIs donatable: umbrella ownership must not make future project transfer
  require a user API migration.

## Current development gate

Before broad product implementation, complete in this order:

1. **F0 prior-art execution spike** — run Sieve and Acto; decide build vs contribute
2. ReconcileBench fixture: clean control + eight seeded-defect variants
3. F1 prior-art challenge, with Sieve and Acto in the candidate set
4. F5a technical-scope challenge
5. F5b market-wedge challenge
6. Gate 1 decision, written into `docs/context/DECISIONS.md`

Do not use planning as an excuse to avoid implementation, but do not skip Gate 1 by creating
the full roadmap at once.

## Blocked work

- **Do not run `kubebuilder init` for k8sbricks and do not commit any public
  `groupversion_info.go` for it.** The project name is provisional but API groups embed it
  permanently. That contradiction must be resolved first.
- The fixture's throwaway domain `fixture.openreconcile.org` is the one sanctioned exception
  and is unblocked.
- Do not create `controller-template`, `kubereserve`, the environment-pool project, or the
  workload runner. Deferred behind Gate 1.

## AI-agent independence

Implementation generation and adversarial test generation must be independently reviewed.

Do not ask one coding context to write a feature and then accept its own tests as sufficient
proof. For adversarial review use a separate chat/context, give it the RFC, API and
invariants but not the builder's reasoning, and ask it to attack retry, crash, stale-state,
concurrency, partial-write, and deletion behaviour. Treat its scenarios as hypotheses, not
automatically correct tests.

**Do not write the fixture's seeded defects.** A bug generated from a description of the bug
comes out as the textbook version. The value is in defects that look like plausible mistakes
and would survive code review; a human writes those.

## Change discipline

Before changing a public API, security boundary, reconciliation semantic, or cross-project
convention:

1. identify the existing decision;
2. explain whether the change is compatible;
3. write or update an RFC or ADR when required;
4. add migration notes if users could be affected.

When a decision changes, update `docs/context/DECISIONS.md` in the same commit. The
repository is the memory; chat history is not.

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

Do not build these unless Gate 1 or later evidence explicitly justifies them:
Forge, Radar, Catalog, Drift, certification, broad conformance machinery, a frozen shared
meta API, `controller-template`, one bespoke operator for every AI framework.

Historical ambition is not current authorization. When a document under `sources/` mentions a
large feature, check current state before implementing it.

## If uncertain

Prefer a small RFC and an explicit question over silently inventing project policy.
