# operator-chaos outreach

**Status:** sent 2026-09-13. **Hold further comments.** Do not comment on
#6 and do not file further issues until `reconcilebench derive` attaches a
real `knowledge.yaml`. See `LAUNCH.md` L0.
**Issue:** https://github.com/opendatahub-io/operator-chaos/issues/6

**To:** maintainers of [`opendatahub-io/operator-chaos`](https://github.com/opendatahub-io/operator-chaos)
**From:** OpenReconcile (`github.com/openreconcile`)
**When:** after the survey is on `main`

## Proposed issue title

Complementary layers: status-contract oracle and source-derived knowledge models

## Proposed body

Hello — thank you for operator-chaos. The four modes and the knowledge-model /
verdict design are the maintained answer to “does this operator restore its
graph after an operator-semantic fault?” We do not intend to build another
injector.

We published a short survey of the controller-resilience testing landscape
(Sieve, Acto, Anvil, operator-chaos, envtest/chainsaw) at:

https://github.com/openreconcile/community/blob/main/docs/surveys/controller-resilience-testing-2026-09.md

Two layers look unclaimed and complementary to your project:

1. **Status-contract oracle.** Sieve and Acto both mask `observedGeneration` and
   `conditions` to control false positives. We think those fields *are* the
   Kubernetes user contract and want an oracle that checks them after your
   experiments run. Happy to emit findings in a form you could optionally
   consume.

2. **Source-derived knowledge models.** We plan a `derive` step that walks CRDs
   and controller source and writes `knowledge.yaml`. If the schema is stable
   enough to target, we would rather produce your format than invent a parallel
   one.

OpenReconcile’s ReconcileBench is explicitly an engine-agnostic scenario /
oracle / evidence layer. operator-chaos would be a first-class engine, not a
fork.

Questions:

- Is the `knowledge.yaml` schema considered stable enough for an external
  generator?
- Would a status-contract check as a post-experiment hook be in-scope for you,
  or better kept out-of-tree?
- Preferred channel for design discussion (issue, discussion, Slack/Google
  group)?

We will not publish third-party operator findings without a disclosure path.

Thanks for reading.
