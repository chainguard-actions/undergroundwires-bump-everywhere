<!-- markdownlint-disable -->

# Hardening Report: undergroundwires--bump-everywhere/1.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **undergroundwires--bump-everywhere/1.3.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references in workflow files use mutable tags or branch names instead of pinned 40-character SHA commit hashes, making the workflows vulnerable to supply-chain attacks if the referenced action is compromised or its tag is moved.

Failing references:
- `.github/workflows/bump-and-release.yaml`: `uses: undergroundwires/bump-everywhere@master`
- `.github/workflows/publish.yaml`: `uses: actions/checkout@v2` (×2), `uses: actions/setup-node@v1` (×2), `uses: docker/build-push-action@v1`
- `.github/workflows/quality-checks.yaml`: `uses: actions/checkout@v2`, `uses: actions/setup-node@v1`
- `.github/workflows/test.yaml`: `uses: actions/checkout@v2` (×2)

Locations:

- `.github/workflows/bump-and-release.yaml:16`
- `.github/workflows/publish.yaml:12`
- `.github/workflows/publish.yaml:18`
- `.github/workflows/publish.yaml:27`
- `.github/workflows/publish.yaml:33`
- `.github/workflows/publish.yaml:37`
- `.github/workflows/quality-checks.yaml:10`
- `.github/workflows/quality-checks.yaml:30`
- `.github/workflows/quality-checks.yaml:32`
- `.github/workflows/test.yaml:10`
- `.github/workflows/test.yaml:20`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` block, and no individual job defines a `permissions:` block. Without explicit permissions, workflows run with the default (potentially broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/bump-and-release.yaml:1`
- `.github/workflows/publish.yaml:1`
- `.github/workflows/quality-checks.yaml:1`
- `.github/workflows/test.yaml:1`

### script-injection (severity: high)

Rule (a): GitHub Actions expressions are interpolated directly inside `run:` shell command strings, allowing an attacker to inject arbitrary shell commands.

1. `publish.yaml` (publish-npm job, 'Checkout to bump commit' step): `run: git checkout "$(git rev-list "${{ github.event.release.tag_name }}"..master | tail -1)"` — the release tag name is controlled by whoever creates the release and is injected directly into the shell command.

2. `publish.yaml` (publish-docker job, 'Checkout to bump commit' step): same pattern — `run: git checkout "$(git rev-list "${{ github.event.release.tag_name }}"..master | tail -1)"`.

3. `quality-checks.yaml` ('Lint' step): `run: ${{ matrix.lint-command }}` — the entire run command is sourced from the matrix context, which is workflow-controllable and injected directly into the shell.

Locations:

- `.github/workflows/publish.yaml:16`
- `.github/workflows/publish.yaml:31`
- `.github/workflows/quality-checks.yaml:38`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all four workflow files:

1. **unpinned-uses**: Pinned all `uses:` references to full 40-char SHAs with original tags as comments: `actions/checkout@v2` → SHA `0717577d...`, `actions/setup-node@v1` → SHA `f1f314fc...`, `docker/build-push-action@v1` → SHA `3e7a4f66...`, `undergroundwires/bump-everywhere@master` → SHA `5b254596...`.

2. **missing-permissions**: Added `permissions: {}` at the top level of all four workflow files. Added minimal job-level permissions: `contents: write` for the bump-and-release job (needs to create releases), `contents: read` for all other jobs.

3. **script-injection**: (a) In `publish.yaml`, moved `${{ github.event.release.tag_name }}` into an `env:` block as `TAG_NAME` and referenced it as `"$TAG_NAME"` in the shell command (both publish-npm and publish-docker jobs). (b) In `quality-checks.yaml`, restructured the matrix to contain just npm script names (e.g., `lint:yaml`) instead of full commands, moved `${{ matrix.lint-command }}` into an `env:` block as `LINT_COMMAND`, and changed the run step to `npm run "$LINT_COMMAND"` to prevent arbitrary shell command injection.

