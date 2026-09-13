# OpenReconcile — Roadmap and Next Steps

Sequenced work after Gate 1. Read `PROJECT-CONTEXT.md` first, `context/GATE1.md` for
the decision, and `adr/ADR-0001-engine-agnostic-architecture.md` for architecture.

Historical Milestone 0 text (F0 → F1 → pick A–F) is superseded. The argument is
preserved in `sources/` and `context/PRIOR_ART.md`.

---

## Repositories

| Repo | Contents | Status |
|---|---|---|
| `openreconcile/.github` | Org profile README, issue templates, code of conduct | create now |
| `openreconcile/community` | Context, conventions, Skills catalog | exists |
| `openreconcile/reconcilebench-fixture` | Widget controller: clean + eight defects | next |
| `openreconcile/reconcilebench` | Scenario, oracle, derive, evidence, MCP, CLI | after fixture skeleton |
| `openreconcile/controller-template` | Seeded from the proven fixture | after oracle |
| `openreconcile/k8sbricks` | Databricks Compute first | after API identity |
| `openreconcile/kubereserve` | Reservation lifecycle, one provider | after RFC; last |

No OpenEnv org repo. Upstream `KubernetesProvider` against `meta-pytorch/OpenEnv`.

Point `openreconcile.org` at the community repo’s docs.

**Launch is not “push everything.”** See [`process/LAUNCH.md`](process/LAUNCH.md).
Announce only after the KubeReserve greenfield example (L3) and the k8sbricks retrofit example (L4).

---

## Part A — Close falsification (this week)

- [x] Gate 1 record — `context/GATE1.md`
- [x] ADR-0001
- [ ] Public survey — `docs/surveys/controller-resilience-testing-2026-09.md`
- [ ] Org profile — `openreconcile/.github`
- [ ] Operator prior-art triage written into `docs/projects/`
- [ ] operator-chaos outreach draft

## Part B — Contribution pipeline

### Phase 1 — Fixture

```bash
mkdir reconcilebench-fixture && cd $_
kubebuilder init --domain fixture.openreconcile.org \
  --repo github.com/openreconcile/reconcilebench-fixture
kubebuilder create api --group testing --version v1alpha1 --kind Widget \
  --resource --controller
```

`fixture.openreconcile.org` is the one sanctioned org-name-in-API-group exception.

**Widget must exercise:** child Deployment, child ConfigMap, simulated asynchronous
external resource, status conditions and `observedGeneration`, a finalizer with
Delete / Orphan policy.

**Eight defect branches** (human-written bugs):

| Branch | Defect |
|---|---|
| `defect/duplicate-create` | Child created again after restart mid-create |
| `defect/finalizer-deadlock` | Finalization blocks forever when dependency is down |
| `defect/missing-observed-generation` | Status never updates `observedGeneration` |
| `defect/split-write` | External create succeeds, status write fails, ID is lost |
| `defect/stale-cache` | Acts on cached state that no longer reflects reality |
| `defect/retry-storm` | Retries a terminal error indefinitely |
| `defect/ownership-conflict` | Two concurrent reconciles corrupt shared state |
| `defect/unobserved-event` | Misses an event and never converges |

Consumable by operator-chaos, envtest, and chainsaw (deployment script + CRD +
knowledge stub).

### Phase 2 — Agent context and Skills

Portable rules plus project Skills: `new-operator`, `contribute`. `verify` and
`disclose` wait for Phase 6 and `DISCLOSURE.md`.

Hero prompt the `new-operator` Skill must handle:
“I have an MLflow model registry I want as a Kubernetes CR.”

### Phases 3–7

3. ReconcileBench core — scenario spec + engine adapters. No owned injector.
4. Status-contract oracle. Must catch the eight defects, especially
   `missing-observed-generation`.
5. `reconcilebench derive`.
6. Evidence format + MCP. Ship `verify` Skill.
6.5. `openreconcile new` CLI — after the template exists.
7. `controller-template` seeded from the fixture.

Do not advertise the NL slogan until Phase 4 passes the fixture.

## Part C — Operators, one at a time

Entry gate (`PROJECT-CONTEXT.md` §10): named maintainer + working e2e. At most two
in alpha. Archive after two untouched release cycles.

1. **k8sbricks Compute** — permanent API identity first. Generator spike against
   `databricks-sdk-go`, evaluate ACK code-generator before writing one.
2. **OpenEnv `KubernetesProvider`** — upstream, no org repo.
3. **KubeReserve** — API/lifecycle RFC first; one provider; TTL, budget, sweeper.
   Reconsider if no second contributor exists when operator 1 is in alpha.

## Part D — Hosted demo

After k8sbricks Compute exists: hosted NL demo. The public line becomes
*Build a Kubernetes operator in natural language* only after L3 has been
replayed by someone other than the author. k8sbricks is the AI/ML proof point.

## Milestone 1 — Conventions

Extract from implementation experience. Publish controller conventions, error
taxonomy, condition conventions. Do **not** freeze shared Go structs yet.

## Milestone 2 — External validation

Runs while building.

- **F2 Trust** — third-party operators under `DISCLOSURE.md`.
- **F3a Convention interest** — plain document; F3b only if demand appears.
- **F6 Economics** — TOC on fixture → k8sbricks → two third-party targets.

Then write Project Plan v3. It must be **narrower** than v2.

## Standing constraints

1. No public CRDs until k8sbricks API identity is resolved. Fixture domain excepted.
2. Compute-budget ceiling before a published scenario matrix.
3. Disclosure policy before F2.
4. Governance and archival policy before the second project repo.
5. Do not make a single personal account the only long-term org owner once a
   trusted co-maintainer exists.

## Documents still to write

- `CONVENTIONS.md` — Milestone 1
- `GOVERNANCE.md`, `MAINTAINERS.md`, `ARCHIVAL.md` — before the second project repo
- `DISCLOSURE.md` — before F2
- `SECURITY.md` — with the first public repo that accepts reports
- RFC/ADR templates — ADR-0001 now exists; keep a template beside it
