# OpenReconcile — Current Falsification Execution Model

## Core rule

The falsification process must not become a procrastination mechanism.

There are two phases.

# Phase A — Pre-Gate-1 falsification

Cheap, fast, allowed to stop/reframe the project before meaningful investment.

## F0 — Prior-art execution spike

**Added September 2026, runs before the fixture.** Two days.

The differentiation claim was originally stated against industry tooling only
(`operator-sdk scorecard`, chainsaw, envtest, Chaos Mesh, Litmus) and is false as stated:
Sieve already performs deterministically-timed mid-reconcile fault injection, and Acto
already performs automated operator-correctness testing with a lower onboarding cost than
anything currently planned here. See `PRIOR_ART.md`.

F0 is execution, not desk research:

- install and run Sieve; establish whether it still works after two years dormant, and
  measure the porting cost in hours;
- install and run Acto against one public operator; measure onboarding cost;
- establish whether either can express a fault at an **external, non-Kubernetes API
  boundary**. This is the load-bearing question, because that class is the strongest
  remaining candidate for genuine novelty;
- search once more for a maintained proxy-based Kubernetes API fault injector;
- decide whether to contact the authors.

### Verdict

F0 answers a question that did not previously exist:

> Should we build at all, or contribute to Acto / revive Sieve?

Upstream-first is a locked decision and it applies to the assurance tooling itself. A
decision to contribute rather than build is a successful F0 outcome costing two days.

F0 also supplies the first Target Onboarding Cost data points for F6, free.

## Build the fixture

Maintain:
- clean control;
- seeded-defect variants.

The fixture is useful regardless of the final ReconcileBench architecture.

## F1 — Prior-art challenge

Attempt to detect seeded defects using existing tools with modest composition effort.

Candidate existing building blocks include:
- **Sieve** and **Acto** — the closest prior art. Non-negotiable inclusions; an F1 verdict
  that ignores the two tools nearest the thesis is worthless;
- chainsaw;
- Chaos Mesh or Litmus;
- envtest;
- operator-sdk scorecard;
- shell/harness orchestration;
- any existing API-proxy/fault-injection mechanism discovered during F0.

For every defect record:
- detected;
- not detected;
- detected only with target-specific custom code.

Verdicts must consider:
1. detection count;
2. significance/distinctness of missed defect classes.

Sub-claim verdicts are allowed.

## F5a — Technical scope

Question:
> Is ReconcileBench technically AI/ML-specific?

A likely healthy outcome is that the core is general Kubernetes controller resilience tooling.

## F5b — Market wedge

Separate question:
> Even if technically general, is AI/ML the best first market?

A plausible combined result is:
- F5a REFRAMED;
- F5b VALIDATED.

## Gate 1

Gate 1 answers:

> What exactly is differentiated enough to build?

Possible outcomes:
- new controller-resilience testing capability;
- narrower hard-failure-class tool;
- scenario library + convergence oracle over existing tools;
- general-purpose controller resilience tool with AI/ML first;
- upstream contribution to Acto, or revival of Sieve, instead of a third tool;
- nothing worth continuing.

# Phase B — Build and collect falsification evidence

After Gate 1, stop creating separate research phases. Build the surviving product and measure the remaining assumptions.

## F2 — Trust

When real findings exist, observe maintainer behaviour:
- ignored;
- reproduced;
- accepted as bug;
- fixed;
- tool adopted by maintainer;
- used in CI/release decision.

One acted-on finding proves initial value, not ecosystem trust.

Trust ladder:

```text
Interesting
→ Reproducible
→ Acted on once
→ Repeated across unrelated projects
→ Independently reproduced
→ Trusted in release/CI decisions
```

## F3a — Convention interest

Publish a lightweight conventions document alongside real engineering output.

Do not build a full suite yet.

Observe whether maintainers:
- adopt conventions;
- reference them;
- object to them;
- ignore them but want the tests.

## F3b — Formal conformance

Only build if F3a shows genuine demand.

F6 may eliminate the need for F3b entirely.

## F4 — Coupling

Collect evidence while OpenReconcile ships both assurance tooling and production controllers.

Look for:
- contributor overlap;
- adoption benefits;
- narrative cost;
- perceived conflict of interest;
- reluctance to trust findings because the org ships competing controllers.

A five-engineer explanation test may be used as supporting evidence only.

## F6 — Economic / Maintenance challenge

This is continuous telemetry, not a separate research program.

Every target onboarding records:
- engineer hours to first reliable test;
- target configuration size;
- target-specific code;
- reusable vs bespoke scenarios;
- false positives/false negatives where measurable;
- manual interpretation;
- compute cost;
- maintenance/version churn.

Possible outcomes:
- VALIDATED: mostly declarative/generic onboarding;
- WEAKENED: useful curated framework with adapters;
- REFRAMED: author-side SDK/framework is the better model;
- KILLED: bespoke onboarding costs approach handwritten fault tests.

## Rule against falsification bureaucracy

No falsification workstream should require an artefact that would not otherwise be useful to the surviving product.

## Gate 2

Use accumulated real-world evidence to produce a **narrower** Project Plan v3.

If v3 is simply longer and broader than v2, the process failed.
