# OpenReconcile — Project Context

**Read this first.** It is the accumulated background for the project: what it is, what has
been decided, what has been deliberately rejected, and what remains open. It exists so that
a new contributor or coding agent does not re-propose approaches that were already
evaluated and dismissed.

**Status:** Gate 1 closed 2026-09-13 (`context/GATE1.md`). Product shape is an
engine-agnostic oracle-and-agent layer. Implementation starts with the fixture.

**Provenance:** consolidated September 2026 from two independent write-ups of the same
planning history. The full argument chain is preserved verbatim in `../sources/`. Where this
file and a source document disagree, this file wins — see `README.md` for precedence.

---

## 1. What OpenReconcile Is

An open, vendor-neutral engineering community for making software operable on Kubernetes,
with AI/ML infrastructure as the initial focus.

**Core thesis:** generating infrastructure code is now cheap. Determining the correct
architecture, and proving the result behaves correctly under failure, is not.

Two tracks:

- **Track A — Assurance.** `ReconcileBench`: a scenario, status-contract oracle, and
  agent loop **above existing perturbation engines** (operator-chaos, envtest, chainsaw;
  Acto optional). It does **not** own a fault-injection engine. The claim is **defect
  discovery, not certification.** See `adr/ADR-0001-engine-agnostic-architecture.md`.
- **Track B — Reference implementations.** A small number of production Kubernetes control
  planes built to shared conventions, used as proving grounds for Track A. **One active at
  a time.**
- **Track C — MLE front door.** Portable Skills, then `openreconcile new`, then a hosted
  NL demo after k8sbricks exists. Hero user: an MLE productionising an existing ML tool.
  The slogan *Build a Kubernetes operator in natural language* is gated on the
  KubeReserve greenfield example being replayed by someone other than the author.
  k8sbricks is the AI/ML proof point.

**Positioning line:** *Capabilities are declared. Resilience is demonstrated.*

Do not use "Capability levels are declared, we measure them" — it implies formal Operator
Capability Level verification, which is not what this is.

### Assets owned

| Asset | Status |
|---|---|
| `openreconcile.org` | Registered — primary, points at community docs |
| `openreconcile.dev` | Registered — held, unused until Gate 1 |
| `github.com/openreconcile` | Registered, created 2026-09-12 |

Gate 1 kept the community + tool split: `.org` remains primary. `.dev` may host the
developer/docs surface and, later, the NL demo. Do not hard-code either into a public
API group.

---

## 2. The Technical Gap

**This section was materially wrong until September 2026 and has been rewritten. Read
`context/PRIOR_ART.md` before relying on any claim here.**

Industry operator testing covers the happy path:

- `operator-sdk scorecard` — functional tests from the operator bundle against a live cluster
- **chainsaw** — declarative before/after cluster-state assertions. The modern, actively
  maintained option; operator-sdk recommends it over kuttl, whose website is defunct
- **envtest** — controller testing against a real API server
- **Chaos Mesh / Litmus** — workload-level fault injection

None of *those* inject faults during reconciliation. But the earlier version of this
document generalised that into "nothing does", and that is false:

