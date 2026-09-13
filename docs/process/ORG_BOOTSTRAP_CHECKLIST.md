# OpenReconcile — GitHub Organisation Bootstrap Checklist

## Organisation

- [ ] Require 2FA for members.
- [ ] Add organisation description and `https://openreconcile.org`.
- [ ] Set a project-controlled contact email when available.
- [ ] Configure conservative Actions permissions.
- [ ] Enable secret scanning/security features available to public repos.
- [ ] Decide default repository visibility: public for open-source projects.
- [ ] Avoid making a single personal account the only long-term owner once a trusted co-maintainer exists.

## Initial repositories

Create first:

- [ ] `.github`
- [x] `community`
- [ ] `reconcilebench-fixture` — after the F0 spike
- [ ] `reconcilebench` — blocked until Gate 1 clarifies architecture

`controller-template` is **deferred** (decided 2026-09-13); its checklist below applies
whenever it is eventually created.

Stage production repos deliberately:

- [ ] `k8sbricks`
- [ ] `kubereserve`

Do not lock OpenEnv/SmolAgents repo names before their generic-abstraction RFCs.

## `.github`

Add:
- [ ] `profile/README.md`
- [ ] shared issue templates
- [ ] PR template
- [ ] security/support defaults
- [ ] reusable workflow templates

## `community`

Add:
- [ ] `GOVERNANCE.md`
- [ ] `MAINTAINERS.md`
- [ ] `CONVENTIONS.md`
- [ ] `SECURITY.md`
- [ ] `DISCLOSURE.md`
- [ ] RFC/ADR templates
- [ ] falsification/gate notes

## `controller-template`

Include:
- [ ] Kubebuilder baseline
- [ ] Makefile
- [ ] lint/test CI
- [ ] container/Helm scaffolding
- [ ] security workflow
- [ ] `.cursor/rules`
- [ ] RFC/ADR templates
- [ ] ReconcileBench test directory placeholder
- [ ] CODEOWNERS/CONTRIBUTING/SECURITY

## Branch/rulesets

Recommended baseline:
- [ ] protect default branch;
- [ ] require PR for non-trivial changes;
- [ ] require CI;
- [ ] prevent force-push on protected branch;
- [ ] require generated artefacts to be in sync.

Do not create governance theatre; keep the process small enough for the current maintainer count.
