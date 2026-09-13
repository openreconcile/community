# Survey: Kubernetes controller resilience testing (September 2026)

**As-of:** 2026-09-13
**Audience:** controller authors, platform engineers, maintainers of operator test tools
**Not a ranking.** A map of who already occupies which layer, so a new tool does not
re-claim solved problems.

OpenReconcile used this survey to close Gate 1. The product decision is recorded in
[`../context/GATE1.md`](../context/GATE1.md). This document is intended to stand alone.

## The question that is already answered

“Can we inject faults *during* reconciliation of a Kubernetes controller and see
whether it converges?”

Yes. Twice.

| Tool | When | Status | What it actually does |
|---|---|---|---|
| [Sieve](https://github.com/sieve-project/sieve) (OSDI ’22) | 2022 | Dormant since 2024-09-26 | Learns event sequences, injects crash / partition / stale-API-server faults at chosen cluster-state transitions. Requires a ported controller and, for stale-state, a patched Kubernetes kind node image. |
| [operator-chaos](https://github.com/opendatahub-io/operator-chaos) | 2026 | Maintained (Open Data Hub / Red Hat) | Twenty operator-semantic injection types, four modes (CLI, chaostransport, ChaosClient, ActionInterceptor), YAML knowledge models, verdicts `Resilient` / `Degraded` / `Failed` / `Inconclusive`. |

Building a third injector is not a contribution. Using one of these as an engine might be.

## The question that is only half answered

“Did the controller become *correct*, not merely *quiet*?”

Happy-path tools answer a weaker form of this:

- **envtest** — real API server, no fault model.
- **chainsaw** — declarative before/after cluster assertions. The maintained successor
  to kuttl (whose site is defunct).
- **operator-sdk scorecard** — bundle-driven functional tests.
- **Chaos Mesh / Litmus / Krkn** — infrastructure faults (pod kill, network, node).
  They test the platform, not whether the operator restored its resource graph.

[Acto](https://github.com/xlab-uiuc/acto) answers a stronger form: mutate the CR,
snapshot the cluster, run consistency / health / differential oracles. It has found
tens of real bugs and onboards from a deployment script. It does **not** time a fault
to a reconcile phase.

[Anvil](https://github.com/xlab-uiuc/anvil) answers the strongest form — Eventually
Stable Reconciliation, formally verified — but only for controllers rewritten in
Rust against Verus. That is a research artefact, not a CI tool for
controller-runtime authors.

## A blind spot shared by Sieve and Acto

Both tools **mask** `status.observedGeneration` and `status.conditions` (Sieve via
`field_path_mask`; Acto via `EXCLUDE_PATH_REGEX`) because those fields churn and
create false positives.

The Kubernetes API convention is the opposite: `observedGeneration` and conditions
*are* the contract that tells a user whether the controller has seen the current
spec. A controller that never writes `observedGeneration` is a defect. A tool that
cannot see that defect is not checking the status contract.

This is cheap to check and, as of this survey, unclaimed as a first-class oracle.

## Knowledge models are still handwritten

operator-chaos’s best idea is the knowledge model: declare what the operator owns,
then judge the graph after a fault. Today that YAML is authored by a human. Deriving
it from CRD schemas and controller source would collapse Target Onboarding Cost —
the same reason Acto’s “give us the deploy script” story wins.

No maintained tool in this survey does that derivation.

## External APIs are still a second-class fault domain

Sieve’s model is the controller’s view of *cluster* state. Acto snapshots cluster
state. operator-chaos injects into Kubernetes objects and RBAC/webhooks. The
failure “external create succeeded, status write failed, the foreign key is gone”
is the everyday cloud-operator bug and is poorly represented.

A fixture that includes a fake asynchronous external service is the cheapest way
to make this class testable. It is not itself a new engine.

## Adjacent systems people confuse with this problem

| System | Related problem | Not this problem |
|---|---|---|
| OpenSandbox `Pool` / `BatchSandbox` | Environment pooling for RL / agents | Operator-correctness testing |
| OpenEnv `KubernetesProvider` (planned) | Running OpenEnv envs on Kubernetes | Same |
| Karpenter ODCR / Capacity Blocks | *Consuming* reserved cloud capacity | *Creating / expiring / budgeting* reservations |
| `glalanne/provider-databricks` | Terraform-derived Databricks as Crossplane MRs | Kubernetes-native Databricks control plane |
| Operator Capability Levels I–V | Self-declared maturity on OperatorHub | Nothing verifies the claim |

## Practical recommendation

If you maintain an operator today:

1. Keep envtest and chainsaw. They are the happy path.
2. If you need operator-semantic faults, try operator-chaos before writing a
   harness.
3. Add an explicit check that `status.observedGeneration == metadata.generation`
   and that `Ready` is consistent with remaining errors. Do not copy Sieve/Acto’s
   ignore list onto those fields.
4. If the operator talks to a cloud API, test the create-then-lose-the-ID window
   with a fake. Chaos Mesh will not find it.

If you are considering a new testing tool, the unclaimed layers are: a
status-contract oracle, source-derived knowledge models, and evidence an agent
can act on. The injector layer is taken.

## Sources

- Sun et al., *Automatic Reliability Testing for Cluster Management Controllers*,
  OSDI 2022 (Sieve).
- Acto repository and porting docs (`xlab-uiuc/acto`), including the documented
  false alarm on control-flow-dependent spec fields.
- Anvil / Eventually Stable Reconciliation (`xlab-uiuc/anvil`).
- operator-chaos documentation and the Red Hat Developer introduction
  (2026-07-11).
- OpenSandbox Kubernetes docs; OpenEnv runtime-provider docs (KubernetesProvider
  planned).
- Karpenter ODCR / Capacity Blocks task docs.
- Upbound marketplace: `lalanne/provider-databricks` v2.5.0 (2026-08-31),
  source `glalanne/provider-databricks`.

Corrections welcome as issues on [openreconcile/community](https://github.com/openreconcile/community).
