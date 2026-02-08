# AGENTS.md

Guidance for AI coding assistants working in `github.com/kubeedge/kubeedge`.

## Primary objective

- Make minimal, reviewable changes that solve the requested problem at root cause.
- Keep behavior and architecture consistent with existing cloud/edge split.
- Prefer existing scripts and targets over ad-hoc one-off command sequences.

## Repository shape (where to work)

- `cloud/`: cloud-side components and controllers (`cloudcore`, `admission`, `controllermanager`, `csidriver`, etc.).
- `edge/`: edge-side runtime components (`edgecore`, `edgemark`, modules, integration tests).
- `keadm/`: installer/bootstrap CLI.
- `pkg/`: shared logic consumed by multiple components.
- `common/`: shared constants and simple shared types.
- `staging/src/github.com/kubeedge/api/`: API types + generated client/lister/informer output.
- `staging/src/github.com/kubeedge/beehive/`, `staging/src/github.com/kubeedge/mapper-framework/`: staging repos included in workspace.
- `manifests/` and `build/crds/`: deploy assets and CRDs.
- `hack/`: build, lint, codegen, vendor, verify, local env scripts.
- `tests/`: e2e, keadm e2e, conformance orchestration.

## Toolchain and environment expectations

- Root module uses Go `1.23.12` (`go.mod` / `go.work`).
- Verification script enforces Go >= `1.21` (`hack/verify-golang.sh` via `hack/lib/golang.sh`).
- Default build path is containerized: `BUILD_WITH_CONTAINER=true` and `kubeedge/build-tools:1.23.12-ke1` (`hack/make-rules/build_with_container.sh`).
- Lint script expects GNU `sed` on macOS (`gsed`) and standard `sed` on Linux.
- Do not assume heavyweight test prerequisites (kind/docker/containerd/kubectl/sudo) are available unless explicitly checked.

## Canonical commands (CI-aligned)

Use Makefile targets first; they are what CI runs.

- Build:
  - `make` / `make all`
  - `make all WHAT=<binary>`
  - `make smallbuild`, `make crossbuild`
- Verification:
  - `make verify` (Go version, vendor, codegen, vendor licenses, CRDs)
  - `make lint`
  - `make spellcheck`
- Tests:
  - `make test`
  - `make test WHAT=cloud|edge|keadm|pkg`
  - `make integrationtest`
  - `make e2e` (heavy)
  - `make keadm_e2e`, `make keadm_compatibility_e2e`, `make keadm_deprecated_e2e` (heavy)

Note: `CONTRIBUTING.md` still mentions `make edge_test` / `make cloud_test`; current Makefile/test scripts use `make test WHAT=edge|cloud`.

## Validation playbook by change type

Run targeted checks first, then broaden only as needed.

- Go logic under `cloud/`, `edge/`, `keadm/`, `pkg/`:
  - Start with `make test WHAT=<component>`.
  - Then run `make verify` and `make lint` for full consistency.
- API or type changes under `staging/src/github.com/kubeedge/api/apis/`:
  - Run `hack/update-codegen.sh`.
  - Run `make verify` to confirm no codegen/CRD drift.
- CRD/schema/manifests touching CRDs:
  - Run `make generate` (or `hack/generate-crds.sh`).
  - Ensure `build/crds/` and `manifests/charts/cloudcore/crds/` are in sync.
  - Run `make verify`.
- DMI proto changes (`.../api/apis/dmi/.../api.proto`):
  - Run `make dmi-proto`.
  - Include generated updates (`api.pb.go` and related proto outputs).
- Dependency updates (`go.mod`, `go.sum`, `vendor/`, `go.work`, staging module deps):
  - Use `hack/update-vendor.sh` instead of only `go mod tidy`.
  - If licenses change, run `hack/update-vendor-licenses.sh`.
  - Confirm with `make verify`.

## Generated/derived files policy

- Never hand-edit files marked `Code generated ... DO NOT EDIT`.
- Regenerate from source inputs and include generated deltas in the same change.
- Common regeneration paths:
  - `hack/update-codegen.sh` -> `staging/src/github.com/kubeedge/api/client/...` and deepcopy outputs.
  - `hack/generate-crds.sh` / `make generate` -> `build/crds/...` and Helm CRD copies.
  - `hack/generate-dmi-proto.sh` / `make dmi-proto` -> DMI protobuf Go outputs.

## Heavy and destructive test notes

- `make e2e` / `tests/scripts/execute.sh` can:
  - create/delete kind clusters,
  - run privileged cleanup,
  - install tools,
  - kill local processes during cleanup.
- `make integrationtest` runs edge + cloud integration flows; cloud integration requires envtest binaries and CRDs.
- keadm e2e targets are expensive and environment-sensitive; run when the task touches keadm/bootstrap/e2e behavior or when explicitly requested.

## Lint and formatting behavior

- `make lint` runs `golangci-lint` with root config and also checks staging repos (`beehive`, `mapper-framework`).
- It also trims trailing whitespace on staged files relative to `master`; expect possible auto-edits.
- Always run `gofmt` on changed Go files (directly or via lint tooling).

## CI parity and known workflow quirks

- Main CI (`.github/workflows/main.yaml`) uses Go `1.23.x`, `make verify`, `make lint`, `make test PROFILE=y`, `make integrationtest`, and e2e matrices.
- Some workflows intentionally drift:
  - `main-arm64.yaml` still uses `kubeedge/build-tools:1.22.9-ke1`.
  - `cilium-e2e.yml` uses Go `1.22.x`.
- For local reproduction, prioritize matching the specific workflow being debugged.

## Change hygiene for assistants

Before editing:

1. Identify the smallest affected package(s) and corresponding tests.
2. Check whether the change touches generated artifacts, CRDs, or dependencies.
3. Prefer existing conventions and helper scripts over introducing new patterns.

Before handoff:

1. Report exactly what commands were run and what passed/failed.
2. Call out heavyweight checks not run (and why).
3. Keep unrelated file churn out of the diff.

## Commit and PR guidance

- Keep commits logically grouped.
- Follow `CONTRIBUTING.md` message style:
  - Subject: `<subsystem>: <what changed>` (<= 70 chars)
  - Blank second line
  - Body explains why
- PR template expects clear problem/solution text and issue linkage (`Fixes #...`) when applicable.
