# OpenReconcile — Project Context

**Read this first.** It is the accumulated background for the project: what it is, what has
been decided, what has been deliberately rejected, and what remains open. It exists so that
a new contributor or coding agent does not re-propose approaches that were already
evaluated and dismissed.

**Status:** Pre-implementation. Nothing is built. The project is before its first decision gate.

---

## 1. What OpenReconcile Is

An open, vendor-neutral engineering community for making software operable on Kubernetes,
with AI/ML infrastructure as the initial focus.

**Core thesis:** generating infrastructure code is now cheap. Determining the correct
architecture, and proving the result behaves correctly under failure, is not.

Two tracks:

- **Track A — Assurance.** `ReconcileBench`: a tool that injects faults *during*
  reconciliation and measures whether a controller converges. The claim is **defect
  discovery, not certification.**
- **Track B — Reference implementations.** A small number of production Kubernetes control
  planes built to shared conventions, used as proving grounds for Track A. **One active at
  a time.**

**Positioning line:** *Capabilities are declared. Resilience is demonstrated.*

### Assets owned

| Asset | Status |
|---|---|
| `openreconcile.org` | Registered — primary, points at community docs |
| `openreconcile.dev` | Registered — held, unused until Gate 1 |
| `github.com/openreconcile` | Registered |

Canonical domain is **not yet declared**. If Gate 1 concludes the project is a tool rather
than a community, `.dev` may become canonical. Do not hard-code either into anything
irreversible.

---

## 2. The Technical Gap

Existing operator testing covers the happy path:

- `operator-sdk scorecard` — functional tests from the operator bundle against a live cluster
- **chainsaw** — declarative before/after cluster-state assertions. This is the modern,
  actively maintained option; operator-sdk recommends it over kuttl, whose website is
  defunct and whose docs now point at its GitHub repo
- **envtest** — controller testing against a real API server
- **Chaos Mesh / Litmus** — workload-level fault injection

None of these inject faults *during* reconciliation. The failure classes that matter live
there: stale informer cache, partial object creation, duplicate ownership, finalizer
deadlock, controller restart mid-loop, external-create-succeeds-status-write-fails.

Separately, **Operator Capability Levels I–V** already exist as a maturity taxonomy.
OperatorHub renders them from what the operator *claims* in its ClusterServiceVersion.
Nothing verifies the claim. That gap is the positioning.

**This differentiation claim is unproven and is the subject of falsification workstream F1
(see ROADMAP).** Do not treat it as settled.

---

## 3. Decisions Locked

These are settled. Do not reopen without new evidence.

1. ReconcileBench is the first technical wedge.
2. The initial claim is defect discovery, not certification.
3. Testing targets failures *during* reconciliation, not generic chaos.
4. **Operator-or-Not** — never assume a project needs an operator. "Do not build an
   operator" is a valid, desirable outcome.
5. **Upstream-first** — prefer contributing to the correct upstream over owning a fork.
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

---

## 4. Approaches Explicitly Rejected

Listed with reasoning, because each looks reasonable and will otherwise be re-proposed.

### Kopf / Python operators
Rejected. Splits the runtime across the org while claiming unified conventions; loses
controller-gen (forcing hand-written CRDs); loses typed clients, conversion webhooks, and
controller-runtime's workqueue and rate-limiting semantics. The stated justification —
"you can import ML libraries into the control loop" — is an anti-pattern: the operator's
dependency tree becomes the union of every framework it supports. Python is for a client
SDK only.

### Org name in the API group (`dbx.openreconcile.io`)
Rejected. Embeds the org identity in every installed API, making donation or transfer a
full API migration for every user. Violates decision 11.

### PAT-in-a-Secret authentication
Rejected as the default path. Every prior Databricks operator shipped PAT-first and every
one is dead. OIDC federation / workload identity from v1.

### Inlining source code in CR spec fields
Rejected. etcd object size limits, YAML escaping, loss of IDE tooling. Reference git with a
pinned commit instead. ConfigMap script injection is a footnote for tiny scripts, not the
headline pattern.

### Upjet / Terraform-derived generation
Rejected for k8sbricks. Inherits Terraform's runtime, state, and failure modes, and would
make us the fourth identical provider behind an incumbent with 209 managed resources.

