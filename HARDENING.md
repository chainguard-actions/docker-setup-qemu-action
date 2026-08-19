<!-- markdownlint-disable -->

# Hardening Report: docker--setup-qemu-action/v4.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **docker--setup-qemu-action/v4.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in ci.yml directly interpolate GitHub Actions expressions into shell commands (rule a). The expressions ${{ steps.qemu.outputs.platforms }}, ${{ toJson(steps.qemu) }}, ${{ steps.qemu.outcome }}, and ${{ steps.qemu.conclusion }} are substituted by the YAML template engine before the shell ever sees them, allowing injection of shell metacharacters. Offending lines: (1) 'run: echo ${{ steps.qemu.outputs.platforms }}' in the 'default' job; (2) 'run: echo ${{ steps.qemu.outputs.platforms }}' in the 'main' job; (3) 'echo "${{ toJson(steps.qemu) }}"' and 'if [ "${{ steps.qemu.outcome }}" != "failure" ] || [ "${{ steps.qemu.conclusion }}" != "success" ]' in the 'error' job; (4) 'run: echo ${{ steps.qemu.outputs.platforms }}' in the 'cache-image' job. These should be moved to env: variables and the env vars double-quoted in the shell script.

Locations:

- `.github/workflows/ci.yml:28`
- `.github/workflows/ci.yml:50`
- `.github/workflows/ci.yml:66`
- `.github/workflows/ci.yml:87`

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved. Failing references: ci.yml — actions/checkout@v6 (×5); test.yml — actions/checkout@v6, docker/bake-action@v6, codecov/codecov-action@v5; publish.yml — actions/checkout@v6, actions/publish-immutable-action@v0.0.4; update-dist.yml — actions/create-github-app-token@v2, actions/checkout@v6, docker/bake-action@v6; validate.yml — actions/checkout@v6, docker/bake-action/subaction/list-targets@v6, docker/bake-action@v6. Each should be pinned to a full SHA digest with the tag as a comment.

Locations:

- `.github/workflows/ci.yml:23`
- `.github/workflows/test.yml:17`
- `.github/workflows/publish.yml:14`
- `.github/workflows/update-dist.yml:10`
- `.github/workflows/validate.yml:13`

### missing-permissions (severity: medium)

Four workflow files have no top-level permissions: block and no job-level permissions: blocks on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) permissions. Each workflow should declare minimal required permissions at the top level or per job. Affected files: ci.yml (jobs: default, main, error, cache-image, version — none have permissions); test.yml (job: test — no permissions); update-dist.yml (job: update-dist — no permissions); validate.yml (jobs: prepare, validate — none have permissions).

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/update-dist.yml:1`
- `.github/workflows/validate.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across five workflow files:

1. script-injection (ci.yml): Moved all ${{ steps.qemu.* }} expressions to env: blocks in the four affected run: steps (default, main, error, cache-image jobs). Variables PLATFORMS, QEMU_JSON, QEMU_OUTCOME, QEMU_CONCLUSION are now referenced as shell env vars.

2. unpinned-uses: Pinned all action references to full 40-char SHAs with tag comments:
   - actions/checkout@v6 → @d23441a48e516b6c34aea4fa41551a30e30af803
   - docker/bake-action@v6 → @5be5f02ff8819ecd3092ea6b2e6261c31774f2b4
   - codecov/codecov-action@v5 → @0fb7174895f61a3b6b78fc075e0cd60383518dac
   - actions/publish-immutable-action@v0.0.4 → @4bc8754ffc40f27910afb20287dbbbb675a4e978
   - actions/create-github-app-token@v2 → @fee1f7d63c2ff003460e3d139729b119787bc349
   - docker/bake-action/subaction/list-targets@v6 → @5be5f02ff8819ecd3092ea6b2e6261c31774f2b4

3. missing-permissions: Added top-level `permissions: {}` to ci.yml, test.yml, update-dist.yml, validate.yml, and publish.yml. Added job-level minimal permissions (contents: read for most jobs, contents: write for update-dist which pushes commits).

