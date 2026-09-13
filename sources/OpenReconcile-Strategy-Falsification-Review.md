# OpenReconcile — Strategy Falsification Review

**Status:** Workstream definition — to be executed before Project Plan v3
**Input:** Project Plan v2 (September 2026) + Review Response & Falsification Addendum
**Date:** September 2026

---

## 0. Purpose and Rules

The previous review cycle improved execution without challenging the thesis. This document exists to attack it.

### Rules of engagement

1. **The objective is disproof, not improvement.** A workstream that produces suggestions has failed. It must produce a verdict.
2. **Every workstream ends with one label**, defined in §1, with the evidence that produced it.
3. **A label may not be assigned on intuition.** Each workstream below names its required evidence. Absent that evidence the workstream is INCOMPLETE, which is not a passing state.
4. **KILLED and REFRAMED are successful outcomes.** They are cheaper now than after implementation.
5. **Disconfirming evidence is sought actively.** Where a workstream involves asking people, the question is phrased to invite refusal, not agreement.

### Deviation from the Addendum

The Addendum (§9) recommends freezing v2 and running falsification before any building. Three of the five workstreams below cannot be answered without artefacts — you cannot test whether maintainers trust evidence without producing evidence.

Therefore: **F1 runs immediately and analytically. The fixture operator is built in parallel, because F1 cannot be answered rigorously without it and it is unambiguously wanted regardless of outcome.** Everything else waits.

If F1 returns KILLED or REFRAMED, the fixture cost is one week and a small controller worth having anyway.

---

## 1. Outcome Labels

| Label | Meaning | Consequence |
|---|---|---|
| **VALIDATED** | Evidence supports the thesis as stated | Proceed; carry the evidence into v3 as justification |
| **WEAKENED** | Direction holds; claims or scope must narrow | Rewrite the claim, reduce the scope, proceed |
| **REFRAMED** | The need is real; the product category is wrong | Redefine what is being built before proceeding |
| **KILLED** | Evidence does not justify continued investment | Stop that thesis; the rest of the plan may or may not survive |

A workstream may return different labels for different sub-claims. Record them separately rather than averaging to a summary verdict.

---

## 2. F1 — Prior-Art Challenge

> **Thesis under test:** mid-reconcile fault injection with invariant evaluation and convergence analysis is a genuine capability gap, not a thin orchestration layer over existing tools.

**Priority: run first.** This is the only workstream answerable without external input, and it is the one most likely to invalidate the project's core differentiation claim.

### 2.1 Method

Build the fixture operator (defect branches plus clean control), then attempt to detect its seeded defects using **only existing tooling**, composed as cheaply as possible:

- **chainsaw** — declarative assertions, before/after cluster state
- **Chaos Mesh** or **Litmus** — pod kill, node drain, network fault
- **envtest** — controller-level testing against a real API server
- **operator-sdk scorecard** — bundle-driven functional tests
- shell orchestration to sequence them

Timebox: one week. The question is not whether it can be made to work with unlimited effort — it is whether a competent engineer gets there quickly.

### 2.2 The seven seeded defects to detect

| # | Defect | Why it is hard to catch |
|---|---|---|
| 1 | Duplicate child creation after controller restart mid-create | Requires killing the process between external create and status write |
| 2 | Finalizer deadlock on delete | Requires a dependency that fails during finalization |
| 3 | Missing `observedGeneration` update | Requires distinguishing stale status from correct status |
| 4 | External create succeeds, status write fails | Requires failing one API call and not others |
| 5 | Stale-cache race | Requires serving the controller outdated state deliberately |
| 6 | Retry storm on terminal error | Requires observing write frequency over time |
| 7 | Ownership conflict between two reconcile passes | Requires concurrency control |

For each: **detected / not detected / detected only with target-specific custom code**. That third column is the interesting one — if every defect needs bespoke glue, the composition is not a substitute for a tool.

### 2.3 Evidence required for a label

| Label | Evidence |
|---|---|
| VALIDATED | Existing tools detect ≤2 of 7 without substantial custom code; the gap is a distinct capability |
| WEAKENED | Existing tools detect 3–5; the gap is real but narrower than the plan claims — narrow the pitch to the specific undetectable classes |
| REFRAMED | Existing tools detect ≥6 with a modest harness; ReconcileBench becomes a **scenario library and orchestration layer** over chainsaw and Chaos Mesh, not a new testing category |
| KILLED | Existing tools detect all 7 easily and the composition is already documented practice somewhere |

