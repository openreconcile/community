# OpenReconcile — Roadmap and Next Steps

Sequenced work. Read `PROJECT-CONTEXT.md` first for why the order is what it is, and
`context/PRIOR_ART.md` for why F0 was inserted ahead of the fixture.

---

## Repositories to create now

| Repo | Contents | Status |
|---|---|---|
| `openreconcile/.github` | Org profile README, issue templates, code of conduct | to create |
| `openreconcile/community` | This context set; later `CONVENTIONS.md`, `GOVERNANCE.md`, `ARCHIVAL.md`, `DISCLOSURE.md` | created |
| `openreconcile/reconcilebench-fixture` | Fixture controller: clean control + seeded defects | after F0 |
| `openreconcile/reconcilebench` | The tool — created after Gate 1 | blocked |

**`controller-template` comes later.** A scaffold template written before building two
things will encode guesses. `context/REPO_TOPOLOGY.md` and
`process/ORG_BOOTSTRAP_CHECKLIST.md` previously put it in the first batch; that conflict is
resolved in favour of deferring it.

Point `openreconcile.org` at the community repo's docs. Leave `.dev` unused until Gate 1.

---

## Milestone 0 — Falsification to Gate 1

Roughly two and a half weeks. Cheap, fast, and allowed to stop or reframe the project before
meaningful investment.

### Step 0 — F0 prior-art spike (two days, runs first)

**Added September 2026.** The differentiation claim was stated against industry tooling only
and is false as stated. Two research tools occupy the same space. Establishing what they
actually do is cheaper and more decisive than building the fixture, and the result changes
the fixture's design.

Execute, do not desk-research:

1. Install **Sieve** (`sieve-project/sieve`). Determine whether it still builds against
   current Kubernetes and `controller-runtime` after two years dormant. Port one trivial
   controller and record the hours.
2. Install **Acto** (`xlab-uiuc/acto`). Run it against one public operator. Record onboarding
   hours and what its oracles catch.
3. Answer the seven questions in `context/PRIOR_ART.md` §7. The load-bearing one: **can
   either tool express a fault at an external, non-Kubernetes API boundary?**
4. Search once more for a maintained proxy-based Kubernetes API fault injector.
5. Decide whether to contact the Sieve and Acto authors.

**Output:** a written update to `context/PRIOR_ART.md` with measured onboarding costs, plus
an explicit answer to open question 2 in `PROJECT-CONTEXT.md` §9 — build, contribute to
Acto, or revive Sieve. Upstream-first is a locked decision and it now applies to the
assurance tooling itself.

**If Sieve turns out to be alive, ergonomic, and covers the external boundary, Gate 1 may
arrive early and return outcome E.** That is a successful outcome costing two days.

### Step 1 — Fixture controller (one week)

```bash
mkdir reconcilebench-fixture && cd $_
kubebuilder init --domain fixture.openreconcile.org \
  --repo github.com/openreconcile/reconcilebench-fixture
kubebuilder create api --group testing --version v1alpha1 --kind Widget \
  --resource --controller
```

`fixture.openreconcile.org` is a throwaway domain on infrastructure we own. It is the one
place a `kubebuilder init` is currently unblocked, and the one sanctioned exception to the
no-org-name-in-API-groups rule — the fixture is never donated and never installed by a user.

**`Widget` must exercise, at minimum:**

- a child Deployment (in-cluster resource with owner reference)
- a child ConfigMap (second child, to expose partial-creation windows)
- a simulated **asynchronous external resource** — a fake remote service with create/get/
  delete and a provisioning delay, so external-boundary failures are reachable
- status conditions and `observedGeneration`
- a finalizer with `Delete` / `Orphan` policy

The external simulator is the important part twice over: without it the highest-value defect
classes cannot be expressed, **and** it is the axis on which Sieve and Acto appear weakest.

**Constraint added by F0:** the fixture must be consumable by every candidate harness —
Sieve, Acto, chainsaw, and a bespoke runner. That means a deployment script Acto can take as
its sole input, a CRD schema rich enough for Acto to mutate meaningfully, and a build that
can be pointed at instrumented dependencies for Sieve. Design for this before writing the
controller, not after.

### Step 2 — Branches

`main` is the **clean control branch** and must be genuinely correct. It measures
false-positive rate. A tool that reports failures on a correct controller is worse than
useless, and specificity decides whether anyone trusts the output. Acto's own documentation
records a false alarm caused by a control-flow dependency between two spec fields — that is
the failure mode this branch exists to catch in ourselves.