### Bare Pods from a controller
Rejected. Bare pods do not reschedule or restart. For a training job that is data loss.

### Shared CRDs across projects
Rejected. A shared CRD means a shared install unit and release train, breaking donatability.
Each project defines its own connection CRD with an identical *shape*.

### Custom condition types
Rejected. Use `metav1.Condition`. ACK carries a custom type only because it predates general
adoption, and is stuck with it. The standard type gives `kubectl wait --for=condition=Ready`
and ecosystem tooling for free.

### Five separate framework operators (smolagents, AutoTrain, OpenCode, …)
Rejected. These are one operator with declarative runtime providers. See §6.

### Publishing to ArtifactHub / OperatorHub first
Deferred, reversibly. ArtifactHub is an index, not a channel — charts live in our own OCI
registry either way. OperatorHub requires ongoing OLM bundle maintenance for OpenShift
console install only. Both may be revisited when discoverability outweighs maintenance cost.

### Building a registry/storefront site early
Rejected. A catalog with two entries advertises smallness.

### An SDK that applies to the cluster from a developer laptop
Rejected. Requires ML engineers to hold cluster credentials, which no platform team grants,
and ArgoCD reverts anything applied imperatively. Any future SDK emits to git or posts to a
thin authenticated service.

---

## 5. Prior Art (verified, September 2026)

### Operator testing
- `operator-sdk scorecard` — bundle-driven functional tests
- **chainsaw** — declarative assertions; the actively maintained successor to kuttl
- kuttl — legacy; site defunct, migration path to chainsaw exists
- envtest — controller-level testing
- Chaos Mesh, Litmus — workload fault injection
- Operator Capability Levels I–V — self-declared, unverified

