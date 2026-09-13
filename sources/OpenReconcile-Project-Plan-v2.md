# OpenReconcile — Project Plan v2 (For Review)

**Status:** Draft for review
**Supersedes:** OpenReconcile — Final Project Plan (v1)
**Date:** September 2026

---

## 1. Executive Summary

OpenReconcile is an open, vendor-neutral engineering community for making AI/ML software operable on Kubernetes. It has two tracks that reinforce each other:

**Track A — Assurance.** ReconcileBench: an adversarial testing tool that injects failures *during* reconciliation and measures whether a controller converges. The claim is defect discovery, not certification.

**Track B — Operators.** A small number of Kubernetes control planes for the ML stack — capacity, data platform, workloads, environments — built to a shared set of API conventions and used as the proving ground for Track A.

The organising thesis: **generating infrastructure code is now cheap; determining the correct architecture and proving the result behaves correctly under failure is not.**

Milestone 0 is deliberately narrow: build ReconcileBench, run it against our own operator, publish the tool and what it found. This requires no external permission, no community, and no frozen specifications.

### Change from v1

v1 positioned OpenReconcile as an independent verification body producing trusted evidence for third-party projects. That framing has an authority problem — certification value is conferred by an ecosystem, not claimed by an unknown org — and it made every milestone dependent on people we have not yet met.

v2 keeps v1's principles and reverses its sequencing. We earn standing by finding real bugs in real controllers, starting with our own. Catalog, Drift, Radar and Forge remain in the roadmap but sit behind a decision gate.

---

## 2. Problem Statement

AI/ML open-source projects are outpacing the Kubernetes production ecosystem around them. Common failure patterns:

- no Kubernetes path, or one that only works for demos
- stale operators and charts
- untested upgrades and poor failure recovery
- duplicated infrastructure where an existing ecosystem would serve
- commercial-only Kubernetes functionality with no open equivalent
- strong application code, insufficient operational knowledge
- no independent evidence that the deployment behaves correctly under failure

AI-assisted development compounds this. Controller code is now cheap to generate and expensive to validate. Passing CI proves manifests applied; it does not prove the control loop converges when a node drains mid-reconcile.

### The specific gap

Existing operator testing tooling covers the happy path:

- `operator-sdk scorecard` runs functional tests against a live cluster from the operator bundle
- **chainsaw** does declarative before/after cluster-state assertions and is now the actively maintained option — operator-sdk recommends it over kuttl, whose site is defunct
- envtest covers controller unit testing

None of these inject faults *during* reconciliation. Stale informer cache, partial object creation, duplicate ownership, finalizer deadlock, controller restart mid-loop — these are where real controller bugs live, and nothing systematically tests for them.

Separately, **Operator Capability Levels I–V** already exist as a maturity taxonomy, and OperatorHub renders them from what the operator claims in its ClusterServiceVersion. Nothing verifies the claim.

**Capability levels are declared. We measure them.**

---

## 3. What OpenReconcile Is Not

- a certification authority
- a generic operator generator
- a replacement for Kubebuilder, Operator SDK, Helm, chainsaw, KServe, KubeRay or Kubeflow
- a repository containing one operator per AI project
- an AI-generated infrastructure marketplace
- a chart or operator directory
- a closed commercial layer around open-source projects

These boundaries are load-bearing and should be enforced in review.

---

## 4. Core Principles

**4.1 Operator-or-Not.** Never assume a project needs an operator. Valid outcomes include integration with an existing controller, a Job, a Helm chart, a provider, or no Kubernetes-specific abstraction at all. "Do not build an operator" is a desirable result.

**4.2 Upstream First.** Prefer contributing to the correct upstream over owning a fork. Own the intelligence, the proof, and the conventions — not unnecessary code.

**4.3 Use Existing Control Planes.** Do not recreate what Kubernetes, KServe, KubeRay, Kubeflow, Crossplane or mature upstream operators already do well.

**4.4 AI-Assisted, Human-Governed.** Agents accelerate research, implementation, test generation and review. Humans retain authority over architecture, security decisions, merge approval and governance.

**Conflict-of-interest rule:** adversarial scenario generation must be independent of implementation generation. If the same agent writes the controller and the tests that validate it, the proof correlates with the artefact it is proving. This constraint is set now, before Forge exists.