Then eight defect branches, one deliberate bug each, documented in the branch README with
what correct behaviour would have been:

| Branch | Defect | Why it is hard to catch |
|---|---|---|
| `defect/duplicate-create` | Child created again after restart mid-create | Needs process kill between external create and status write |
| `defect/finalizer-deadlock` | Finalization blocks forever when dependency is down | Needs a dependency failure during delete |
| `defect/missing-observed-generation` | Status never updates `observedGeneration` | Needs distinguishing stale status from correct status |
| `defect/split-write` | External create succeeds, status write fails, ID is lost | Needs failing one specific API call |
| `defect/stale-cache` | Acts on cached state that no longer reflects reality | Needs serving the controller outdated state |
| `defect/retry-storm` | Retries a terminal error indefinitely | Needs observing write frequency over time |
| `defect/ownership-conflict` | Two concurrent reconciles corrupt shared state | Needs concurrency control |
| `defect/unobserved-event` | Misses an event and never converges — not truly level-triggered | Needs control over informer/reconciler interleaving |

The eighth branch is new. It corresponds to Sieve's unobserved-state pattern and had no
equivalent in the original seven, which was a gap in our fixture rather than a gap in Sieve.

**Write the defect branches by hand.** A bug generated from a description of the bug tends
to be the textbook version. The value is in defects that look like plausible mistakes and
would pass review. Let an agent write the surrounding boilerplate; write the bugs yourself.

Review `main` line by line against `.cursor/rules/10-controller-conventions.mdc` before
creating any defect branch. If the clean branch has an accidental bug the whole ground-truth
exercise is compromised.

### Step 3 — F1 prior-art challenge (one week, timeboxed)

Attempt to detect all eight defects using **only existing tools**. The candidate set is now:

- **Sieve** and **Acto** — the closest prior art, and non-negotiable inclusions. A verdict
  that ignores them is worthless
- chainsaw, Chaos Mesh or Litmus, envtest, `operator-sdk scorecard`
- shell orchestration to sequence them

Record per defect: **detected / not detected / detected only with substantial
target-specific code.** That third column is the interesting one.

Score on two dimensions, not one:

- **coverage quantity** — how many detected without substantial custom code
- **defect-class significance** — whether what remains undetected is a distinct,
  operationally important class

Numerical thresholds are guidance, not a scoring function. Six of eight detected still
supports a narrower VALIDATED if the remainder is partial-success-across-an-external-boundary,
which is the hardest and most common real-world class.

Report sub-claims separately. Do not average to one verdict. A healthy F1 might read:
generic fault injection REFRAMED, external-boundary partial-success VALIDATED, scenario
orchestration WEAKENED.

Also record **Target Onboarding Cost** for each tool against the fixture. This is F6's first
data point and it arrives free here.

### Step 4 — F5a/F5b scope challenge (two days, parallel)

**F5a:** would any scenario, invariant class, or convergence semantic change if the targets
were database, networking, or storage operators? If not, the tool is general-purpose. Note
that Sieve and Acto were both evaluated on databases and messaging systems — Cassandra,
RabbitMQ, TiDB, ZooKeeper — which is itself evidence for REFRAMED.

**F5b:** is AI/ML still the best *first market* even if the tool is general? Consider that
more mature Kubernetes domains have more complex controllers and stronger engineering teams,
and that the research tools already harvested the mature-database operators.

Likely and healthy outcome: **F5a REFRAMED, F5b VALIDATED** — a general-purpose Kubernetes
controller resilience tool, launched into AI/ML infrastructure first.

### Gate 1

Answer explicitly, in writing, in `context/DECISIONS.md`:

- **A.** A genuinely distinct testing category
- **B.** A narrower tool targeting a few hard failure classes
- **C.** A scenario library and convergence oracle over existing tools
- **D.** A general controller-testing tool with AI/ML as first market
- **E.** Nothing sufficiently differentiated remains
- **F.** *(new)* Upstream contribution — the right move is improving Acto or reviving Sieve
  rather than building a third tool

C, D, and F are the most likely. Do not proceed until one is chosen.

---

## Milestone 0B — Build to the Gate 1 conclusion

1. Build ReconcileBench at the scope Gate 1 justifies — or open the upstream PRs, if F
2. Resolve the k8sbricks name / API-identity contradiction — **blocks the first public CRD**
3. Generator spike: evaluate forking `aws-controllers-k8s/code-generator` and replacing its
   AWS model front end with a `databricks-sdk-go` type walker. Prove one representative
   resource family end-to-end before committing to the architecture