### Databricks (for k8sbricks)
- `Azure/azure-databricks-operator` — Microsoft, Kubebuilder + Go SDK, experimental, dormant for years
- `mach-kernel/databricks-kube-operator` — kube-rs, archived June 2025; README points users to Upjet-generated Crossplane providers
- `upbound/provider-databricks` — Upjet, 69 managed resources, stalled at v0.1.x
- `lalanne/provider-databricks` — Upjet, v2.x, 209 managed resources. **The live incumbent.**
- Databricks DABs (renamed 2026 from "Databricks Asset Bundles" to "Declarative Automation
  Bundles") — `bundle plan -t prod -o json` for GitOps apply with approval gate

**Both the Databricks Go SDK and the Databricks Terraform provider are generated from
Databricks' internal OpenAPI spec, which is not published.** SDK releases reference an
internal OpenAPI SHA. This is why the generator input must be the SDK's public generated Go
types in `service/*`, not a spec document.

### Reference architecture
**AWS Controllers for Kubernetes (ACK)** is the model for k8sbricks: per-service controllers
and Helm charts, `generator.yaml` for field overrides and hooks, late initialization,
adoption of pre-existing resources, cross-account credential mapping by namespace, field
export to ConfigMap/Secret. `aws-controllers-k8s/code-generator` is Apache-2.0 and its output
is already used by both ACK and Crossplane, so the emitter may be retargetable — evaluate
forking it and replacing the AWS model front end before writing a generator from scratch.

### Name collision note
An older, dormant "Open Reconcile" exists — Rebecca Lawler's project, cloned at
`OpenRefine/open-reconcile`, Java package `com.googlecode.openreconcile`. Dead since ~2012.
In the data world "reconciliation" means entity matching (OpenRefine's Reconciliation API).
Mitigation: never let "reconcile" stand alone in titles, descriptions, or og:tags — always
"Kubernetes controller reconciliation". Avoid the hyphenated form `open-reconcile` entirely.

---

## 6. Track B — Project Designs

Designs are settled; **build order is in ROADMAP.md.** Only one is active at a time.

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

**Encoded domain knowledge — a differentiator generated providers cannot match:**
- `existing_cluster_id` silently drops env vars, image, and Spark conf. Gate behind an
  explicit acknowledgement field or emit an admission warning.
- `git_source` checkout happens on the control plane, not the cluster, and requires commit
  pinning (not branch) for reproducibility. Make branch-tracking the escape hatch.

**Unresolved contradiction — blocks first public CRD.** The name is held provisional, but
API groups embed it permanently. Either make `k8sbricks` the final API identity, or choose
a stable API identity independent of the cosmetic project name. Do not ship a provisional
brand in a permanent API group.

**Trademark position: recorded and accepted, not zero-risk.** Infringement turns on
likelihood of confusion between marks on related goods, not on whether "bricks" is
separately registrable. Mitigations: no Databricks logo, colours, or wordmark; prominent
non-affiliation notice in README and docs; no company registration under the name. The
Kubernetes wheel is a Linux Foundation mark with usage guidelines constraining derivative
logos.

### kubereserve — cloud capacity (candidate)

Core plus provider binaries in separate repos (Karpenter model), because credentials and
release cadence differ per vendor. Locked decisions: no Karpenter dependency (must work with
default kube-scheduler); `dryRun: true` as install default; mandatory TTL on every claim;
budget ceiling enforced at admission; independent sweeper. Demand from a separate
`CapacityAutoReserver` doing arithmetic aggregation of pending GPU pods, not scheduling
simulation. Multi-cluster is v2, made additive by six v1 choices (UID-based tagging,
kube-system UID as cluster identity, split binaries, tag-as-source-of-truth, cluster label on
metrics, stay on v1alpha1).

### Environment pooling — "the OpenEnv operator" (candidate)

**The CR boundary is the pool, never the episode.** RL rollouts check out and release
environments thousands of times per second; leases as custom resources would melt etcd.
Control plane owns pool shape; an in-memory data-plane broker owns leases.

`EnvironmentClass` + `EnvironmentPool`. Two binaries — manager and broker — which scale
differently and must not share a chart lifecycle. Protocol-agnostic behind a small interface
(reset/step/health), shipping only the OpenEnv provider. OpenEnv has multi-org governance
(committee including Modal and Prime Intellect) and evolves by RFC, so protocol versions
should be providers rather than breaking CRD changes.

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

## 7. Cross-Project API Conventions

Full text belongs in `community/CONVENTIONS.md`. Summary:

- **No shared CRDs.** Shared shape, separate definitions.
- **Conventions lock now; shared Go structs are extracted later**, after two production
  controllers prove which structures are genuinely identical. Do not create a frozen
  `apis/meta` module yet.
- **Lock now:** API group naming policy; `metav1.Condition`; same-namespace references by
  default for credentials; controllers never write to `spec`; conversion webhook written and
  tested before version promotion; explicit terminal vs recoverable error handling.
- **Learn before extracting:** `ManagedSpec` / `ManagedStatus`, connection reference shapes,
  common policy structs.
- API groups: `<subcomponent>.<project>.io`. No org name. No ACK-style infix — we own the
  domains outright.
- Cross-project references are typed and resolved unstructured via discovery. Zero
  compile-time coupling; a missing project yields `Ready=False` with a recoverable reason.
- Controllers never write to `spec` — only `status`. Defaulting goes in the admission
  webhook. Violating this gives every ArgoCD user an infinite sync loop.

---

## 8. Open Questions

Unresolved. Do not assume answers.

1. **Is ReconcileBench genuinely differentiated?** Falsification workstream F1. Most likely
   outcome is REFRAMED: a scenario library and convergence oracle over chainsaw and
   Chaos Mesh rather than a new testing category.
2. **Will anyone trust findings from an unknown org?** F2.
3. **Does anyone want the conformance layer, or only the test packs?** F3.
4. **Do the assurance work and the operators belong in one org?** F4. There is a real
   conflict-of-interest exposure in grading others' controllers while shipping your own.
5. **Is the AI/ML framing technical or go-to-market?** F5a/F5b. Likely outcome: general
   -purpose tool, AI/ML as first market.
6. **Does target onboarding cost scale?** F6. If every target needs bespoke proxies, ignore
   rules, and convergence logic, the generic-tool thesis fails economically.
7. **k8sbricks name vs API identity.** Blocks the first public CRD.
8. **Compute budget ceiling.** Undecided, and it constrains what coverage can honestly be
   claimed.

---

## 9. Working Rules

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
