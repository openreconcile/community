# OpenReconcile — Prior Art Register

**Purpose.** ReconcileBench's differentiation claim is a claim about what does not already
exist. That claim is only as good as this file. Every tool that plausibly overlaps belongs
here, with its maintenance status and its actual mechanism, so the claim can be checked
rather than repeated.

**Status of this register:** desk research and source-level F0 complete as of 2026-09-13.
Gate 1 is closed — see `GATE1.md`. operator-chaos, OpenSandbox, Karpenter ODCR support, and
the live Databricks Crossplane provider were added the same day. Execution of Sieve/Acto
against a live operator remains useful F6 telemetry; it is no longer load-bearing for
build-vs-contribute.

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

**Path correction, 2026-09-13.** The live incumbent is `glalanne/provider-databricks`
(Upbound marketplace id `lalanne/provider-databricks`), not `lalanne/provider-databricks`
or `upbound/provider-databricks` — those GitHub paths 404. An earlier note that the
Databricks operator space “may be vacant” was wrong.

- `Azure/azure-databricks-operator` — Microsoft, Kubebuilder + Go SDK, archived, last
  push 2021-06-04.
- `mach-kernel/databricks-kube-operator` — kube-rs, archived June 2025; README points
  users at Upjet-generated Crossplane providers.
- `glalanne/provider-databricks` — **the live incumbent.** Upjet / Crossplane, community
  provider, latest marketplace release **v2.5.0 (2026-08-31)**. Terraform-derived
  managed resources. Kubernetes-native reconciliation, OIDC-first auth, and encoded
  Databricks domain knowledge (`existing_cluster_id`, commit-pinned `git_source`) are
  still the k8sbricks gap — not “no Databricks provider exists”.
- `upbound/provider-azure-databricks` — official Upbound provider for *Azure Databricks
  workspace infrastructure*, not workspace-level compute/jobs/serving.
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

## 6. F0 findings — source-level analysis

**Date: 2026-09-13. Method: read the source of both tools at `HEAD`. Neither has been
executed. Treat everything here as provisional until it is.**

### Sieve's onboarding cost is far worse than "port the controller"

`build.py` reveals the real setup path: `download_kubernetes` → `instrument_kubernetes` →
`build_kubernetes` → `kind build node-image`. Sieve **downloads Kubernetes source, patches
it with its own instrumentation tool, and builds a custom kind node image.** Interposing at
the API server is achieved by shipping a modified Kubernetes.

Version evidence: `go.work` declares go 1.19, the three Go modules declare go 1.13,
`sieve_server/go.mod` pins `k8s.io/api v0.18.9`, `docs/demo.md` uses Kubernetes branch
`v1.18.9`, and `docs/port.md` gives `v1.23.1` as the example. Against current Kubernetes
(1.34/1.35) that is 11–16 minor versions of drift in a tool that compiles Kubernetes from
source, with no commit since September 2024.

Stale-state testing additionally requires a three-node control plane —
`config.json` sets `leading_api: kind-control-plane` and
`following_api: kind-control-plane3`.

**Assessment:** Sieve is very unlikely to run as-is, and reviving it means maintaining a
patched Kubernetes fork. That is not a cost this project can carry, and it is strong
evidence for *why* a rigorous approach was abandoned rather than adopted.

### Sieve has exactly three fault policies, and no external-boundary concept

`sieve_perturbation_policies/` contains precisely `intermediate_state.py`, `stale_state.py`,
and `unobserved_state.py`. Nothing else.

A search across the entire Sieve source and docs for external API, non-Kubernetes, cloud
API, HTTP proxy, or egress concepts returned **zero matches**. Sieve's model is the
controller's view of *cluster* state. It has no vocabulary for a fault at an external
service boundary.

**This confirms the strongest remaining novelty candidate in `PROJECT-CONTEXT.md` §2.**

### Both tools deliberately mask `observedGeneration` — and it is one of our defect classes

This is the most useful finding and it was not anticipated.

Sieve's `config.json` applies a global `field_path_mask` for `*/*/*` that includes
`["status", "observedGeneration"]` and `["status", "conditions"]`.