**4.5 Proof Before Distribution.** Passing CI is not evidence of production readiness. Every supported integration should produce structured evidence of behaviour under failure, recovery, upgrade and lifecycle transition.

**4.6 Donatability.** Every project must be transferable to another org or foundation without an API migration for its users. This drives the API group policy in §8.

---

## 5. Program Structure

Two tracks, deliberately coupled.

```
Track A — Assurance          Track B — Operators
─────────────────────        ──────────────────────
ReconcileBench          ←→   k8sbricks     (Databricks control plane)
Bench target files           kubereserve   (cloud capacity)
Findings & case studies      env pooling   (OpenEnv-compatible)
                             workload runner (MLRuntime providers)
                                    ↑
                        API conventions + frozen meta module
```

Track B gives Track A subjects we own and may break publicly. Track A gives Track B evidence that its controllers actually work. Neither depends on an external maintainer's goodwill to produce its first result.

---

## 6. Track A — ReconcileBench

### 6.1 Scope

A standalone open-source tool. Point it at any operator; it installs the operator, applies custom resources, injects faults during reconciliation, and reports whether the controller converged.

Built on existing infrastructure:

- **chainsaw** for declarative assertions
- **Chaos Mesh** or Litmus for fault primitives
- our layer: convergence detection, invariant checking, mid-reconcile fault timing

We write the reconciliation-specific logic. We do not reimplement pod kill and node drain.

### 6.2 The hard problem

**Defining convergence.** Everything ReconcileBench reports rests on a precise, non-gameable definition of "the controller reached correct state." This is the single genuine design question in Milestone 0 and should be settled before scenario work begins.

Working definition to be challenged in review: given a declared invariant set, the target converges if within a bounded time after fault resolution all invariants hold, `observedGeneration` matches `generation`, and no further writes occur. Requires care around legitimately long operations (a 30-minute serving endpoint provision is not a hang).

### 6.3 Scenario set

Full catalogue (v1 §7) retained as roadmap. **v0.1 ships seven:**

1. pod deletion mid-reconcile
2. controller restart mid-reconcile
3. failed child resource creation
4. partial object creation (create succeeds, status write fails)
5. stale informer cache
6. finalizer failure on delete
7. scale change during reconcile

Selection criterion: each must be capable of failing on a real, competently written operator. A scenario nothing ever fails is not a test.

### 6.4 Bench target file

ReconcileBench needs a per-target declaration of what correct means. This is the surviving subset of v1's ProjectOperationalProfile:

- lifecycle transitions
- reconciliation invariants
- failure model
- expected conditions per phase

**Deliberately not a versioned specification.** It is a tool input file, changeable freely as scenarios teach us what it needs. Promoting it to a spec creates a maintenance obligation before we know the shape is right.

The remainder of v1 §5.2 — compute, state, networking, security posture, observability, distributed execution, topology — is Architect input and defers to Phase 3+.

### 6.5 Findings output

Per run: target and version, Kubernetes version, scenarios executed, results, logs, reproduction command.

**Every output carries an as-of date and validity basis from v0.1.** A stale readiness claim is worse than no claim. Signing, SBOM and provenance are real requirements but are features of a trusted artefact; they follow trust rather than creating it.

### 6.6 Disclosure policy

Required before ReconcileBench is run against any code we do not own.

Private report to maintainers → agreed remediation window → publication. Modelled on security embargo practice. Publishing "failed 12 of 24" without warning makes an adversary on day one and teaches the ecosystem to avoid us.

---

## 7. Track B — Operators

### 7.1 k8sbricks — Databricks control plane

**Scope:** the full Databricks surface, not a single resource.

**Architecture:** AWS Controllers for Kubernetes (ACK) as the reference model.

- code generation from `databricks-sdk-go`'s public `service/*` packages. Both the Go SDK and the Databricks Terraform provider are generated from Databricks' internal OpenAPI spec, which is not published — so the SDK's generated Go types are the closest available machine-readable model.
- evaluate forking `aws-controllers-k8s/code-generator` (Apache-2.0) and replacing its AWS model front end while keeping the emitter, hooks system and runtime contract. Spike this before writing a generator from scratch.
- per-service controllers and Helm charts, so users install ~20 CRDs rather than 200+. CRD count is an operational problem: discovery-document bloat, etcd load, kubectl latency cluster-wide. Upbound split into provider families for exactly this reason.
- per-service `generator.yaml` equivalent for field overrides and hooks — this replaces a hard hand-written/generated tier split with a dial.
- late initialization is mandatory. Databricks defaults server-side extensively; without it, controllers hot-loop against spurious drift.
- adoption of pre-existing resources from v1. Nobody starts greenfield on Databricks.

