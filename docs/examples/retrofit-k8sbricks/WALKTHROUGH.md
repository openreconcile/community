# Retrofit example: k8sbricks

This is a recorded run of the **onboard** path against the existing
handwritten k8sbricks spike. It does not claim the operator was generated
from English.

## What was already there

A Kubebuilder scaffold plus a stub controller that encoded
`existingClusterId` acknowledgement and the status contract, then stopped
at `SDKNotWired`.

## Commands

```bash
cd reconcilebench
go run ./cmd/openreconcile onboard ../k8sbricks

# Wrote k8sbricks/knowledge.yaml and k8sbricks/ONBOARD.md.
# Did not restructure the repo or change compute.k8sbricks.io.

# Then: add internal/databricks Clusters + recording Fake, wire the
# controller off SDKNotWired, write tests first against the naive client.

go test ./internal/controller -count=1   # from k8sbricks

go run ./cmd/reconcilebench verify \
  --object ../community/docs/examples/retrofit-k8sbricks/objects/pass.yaml

go run ./cmd/reconcilebench verify \
  --object ../community/docs/examples/retrofit-k8sbricks/objects/fail-missing-og.yaml
```

## Oracle result

The real happy-path object (externalID persisted, then `phase=RUNNING`,
`observedGeneration` set) **converged**. The pipeline confirmed clean on
this snapshot. It did not invent a bug.

The fail file is a labelled mutant (`observedGeneration` omitted).

## Tests

- Unacked `existingClusterId` → `Ready=False`, `ExistingClusterSemanticsUnacked`
- Happy path against the recording fake: `externalID` before `RUNNING`
- `observedGeneration == generation`
- Delete/finalizer calls `Clusters.Delete`

No Databricks credentials. Not affiliated with Databricks. Incumbent is
`glalanne/provider-databricks`. Compute-only alpha.
