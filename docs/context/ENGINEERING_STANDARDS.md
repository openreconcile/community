# OpenReconcile — Engineering Standards

## Language/runtime

Production Kubernetes controllers:
- Go
- Kubebuilder
- controller-runtime

Python may be used for client SDKs, analysis, or tooling, but should not become the default controller runtime without an approved architectural reason.

## Common repository shape

```text
project/
├── .cursor/rules/
├── api/
├── cmd/
├── internal/
│   ├── controller/
│   ├── provider/
│   ├── admission/
│   └── conditions/
├── config/
├── charts/
├── test/
│   ├── envtest/
│   ├── e2e/
│   └── reconcilebench/
├── docs/
├── rfcs/
├── hack/
├── .github/workflows/
├── Dockerfile
├── Makefile
├── PROJECT
├── go.mod
├── README.md
├── SECURITY.md
├── CONTRIBUTING.md
└── CODEOWNERS
```

Not every directory is mandatory on first commit; keep the structure consistent as features appear.

## CI baseline

Target baseline:
- formatting;
- generated-file check;
- `go vet`;
- lint;
- unit tests;
- envtest;
- kind e2e;
- Helm validation;
- container build;
- vulnerability scanning;
- CodeQL/static analysis as appropriate.

Expensive ReconcileBench matrices should normally run nightly/on-demand until costs justify broader PR coverage.

## Releases

Target public-release artefacts:
- container image;
- Helm OCI chart;
- SBOM;
- signature;
- provenance/verification instructions.

## Documentation

Durable design decisions belong in repository documentation/RFCs, not only Cursor chat history.
