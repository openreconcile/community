# ReconcileBench Testing Rules

The first goal is defect discovery, not certification.

## Prior art is mandatory reading

`docs/context/PRIOR_ART.md` and `docs/context/GATE1.md`. **Sieve** already performs
deterministically-timed mid-reconcile fault injection; **operator-chaos** is the
maintained engine we adopt; **Acto** already performs automated state-centric
operator-correctness testing and masks `observedGeneration`. Do not design a
mechanism, or write a claim about what does not exist, without checking against
those three. Do not implement a new fault injector.

## Ground truth

The fixture project must contain:
- a clean control;
- seeded-defect variants.

Use the clean control to measure false positives/specificity.
Use defect variants to measure sensitivity/detection.

Acto's own documentation records a false alarm caused by a control-flow dependency between
two spec fields — the operator correctly ignored a field because a sibling was unset, and the
resulting inconsistency was reported as a bug. That is the failure mode the clean control
exists to catch in ourselves.

## Convergence

Do not define convergence as "no writes occur."

Use semantic convergence:
- declared invariants hold;
- observed/status state corresponds to the current desired generation;
- no unresolved actionable error remains;
- subsequent reconciliation causes no semantically meaningful desired-state mutation.

Ignore-list fields such as `resourceVersion`, `managedFields`, observation timestamps,
heartbeats, and project-declared non-semantic fields only when justified. The ignore list is
part of the target definition, and getting it wrong in either direction is a tool defect:
ignoring too much hides real failures, ignoring too little manufactures false positives.

Define this against the Anvil / Eventually Stable Reconciliation literature rather than from
scratch.

## Invariant vocabulary

Support/think in terms of:
- resource invariants;
- state invariants;
- safety invariants;
- progress/liveness invariants;
- idempotency invariants;
- recovery invariants;
- deletion invariants.

Safety and liveness map to the classic distributed-systems distinction. Safety invariants are
the highest-value class because before/after testing rarely exercises them.

## F1

When comparing ReconcileBench with existing tools, record both:
1. number of seeded defects detected without substantial custom code;
2. significance/distinctness of defect classes not detected.

Do not reduce F1 to a single raw score. Sub-claim verdicts are expected and must be reported
separately rather than averaged.

operator-chaos, envtest, chainsaw, and Acto are the comparison set. Sieve is
historical reference, not a required CI target.

## Fault timing

Black-box mode ships first.

Prefer an engine adapter (operator-chaos `chaostransport` / ActionInterceptor,
envtest interceptors) over inventing an API-server proxy. Sieve already interposes
at the API server; we do not rebuild that.

## Target Onboarding Cost

Record it for every target and every tool: engineer-hours to first reliable adversarial test,
target-specific code and configuration, and ongoing maintenance burden. Acto onboards from
the operator's deployment script alone; that is the benchmark to beat.

## Evidence

Record:
- target/version;
- Kubernetes version;
- scenario;
- repetitions;
- exact fault;
- invariants;
- result;
- logs/reproducer;
- date and validity basis.

Never imply universal resilience from a narrow matrix.

## Avoid

- claiming certification;
- building a generic chaos engine;
- reimplementing fault primitives that mature tools already provide;
- reporting third-party findings publicly before the disclosure policy exists and has been
  followed.
