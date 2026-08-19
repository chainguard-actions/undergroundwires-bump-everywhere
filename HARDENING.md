<!-- markdownlint-disable -->

# Hardening Report: undergroundwires--bump-everywhere/1.1.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **undergroundwires--bump-everywhere/1.1.2** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tags or branch names instead of pinned 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the referenced action is compromised or the tag is moved. Failing references: bump-and-release.yaml: `undergroundwires/bump-everywhere@master`; publish.yaml: `actions/checkout@v2`, `actions/setup-node@v1` (×2), `docker/build-push-action@v1`; quality-checks.yaml: `actions/checkout@v2`, `actions/setup-node@v1`; test.yaml: `actions/checkout@v2`.

Locations:

- `.github/workflows/bump-and-release.yaml:13`
- `.github/workflows/publish.yaml:11`
- `.github/workflows/publish.yaml:18`
- `.github/workflows/publish.yaml:27`
- `.github/workflows/publish.yaml:34`
- `.github/workflows/publish.yaml:40`
- `.github/workflows/quality-checks.yaml:11`
- `.github/workflows/quality-checks.yaml:28`
- `.github/workflows/quality-checks.yaml:30`
- `.github/workflows/test.yaml:10`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` block, and no individual job defines its own `permissions:` block. Without explicit permissions, workflows run with the default (potentially broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/bump-and-release.yaml:1`
- `.github/workflows/publish.yaml:1`
- `.github/workflows/quality-checks.yaml:1`
- `.github/workflows/test.yaml:1`

### script-injection (severity: high)

Direct expression interpolation inside `run:` shell commands. (a) In publish.yaml, `${{ github.event.release.tag_name }}` is interpolated directly into a shell command string in two separate steps: `run: git checkout "$(git rev-list "${{ github.event.release.tag_name }}"..master | tail -1)"`. An attacker who can control the release tag name could inject arbitrary shell commands. (b) In quality-checks.yaml, `${{ matrix.lint-command }}` is used as the entire `run:` value: `run: ${{ matrix.lint-command }}`. Although the matrix values are defined statically in the same file, any expression interpolated directly into a `run:` block is a script-injection risk per the check rules.

Locations:

- `.github/workflows/publish.yaml:15`
- `.github/workflows/publish.yaml:31`
- `.github/workflows/quality-checks.yaml:38`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all four workflow files: (1) Pinned all action references to full 40-char commit SHAs with tag comments preserved: undergroundwires/bump-everywhere@master→5b25459..., actions/checkout@v2→0717577..., actions/setup-node@v1→f1f314f..., docker/build-push-action@v1→3e7a4f6.... (2) Added `permissions: {}` top-level block to all four workflow files. (3) Fixed script injection in publish.yaml by moving `${{ github.event.release.tag_name }}` into env var TAG_NAME in both publish-npm and publish-docker jobs; fixed script injection in quality-checks.yaml by moving `${{ matrix.lint-command }}` into env var LINT_COMMAND and running `$LINT_COMMAND` in the shell.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell expansion in hardened/action/.github/workflows/quality-checks.yaml line 43. Changed `run: $LINT_COMMAND` to `run: "$LINT_COMMAND"` so the shell treats the value as a single quoted word, preventing metacharacter parsing and command injection from the workflow-controllable `matrix.lint-command` context value.

