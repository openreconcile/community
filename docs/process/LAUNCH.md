# Launch readiness

**Status:** Not ready. Do not announce. Do not enable the NL slogan.
**Decided 2026-09-13:** Launch only when the two real operators are the
worked examples — KubeReserve regenerated greenfield through the pipeline,
k8sbricks onboarded through the retrofit path — and each has passing e2e.
OpenEnv is optional (upstream contribution, not an org repo).

This is the go / no-go list. It is narrower than “push every local folder.”

## What launch means

A public **alpha**, not a product:

- GitHub repos exist, Apache-2.0, honest READMEs.
- Someone else can clone, test, and see the same oracle evidence we saw.
- We do **not** claim Databricks-on-Kubernetes is production, or that
  reservations spend real money, or that English always yields a correct operator.

Announcement line until the gates below pass remains:

*Capabilities are declared. Resilience is demonstrated.*

*Build a Kubernetes operator in natural language* is allowed only after Gate L3
has been replayed by someone other than the author. k8sbricks is the AI/ML
proof point, not the slogan.

## Operators in scope

| Operator | Ship form | How it is the example | Must pass before launch |
|---|---|---|---|
| **KubeReserve** | `openreconcile/kubereserve`, `capacity.kubereserve.io` | **Greenfield.** Regenerated from `KUBERESERVE-RFC.md` via `new-operator` / `openreconcile new`. Current handwritten stub is not consulted. | `dryRun: true` e2e: TTL required, budget field present, `expiresAt` derived, `observedGeneration`, no cloud SDK call; `dryRun: false` still refuses |
| **k8sbricks** Compute | `openreconcile/k8sbricks`, `compute.k8sbricks.io` | **Retrofit.** Existing spike onboarded via `onboard` / `openreconcile onboard`. | Fake or recorded Databricks client; envtest e2e for create / ack-existing-cluster / status contract / delete |
| **OpenEnv** | Optional. Draft under `contrib/`, PR to `meta-pytorch/OpenEnv` if their interface still matches | Not an org example. | kind or envtest: start pod, `wait_for_ready`, stop. **No org repo.** |

Do not create a third alpha org operator. OpenEnv stays upstream-shaped.

k8sbricks and KubeReserve may both be public **only** as alpha, and only after
each has a named maintainer (can be the same person at launch) and a working
e2e. Archive policy still applies.

## Honesty rules

- KubeReserve regeneration takes the RFC, the Skill, and `.cursor/rules` as
  input. The current `kubereserve/internal/controller/reservation_controller.go`
  is moved aside locally and not consulted. If the RFC is missing something the
  stub knew, the RFC is fixed first.
- Every evidence JSON checked into an example is the real output of the run,
  including runs where the oracle finds nothing. No mutant is introduced to
  manufacture a finding, except where explicitly labelled as a mutant.
- k8sbricks was handwritten. The retrofit write-up describes that. Do not claim
  it was generated from English.

## Gates (all required except L5)

### L0 — Public story (not a launch)

- [ ] `community` on `main` includes the survey, `GATE1.md`, ADR-0001, prior art,
      Skills, `DISCLOSURE.md`, `SECURITY.md`, this file.
- [ ] `openreconcile/.github` profile README is live and does not use the NL slogan.
- [ ] Org description is not “AI/ML Operator Community” as if that were the product.

Do **not** comment on `opendatahub-io/operator-chaos#6`. Do not file further
issues on operator-chaos until `derive` attaches a real `knowledge.yaml`.

### L1 — Pipeline is mechanically real

- [ ] Fixture: unit tests for **all eight** defects, not three.
- [ ] `reconcilebench verify` fails on each defective snapshot and passes on clean.
- [ ] `reconcilebench derive` emits `knowledge.yaml` from fixture CRDs.
- [ ] `openreconcile new` copies `controller-template` and writes `PROMPT.md`.
- [ ] Scaffolded Ginkgo envtest suites in fixture, k8sbricks, and kubereserve
      create valid CRs and assert the status contract.
- [ ] CI on `reconcilebench` and `reconcilebench-fixture`: `go test ./...` on PR.

### L2 — Template and onboard path

- [ ] `controller-template` has a real Kubebuilder baseline (`api/`,
      `internal/controller/`, `cmd/`, `Makefile`, `PROJECT`, `config/`).
- [ ] `hack/scaffold.sh` refuses `*.openreconcile.io` API groups.
- [ ] Template envtest asserts the status contract out of the box.
- [ ] `onboard` Skill exists and refuses to restructure the host repo or change
      its API group.
- [ ] `openreconcile onboard <path>` derives `knowledge.yaml` from
      `config/crd/bases` and writes `ONBOARD.md`. No scaffolding, no file rewrites
      of the host controller.
