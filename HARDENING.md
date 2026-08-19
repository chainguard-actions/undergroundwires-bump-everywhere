<!-- markdownlint-disable -->

# Hardening Report: undergroundwires--bump-everywhere/1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **undergroundwires--bump-everywhere/1.2.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tags or branch names instead of full 40-character commit SHAs. This exposes the workflow to supply-chain attacks where a tag or branch can be silently updated to point to malicious code.

Failing references:
- .github/workflows/bump-and-release.yaml: `undergroundwires/bump-everywhere@master`
- .github/workflows/publish.yaml: `actions/checkout@v2`, `actions/setup-node@v1`, `docker/build-push-action@v1`
- .github/workflows/quality-checks.yaml: `actions/checkout@v2`, `actions/setup-node@v1`
- .github/workflows/test.yaml: `actions/checkout@v2`

All should be pinned to their full SHA digest, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/bump-and-release.yaml:15`
- `.github/workflows/publish.yaml:12`
- `.github/workflows/publish.yaml:18`
- `.github/workflows/publish.yaml:27`
- `.github/workflows/publish.yaml:33`
- `.github/workflows/publish.yaml:40`
- `.github/workflows/quality-checks.yaml:11`
- `.github/workflows/quality-checks.yaml:16`
- `.github/workflows/test.yaml:11`

### missing-permissions (severity: medium)

None of the four workflow files define a top-level `permissions:` block, and no job within any of them defines job-level permissions. Without explicit permissions, workflows run with the default token permissions (which may be read/write depending on repository settings), violating the principle of least privilege.

Locations:

- `.github/workflows/bump-and-release.yaml:1`
- `.github/workflows/publish.yaml:1`
- `.github/workflows/quality-checks.yaml:1`
- `.github/workflows/test.yaml:1`

### script-injection (severity: high)

Two workflow files directly interpolate GitHub Actions expressions inside `run:` shell command strings, enabling script injection:

(a) `.github/workflows/publish.yaml` lines 16 and 31: `${{ github.event.release.tag_name }}` is interpolated directly inside a `run:` block — `run: git checkout "$(git rev-list "${{ github.event.release.tag_name }}"..master | tail -1)"`. An attacker who controls the release tag name could inject arbitrary shell commands.

(a) `.github/workflows/quality-checks.yaml` line 27: `run: ${{ matrix.lint-command }}` — the entire shell command is a matrix expression. Any workflow-controllable value in the matrix flows directly into the shell without quoting or sanitization.

Fix: move the values into `env:` variables and reference them as double-quoted shell variables, e.g.:
```yaml
env:
  TAG_NAME: ${{ github.event.release.tag_name }}
run: git checkout "$(git rev-list "$TAG_NAME"..master | tail -1)"
```

Locations:

- `.github/workflows/publish.yaml:16`
- `.github/workflows/publish.yaml:31`
- `.github/workflows/quality-checks.yaml:27`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all findings across 4 workflow files:

1. unpinned-uses: Pinned all 6 unique action references to full 40-char SHAs using lookup_action_sha: undergroundwires/bump-everywhere@master→5b25459..., actions/checkout@v2→0717577..., actions/setup-node@v1→f1f314f..., docker/build-push-action@v1→3e7a4f6...

2. missing-permissions: Added top-level `permissions: {}` to all 4 workflow files. Added job-level `permissions: contents: write` for bump-and-release (needs to push tags/releases) and `permissions: contents: read` for all other jobs.

3. script-injection: In publish.yaml, moved `${{ github.event.release.tag_name }}` into env var TAG_NAME in both publish-npm and publish-docker jobs. In quality-checks.yaml, moved `${{ matrix.lint-command }}` into env var LINT_COMMAND and referenced it as $LINT_COMMAND in the run step.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell expansion in `.github/workflows/quality-checks.yaml` line 43: changed `run: $LINT_COMMAND` to `run: "$LINT_COMMAND"`. This prevents shell metacharacter injection from the `matrix.lint-command` workflow-controllable context value.

