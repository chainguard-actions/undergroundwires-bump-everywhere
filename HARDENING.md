<!-- markdownlint-disable -->

# Hardening Report: undergroundwires--bump-everywhere/1.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **undergroundwires--bump-everywhere/1.4.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in run: blocks. In publish.yaml, two steps interpolate `${{ github.event.release.tag_name }}` directly inside shell command strings: `git checkout "$(git rev-list "${{ github.event.release.tag_name }}"..master | tail -1)"`. An attacker who can control the release tag name could inject arbitrary shell commands.

Locations:

- `.github/workflows/publish.yaml:14`
- `.github/workflows/publish.yaml:31`

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in run: block. In quality-checks.yaml, the step `run: ${{ matrix.lint-command }}` interpolates a matrix value directly as the entire shell command. Any workflow-controllable matrix value flows through YAML template substitution before the shell executes it, enabling script injection.

Locations:

- `.github/workflows/quality-checks.yaml:33`

### unpinned-uses (severity: high)

All uses: references across workflow files use mutable tag or branch refs instead of pinned 40-character SHA commit hashes, making the workflows vulnerable to supply-chain attacks if the referenced action is compromised or the tag is moved. Failing references include: `undergroundwires/bump-everywhere@master` (bump-and-release.yaml), `actions/checkout@v2` (publish.yaml, quality-checks.yaml, test.yaml), `actions/setup-node@v1` (publish.yaml, quality-checks.yaml), `docker/build-push-action@v1` (publish.yaml).

Locations:

- `.github/workflows/bump-and-release.yaml:13`
- `.github/workflows/publish.yaml:11`
- `.github/workflows/publish.yaml:17`
- `.github/workflows/publish.yaml:28`
- `.github/workflows/publish.yaml:34`
- `.github/workflows/publish.yaml:38`
- `.github/workflows/quality-checks.yaml:11`
- `.github/workflows/quality-checks.yaml:26`
- `.github/workflows/quality-checks.yaml:28`
- `.github/workflows/test.yaml:10`
- `.github/workflows/test.yaml:19`

### missing-permissions (severity: medium)

None of the four workflow files define a top-level `permissions:` block, and none of the individual jobs define job-level `permissions:` blocks. Without explicit permissions, workflows run with the default (potentially broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/bump-and-release.yaml:1`
- `.github/workflows/publish.yaml:1`
- `.github/workflows/quality-checks.yaml:1`
- `.github/workflows/test.yaml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four findings across four workflow files:

1. **script-injection (publish.yaml)**: Moved `${{ github.event.release.tag_name }}` into `env: TAG_NAME:` blocks in both publish-npm and publish-docker jobs. Shell commands now reference `"$TAG_NAME"` instead of the raw expression.

2. **script-injection (quality-checks.yaml)**: Changed matrix values from full `npm run <script>` commands to just the npm script names (e.g., `lint:yaml`). The Lint step now uses `env: LINT_SCRIPT: ${{ matrix.lint-command }}` and runs `npm run "$LINT_SCRIPT"`, preventing arbitrary shell command injection.

3. **unpinned-uses**: Pinned all action references to full 40-char SHAs with tag comments:
   - `undergroundwires/bump-everywhere@master` → `@5b25459619298d93b61472550ee5ef1230ecac93 # master`
   - `actions/checkout@v2` → `@0717577d45739eb3c851188b29f50ed6c0b2194e # v2`
   - `actions/setup-node@v1` → `@f1f314fca9dfce2769ece7d933488f076716723e # v1`
   - `docker/build-push-action@v1` → `@3e7a4f6646880c6f63758d73ac32392d323eaf8f # v1`

4. **missing-permissions**: Added `permissions: {}` at workflow top-level for all four files. Added job-level permissions: `contents: write` for bump-and-release (needs to push tags/releases), `contents: read` for all other jobs.

