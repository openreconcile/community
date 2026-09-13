# OpenReconcile — community

Governance, conventions, and the durable project context for
[OpenReconcile](https://openreconcile.org).

OpenReconcile is an open, vendor-neutral engineering community for making software operable
on Kubernetes, with AI/ML infrastructure as the initial focus. The thesis: generating
infrastructure code is now cheap; determining the correct architecture, and proving the
result behaves correctly under failure, is not.

*Capabilities are declared. Resilience is demonstrated.*

> **Status: pre-implementation.** Nothing is built. The project is before its first decision
> gate, and its core differentiation claim is under active falsification. Nothing here should
> be read as a description of working software.

## Where to start

| If you are | Read |
|---|---|
| new to the project | `docs/PROJECT-CONTEXT.md` |
| picking up work | `docs/ROADMAP.md` |
| a coding agent | `AGENTS.md` |
| checking a differentiation claim | `docs/context/PRIOR_ART.md` |
| wondering why a decision was made | `docs/context/DECISIONS.md`, then `sources/` |

## Layout

```text
AGENTS.md                          engineering instructions + precedence order
docs/
  PROJECT-CONTEXT.md               background, locked decisions, rejected approaches
  ROADMAP.md                       sequenced work, milestones, operator queue
  context/
    CURRENT_STATE.md               consolidated latest state
    DECISIONS.md                   decision register + dated change log
    PRIOR_ART.md                   what already exists, and what that costs the thesis
    FALSIFICATION_EXECUTION.md     F0–F6 workstreams and gate model
    PROJECT_HISTORY.md             how the current direction emerged
    ENGINEERING_STANDARDS.md       repo shape, CI baseline, release artefacts
    REPO_TOPOLOGY.md               which repos exist and which must not yet
  projects/                        per-project context: reconcilebench, k8sbricks, …
  process/                         Cursor workflow, org bootstrap checklist
sources/                           historical documents, verbatim, superseded
.cursor/rules/                     agent rules, vendored into every OpenReconcile repo
```

## A note on `sources/`

This context set was consolidated in September 2026 from two independent write-ups of the
same planning history, plus the four-document argument chain that produced it — Project Plan
v2, the review response and falsification addendum, the strategy falsification review, and
its requested changes.

Those originals are kept verbatim because the *reasoning* behind a rejected approach is what
stops it being re-proposed. They contain superseded decisions and are the lowest-precedence
tier in `AGENTS.md`. Do not implement from them.

## Not yet written

Tracked honestly rather than created as empty files: `CONVENTIONS.md`, `GOVERNANCE.md`,
`MAINTAINERS.md`, `ARCHIVAL.md`, `DISCLOSURE.md`, `SECURITY.md`, RFC/ADR templates. Each has
a trigger recorded at the end of `docs/ROADMAP.md`.

## Non-affiliation

OpenReconcile is not affiliated with, endorsed by, or sponsored by Databricks, Amazon Web
Services, Hugging Face, the Cloud Native Computing Foundation, or the Linux Foundation.
Project and product names referenced in these documents are the marks of their respective
owners.