- **Sieve** (OSDI '22, `sieve-project/sieve`) does deterministically-timed mid-reconcile
  fault injection against unmodified controller logic, covering intermediate-state,
  unobserved-state, and stale-state bugs. **Dormant since September 2024.**
- **Acto** (`xlab-uiuc/acto`) does state-centric operator testing with consistency and
  differential oracles, has found 50–80+ real bugs across 11 popular operators, and needs
  only the operator's deployment script to onboard a target. **Actively maintained.**
- **Anvil** (`xlab-uiuc/anvil`) formally verifies controller liveness via Eventually Stable
  Reconciliation.

The failure classes that matter still live in the same place: stale informer cache, partial
object creation, duplicate ownership, finalizer deadlock, controller restart mid-loop,
external-create-succeeds-status-write-fails. What has changed is who else is already there.

**The narrower claim now under test:** the approach was proven by research and then
abandoned; nothing in the field combines Sieve's fault-timing precision with Acto's
onboarding cost, and nothing is packaged for a controller author's CI. The
external-API-boundary defect class — where the external system's state, not the cluster's,
is what gets corrupted — is the strongest remaining candidate for genuine novelty, because
both research tools model the controller's view of cluster state.

Separately, **Operator Capability Levels I–V** exist as a maturity taxonomy. OperatorHub
renders them from what the operator *claims* in its ClusterServiceVersion. Nothing verifies
the claim. That part of the positioning is unaffected.

**Gate 1 closed this claim.** Generic mid-reconcile injection is REFRAMED (Sieve did it;
operator-chaos does it). The unclaimed remainder is the status-contract oracle,
source-derived target models, and machine-actionable evidence. Do not write docs or
comments that reassert “we inject faults during reconciliation” as a novelty claim.

A fourth tool, **operator-chaos** (`opendatahub-io/operator-chaos`), is the maintained
engine we adopt. See `context/PRIOR_ART.md` §7.

---

## 3. Decisions Locked

These are settled. Do not reopen without new evidence.

1. ReconcileBench is the first technical wedge.
2. The initial claim is defect discovery, not certification.
3. Testing targets failures *during* reconciliation, not generic chaos.
4. **Operator-or-Not** — never assume a project needs an operator. "Do not build an
   operator" is a valid, desirable outcome.
5. **Upstream-first** — prefer contributing to the correct upstream over owning a fork.
   Applies to assurance tooling: adopt operator-chaos as an engine; contribute
   complementary layers (oracle, derive, evidence) rather than a fifth injector.
   OpenEnv work is an upstream `KubernetesProvider`, not a new org repo.
6. **Reuse existing control planes** — do not recreate Kubernetes, KServe, KubeRay,
   Kubeflow, Crossplane, or mature upstream operators.
7. Adversarial scenario generation must be independent of implementation generation.
   If the same agent writes the controller and the tests validating it, the proof
   correlates with the artefact it proves.
8. Human maintainers retain architecture and security authority.
9. All published evidence carries target version, Kubernetes version, execution context,
   and an as-of date.
10. Responsible disclosure before publishing findings against third-party code.
11. **Donatability** — every project must be transferable to another org or foundation
    without an API migration for its users. This drives API group policy.
12. **Go + controller-runtime/Kubebuilder** for all project-owned production controllers.
    This constrains controller runtimes, not test harnesses.
13. **Semantic convergence, not literal write quiescence** — see §7.
14. The fixture carries both a clean control and seeded-defect variants. Specificity is
    measured, not assumed.

---

## 4. Approaches Explicitly Rejected

Listed with reasoning, because each looks reasonable and will otherwise be re-proposed.

### Kopf / Python operators
Rejected. Splits the runtime across the org while claiming unified conventions; loses
controller-gen (forcing hand-written CRDs); loses typed clients, conversion webhooks, and
controller-runtime's workqueue and rate-limiting semantics. The stated justification —
"you can import ML libraries into the control loop" — is an anti-pattern: the operator's
dependency tree becomes the union of every framework it supports. Python is for client SDKs
and tooling only. **This does not extend to test harnesses** — Sieve and Acto are Python and
must be evaluated on merit.

### Org name in the API group (`dbx.openreconcile.io`)
Rejected. Embeds the org identity in every installed API, making donation or transfer a
full API migration for every user. Violates decision 11. Note that Plan v2's proposed
`openreconcile.io/apis/meta/v1alpha1` meta module violated this rule too, which is a second
reason it is dead.

### PAT-in-a-Secret authentication
Rejected as the default path. Every prior Databricks operator shipped PAT-first and every
one is dead. OIDC federation / workload identity from v1.

### Inlining source code in CR spec fields
Rejected. etcd object size limits, YAML escaping, loss of IDE tooling. Reference git with a
pinned commit instead. ConfigMap script injection is a footnote for tiny scripts, not the
headline pattern.

### Upjet / Terraform-derived generation
Rejected for k8sbricks. Inherits Terraform's runtime, state, and failure modes.
`glalanne/provider-databricks` is the live Crossplane incumbent (v2.5.0, 2026-08-31).
k8sbricks exists only if Kubernetes-native reconciliation, OIDC-first auth, and encoded
Databricks domain knowledge are real differentiators — not because the space is empty.

### Bare Pods from a controller
Rejected. Bare pods do not reschedule or restart. For a training job that is data loss.

### Shared CRDs across projects
Rejected. A shared CRD means a shared install unit and release train, breaking donatability.
Each project defines its own connection CRD with an identical *shape*.

### A frozen shared meta Go module
Rejected **for now**, not permanently. Plan v2 specified
`openreconcile.io/apis/meta/v1alpha1` with `ManagedSpec` / `ManagedStatus` as additive-only
forever. Superseded: conventions lock early because retrofitting them costs users a
migration, but shared Go structs are extracted only after two production controllers prove
which structures are genuinely identical.

### Custom condition types
Rejected. Use `metav1.Condition`. ACK carries a custom type only because it predates general
adoption, and is stuck with it. The standard type gives `kubectl wait --for=condition=Ready`
and ecosystem tooling for free.

### Literal write quiescence as the convergence definition
Rejected. Correct controllers legitimately write after desired state is reached —
`lastSyncTime`, heartbeats, lease renewal, token refresh, condition timestamps, periodic
resync metadata. See §7.

### Five separate framework operators (smolagents, AutoTrain, OpenCode, …)
Rejected. These are one operator with declarative runtime providers. See §6.

### Publishing to ArtifactHub / OperatorHub first
Deferred, **reversibly**. ArtifactHub is an index, not a channel — charts live in our own
OCI registry either way. OperatorHub requires ongoing OLM bundle maintenance for OpenShift
console install only. Plan v2's "No ArtifactHub. Decided." was stronger than its reasoning;
both may be revisited when discoverability outweighs maintenance cost.

### Building a registry/storefront site early
Rejected. A catalog with two entries advertises smallness.

### An SDK that applies to the cluster from a developer laptop
Rejected. Requires ML engineers to hold cluster credentials, which no platform team grants,
and ArgoCD reverts anything applied imperatively. Any future SDK emits to git or posts to a
thin authenticated service.

---

## 5. Prior Art

Moved to **`context/PRIOR_ART.md`** and substantially expanded. That file is the single
place prior art is recorded; do not re-summarise it here, because a second copy will drift
and the differentiation claim depends on it being right.

---

## 6. Track B — Project Designs

Designs are settled; **build order is in `ROADMAP.md`.** Only one is active at a time.
Per-project detail lives in `projects/`.

### k8sbricks — Databricks control plane (first active)

ACK-architecture, generated from `databricks-sdk-go`'s `service/*` packages.

- Per-service controllers and charts. CRD count is an operational problem: 200+ CRDs bloat
  discovery documents, slow kubectl cluster-wide, and inflate etcd. Upbound split into
  provider families for exactly this reason.
- API groups mirror SDK service package names: `serving.`, `compute.`, `jobs.`,
  `pipelines.`, `catalog.`, `sql.`, `iam.`, `core.`
- Nested `apis/` Go module so consumers don't pull controller-runtime transitively.
- Late initialization mandatory — Databricks defaults server-side extensively; without it
  controllers hot-loop on spurious drift.
- Adoption of pre-existing resources from v1. Nobody starts greenfield on Databricks.
- OIDC federation, no PAT.
- **Staged: Compute/Cluster first, Serving second.** Compute is chosen over Serving because
  it has a genuine async state machine (`PENDING` → `RUNNING` → `RESTARTING` →
  `TERMINATING` → `TERMINATED` → failure states) with a far smaller spec.

**Encoded domain knowledge — a differentiator generated providers cannot match:**
- `existing_cluster_id` silently drops env vars, image, and Spark conf. Gate behind an
  explicit acknowledgement field or emit an admission warning.
- `git_source` checkout happens on the control plane, not the cluster, and requires commit
  pinning (not branch) for reproducibility. Make branch-tracking the escape hatch.

**Unresolved contradiction — blocks first public CRD.** The name is held provisional, but
API groups embed it permanently. Either make `k8sbricks` the final API identity, or choose
a stable API identity independent of the cosmetic project name. There is no safe third
option where the name stays provisional and the API group is treated as permanent. Do not
ship a provisional brand in a permanent API group.

**Trademark position: recorded and accepted, not zero-risk.** Infringement turns on
likelihood of confusion between marks on related goods, not on whether "bricks" is
separately registrable. Mitigations: no Databricks logo, colours, or wordmark; prominent
non-affiliation notice in README and docs; no company registration under the name. The
Kubernetes wheel is a Linux Foundation mark with usage guidelines constraining derivative
logos.

**k8sbricks is production software, not a disposable test fixture.** Real users running it
is what makes it a valuable ReconcileBench subject. The fixture is the artificial subject.

### kubereserve — cloud capacity (candidate)

Core plus provider binaries in separate repos (Karpenter model), because credentials and
release cadence differ per vendor. Locked decisions: no Karpenter dependency (must work with
default kube-scheduler); `dryRun: true` as install default; mandatory TTL on every claim;
budget ceiling enforced at admission; independent sweeper. Demand from a separate
`CapacityAutoReserver` doing arithmetic aggregation of pending GPU pods, not scheduling
simulation. Multi-cluster is v2, made additive by six v1 choices (UID-based tagging,
kube-system UID as cluster identity, split binaries, tag-as-source-of-truth, cluster label on
metrics, stay on v1alpha1). Start with one provider.

### Environment pooling — REFRAMED to upstream OpenEnv (2026-09-13)

Do **not** create an OpenReconcile `EnvironmentPool` operator. Alibaba’s OpenSandbox
already ships `Pool` / `BatchSandbox` with pre-warmed buffers and batch RL delivery.
OpenEnv core lists `KubernetesProvider` as an unimplemented placeholder. Track B work
here is an **upstream contribution** to `meta-pytorch/OpenEnv`. See
`projects/OPENENV_ENVPOOL.md`.

### Workload runner — "the SmolAgent operator" (candidate)

**There is no SmolAgent operator.** smolagents, AutoTrain, OpenCode and similar are
*declarative runtime providers* of one operator, not separate controllers.

`MLRuntime` (cluster-scoped, community-contributable — base image, entrypoint contract,
dependency strategy, config schema) + `MLWorkload` (namespaced, user-authored). Adding a
framework is a YAML PR, not a Go controller. This is the contributor-leverage mechanism for
a small maintainer team.

Zero-Docker path: curated base images, git-sync with commit pinning, PVC-cached dependency
layers. Production requires a bake-to-image promotion step; `pip install` at pod start is a
development path only and must be blocked by policy in production namespaces.

---

## 7. ReconcileBench Core Semantics

### Semantic convergence

A target has converged when, within the scenario's bounded convergence window after fault
resolution:

- all declared invariants hold;
- observed/status state represents the current desired generation;
- no unresolved actionable error remains;
- subsequent reconciliation produces no semantically meaningful desired-state mutation.

Per-target ignore lists may exclude non-semantic fields — `metadata.resourceVersion`,
`metadata.managedFields`, observation timestamps, heartbeats, lease renewals, known
controller-maintained sync timestamps. **The ignore list is part of the target definition
and getting it wrong in either direction is a tool defect:** ignoring too much hides real
failures, ignoring too little manufactures false positives.

Define this against the Anvil/ESR literature rather than from scratch — Eventually Stable
Reconciliation is a more rigorous formulation of the same property.

### Invariant classes

Vocabulary for the target file, **not yet a formal specification**:

- **resource** — resources that must exist, with correct ownership and configuration
- **state** — observed/status state accurately represents desired and actual state
- **safety** — states that must never occur (duplicate external resources; an orphaned
  cloud resource contrary to policy; two controllers simultaneously owning one exclusive
  resource). Highest-value class, because before/after testing rarely exercises it
- **progress / liveness** — states that must eventually advance
- **idempotency** — repeated reconciliation must not duplicate or corrupt state
- **recovery** — after the injected fault is removed, the system returns to an acceptable state
- **deletion** — finalization and deletion-policy behaviour must terminate correctly

Safety and liveness map to the classic distributed-systems distinction. Use it.

### Modes

- **Black-box** — no controller modification. **The only mode that ships initially.**
- **Grey-box** — controller exposes reconciliation checkpoints or richer traces.
- **Instrumented** — controller integrates against a ReconcileBench SDK.

The architecture may anticipate all three without implementing the latter two in
Milestone 0. Note that Sieve's instrumentation of `client-go`/`controller-runtime` sits
between black-box and grey-box, and F0 should establish whether that is a better boundary.

### Fault timing

Random injection under load gives only probabilistic coverage of windows like "status
persistence begins". The higher-leverage middle path is **API-server-boundary
interception**: a proxy between controller and API server that can allow a child create to
succeed then fail the following status update, delay a specific `GET`, return stale data,
inject `409 Conflict`, inject throttling, terminate the controller after a specific call, or
delay finalizer writes.

This was assessed as possibly the highest-leverage architectural decision in ReconcileBench.
**F0 must establish whether it already exists** — Sieve interposes at the API server for
stale-state testing, so partial prior art is confirmed.

### Bench target file

A per-target declaration of what correct means: lifecycle transitions, reconciliation
invariants, failure model, expected conditions per phase, ignored non-semantic fields.

**Deliberately not a versioned specification.** It is a tool input file, changeable freely
as scenarios teach us what it needs. Promoting it to a spec creates a maintenance obligation
before the shape is known. It is the surviving subset of v1's `ProjectOperationalProfile`.

### Evidence

Every run records target and version, Kubernetes version, scenario, exact fault, invariants,
repetitions, result, logs, reproducer, and an as-of date with validity basis. Never imply
general resilience from a narrow matrix.

---

## 8. Cross-Project API Conventions

Full text belongs in `CONVENTIONS.md` (not yet written). Summary:

- **No shared CRDs.** Shared shape, separate definitions.
- **Conventions lock now; shared Go structs are extracted later**, after two production
  controllers prove which structures are genuinely identical. Do not create a frozen
  `apis/meta` module yet.
- **Lock now:** API group naming policy; `metav1.Condition`; same-namespace references by
  default for credentials; controllers never write to `spec`; conversion webhook written and
  tested before version promotion; explicit terminal vs recoverable error handling;
  donatability rules.
- **Learn before extracting:** `ManagedSpec` / `ManagedStatus`, connection reference shapes,
  common policy structs.
- API groups: `<subcomponent>.<project>.io`. No org name. No ACK-style infix — we own the
  domains outright.
- Cross-project references are typed and resolved unstructured via discovery. Zero
  compile-time coupling; a missing project yields `Ready=False` with a recoverable reason.
- Controllers never write to `spec` — only `status`. Defaulting goes in the admission
  webhook. Violating this gives every ArgoCD user an infinite sync loop.
- Group version tracks the weakest resource in the group; a per-resource maturity matrix
  carries the real signal.

---

## 9. Open Questions

Closed at Gate 1 unless marked open. See `context/GATE1.md`.

1. **Is ReconcileBench a new injection category?** **No.** Closed. Engine-agnostic oracle
   layer (`ADR-0001`).
2. **Build vs contribute?** **Both, split.** Own the oracle/derive/evidence/agent layers;
   adopt operator-chaos as an engine; OpenEnv is upstream-only.
3. **Will anyone trust findings from an unknown org?** **Open — F2.**
4. **Does anyone want the conformance layer, or only the test packs?** **Open — F3.**
5. **Do the assurance work and the operators belong in one org?** **Resolved by
   donatability**, not by a split. Watch conflict-of-interest; do not delay the oracle.
6. **Is the AI/ML framing technical or go-to-market?** **Closed.** F5a general tool,
   F5b AI/ML first market and MLE hero user.
7. **Does target onboarding cost scale?** **Provisionally yes if we adopt engines.**
   Still measure F6 on fixture → k8sbricks → two third-party targets.
8. **k8sbricks name vs API identity.** Still blocks the first public CRD.
9. **Compute budget ceiling.** Undecided. Set before the scenario matrix is published.
10. **Does KubeReserve survive Karpenter ODCR support?** Lifecycle (create/TTL/budget/
    sweeper) is the remaining wedge. Confirm in the API RFC before cloud-SDK work.

---

## 10. Working Rules

- **A falsification result that narrows the project is a success.** Plan v3 must be
  materially narrower than v2. If every workstream returns VALIDATED, the questions were
  asked to be passed.
- **Nothing enters the org without a named maintainer and a working e2e test.** Nothing
  leaves alpha without a second contributor. At most two projects in alpha concurrently.
  Anything untouched for two release cycles gets an explicit archive notice.
- **Every published result states its exact tested matrix and repetition count.** Never
  imply general resilience from a narrow matrix.
- **Reviewer acquisition is an output problem, not a recruitment problem.** A maintainer who
  receives a credible bug report in their own project is a reviewer with something at stake.
  Do not build a reviewer panel; produce findings and let respondents self-select.
- **The repository is the memory.** When a decision is made, update `context/DECISIONS.md`
  and the relevant document in the same commit. Chat history is not durable project memory.
- **Claims must match evidence.** "Finds reconciliation defects that ordinary tests miss" is
  supportable once demonstrated. "Verified", "certified", "production-ready" are not, and
  "resilient" is a property to demonstrate, never to assert.
- **No falsification workstream may require an artefact that would not otherwise be useful
  to the surviving product.** Falsification must not become a procrastination mechanism.
