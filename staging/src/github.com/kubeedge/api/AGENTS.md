# AGENTS.md

Assistant guidance for changes under `staging/src/github.com/kubeedge/api/`.

## Scope

- Applies to all files in `staging/src/github.com/kubeedge/api/**`.
- Follow root `/AGENTS.md` first; this file adds staging API-specific rules.

## What lives here

- Source API types under `apis/**`.
- Generated clients/listers/informers under `client/**`.
- Additional generated artifacts (e.g., openapi/proto outputs) in subtrees.

## Hard rules for generated files

- Never hand-edit files marked `Code generated ... DO NOT EDIT`.
- Update source types/protos first, then regenerate.
- Include regenerated output in the same change that modifies source definitions.

## Required regeneration flows

When touching API types under `apis/**`:

- Run `hack/update-codegen.sh` from repository root.
- Then run `make verify` (or at minimum `hack/verify-codegen.sh`) to confirm no drift.

When touching CRD-related schema fields:

- Run `make generate` (or `hack/generate-crds.sh`).
- Confirm generated CRDs are updated in both:
  - `build/crds/**`
  - `manifests/charts/cloudcore/crds/**`

When touching DMI proto (`apis/dmi/**/api.proto`):

- Run `make dmi-proto`.
- Commit updated protobuf-generated outputs.

## Validation expectations

- Run targeted tests for impacted consumers (`make test WHAT=cloud|edge|pkg` as relevant).
- Always run `make verify` before handoff for API-generation sensitive changes.

## Dependency/workspace notes

- This module is used via `go.work` and `replace` directives from the repo root.
- Avoid changes that desynchronize staging module expectations from root `go.mod`/`go.work`.
