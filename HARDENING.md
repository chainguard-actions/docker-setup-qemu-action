<!-- markdownlint-disable -->

# Hardening Report: docker--setup-qemu-action/v3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **docker--setup-qemu-action/v3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): ${{ }} expressions are interpolated directly inside run: shell commands. In the 'default', 'main', and 'cache-image' jobs: `run: echo ${{ steps.qemu.outputs.platforms }}` — the step output is injected into the shell command string before the shell sees it. In the 'error' job 'Check' step: `echo "${{ toJson(steps.qemu) }}"` and `if [ "${{ steps.qemu.outcome }}" != "failure" ] || [ "${{ steps.qemu.conclusion }}" != "success" ]` — all these expressions flow through YAML template substitution directly into the shell.

Locations:

- `.github/workflows/ci.yml:30`
- `.github/workflows/ci.yml:52`
- `.github/workflows/ci.yml:68`
- `.github/workflows/ci.yml:89`

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags/versions instead of full 40-character SHA commit hashes. Failing references: ci.yml — actions/checkout@v5 (×5); test.yml — actions/checkout@v5, docker/bake-action@v6, codecov/codecov-action@v5; validate.yml — actions/checkout@v5, docker/bake-action/subaction/list-targets@v6, docker/bake-action@v6; publish.yml — actions/checkout@v5, actions/publish-immutable-action@v0.0.4.

Locations:

- `.github/workflows/ci.yml:24`
- `.github/workflows/test.yml:18`
- `.github/workflows/test.yml:22`
- `.github/workflows/test.yml:26`
- `.github/workflows/validate.yml:18`
- `.github/workflows/validate.yml:23`
- `.github/workflows/validate.yml:38`
- `.github/workflows/publish.yml:14`
- `.github/workflows/publish.yml:16`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` block and no job-level `permissions:` blocks on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially write) permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/validate.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across four workflow files:

1. script-injection (ci.yml): Moved all ${{ }} expressions from run: shell commands into env: blocks. The 'default', 'main', and 'cache-image' jobs' 'Available platforms' steps now use PLATFORMS env var. The 'error' job 'Check' step now uses QEMU_JSON, QEMU_OUTCOME, and QEMU_CONCLUSION env vars.

2. unpinned-uses: Pinned all mutable tag references to full 40-char SHAs with tag comments: actions/checkout@v5 → @93cb6efe18208431cddfb8368fd83d5badbf9bfd, docker/bake-action@v6 → @5be5f02ff8819ecd3092ea6b2e6261c31774f2b4, codecov/codecov-action@v5 → @0fb7174895f61a3b6b78fc075e0cd60383518dac, actions/publish-immutable-action@v0.0.4 → @4bc8754ffc40f27910afb20287dbbbb675a4e978.

3. missing-permissions: Added top-level 'permissions: contents: read' block to ci.yml, test.yml, and validate.yml. publish.yml already had job-level permissions.

