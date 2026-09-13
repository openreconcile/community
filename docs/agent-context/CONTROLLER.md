# Controller and API Conventions

Most operators in public training data violate several of these. Generate to this file, not
to the common pattern.

## Status and conditions

- Use `metav1.Condition` and `meta.SetStatusCondition`. Never a custom condition struct, and
  never a project-specific condition type without an approved RFC.
- Standard types: `Ready`, `Synced`, `Progressing`. Reasons are CamelCase and stable —
  they are API surface, not log text.
- **Always set `status.observedGeneration`** to `metadata.generation` when status is written.
  A status without it is indistinguishable from stale, and this is one of the defects
  ReconcileBench specifically hunts.
- Status must represent observed state for the current desired generation.
- Write status through the `/status` subresource, never as part of the object update.

## Never write to spec

Controllers write `status` only. Defaulting belongs in a mutating admission webhook or in
CRD schema defaults, not in reconciliation. A controller that mutates `spec` gives every
ArgoCD or Flux user an infinite sync loop.

## Error classification

Distinguish **terminal** from **recoverable** errors explicitly.

- Terminal: the remote will never accept this spec. Set a condition, emit an event, do not
  requeue. Returning an error here burns API quota in a hot loop forever.
- Recoverable: transient. Return the error and let the workqueue back off.
- Never `return ctrl.Result{Requeue: true}, err` — that double-requeues.
- Never swallow an error into a plain `ctrl.Result{}`.

## Idempotency and ownership

- Every create path must be safe to run twice. Assume the process dies between the external
  call and the status write, because ReconcileBench will make that happen.
- Set owner references for garbage collection where the child is cluster-local.
- For external resources, record the external ID in status **and** tag the remote resource
  with the owning object's UID, so orphans are identifiable after a status write is lost.
- Support adoption of pre-existing external resources where applicable.

## Finalizers

- Add the finalizer before the first external create, not after.
- Finalization must be able to complete when the external dependency is unavailable —
  either succeed, or surface a clear condition and remain removable by policy.
- Honour a `deletionPolicy` of `Delete` or `Orphan`.
- A finalizer that can deadlock is a defect, not a safety feature.
- Treat deletion as first-class reconciliation, not cleanup code appended later.

## Late initialization

When the remote system defaults a field the user left empty, record the server-side value
so subsequent diffs do not see phantom drift. Without this the controller fights the remote
API in a hot loop. This is mandatory for any controller managing an external service.

## External API discipline

- Rate-limit outbound calls with a shared limiter, and jitter requeues. Many CRs reconciling
  against one external API will hit quotas.
- Never call an external API from a webhook.
- Time out every outbound call and propagate context cancellation.

## API design

- Start at `v1alpha1`. Versions are per-group and advance independently.
- Group names: `<subcomponent>.<project>.io`. No org name in the group. Do not put all
  projects under an umbrella API group for branding.
- Public API groups must not be committed under a provisional domain or name.
- Namespace-scoped by default. Cluster-scoped requires justification — it breaks
  multi-tenancy.
- Credential and connection references resolve **same-namespace only** unless an explicit
  grant mechanism exists. Cross-namespace credential references are privilege escalation.
- Add `additionalPrinterColumns` for anything a user would want from `kubectl get` —
  at minimum a readiness column and the external ID.
- Do not promote a version without a conversion webhook written and tested. A plan is not a
  conversion webhook.
- Group version tracks the weakest resource in the group; carry real maturity signal in a
  per-resource matrix.
- Prefer explicit references and lifecycle semantics over hidden magic.

## Generated artefacts

Generated CRDs are generated artefacts. Do not hand-edit them. After Go API changes, run the
repository's generation and manifests targets.

## Testing

- envtest for controller logic; chainsaw for declarative end-to-end assertions.
- Every controller needs a test that kills and restarts the manager mid-reconcile.
- Do not test only the happy path. The failure paths are the point of this project.