Acto's `EXCLUDE_PATH_REGEX` in `acto/common.py` includes `observed_generation`,
`observedGeneration`, `generation`, `resourceVersion`, and `managedFields`.

Both exclude these fields for the same reason this project already understands: they change
for non-semantic reasons and produce false positives. But the consequence is that **neither
tool can detect a controller that never updates `observedGeneration`, or one that reports
incorrect conditions.** `defect/missing-observed-generation` is invisible to both by
configuration, not by accident.

Separately, Acto's `EXCLUDE_ERROR_REGEX` suppresses log messages matching
`failed to sync(.)*status` — its log-based oracle is explicitly tuned to ignore status-sync
failures as noise. This does not prove its consistency oracle would miss a split write, but
it is a pointer at the same blind spot.

**Why this matters.** The project's semantic-convergence definition already distinguishes
non-semantic churn from semantic meaning, and its convention rules already treat
`observedGeneration` and condition correctness as load-bearing API surface. That is
precisely the distinction both tools collapsed in order to control false positives. A
convergence oracle that can tell "this field changed for a non-semantic reason" from "this
controller never reports which generation it observed" is a real capability neither has —
and it is cheap.

### Acto is the better-engineered starting point

Acto takes standard `kindest/node:{version}` images with a configurable version
(`acto/kubernetes_engine/kind.py`), needs only the operator's deployment script, is
Apache-2.0, and has commits into 2026. It has also grown beyond the paper:
`acto/runner/fault_injection_runner.py` and `acto/post_process/simple_crash_test.py` add
crash injection, though as a post-processing pass rather than a fault timed to a
reconciliation phase.

If Gate 1 returns outcome F (contribute upstream), Acto is the target, not Sieve.

### Provisional coverage map

Against the eight fixture defect branches. **Predicted from source, not measured.**

| Branch | Sieve | Acto |
|---|---|---|
| `defect/duplicate-create` | likely — intermediate-state | partial — crash test, not phase-timed |
| `defect/unobserved-event` | **yes** — unobserved-state | no |
| `defect/stale-cache` | yes, at the cost of a patched Kubernetes and 3 control planes | no |
| `defect/missing-observed-generation` | **no — masked by config** | **no — masked by EXCLUDE_PATH_REGEX** |
| `defect/split-write` | no external-boundary model | uncertain; log oracle suppresses status-sync errors |
| `defect/finalizer-deadlock` | partial | partial — deletion is checked |
| `defect/ownership-conflict` | partial, via intermediate-state | no |
| `defect/retry-storm` | no — no write-frequency oracle | no |

If this map survives execution, F1 returns something like: generic mid-reconcile fault
injection **REFRAMED** (Sieve did it first, better, and it died of maintenance cost);
external-boundary partial success **VALIDATED**; status-semantics oracles — `observedGeneration`,
conditions, write-frequency — **VALIDATED** and cheap.

### Remaining F0 work

1. Run Acto end-to-end against one public operator. Measure onboarding hours.
2. Attempt Sieve's build and record exactly where it fails. A precise failure is publishable
   evidence about maintenance cost.
3. Confirm from execution, not source, whether Acto's consistency oracle catches a split
   write.
4. Search once more for a maintained proxy-based Kubernetes API fault injector.
5. Decide whether to open a conversation with the Acto maintainers.
   **Superseded 2026-09-13:** complementary outreach is to operator-chaos, not Acto
   as the primary upstream. See `GATE1.md` and
   `process/OPERATOR_CHAOS_OUTREACH.md`.

---

## 7. operator-chaos — `opendatahub-io/operator-chaos`

Added 2026-09-13. This is the maintained industry engine that occupies the slot
ReconcileBench originally claimed.

| | |
|---|---|
| Licence | Apache-2.0 (confirm at HEAD before depending) |
| Language | Go 1.26+ |
| Home | https://opendatahub-io.github.io/operator-chaos/ |
| Article | Red Hat Developer, 2026-07-11 |