**Auth:** OIDC federation, workload identity, no PAT in a Secret. Every prior Databricks operator shipped PAT-first and every one is dead. This is the security-review gate.

**Encoded domain knowledge** — a differentiator generated providers cannot match:

- `existing_cluster_id` silently drops env vars, image and Spark conf; gate behind explicit acknowledgement or emit an admission warning
- `git_source` requires commit pinning for reproducibility; make branch-tracking the escape hatch, not the documented path

**Prior art (verified):** Azure/azure-databricks-operator (dormant), mach-kernel/databricks-kube-operator (archived June 2025), upbound/provider-databricks (Upjet, stalled at v0.1.x), lalanne/provider-databricks (Upjet, v2.x, 209 managed resources — the live incumbent). Positioning is *generated from the SDK, not from Terraform*, plus execution semantics and identity.

**Naming risk — recorded, accepted.** Infringement turns on likelihood of confusion between marks on related goods, not on whether "bricks" is separately registrable. Mitigations: no Databricks logo, colours or wordmark; prominent non-affiliation notice in README and docs; no company registration under the name. The Kubernetes wheel is a Linux Foundation mark with usage guidelines that constrain derivative logos.

### 7.2 kubereserve — cloud capacity

Core plus provider binaries in separate repos (Karpenter model), because credentials and release cadence differ per vendor. Decisions already locked: no Karpenter dependency, `dryRun` install default, mandatory TTL on claims, budget ceiling at admission, independent sweeper, v2 multi-cluster made additive by six v1 choices.

### 7.3 Environment pooling

For OpenEnv-compatible RL environments. **The CR boundary is the pool, never the episode** — rollouts checkout and release thousands of times per second, and leases as custom resources would melt etcd. Control plane owns pool shape; an in-memory data-plane broker owns leases.

`EnvironmentClass` + `EnvironmentPool`. Protocol-agnostic behind a small interface (reset/step/health), shipping only the OpenEnv provider. Given OpenEnv's RFC-driven multi-org governance, protocol versions should be providers rather than breaking CRD changes.

### 7.4 Workload runner

`MLRuntime` (cluster-scoped, community-contributable) + `MLWorkload` (namespaced). Runtime providers are declarative CRs, not Go — adding smolagents, AutoTrain or a new framework is a YAML PR. This is the contributor-leverage mechanism for a small maintainer team.

Zero-Docker path: curated base images, git-sync with commit pinning, PVC-cached dependency layers. Production requires a bake-to-image promotion step; `pip install` at pod start is a development path only and must be blocked by policy in production namespaces.

### 7.5 Language

**Go and Kubebuilder throughout.** Python only for a client SDK.

Rejecting Kopf: k8sbricks and kubereserve need Go SDKs and generated controllers; splitting runtimes contradicts the unified-conventions claim; importing ML libraries into the control loop is an anti-pattern that makes the operator's dependency tree the union of every supported framework; and losing controller-gen means hand-written CRDs, which is why the earlier drafts' Job schema covered a fraction of the real API.

### 7.6 Python SDK (roadmap, not Milestone 0)

Decorator-style deployment for ML scientists. **Must emit to git, not apply to the cluster.** An SDK holding cluster credentials on a scientist's laptop cannot pass platform review and would be reverted immediately by ArgoCD. PR-based emission, or a thin authenticated service holding the credentials.

---

## 8. API and Convention Standards

The org's actual intellectual content, and the mechanism by which external projects can participate.

**No shared CRDs.** A shared CRD means a shared install unit and release train, which breaks donatability.

**One frozen meta module** — `openreconcile.io/apis/meta/v1alpha1` — containing embeddable structs only, no CRDs, no group. `ManagedSpec` (connectionRef, managementPolicy, deletionPolicy, suspend) and `ManagedStatus` (observedGeneration, conditions, externalID, lastSyncTime). Additive changes only, forever; any change is an org-level RFC.

