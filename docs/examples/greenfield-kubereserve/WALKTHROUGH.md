# Greenfield example: KubeReserve

This is a recorded run of the **new-operator** path. Someone else should be
able to replay the commands. The handwritten stub controller was moved aside
and not consulted; the RFC and Skills were the input.

## Prompt

See [PROMPT.md](PROMPT.md):

> I need a Kubernetes CR for AWS capacity reservations with TTL, budget, and dry-run default true

API identity from [KUBERESERVE-RFC.md](../../projects/KUBERESERVE-RFC.md):
`capacity.kubereserve.io`, kind `Reservation`. Not `*.openreconcile.io`.

## Commands

From the workspace root, with `OPENRECONCILE_TEMPLATE` pointing at
`controller-template`:

```bash
cd reconcilebench
go run ./cmd/openreconcile new \
  "I need a Kubernetes CR for AWS capacity reservations with TTL, budget, and dry-run default true"

# In the generated tree (or the kubereserve repo, same API):
# implement Reservation from the RFC. dryRun defaults true. No AWS SDK.

go run ./cmd/reconcilebench derive --name kubereserve \
  --crds ../kubereserve/config/crd/bases

go run ./cmd/reconcilebench verify \
  --object ../community/docs/examples/greenfield-kubereserve/objects/pass.yaml

go run ./cmd/reconcilebench verify \
  --object ../community/docs/examples/greenfield-kubereserve/objects/fail-missing-og.yaml
```

`make test` in `kubereserve` covers envtest status contract plus:

- `ttl` missing or zero → `Ready=False`, `TTLRequired`, no cloud call
- `dryRun: true` → no cloud call, `expiresAt` derived
- `dryRun: false` → still refuses, no cloud call

## Evidence

| File | Result |
|---|---|
| [evidence/pass.json](evidence/pass.json) | `converged: true` — real `verify` output |
| [evidence/fail.json](evidence/fail.json) | `missing-observed-generation` on the labelled mutant |

The fail object is explicitly a mutant. The pass object is the dry-run
status the regenerated controller writes.

## Honesty

- Regenerated from the RFC + `new-operator` / `openreconcile new`.
- Karpenter consumes reservations. KubeReserve only manages lifecycle.
- No AWS credentials. Alpha does not spend.
- Do not call this resilient.