**REFRAMED is the most likely outcome and is not a bad one.** A well-designed scenario library with a convergence oracle is still valuable and considerably cheaper to build. The pitch changes from "a new category of testing" to "the missing scenarios and the missing oracle."

### 2.4 Secondary question

Does the **API-server boundary interception** approach (Addendum §2) already exist? Check for existing proxy-based Kubernetes fault injection before designing one. If it exists, use it. If it does not, that absence is itself evidence supporting VALIDATED, because it is the mechanism the composition above cannot replicate.

---

## 3. F2 — Trust Challenge

> **Thesis under test:** reproducible evidence from an unknown open-source project is sufficient to make a maintainer act.

**Runs after F1 and after the first real findings exist.** Cannot be answered hypothetically.

### 3.1 Method

Do not survey opinions. Produce a real finding against a real controller and observe behaviour.

1. Run ReconcileBench against 2–3 third-party operators of moderate profile
2. Report through the disclosure policy
3. Measure what happens

Ask maintainers a question designed to elicit refusal: *"What would have made you ignore this report?"* — not *"was this useful?"*

### 3.2 Evidence required for a label

| Label | Evidence |
|---|---|
| VALIDATED | ≥1 maintainer accepts the finding as a bug and changes code, without institutional backing being requested |
| WEAKENED | Findings acknowledged but deprioritised; trust exists but urgency does not — the claim becomes "surfaces real issues," not "changes release decisions" |
| REFRAMED | Maintainers want the tool to run themselves rather than receiving external reports; the product is a tool, not an assurance service, and Catalog/Drift die with it |
| KILLED | Findings dismissed or ignored across all targets, with reproducibility explicitly not the blocker |

### 3.3 What to watch for

If the common response is *"who are you"* rather than *"is this correct"*, the constraint is institutional standing rather than evidence quality, and the roadmap needs an explicit trust-acquisition path — a CNCF sandbox application, a co-maintainer with existing standing, or a known project's endorsement. Note this as a required roadmap item rather than treating it as a soft factor.

---

## 4. F3 — Conformance Challenge

> **Thesis under test:** external projects want to conform to OpenReconcile conventions, making conformance a viable participation mechanism.

### 4.1 Method

Publish the conventions and the conformance suite. Then observe the split between two behaviours:

- projects that run the conformance suite and adopt conventions
- projects that take the ReconcileBench scenario packs and ignore the conventions entirely

The second group is the falsifying evidence. It indicates the tool has value and the standards layer does not.

Ask directly, with an easy exit: *"Would you change your API conventions to conform, or would you only want the test packs?"*

### 4.2 Evidence required for a label

| Label | Evidence |
|---|---|
| VALIDATED | ≥2 external projects adopt conventions or state clear intent to conform |
| WEAKENED | Interest in conventions as documented guidance, but no adoption; publish as recommendations, drop the conformance framing |
| REFRAMED | Only the test packs are wanted; **drop the standards layer entirely** and with it the community-participation model built on it |
| KILLED | Neither conventions nor packs attract external use |

### 4.3 Consequence if REFRAMED

The community model in Plan v2 §10 rests entirely on conformance-as-participation. If that fails, OpenReconcile has no mechanism for external contribution beyond ordinary PRs, and the "community" framing should be dropped rather than maintained aspirationally. An org that calls itself a community and functions as one person's portfolio damages credibility more than an honestly-labelled personal project.

---

## 5. F4 — Track-B Coupling Challenge

> **Thesis under test:** the assurance work and the operator portfolio benefit from shared organisational identity, not merely a technical relationship.

### 5.1 Method

Analytical, but stated as a falsifiable claim. Ask what the coupling actually buys, and whether each benefit survives decoupling:

| Claimed benefit of coupling | Survives if separated? |
|---|---|
| A controller to test on day one | Yes — technical relationship suffices |
| Credible founding story | Partly — "we test our own operator" works across orgs |
| Conventions proven by implementation | Yes — conventions can be published independently |
| Shared contributor base | Unclear — the audiences differ (platform engineers vs data engineers) |
| Simpler narrative | **No — coupling makes the narrative harder to explain** |

Test the last row directly: describe OpenReconcile to five engineers in one sentence and record how many ask a clarifying question about scope.

### 5.2 Evidence required for a label

