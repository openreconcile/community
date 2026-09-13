# Historical sources — superseded, preserved verbatim

**Do not implement from these files.** They are the lowest-precedence tier in `AGENTS.md`.

They are kept because the *reasoning* behind a rejected approach is what stops it being
re-proposed. When a document here conflicts with `docs/`, `docs/` wins.

## The argument chain

Read in this order to follow how the current direction emerged:

1. **`OpenReconcile-Project-Plan-v2.md`** — the plan that froze as the working hypothesis.
   Two tracks, seven v0.1 scenarios, the conformance-as-participation community model.
2. **`OpenReconcile-Review-Response-and-Strategy-Falsification.md`** — the response that
   accepted the clean-control branch, replaced write quiescence with semantic convergence,
   proposed API-server-boundary interception, surfaced the k8sbricks name/API contradiction,
   and concluded that v3 must not be written until the thesis survives falsification.
3. **`OpenReconcile-Strategy-Falsification-Review.md`** — the falsification protocol. Outcome
   labels, F1–F5, the gate model, and the explicit statement of what would make the review
   itself a failure.
4. **`OpenReconcile-Strategy-Falsification-Review-Requested-Changes.md`** — two-dimensional
   F1 verdicting, the trust ladder, F5 split into technical scope and market wedge, and the
   addition of F6 with Target Onboarding Cost.

## Consolidation inputs

- `Claude-PROJECT-CONTEXT-original.md`, `Claude-ROADMAP-original.md` — one of the two
  parallel write-ups consolidated in September 2026.
- `OpenReconcile-Full-Context-Export.md` — the other write-up's consolidated export.

## Known superseded content

Specific things in these files that are no longer policy:

- **"No ArtifactHub. Decided."** — now deferred and reversible.
- **The frozen `openreconcile.io/apis/meta/v1alpha1` module** — dead. Shared Go structs are
  extracted only after two production controllers prove the shape. It also embedded the org
  name in an API path, which the donatability rule now forbids.
- **Convergence as "no further writes occur"** — replaced by semantic convergence.
- **F1's purely numeric verdict thresholds** (≤2 VALIDATED, 3–5 WEAKENED, ≥6 REFRAMED) —
  replaced by coverage quantity plus defect-class significance.
- **"Capability levels are declared. We measure them."** — replaced by "Capabilities are
  declared. Resilience is demonstrated."
- **"None of these inject faults during reconciliation"** and the prior-art list behind it —
  false as stated. See `../docs/context/PRIOR_ART.md`.
- **`controller-template` in the first batch of repos** — deferred.
- **Seven seeded defect branches** — now eight, with `defect/unobserved-event` added.
