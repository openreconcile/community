# Cursor Workflow for OpenReconcile

## 1. Start each project session with context

Ask Cursor to read:

```text
AGENTS.md
docs/PROJECT-CONTEXT.md
docs/ROADMAP.md
docs/context/CURRENT_STATE.md
docs/context/DECISIONS.md
docs/context/PRIOR_ART.md
docs/context/FALSIFICATION_EXECUTION.md
docs/projects/<relevant-project>.md
```

In a project repo other than `community`, the same applies to the copies of
`PROJECT-CONTEXT.md` and `.cursor/rules/` vendored into that repo.

## 2. Builder session

The builder session may see:
- RFC;
- API design;
- implementation;
- existing tests.

Ask it to produce small, reviewable changes.

## 3. Adversarial session

Use a separate Cursor context/chat.

Give it:
- RFC;
- API contract;
- invariants;
- public behaviour.

Ask it to attack:
- retry semantics;
- crash/restart;
- stale cache;
- partial writes;
- external create/status gap;
- conflict/concurrency;
- finalization/deletion;
- timeouts/rate limiting;
- external drift.

Do not automatically expose the builder's reasoning if the goal is independent adversarial thinking.

## 4. Repository is the memory

When a decision is made:
- update `DECISIONS.md`, relevant RFC, or project context;
- do not rely on Cursor's chat history as durable project memory.

## 5. Commands

Cursor should prefer repository commands such as:

```bash
make generate
make manifests
make fmt
make vet
make lint
make test
make test-envtest
make test-e2e
make test-reconcile
make verify
```

rather than inventing ad hoc local command sequences.

## 6. Scope guard

When a historical source mentions a large feature, Cursor must check current state before implementing it.

Historical ambition is not current authorization.