- [ ] Skill is vendored into `controller-template` and linked from org
      `CONTRIBUTING.md`.

### L3 — Greenfield example: KubeReserve

A **recorded** run, checked into `community/docs/examples/greenfield-kubereserve/`,
that an outsider can replay:

1. English / RFC: reservation lifecycle (TTL, dry-run, budget, adopt, sweep).
2. Skill or `openreconcile new` scaffolds from `controller-template`.
3. Implement against `dryRun: true` only. No AWS credentials.
4. `reconcilebench derive` → target model.
5. `reconcilebench verify` on a clean object (pass) and on a labelled mutant
   (fail with evidence JSON).
6. envtest: TTL required; `dryRun: true` makes no cloud calls; `dryRun: false`
   still refuses; `expiresAt` derived from `ttl`; oracle pass.
7. `WALKTHROUGH.md` with the exact commands, the prompt, and the evidence files.

README: Karpenter consumes reservations; we only manage lifecycle.

### L4 — Retrofit example: k8sbricks

A **recorded** run, checked into `community/docs/examples/retrofit-k8sbricks/`,
that an outsider can replay:

1. `openreconcile onboard` against the existing handwritten spike.
2. Databricks `Clusters` interface plus a recording fake, written naively first.
3. Controller wired off `SDKNotWired`.
4. Capture the real oracle evidence. Fix what it reports. If it finds nothing,
   the example says the pipeline confirmed clean, not that it found bugs.
5. envtest: `existingClusterId` without acknowledgement → `Ready=False`,
   `ExistingClusterSemanticsUnacked`; happy path; `externalID` persisted before
   `phase=RUNNING`; `observedGeneration`; delete/finalizer.
6. `WALKTHROUGH.md` stating the spike was handwritten and this pass is retrofit.

README: not affiliated with Databricks; incumbent is Crossplane/Upjet;
this is Compute-only alpha.

### L5 — OpenEnv (optional, does not block L3 or L4)

- [ ] `KubernetesProvider` matches current OpenEnv provider methods.
- [ ] Test starts and stops a pod on kind **or** a documented skip if no cluster.
- [ ] Upstream issue/PR opened only after that test exists. No `openreconcile/openenv` repo.

### L6 — Announce (only after L0–L4)

- [ ] Org profile lists community, fixture, reconcilebench, controller-template,
      k8sbricks, kubereserve.
- [ ] Each code README links its pipeline example, not the gated demo.
- [ ] No *Build a Kubernetes operator in natural language* until L3 is on `main`
      and has been followed by someone other than the author (even a friend).
- [ ] No third-party operator bug-count marketing.

## Test matrix (minimum)

| Target | Unit | Oracle | envtest / fake client | kind |
|---|---|---|---|---|
| Fixture clean + 8 defects | required | required | optional for launch | no |
| ReconcileBench verify/derive | required | n/a | n/a | no |
| controller-template | required | required | required | no |
| KubeReserve (greenfield example) | required | required | required (dry-run) | no |
| k8sbricks (retrofit example) | required | required | required (fake SDK) | no |
| OpenEnv provider | optional | n/a | n/a | optional |

No AWS or Databricks credentials are required to launch. If a test needs them,
the test is wrong for alpha.

Old L2 (synthetic MLflow example) and old L4 (operator e2e as a separate gate)
are collapsed: each operator's e2e *is* its example's evidence.

## Sequence

```text
L0 community + .github  (no operator-chaos comment)
        ↓
L1 fixture defects + envtest fix + CI
        ↓
L2 controller-template + onboard path
        ↓
   ┌────┴────┐
   ↓         ↓
L3 KubeReserve     L4 k8sbricks
   greenfield         retrofit
   └────┬────┘
        ↓
L5 OpenEnv if time
        ↓
L6 announce alpha
```

Do not invert this: do not push k8sbricks/kubereserve as the “we shipped
operators” story before L2. The differentiator is that the operators *are*
the pipeline examples.

## Explicitly not launch

- Hosted NL demo with a working Generate button.
- A throwaway MLflow ModelRegistry example. The operators are the examples.
- Helm/OCI/SBOM (post-alpha).
- `k8sbricks.io` as a marketed domain before it is registered (API group may
  already be locked; do not sell the website).
- Filing more issues on operator-chaos until `derive` attaches a real
  `knowledge.yaml`.
- Commenting on `opendatahub-io/operator-chaos#6`.

## Today vs launch

Today we can do L0 (docs + org profile). That is **not** launch.

Launch is L3 + L4. That is several days of tests and two honest worked
examples, not a same-day push of the current spikes.
