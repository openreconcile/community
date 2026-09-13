# OpenReconcile — Roadmap and Next Steps

Sequenced work. Read `PROJECT-CONTEXT.md` first for why the order is what it is.

---

## Repositories to create now

| Repo | Contents |
|---|---|
| `openreconcile/.github` | Org profile README, issue templates, code of conduct |
| `openreconcile/community` | `CONVENTIONS.md`, `GOVERNANCE.md`, `ARCHIVAL.md`, `DISCLOSURE.md` |
| `openreconcile/reconcilebench-fixture` | Test fixture controller with seeded defects |
| `openreconcile/reconcilebench` | The tool — created after Gate 1 |

Four repos. Three have real content on day one. **`template` comes later** — a scaffold
template written before building two things will encode guesses.

Point `openreconcile.org` at the community repo's docs. Leave `.dev` unused until Gate 1.

---

## Milestone 0A — Fixture and Falsification

The immediate work. Roughly two weeks.

### Step 1 — Fixture controller

```bash
mkdir reconcilebench-fixture && cd $_
kubebuilder init --domain fixture.openreconcile.org \
  --repo github.com/openreconcile/reconcilebench-fixture
kubebuilder create api --group testing --version v1alpha1 --kind Widget \
  --resource --controller
```

`fixture.openreconcile.org` is a throwaway domain on infrastructure we own. This is the one
place a `kubebuilder init` is currently unblocked.

**`Widget` must exercise, at minimum:**

- a child Deployment (in-cluster resource with owner reference)
- a child ConfigMap (second child, to expose partial-creation windows)
- a simulated **asynchronous external resource** — a fake remote service with create/get/
  delete and a provisioning delay, so external-boundary failures are reachable
- status conditions and `observedGeneration`
- a finalizer with `Delete` / `Orphan` policy

The external simulator is the important part. Without it, the highest-value defect classes
cannot be expressed.

### Step 2 — Branches

`main` is the **clean control branch** and must be genuinely correct. It measures
false-positive rate. A tool that reports failures on a correct controller is worse than
useless, and specificity decides whether anyone trusts the output.

Then seven defect branches, one deliberate bug each, documented in the branch README with
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

**Write the defect branches by hand.** A bug generated from a description of the bug tends
to be the textbook version. The value is in defects that look like plausible mistakes and
would pass review. Let an agent write the surrounding boilerplate; write the bugs yourself.

### Step 3 — F1 prior-art spike (one week, timeboxed)

Attempt to detect all seven using **only existing tools**: chainsaw, Chaos Mesh or Litmus,
envtest, `operator-sdk scorecard`, and shell orchestration.

Record per defect: **detected / not detected / detected only with substantial
target-specific code.** That third column is the interesting one.

Score on two dimensions, not one:

- **coverage quantity** — how many detected without substantial custom code
- **defect-class significance** — whether what remains undetected is a distinct,
  operationally important class

Numerical thresholds are guidance, not a scoring function. Six of seven detected still
supports a narrower VALIDATED if the seventh is partial-success-across-an-external-boundary,
which is the hardest and most common real-world class.

Report sub-claims separately. Do not average to one verdict.

**Secondary question:** does proxy-based API-server boundary interception already exist? A
proxy between controller and apiserver can observe, delay, and fail specific requests,
making several fault windows deterministic with no controller changes — most of grey-box
precision without controller cooperation. If it exists, use it. If not, that absence is
itself evidence for the gap.

### Step 4 — F5a/F5b scope challenge (two days, parallel)

**F5a:** would any scenario, invariant class, or convergence semantic change if the targets
were database, networking, or storage operators? If not, the tool is general-purpose.

**F5b:** is AI/ML still the best *first market* even if the tool is general? Consider that
more mature Kubernetes domains have more complex controllers and stronger engineering teams.

Likely and healthy outcome: **F5a REFRAMED, F5b VALIDATED** — a general-purpose Kubernetes
controller resilience tool, launched into AI/ML infrastructure first.

### Gate 1

Answer explicitly, in writing:

- **A.** A genuinely distinct testing category
- **B.** A narrower tool targeting a few hard failure classes
- **C.** A scenario library and convergence oracle over existing tools
- **D.** A general controller-testing tool with AI/ML as first market
- **E.** Nothing sufficiently differentiated remains

C and D are the most likely. Do not proceed until one is chosen.

---

## Milestone 0B — Build to the Gate 1 conclusion

1. Build ReconcileBench at the scope Gate 1 justifies
2. Resolve the k8sbricks name / API-identity contradiction — **blocks the first public CRD**
3. Generator spike: evaluate forking `aws-controllers-k8s/code-generator` and replacing its
   AWS model front end with a `databricks-sdk-go` type walker
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
  transfers.
- **F3 Conformance** — publish conventions as a plain document alongside first findings
  (one day of work). Build the suite only if interest appears. Watch for projects that take
  the scenario packs and ignore the conventions — that is the falsifying signal.
- **F4 Coupling** — does the assurance work benefit from sharing an org with the operators,
  or only from the technical relationship? Watch for perceived conflict of interest.
- **F6 Economics** — track **Target Onboarding Cost**: engineer-hours to first reliable
  adversarial test, plus target-specific code, plus ongoing maintenance. Measure across
  fixture → k8sbricks → two third-party targets. If every target needs bespoke work, the
  generic-tool thesis fails and the answer is an SDK for operator authors.

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
for a stranger to evaluate, which is why it is not first.

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
certification. All post-gate.

---

## Standing constraints during all of the above

1. No public CRDs or registered API groups until the k8sbricks name/API-identity question is
   resolved.
2. Set the compute-budget ceiling before designing the scenario matrix. Every published
   result states its exact tested matrix and repetition count.
3. Disclosure policy written and published before F2 begins.
4. Governance, entry criteria, and archival policy published before the second project repo
   exists.