**What it does.** Tests that an operator restores its *managed resource graph* after
operator-semantic faults, not merely that pods restart. Knowledge models (`knowledge.yaml`)
declare what the operator owns. Verdicts are `Resilient` / `Degraded` / `Failed` /
`Inconclusive`.

**Four modes.** CLI experiments on a live cluster; SDK middleware (API-level faults);
`chaostransport` (zero-dependency transport interceptor, no controller-runtime required);
ActionInterceptor / fuzz against a fake client.

**Twenty injection types** across infrastructure (PodKill, NetworkPartition, …),
configuration (ConfigDrift, CRDMutation, LabelStomping, …), access control (RBACRevoke,
WebhookDisrupt), and lifecycle (FinalizerBlock, OwnerRefOrphan, SecretDeletion,
LeaderElectionDisrupt, …).

**What it does not do — the ReconcileBench remainder.**

- No status-contract oracle. Knowledge models describe the resource graph, not
  `observedGeneration` / condition semantics.
- Knowledge models are hand-written YAML. No derivation from source or CRDs.
- Verdicts are human-oriented enums, not machine-actionable fix hints for an agent loop.
- No MCP / agent interface.
- No first-class model of an external (non-Kubernetes) API boundary.

**Implication.** Adopt as an engine. Do not compete on injection types. Produce
`knowledge.yaml` from `reconcilebench derive` so we are a producer in their ecosystem.

---

## 8. Adjacent prior art found the same day

### OpenSandbox — environment pooling

[opensandbox-group/opensandbox](https://github.com/opensandbox-group/opensandbox)
(Alibaba lineage) already ships a Kubernetes operator with `Pool` and `BatchSandbox`
CRs: pre-warmed pod buffers, `PoolMin` / `PoolMax`, allocation and deallocation, batch
delivery for “high-throughput agentic-RL scenarios”, pause/resume via rootfs snapshots.
`PoolReconciler.scalePool` computes `desiredSchedulableCnt` against buffer counts.

The design call in `projects/OPENENV_ENVPOOL.md` — “the CR boundary is the pool, never
the episode” — is already implemented.

OpenEnv core (`meta-pytorch/OpenEnv`) lists `KubernetesProvider` as 🚧 planned, a
placeholder class. Technical committee includes Meta-PyTorch, Nvidia, Microsoft,
Hugging Face, Modal, Prime Intellect, and others.

**Implication.** Do not build an OpenReconcile environment-pool operator. Contribute
`KubernetesProvider` upstream.

### Karpenter capacity reservations

Karpenter consumes EC2 On-Demand Capacity Reservations (v1.3), Capacity Blocks for ML
(v1.6), and interruptible reservations (v1.10) via `ReservedCapacity` and
`capacityReservationSelectorTerms`. It does **not** create, expire, budget, or sweep
the reservations themselves.

**Implication.** KubeReserve’s remaining wedge is reservation *lifecycle* (create /
TTL / budget / sweeper / adoption / orphan recovery), one cloud at a time — not
“Kubernetes that uses reserved capacity”.

---


## 9. Name collision

An older, dormant "Open Reconcile" exists — Rebecca Lawler's project, cloned at
`OpenRefine/open-reconcile`, Java package `com.googlecode.openreconcile`. Dead since ~2012.
In the data world "reconciliation" means entity matching (OpenRefine's Reconciliation API).

Mitigation: never let "reconcile" stand alone in titles, descriptions, or `og:` tags —
always "Kubernetes controller reconciliation". Avoid the hyphenated form `open-reconcile`
entirely.

---

## 10. Open prior-art questions

F0's product decision is closed (`GATE1.md`). These remain as measurement questions
for fixture-era F1/F6, not as blockers.

1. How many of the eight fixture defects does operator-chaos detect with a derived
   `knowledge.yaml` and no custom oracle?
2. How many does Acto detect? chainsaw? envtest?
3. What is Target Onboarding Cost, in hours, for the fixture on each engine?
4. Does operator-chaos grow a status-contract check if we propose one upstream?
5. Confirm `operator-chaos` licence and Go-version floor at the commit we pin.
