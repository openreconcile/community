# OpenReconcile — Standing Constraints

Full background is in `docs/PROJECT-CONTEXT.md`; in other repos, in the vendored copy at the
repo root. Read it before any architectural work. This file is the short form that always
applies.

## Source-of-truth precedence

`AGENTS.md` → `docs/PROJECT-CONTEXT.md` → `docs/context/GATE1.md` and `docs/adr/*` →
`docs/ROADMAP.md` → `docs/context/*` → `docs/projects/*` → `docs/process/*` →
`sources/*` (historical only, contains superseded decisions, never a current specification).

## Core principles

- Open-source and vendor-neutral.
- Defect discovery before certification.
- Operator-or-not: an operator is one possible answer, not the default answer.
- Upstream-first — including for the assurance tooling itself.
- Reuse existing control planes.
- Human-governed architecture and security decisions.
- Independent adversarial review.
- Donatability: avoid API identities that make future transfer require user migration.
- Build narrowly; evidence unlocks scope.

## Project state

Gate 1 closed 2026-09-13. ReconcileBench is an engine-agnostic oracle-and-agent layer
above operator-chaos, envtest, and chainsaw. Do not claim novelty for mid-reconcile
fault injection. Read `docs/context/PRIOR_ART.md` and `docs/context/GATE1.md` before
making any claim about what does not exist.

## Stack — not negotiable

- **Go** with **controller-runtime / Kubebuilder** for all production controllers
- **chainsaw** for declarative assertions (not kuttl — deprecated, site defunct)
- **operator-chaos** as the first-class operator-semantic perturbation engine — do not
  reimplement its injection types
- **Chaos Mesh** or **Litmus** only as primitives an engine already uses
- **envtest** for controller-level tests
- **cobra** for CLIs
- Helm charts distributed via OCI, signed with cosign

This constrains controller runtimes, not test harnesses. Sieve and Acto are Python and are
evaluated on merit.

## Never propose these

Each was evaluated and rejected. Reasoning is in `docs/PROJECT-CONTEXT.md` §4.

- Kopf or any Python-based production operator
- Importing ML libraries into a controller's dependency tree
- The org name inside an API group (e.g. `dbx.openreconcile.io`)
- A frozen shared `apis/meta` Go module — conventions lock now, structs are extracted after
  two production controllers
- A custom condition type instead of `metav1.Condition`
- Literal write quiescence as the definition of convergence
- PAT / static token in a Secret as the default auth path
- Source code inlined into a CR spec field
- Upjet or Terraform-derived resource generation
- Bare Pods created directly by a controller
- CRDs shared across projects
- A separate operator per ML framework (they are declarative `MLRuntime` CRs)
- Building a registry or storefront site
- An SDK that applies manifests to a cluster from a developer machine

## Scope discipline

- **One Track B project active at a time.** Currently: the fixture, then k8sbricks Compute.
- Do not implement a ReconcileBench fault injector.
- Do not create an OpenReconcile environment-pool repo.
- Do not start KubeReserve cloud-SDK work before its RFC.
- Do not build Forge, Radar, Catalog, or Drift.

## Blocked until resolved

**Do not run `kubebuilder init` for k8sbricks and do not commit any public
`groupversion_info.go` for it.** The project name is provisional but API groups embed it
permanently. This contradiction must be resolved first. The fixture uses the throwaway domain
`fixture.openreconcile.org` and is unaffected.

## Writing style for docs and READMEs

Claims must match evidence. "Finds reconciliation defects that ordinary tests miss" is
supportable once demonstrated. "Verified", "certified", "production-ready" are not, and
"resilient" is a property to demonstrate, never to assert. Positioning line:
*Capabilities are declared. Resilience is demonstrated.*

Never let "reconcile" stand alone in titles, descriptions, or `og:` tags — always "Kubernetes
controller reconciliation". A dormant data-reconciliation project of the same name exists,
and in the data world "reconciliation" means entity matching. Avoid the hyphenated form
`open-reconcile` entirely.
