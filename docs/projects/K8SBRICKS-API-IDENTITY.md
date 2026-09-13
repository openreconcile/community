# k8sbricks — API identity RFC (draft)

**Status:** Draft. Blocks `kubebuilder init`.
**Date:** 2026-09-13

## Decision to lock

| Item | Value |
|---|---|
| Cosmetic project name | `k8sbricks` |
| GitHub repo | `github.com/openreconcile/k8sbricks` (donatable; repo can move) |
| Public API groups | `compute.k8sbricks.io` first; later `serving.k8sbricks.io`, `jobs.k8sbricks.io`, … |
| Module path | `github.com/openreconcile/k8sbricks` until donation |
| OpenReconcile in API groups | **Forbidden** |

## Why this and not a provisional group

API groups are user-facing and permanent. A provisional brand in a permanent
group is the contradiction Gate 1 left open. Either `k8sbricks` is the identity
or we pick another word now. We pick `k8sbricks`.

## Domain

Register `k8sbricks.io` (or document a registrar failure and an alternate group
in this file) **before** the first public CRD. Until then, generated
`groupversion_info.go` must not land on `main`.

## Non-affiliation

README, CRD descriptions, and website must carry the community non-affiliation
notice. No Databricks marks, colours, or logos.

## Unlock criterion

This file’s table is copied into `K8SBRICKS.md` as locked **and** the domain is
registered (or an alternate is written). Then `kubebuilder init` is allowed.