**Standard `metav1.Condition`**, not a custom type. Gives `kubectl wait --for=condition=Ready` and ecosystem tooling for free. ACK carries a custom type only because it predates general adoption.

**Terminal vs recoverable error classification.** Load-bearing at scale — it is what stops a controller burning API quota retrying a spec the remote will never accept.

**API groups: `<subcomponent>.<project>.io`.**

| | |
|---|---|
| k8sbricks | `serving.k8sbricks.io`, `compute.`, `jobs.`, `pipelines.`, `catalog.`, `sql.`, `iam.`, `core.` |
| kubereserve | `kubereserve.io`, `aws.kubereserve.io`, `azure.kubereserve.io` |

Group names align to the SDK's `service/*` package names, so membership is derived rather than decided. No org name in any API group — `dbx.openreconcile.io` would make donation an API migration for every user. No ACK-style infix; we own each domain outright.

**Per-project connection CRDs** with identical shape (`WorkspaceConnection`, `CloudConnection`, `RegistryConnection`). Three CRDs installed rather than one is the price of independent releasability.

**Cross-project references** are typed and unstructured — resolved at runtime via discovery, zero compile-time coupling. A missing project yields `Ready=False` with a recoverable reason, not a build failure.

**Same-namespace references by default.** Cross-namespace credential references are privilege escalation; Gateway API had to retrofit ReferenceGrant. Easier to relax than to tighten.

**Controllers never write to `spec`.** Only `status`. Defaulting goes in the admission webhook. Violating this gives every ArgoCD user an infinite sync loop.

**Versioning:** v1alpha1 everywhere; per-group and independent. Group version tracks the weakest resource in the group; a per-resource maturity matrix carries the real signal. Promotion requires the conversion webhook *written and tested*, not planned.

**Prerequisite:** register the domains before committing the first `groupversion_info.go`. Changing an API group post-release is a full migration.

---

## 9. Distribution

**No ArtifactHub or OperatorHub listing.** Decided.

ArtifactHub is an index rather than a channel — charts live in our own registry either way — so declining costs search visibility, not control, and the option remains open indefinitely. OperatorHub requires ongoing OLM bundle maintenance for OpenShift console install only.

Consequence: we publish our own security posture rather than inheriting a third-party scan report. Cosign signatures, SBOM per release, verification command in the README. Enterprise procurement will ask.

Helm OCI, no chart repo, no index.yaml:

```
helm install k8sbricks oci://ghcr.io/openreconcile/charts/k8sbricks --version 0.1.0
```

---

## 10. Community Model

**OpenReconcile is a standard with reference implementations, not a catalog of one person's repos.**

The published artefact is the conventions in §8 plus a conformance suite. Any operator can conform, including ones we did not write. This is what makes external participation possible without donation — listing costs a test run.

A community org where one person writes everything is a portfolio. Conformance is the mechanism that makes other people's work count.

**Write the conformance suite against the spec, not against k8sbricks.** If it encodes our implementation details it becomes a barrier rather than an invitation, even though k8sbricks will be its first subject.

### Org repositories

```
openreconcile/
  .github/       community health files, profile
  community/     governance, conventions, conformance spec, maturity ladder
  apis/          frozen meta module
  reconcilebench/
  template/      repo scaffold + reusable workflows
```

Projects live in their own repos. Commercial work stays in a separate org — mixing it in makes the community org look like a vendor funnel and complicates any future donation.

### Governance minimum — before the second repo exists

1. **Decision-making:** who decides, how disputes resolve
2. **Maintainer path:** how someone becomes one
3. **Archival policy:** what happens to stale projects

Draft entry criteria: named maintainer and working e2e test to enter; second contributor to leave alpha; at most two projects in alpha concurrently; explicit archive notice after two release cycles untouched.

Publishing the archival policy is itself a credibility signal. This category's characteristic failure is abandonment — two dead Databricks operators are the immediate evidence.

---

## 11. Delivery Plan

### Milestone 0 — ReconcileBench (no external dependencies)

1. Settle the convergence definition
2. Build ReconcileBench with the seven v0.1 scenarios on chainsaw + Chaos Mesh
3. Write the bench target file for k8sbricks
4. Run it; fix what it finds
5. Publish tool, findings and method

Exit: the tool distinguishes an operator that deploys from one that recovers, and has found at least one real bug in our own code.

