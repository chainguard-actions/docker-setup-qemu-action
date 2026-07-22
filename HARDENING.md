<!-- markdownlint-disable -->

# Hardening Report: docker--setup-qemu-action/v4.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **docker--setup-qemu-action/v4.2.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in ci.yml directly interpolate `${{ ... }}` expressions (sub-rule a). Any `${{ ... }}` expression inside a `run:` shell command is a script injection risk because the value is substituted by the YAML template engine before the shell processes it, allowing shell metacharacters to be injected.

1. Line 35 (`default` job, "Available platforms" step): `run: echo ${{ steps.qemu.outputs.platforms }}`
2. Line 61 (`main` job, "Available platforms" step): `run: echo ${{ steps.qemu.outputs.platforms }}`
3. Line 81 (`error` job, "Check" step): `echo "${{ toJson(steps.qemu) }}"`
4. Line 82 (`error` job, "Check" step): `if [ "${{ steps.qemu.outcome }}" != "failure" ] || [ "${{ steps.qemu.conclusion }}" != "success" ]; then`
5. Line 108 (`cache-image` job, "Available platforms" step): `run: echo ${{ steps.qemu.outputs.platforms }}`
6. Line ~130 (`reset` job, "Available platforms" step): `run: echo ${{ steps.qemu.outputs.platforms }}`

Fix: Move the values into `env:` variables and reference them as shell variables (e.g., `echo "$PLATFORMS"` with `env: PLATFORMS: ${{ steps.qemu.outputs.platforms }}`).

Locations:

- `.github/workflows/ci.yml:35`
- `.github/workflows/ci.yml:61`
- `.github/workflows/ci.yml:81`
- `.github/workflows/ci.yml:82`
- `.github/workflows/ci.yml:108`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all 5 script injection locations in .github/workflows/ci.yml by moving ${{ ... }} expressions out of run: shell commands and into step env: blocks. Specifically: (1) default job 'Available platforms': PLATFORMS env var; (2) main job 'Available platforms': PLATFORMS env var; (3) error job 'Check': QEMU_JSON, QEMU_OUTCOME, QEMU_CONCLUSION env vars replacing toJson(steps.qemu), steps.qemu.outcome, and steps.qemu.conclusion; (4) cache-image job 'Available platforms': PLATFORMS env var; (5) reset job 'Available platforms': PLATFORMS env var. Shell scripts now reference plain environment variables instead of template expressions.

