# AGENTS.md

Assistant guidance for changes under `keadm/`.

## Scope

- Applies to all files in `keadm/**`.
- Follow root `/AGENTS.md` first; this file adds keadm-specific rules.

## Keadm component map

- CLI entrypoint is `keadm/cmd/keadm`.
- Keadm-related logic and tests live under `keadm/` packages.
- End-to-end keadm scenarios are orchestrated from `tests/scripts/keadm_*.sh` and suites in `tests/e2e_keadm/`.

## How to validate keadm changes

Run from repository root:

- Fast path: `make test WHAT=keadm`
- Broader checks: `make verify` and `make lint`

For bootstrap/join/upgrade workflow changes, run relevant heavy suites:

- `make keadm_e2e`
- `make keadm_compatibility_e2e`
- `make keadm_deprecated_e2e`

Only run heavy e2e targets when needed or requested; they require substantial environment setup.

## Safety and behavior notes

- Keadm e2e scripts create kind clusters, load images, and perform install/join flows; they are long-running and environment-sensitive.
- Keep platform/runtime-specific behavior explicit and avoid hidden side effects in CLI paths.

## Editing boundaries

- Keep keadm concerns isolated from unrelated cloud/edge runtime refactors unless the task explicitly spans components.
- When changing user-facing flags or command behavior, ensure help text/docs in scope are updated consistently.