Founding story: *we built an operator, then built a tool that found bugs in it, here is the evidence.* Better than an unknown org grading strangers.

### Milestone 1 — Conventions and first release

6. Freeze the meta module; publish CONVENTIONS.md and the conformance suite
7. k8sbricks generator spike (evaluate the ack-generate fork)
8. First k8sbricks service controller with OIDC auth and adoption
9. Governance documents published

### Milestone 2 — External validation

10. Prior-art research on candidate lighthouse targets — the check that catches "there is no Databricks operator" before it becomes a premise
11. **Lighthouse A (OpenEnv):** bench target file, architecture assessment, upstream engagement. **Exit condition is PR opened, maintainer engaged, findings published** — not "merged." Never gate our milestone on someone else's merge button.
12. **Lighthouse B (no-operator verdict):** on a project people are *actively building operators for right now*. Ludwig is the wrong choice — a no-build recommendation for a project nobody deploys demonstrates nothing.
13. **Lighthouse C (prove-only):** third-party operator, disclosure policy in force

### Decision gate

Proceed to platform scale only on evidence of: real upstream demand; repeatable architecture analysis; defects found that normal CI missed; successful ecosystem collaboration; third parties reusing our evidence; agent workflows that accelerate without degrading quality.

If met, next investment is Forge, then Radar, Catalog, Drift. If not, OpenReconcile remains a proof toolkit and a small set of well-run operators — which is a legitimate outcome, not a failure.

### Explicitly deferred past the gate

Radar · Forge · Catalog · Drift · Operating Knowledge Packs · full ProjectOperationalProfile schema · ArchitectureDecision schema · Python SDK · community portal, reputation, certification

The Upstream Router is deleted as a *component* — it is a policy decision, not software.

---

## 12. Risks

| Risk | Mitigation |
|---|---|
| Becoming an operator factory | Architecture assessment mandatory before implementation |
| Building another Kubernetes platform | Integration-first and upstream-first policy; §3 enforced in review |
| AI-generated low-quality code | Independent proof, mandatory review, §4.4 conflict-of-interest rule |
| Excessive scope | Decision gate; two-projects-in-alpha cap |
| Proof becomes just another CI suite | Focus strictly on reconciliation invariants and mid-loop fault injection |
| Upstream rejects external infrastructure | Engage maintainers before writing anything; exit conditions we control |
| **Adversarial findings damage relationships** | Disclosure policy (§6.6) before any third-party run |
| **Stale evidence misleads** | As-of date and validity basis on every output from v0.1 |
| **Abandonment across projects** | Published entry criteria and archival policy before second repo |
| **Compute cost** | Set the ceiling before designing the scenario matrix — it constrains coverage claims |
| **Trademark exposure** | §7.1 mitigations; no logo, colours, wordmark, or company registration |
| Existing projects build similar capability | Moat is cross-project reconciliation failure data, not the tool |

---

## 13. Open Questions for Reviewers

1. Is the convergence definition (§6.2) precise enough to be non-gameable, and does it handle legitimately long operations?
2. Are the seven v0.1 scenarios the right seven? Which would a competent operator actually fail?
3. Is "capability levels are declared, we measure them" a strong enough position given operator-sdk scorecard and chainsaw already exist?
4. Is the two-track structure coherent, or does Track B dilute Track A's focus?
5. Is declining ArtifactHub a mistake we will regret at adoption time?
6. Does conformance-as-participation actually attract external projects, or is it a mechanism nobody uses?
7. Is one frozen meta module enough shared surface, or too little to make the conventions real?
8. Which candidate is the right Lighthouse B — where are people currently building operators they should not build?
9. What would make a platform engineering team trust ReconcileBench output from an unknown org?
10. What is the minimum evidence before we describe anything as verified — and should we use that word at all?
11. Which assumptions could invalidate the thesis entirely?
12. Is there an existing CNCF or Kubernetes project already solving the mid-reconcile fault injection problem that we have missed?

---

## 14. Final Position

OpenReconcile should not try to win by producing more Kubernetes code than anyone else.

It should become the place that answers three questions well:

**What is the correct way to operate this AI/ML project on Kubernetes?**

**Can we get there without duplicating an existing ecosystem?**

**Can we prove, reproducibly, that the result behaves correctly when reality goes wrong?**

Only the third needs nobody's permission. That is where we start.
