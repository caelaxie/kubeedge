# AGENTS.md

Assistant guidance for changes under `cloud/`.

## Scope

- Applies to all files in `cloud/**`.
- Follow root `/AGENTS.md` first; this file adds cloud-specific rules.

## Cloud component map

- Entrypoints are under `cloud/cmd/`.
- Main cloud binaries in this repo include:
  - `cloudcore`
  - `admission`
  - `controllermanager`
  - `csidriver`
  - `iptablesmanager`
- Core implementation code is under `cloud/pkg/`.
- Integration tests are under `cloud/test/integration/`.

## How to validate cloud changes

Run from repository root:

- Fast path: `make test WHAT=cloud`
- Broader checks: `make verify` and `make lint`

When controller-runtime behavior, CRDs, or reconciliation logic changes:

- Run cloud integration flow: `cloud/test/integration/scripts/execute.sh`
- Or run full integration target: `make integrationtest`

## CRD and API coupling

- Cloud controllers depend on generated CRDs in `build/crds/`.
- If API schema changes affect cloud controllers, regenerate and verify:
  - `make generate`
  - `make verify`

## Editing boundaries

- Do not hand-edit generated artifacts in `staging/src/github.com/kubeedge/api/client/**`.
- Keep cloud-specific logic in `cloud/` packages; avoid unrelated cross-component refactors.
- If behavior changes are user-facing, ensure docs/manifests impacted by cloud behavior are updated in the same change when applicable.
