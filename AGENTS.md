# Agent guidance

This repository publishes the Agent Client Protocol registry. Each top-level
agent directory contains an `agent.json` manifest and a 16 by 16 monochrome
`icon.svg`. Treat `agent.schema.json` and the validation scripts as the
authoritative format contract.

## Validate changes

Run the narrow checks first, then the complete workflow suite:

```sh
uv run --with jsonschema .github/workflows/build_registry.py --dry-run
cd .github/workflows
uv run --with ruff ruff check .
uv run --with ruff ruff format --check .
uv run --with pytest pytest tests/ -v
```

The Docker wrapper reproduces CI dependencies and architecture more closely:

```sh
.github/workflows/scripts/run-workflows-tests.sh
.github/workflows/scripts/run-registry-docker.sh \
  uv run --with jsonschema .github/workflows/build_registry.py
```

Use `.github/workflows/scripts/run-protocol-matrix.sh` only when agent runtime
or authentication behavior changes. It creates isolated state by default.
Set `ACP_PROTOCOL_MATRIX_KEEP_STATE=1` only for intentional debugging.

## Update versions

Use the updater instead of editing distribution URLs independently:

```sh
uv run .github/workflows/update_versions.py --agents harn
uv run .github/workflows/update_versions.py --apply --agents harn
uv run --with jsonschema .github/workflows/build_registry.py --dry-run
```

The scheduled workflow checks releases hourly. GitHub binary updates require a
GitHub `repository` URL; npm and PyPI packages use their registries. Entries in
`quarantine.json` are intentionally excluded until their recorded failure is
resolved.

## Registry rules

- Directory names and manifest `id` values use lowercase letters, digits, and
  hyphens and must match.
- `version` uses semantic versioning.
- Every manifest provides at least one `binary`, `npx`, or `uvx`
  distribution.
- Binary archives use a supported archive format or a raw executable. The
  validator rejects installers such as DMG, PKG, DEB, RPM, MSI, and AppImage.
- Icons are 16 by 16 SVGs using `currentColor`; do not add fixed colors.
- Agents must return at least one `agent` or `terminal` authentication method
  during ACP initialization. See [AUTHENTICATION.md](./AUTHENTICATION.md).

Set `SKIP_URL_VALIDATION=1` only for an offline diagnostic run. Do not use it
as evidence that a registry change is ready to merge.

## Publication

Pull requests validate the registry. Changes on `main` run
`.github/workflows/build-registry.yml`, which publishes versioned and `latest`
registry releases. The version updater stages only expected `agent.json`
files, verifies them, pushes the commit, and explicitly dispatches the build
workflow.
