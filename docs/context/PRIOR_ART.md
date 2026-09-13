# OpenReconcile — Prior Art Register

**Purpose.** ReconcileBench's differentiation claim is a claim about what does not already
exist. That claim is only as good as this file. Every tool that plausibly overlaps belongs
here, with its maintenance status and its actual mechanism, so the claim can be checked
rather than repeated.

**Status of this register:** desk research complete as of September 2026. **Nothing in the
research-tool section below has been executed yet.** Running it is workstream F0.

---

## 1. The correction that produced this file

Project Plan v2 §2 and the original `PROJECT-CONTEXT.md` §2 both stated the gap as:

> Existing operator testing covers the happy path. None of these inject faults *during*
> reconciliation.

The prior-art list backing that sentence contained only `operator-sdk scorecard`, chainsaw,
kuttl, envtest, Chaos Mesh, and Litmus — all industry tooling. It omitted the academic
Kubernetes-controller-testing literature, which is precisely where mid-reconcile fault
injection was already solved.

**The sentence as written is false.** Sieve (OSDI '22) injects deterministically-timed
faults during reconciliation against unmodified controller logic, and its three bug patterns
map almost one-to-one onto three of the seven defect branches planned for the fixture.

This does not end the project, but it changes the question. See §5.

---

## 2. Research tools — the real comparison set

### Sieve — `sieve-project/sieve`

Automatic reliability testing for Kubernetes controllers. OSDI '22, Sun et al., UIUC.

| | |
|---|---|
| Licence | BSD-2-Clause |
| Language | Python |
| Stars / forks | 348 / 21 |
| Last commit | **2024-09-26** |
| Archived | No, but ~2 years without a commit |
| Open issues | 11 |

**Mechanism.** Two stages. In the *learning* stage Sieve runs an end-to-end test case
against an instrumented controller and analyses the resulting event sequence to identify
promising fault-injection points. That produces *test plans* — a triggering condition
expressed as a cluster-state transition, plus a fault to inject when it matches. In the
*testing* stage a coordinator runs the workload, watches every state transition on both the
controller's view and the API servers' view, injects the fault the moment the condition
matches, and compares the resulting state trace against the reference run. Divergence is a
bug report.

Instrumentation intercepts `client-go` and `controller-runtime` event handlers. Reconciler
logic is not modified, but **the controller must be "ported"** — built against Sieve's
instrumented dependencies and supplied with e2e test cases.

**Fault mechanisms.** Crash/restart the controller; disconnect/reconnect it from an API
server; block/unblock it from processing events; block/unblock an API server from processing
events.

**Bug patterns, and how they map to our planned defect branches:**

| Sieve pattern | Mechanism | Our branch |
|---|---|---|
| Intermediate-state | Restarts the controller mid-reconcile so it resumes on partially-updated cluster state | `defect/duplicate-create`, `defect/split-write` |
| Unobserved-state | Manipulates informer/reconciler goroutine interleaving so the controller misses events; a level-triggered controller must still converge | no direct equivalent — **we are missing this class** |
| Stale-state | Redirects the controller to a deliberately lagged API server in an HA cluster | `defect/stale-cache` |

**What it does not appear to cover.** Failures at an *external* (non-Kubernetes) API
boundary — the "external create succeeds, status write fails, external ID is lost" class.
Sieve's model is the controller's view of *cluster* state. This is the most important thing
to confirm in F0, because it is the class our design treats as highest-value.

**Constraints.** Stale-state testing requires multiple API servers. Porting cost is real
and is exactly the quantity F6 calls Target Onboarding Cost.

### Acto — `xlab-uiuc/acto`

Push-button end-to-end testing for operation correctness of Kubernetes operators. UIUC.

| | |
|---|---|
| Licence | Apache-2.0 |
| Language | Python |
| Stars / forks | 135 / 47 |
| Last commit | **2026-02-22** (repo pushed 2026-08-13) |
| Open issues | 35 |

**Mechanism.** State-centric rather than fault-centric. Acto mutates CR spec fields to
generate state transitions, applies each declaratively, waits for reconciliation, snapshots
system state, and runs a bank of oracles over the snapshot. It checks three properties:
always reconciles to the desired state; always recovers from undesired or error states;
always resilient to misoperations.

**Oracles.** `crash` (container crashed), `health` (ready vs desired replicas on
Deployments/StatefulSets), `consistency` (desired system state matches actual),
`operator_log` (log indicates invalid input), `differential` (recovery succeeds after an
error state), plus user-defined `custom` oracles.

**Onboarding cost.** Input is the operator's deployment script. Nothing else. Runs on kind,
minikube, or k3d. **This is a lower onboarding cost than anything currently planned for
ReconcileBench**, and it is the strongest single competitive fact in this register.

**Track record.** Repo claims 50+ new bugs across 11 popular operators, 28 fixed; the
USENIX write-up claims more than 80. Either way, substantially more real-world validation
than an unknown org starts with.

**Known weakness — and it is our weakness too.** Acto's own porting documentation walks
through a false alarm caused by a control-flow dependency between two spec fields: the
operator correctly ignored a field because a sibling field was not set, and Acto flagged the
resulting inconsistency as a bug. This is direct evidence that the specificity problem is
real and that the clean-control branch is the right instrument for it.

**What it does not do.** Choose *when* during a reconcile to inject a fault. There is no
mid-reconcile crash at a chosen point, no partial-write window, no deliberate conflict or
throttle injection.

### Anvil — `xlab-uiuc/anvil`

Formal verification rather than testing. Controllers are written in Rust against `kube-rs`
as explicit state machines, correctness properties are specified in a formal language, and
Verus produces machine-checkable proofs. Its headline property is **Eventually Stable
Reconciliation** — the controller eventually drives the system to the desired state and
stays there despite failures.

Last commit 2024-10-24. Effectively a research artefact.

**Why it matters to us.** Not a competitor — a Rust-only rewrite is not a path our audience
will take. But ESR is a far more rigorous formulation of the same idea as our "semantic
convergence", and the convergence oracle should be defined with the ESR literature in hand
rather than invented from scratch.

### Language note

Sieve and Acto are both Python. The project rule against Python is about *controller
runtimes*, not test harnesses. Do not reject or ignore either tool on language grounds.

---

## 3. Industry tooling

- **`operator-sdk scorecard`** — functional tests driven from the operator bundle against a
  live cluster. Happy path.
- **chainsaw** — declarative before/after cluster-state assertions. The actively maintained
  option; operator-sdk recommends it over kuttl. No fault injection of its own.
- **kuttl** — legacy. Website defunct, docs redirect to the GitHub repo. Migration path to
  chainsaw exists. Do not build on it.
- **envtest** — controller-level testing against a real API server. No fault injection.
- **Chaos Mesh / Litmus** — workload-level fault primitives: pod kill, node drain, network
  fault. Infrastructure-level, not reconcile-phase-aware. Use these rather than
  reimplementing pod kill.
- **Operator Capability Levels I–V** — a maturity taxonomy. OperatorHub renders what the
  operator claims in its ClusterServiceVersion. Nothing verifies the claim. This part of the
  positioning survives intact.

---

## 4. Databricks prior art (for k8sbricks)

- `Azure/azure-databricks-operator` — Microsoft, Kubebuilder + Go SDK, experimental, dormant
  for years.
- `mach-kernel/databricks-kube-operator` — kube-rs, archived June 2025; README now points
  users at Upjet-generated Crossplane providers.
- `upbound/provider-databricks` — Upjet, 69 managed resources, stalled at v0.1.x.
- `lalanne/provider-databricks` — Upjet, v2.x, 209 managed resources. **The live incumbent.**
- Databricks DABs — renamed in 2026 from "Databricks Asset Bundles" to "Declarative
  Automation Bundles"; `bundle plan -t prod -o json` supports a GitOps apply with an
  approval gate.

**Both the Databricks Go SDK and the Databricks Terraform provider are generated from
Databricks' internal OpenAPI spec, which is not published.** SDK releases reference an
internal OpenAPI SHA. This is why generator input must be the SDK's public generated Go
types in `service/*` rather than a spec document.

### Reference architecture

**AWS Controllers for Kubernetes (ACK)** remains the model: per-service controllers and
charts, `generator.yaml` for field overrides and hooks, late initialization, adoption of
pre-existing resources, cross-account credential mapping by namespace, field export to
ConfigMap/Secret. `aws-controllers-k8s/code-generator` is Apache-2.0 and its output is
already used by both ACK and Crossplane, so the emitter may be retargetable — evaluate
forking it and replacing the AWS model front end before writing a generator from scratch.

---

## 5. What this means for the thesis

The original claim was *"nothing injects faults during reconciliation."* That is wrong.

The defensible claim is narrower and needs F0 to confirm it:

> The approach was validated by peer-reviewed research and then abandoned. Sieve proved
> deterministic mid-reconcile fault injection finds real bugs, and has been dormant since
> September 2024. Acto is alive and has the best onboarding cost in the field, but does not
> control fault timing. Neither is packaged for routine use in a controller author's CI.

That points at Gate 1 outcome **C or D** — a scenario library and convergence oracle, or a
general-purpose tool with AI/ML as first market — reached on far better evidence than a
chainsaw-plus-Chaos-Mesh bake-off would have produced.

Three consequences to carry forward:

1. **F1 must run against Sieve and Acto**, not only the industry tools. A verdict that
   ignores the two tools closest to the thesis is worthless. See `FALSIFICATION_EXECUTION.md`.
2. **Ergonomics and maintenance are now candidate differentiators, and novelty is not.**
   Acto's push-button onboarding is the bar. If ReconcileBench needs more per-target work
   than Acto, F6 is in trouble before it starts.
3. **The external-boundary defect class is the strongest remaining candidate for genuine
   novelty**, because both research tools model the controller's view of *cluster* state
   rather than an external service's state. Confirm this in F0 before relying on it.

Also note: Sieve tests an **unobserved-state** class — a controller missing an event and
failing to converge because it is not truly level-triggered — that has no equivalent in the
seven planned defect branches. That is a gap in our fixture, not in Sieve.

---

## 6. Name collision

An older, dormant "Open Reconcile" exists — Rebecca Lawler's project, cloned at
`OpenRefine/open-reconcile`, Java package `com.googlecode.openreconcile`. Dead since ~2012.
In the data world "reconciliation" means entity matching (OpenRefine's Reconciliation API).

Mitigation: never let "reconcile" stand alone in titles, descriptions, or `og:` tags —
always "Kubernetes controller reconciliation". Avoid the hyphenated form `open-reconcile`
entirely.

---

## 7. Open prior-art questions for F0

1. Does Sieve still build and run against current Kubernetes (1.34/1.35) and current
   `controller-runtime`, or has it bit-rotted since September 2024?
2. What is the real porting cost for Sieve, in hours, on our own fixture?
3. Can Sieve express a fault at an **external** API boundary at all?
4. How many of our seven seeded defects does Sieve detect? Acto? Either with custom oracles?
5. Does Acto's `custom` oracle hook plus its `differential` oracle already amount to a
   convergence oracle?
6. Does any maintained proxy-based Kubernetes API fault injector exist that we have still
   not found? The Addendum's API-server-boundary interception idea assumed none.
7. Are the authors of either project reachable and interested? An upstream contribution to
   Acto, or adopting Sieve, may be a better first move than a new tool — the project's own
   upstream-first rule points that way.
