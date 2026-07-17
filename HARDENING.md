<!-- markdownlint-disable -->

# Hardening Report: docker--setup-qemu-action/v3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **docker--setup-qemu-action/v3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple `run:` blocks in ci.yml directly interpolate GitHub Actions expressions inside shell commands. The expressions `${{ steps.qemu.outputs.platforms }}`, `${{ toJson(steps.qemu) }}`, `${{ steps.qemu.outcome }}`, and `${{ steps.qemu.conclusion }}` are all `steps.*.outputs.*` context values that flow through YAML template substitution before the shell sees them, enabling script injection. Offending lines: `run: echo ${{ steps.qemu.outputs.platforms }}` (lines 31, 55, 90) and `run: | echo "${{ toJson(steps.qemu) }}" / if [ "${{ steps.qemu.outcome }}" ... ] || [ "${{ steps.qemu.conclusion }}" ... ]` (lines 73–78).

Locations:

- `.github/workflows/ci.yml:31`
- `.github/workflows/ci.yml:55`
- `.github/workflows/ci.yml:73`
- `.github/workflows/ci.yml:90`

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable tags instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks. Unpinned references found:
- ci.yml: `actions/checkout@v5` (appears 5 times)
- test.yml: `actions/checkout@v5`, `docker/bake-action@v6`, `codecov/codecov-action@v5`
- validate.yml: `actions/checkout@v5`, `docker/bake-action/subaction/list-targets@v6`, `docker/bake-action@v6`
- publish.yml: `actions/checkout@v5`, `actions/publish-immutable-action@v0.0.4`

Locations:

- `.github/workflows/ci.yml:25`
- `.github/workflows/test.yml:17`
- `.github/workflows/test.yml:21`
- `.github/workflows/test.yml:25`
- `.github/workflows/validate.yml:19`
- `.github/workflows/validate.yml:24`
- `.github/workflows/validate.yml:38`
- `.github/workflows/publish.yml:14`
- `.github/workflows/publish.yml:17`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` key and no job-level `permissions:` keys on any of their jobs. Without explicit permissions, workflows inherit the default repository token permissions (which may be broad), violating the principle of least privilege.
- ci.yml: no permissions at top level or on any of its 5 jobs (default, main, error, cache-image, version)
- test.yml: no permissions at top level or on its test job
- validate.yml: no permissions at top level or on its prepare/validate jobs

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/validate.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across four workflow files:

1. **script-injection** (ci.yml): Moved all `${{ steps.qemu.outputs.platforms }}`, `${{ toJson(steps.qemu) }}`, `${{ steps.qemu.outcome }}`, and `${{ steps.qemu.conclusion }}` expressions out of `run:` blocks into `env:` blocks (as PLATFORMS, QEMU_JSON, QEMU_OUTCOME, QEMU_CONCLUSION), then referenced them as plain shell variables.

2. **unpinned-uses**: Pinned all mutable tag references to full 40-char SHAs with tag comments:
   - actions/checkout@v5 → @93cb6efe18208431cddfb8368fd83d5badbf9bfd # v5
   - docker/bake-action@v6 → @5be5f02ff8819ecd3092ea6b2e6261c31774f2b4 # v6
   - codecov/codecov-action@v5 → @0fb7174895f61a3b6b78fc075e0cd60383518dac # v5
   - actions/publish-immutable-action@v0.0.4 → @4bc8754ffc40f27910afb20287dbbbb675a4e978 # v0.0.4

3. **missing-permissions**: Added `permissions: contents: read` at the top level of ci.yml, test.yml, and validate.yml. publish.yml already had appropriate job-level permissions.