| Label | Evidence |
|---|---|
| VALIDATED | The coupling demonstrably drives adoption or contribution in at least one direction |
| WEAKENED | Coupling is neutral; keep it but stop featuring it in positioning |
| REFRAMED | Coupling costs explanatory clarity without benefit; **separate the orgs, keep the technical relationship** |
| KILLED | The operator portfolio actively impedes the assurance work's credibility (perceived conflict of interest when grading others' controllers while shipping your own) |

### 5.3 Note on the KILLED condition

There is a genuine conflict-of-interest exposure here that Plan v2 does not address: an organisation that publishes resilience findings on operators while also shipping a competing operator has an obvious incentive problem. If k8sbricks ever overlaps with a controller being tested, this becomes a live issue rather than a theoretical one. Consider it as part of this workstream, not separately.

---

## 6. F5 — Scope Challenge (new)

> **Thesis under test:** the AI/ML framing is technically motivated rather than a branding choice that narrows the audience.

Not present in the Addendum. Added because nothing about mid-reconcile fault injection is specific to ML workloads.

### 6.1 Method

For each element of the plan, ask whether it would differ if the target were databases, networking, or storage operators:

- ReconcileBench scenarios — do any depend on ML-specific behaviour?
- Invariant classes — any ML-specific class?
- Convergence definition — differs for ML controllers?
- The conventions in Plan v2 §8 — ML-specific in any respect?

Then check where the strongest early adopters actually are. Operator maturity is generally lower in ML tooling, which argues for the framing — but the tool's value is highest where controllers are complex and widely deployed, which may not be the same place.

### 6.2 Evidence required for a label

| Label | Evidence |
|---|---|
| VALIDATED | ≥2 plan elements are genuinely ML-specific; the framing reflects technical reality |
| WEAKENED | Framing is a go-to-market focus rather than a technical boundary; keep it, but say so explicitly and do not build ML assumptions into the tool |
| REFRAMED | The tool is general-purpose; the ML framing narrows the audience for no technical reason. **Broaden the tool; the AI/ML focus becomes a first market, not an identity** |
| KILLED | n/a — this workstream cannot kill the project, only rescope it |

### 6.3 Why this matters for naming

If F5 returns REFRAMED, the org identity question reopens — "OpenReconcile" survives a broadening fine, but the AI/ML positioning in the plan's problem statement does not. Better to discover this before domains and API groups are registered.

---

## 7. Execution Sequence

| Phase | Workstream | Depends on | Timebox |
|---|---|---|---|
| **Now** | Build fixture operator (defect + clean branches) | — | 1 week |
| **Now** | **F1 — Prior art** | Fixture | 1 week |
| **Now** | **F5 — Scope** | — (analytical, run in parallel) | 2 days |
| **Gate 1** | Decide whether ReconcileBench is a tool, a scenario library, or nothing | F1, F5 | — |
| Next | Build to the Gate 1 conclusion | Gate 1 | — |
| Next | **F2 — Trust** | Real findings on 2–3 third-party operators | — |
| Next | **F3 — Conformance** | Published conventions + suite | — |
| Next | **F4 — Coupling** | F2 (perceived conflict is only testable once findings exist) | — |
| **Gate 2** | Write Project Plan v3 | All five labelled | — |

Gate 1 is the important one. It is cheap, fast, and determines whether anything else is worth doing.

---

## 8. Standing Constraints During Falsification

These apply regardless of outcomes and should not wait for v3:

1. **Do not register API groups or ship public CRDs** until the k8sbricks name/API-identity contradiction is resolved. F5 may also bear on this.
2. **Set the compute-budget ceiling** before designing the scenario matrix. Every published result states its exact tested matrix and repetition count; never imply general resilience from a narrow matrix.
3. **Disclosure policy must be written and published** before F2 begins.
4. **Reviewer acquisition is an output problem, not a recruitment problem.** A maintainer who receives a credible bug report in their own project is a reviewer with something at stake. Do not build a reviewer panel; produce findings and let respondents self-select.

---

## 9. What Would Make This Review Have Failed

Stated explicitly, so it can be checked afterwards:

- Every workstream returns VALIDATED. Real falsification attempts on a young project almost never do; uniform validation indicates the questions were asked to be passed.
- Labels are assigned without the evidence each workstream specifies.
- A REFRAMED result is treated as a setback and quietly re-litigated back to VALIDATED.
- The review produces a longer plan rather than a shorter one.

The expected healthy outcome is one or two REFRAMED, one or two WEAKENED, and a v3 that is meaningfully narrower than v2.
