---
name: new-operator
description: >-
  Scaffold a Kubernetes operator from an MLE's natural-language description of
  an existing ML tool. Use when the user wants to productionise MLflow, a
  training job, serving endpoint, feature store, eval harness, or similar as a
  custom resource.
---

# new-operator

You are helping an MLE productionise an **existing** ML tool on Kubernetes.
You are not inventing a platform.

## Collect before generating

1. What is the external system (API, lifecycle, identity)?
2. What is the desired CR kind and scope (namespaced default)?
3. How does auth work? Prefer OIDC / workload identity. Refuse PAT-in-a-Secret as default.
4. What does deletion mean (delete vs orphan the external object)?
5. **API group.** Must be donatable. Never `*.openreconcile.io`. Never invent a public group. If unset, stop and ask. Point at `docs/projects/K8SBRICKS-API-IDENTITY.md` as the pattern.

## Then do

1. Copy `controller-template` (or `openreconcile new "<prompt>"`).
2. Implement Go + Kubebuilder/controller-runtime only.
3. Status: `observedGeneration`, `metav1.Condition` (`Ready`), never write `spec`.
4. External create must survive "create succeeded, status write failed".
5. Finalizer must have a termination path.
6. Run `reconcilebench derive` and `reconcilebench verify` once those binaries exist.
7. Do not claim the operator is resilient.

## Refuse

- A new operator when KServe / KubeRay / Kubeflow / Crossplane already solves it.
- An environment-pool operator (OpenSandbox exists; OpenEnv is upstream-only).
- Inlining source in a CR spec field.
- Python/Kopf control loops.