4. Build **k8sbricks Compute (Cluster) only** — a real async state machine (PENDING →
   RUNNING → TERMINATING), a small API surface, and the `existing_cluster_id` admission
   behaviour ships in the first release
5. Run normal tests, then ReconcileBench. Fix what only ReconcileBench found
6. Publish results

**Exit:** ReconcileBench finds at least one meaningful issue that ordinary test coverage
missed, or produces compelling evidence the methodology adds information ordinary testing
does not.

---

## Milestone 1 — Conventions

Extract from implementation experience, not up front. Publish controller conventions, error
taxonomy, condition conventions, reference and security rules, initial conformance checks.

Do **not** freeze shared Go structs yet.

---

## Milestone 2 — External validation

Runs **while building**, not before it. F6 in particular is measured as a by-product of work
happening anyway.

- **F2 Trust** — run against 2–3 third-party operators under the disclosure policy. Ask
  maintainers *"what would have made you ignore this?"*, not *"was this useful?"*. One
  maintainer acting proves reproducible findings create action; it does not prove trust
  transfers. Use the trust ladder in `context/FALSIFICATION_EXECUTION.md`.
- **F3a Convention interest** — publish conventions as a plain document alongside first
  findings (one day of work). Watch for projects that take the scenario packs and ignore the
  conventions — that is the falsifying signal. **F3b**, a formal conformance suite, is built
  only if F3a shows demand.
- **F4 Coupling** — does the assurance work benefit from sharing an org with the operators,
  or only from the technical relationship? Watch for perceived conflict of interest. The
  five-engineer narrative test is supporting evidence only and may not determine the verdict.
- **F6 Economics** — track **Target Onboarding Cost**: engineer-hours to first reliable
  adversarial test, plus target-specific code, plus ongoing maintenance. Measure across
  fixture → k8sbricks → two third-party targets. Acto's push-button onboarding is the
  benchmark. If every target needs bespoke work, the generic-tool thesis fails and the
  answer is an SDK for operator authors.

Then write Project Plan v3. It must be **narrower** than v2.

---

## Operator Queue

Order and rationale. **One active at a time.**

### 1. k8sbricks — active after Gate 1

Staged: **Compute first**, Serving second, remaining service families only after the
generator architecture and control-loop conventions are stable. Compute is chosen over
Serving because it has a genuine async state machine with a far smaller spec.

### 2. kubereserve — candidate

Design settled. More defensible commercially than k8sbricks but harder to demo and harder
for a stranger to evaluate, which is why it is not first. Write the API/lifecycle RFC before
touching a cloud SDK.

### 3. Environment pooling — candidate ("the OpenEnv operator")

Design settled. Gated on verifying OpenEnv's current Kubernetes story — Modal and Prime
Intellect host environments commercially and may already own that path. Its multi-org RFC
governance also makes it a slow first upstream partner.

### 4. Workload runner — candidate ("the SmolAgent operator")

**There is no separate SmolAgent operator, and there will not be one.** smolagents,
AutoTrain, OpenCode, OpenClaw and similar are declarative `MLRuntime` CRs of this single
operator. Adding a framework is a YAML PR.

OpenClaw specifically is a poor wrap target for now: it ships multiple times per week and a
single release block in 2026 included six breaking changes. Revisit when it stabilises.

### Not queued

Forge, Radar, Catalog, Drift, Operating Knowledge Packs, the full ProjectOperationalProfile
schema, the ArchitectureDecision schema, the Python SDK, community portal, reputation,
certification, `controller-template`. All post-gate.

---

## Standing constraints during all of the above

1. No public CRDs or registered API groups until the k8sbricks name/API-identity question is
   resolved. The fixture's throwaway domain is the only exception.
2. Set the compute-budget ceiling before designing the scenario matrix. Every published
   result states its exact tested matrix and repetition count.
3. Disclosure policy written and published before F2 begins.
4. Governance, entry criteria, and archival policy published before the second project repo
   exists.
5. Do not make a single personal account the only long-term org owner once a trusted
   co-maintainer exists.

---

## Documents still to write

Tracked here rather than created as empty files, because governance theatre is worse than an
honest gap.

- ~~`LICENSE`~~ — Apache-2.0, added 2026-09-13

- `CONVENTIONS.md` — Milestone 1, extracted from implementation experience
- `GOVERNANCE.md`, `MAINTAINERS.md`, `ARCHIVAL.md` — before the second project repo
- `DISCLOSURE.md` — before F2
- `SECURITY.md` — with the first public repo that accepts reports
- RFC/ADR templates — before the first RFC
- Gate 1 decision record — output of Milestone 0
