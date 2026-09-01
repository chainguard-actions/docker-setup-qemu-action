<!-- markdownlint-disable -->

# Hardening Report: docker--setup-qemu-action/v4.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **docker--setup-qemu-action/v4.3.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` steps in ci.yml directly interpolate `${{ ... }}` expressions inside shell command strings (sub-rule a). Although `steps.*` outputs are not directly attacker-controlled, ANY `${{ }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the template engine before the shell ever sees it, bypassing shell quoting. Affected steps:
- "Available platforms" (default job, line 34): `run: echo ${{ steps.qemu.outputs.platforms }}`
- "Available platforms" (main job, line 55): `run: echo ${{ steps.qemu.outputs.platforms }}`
- "Check" (error job, lines 71–72): `echo "${{ toJson(steps.qemu) }}"` and `if [ "${{ steps.qemu.outcome }}" != "failure" ] || [ "${{ steps.qemu.conclusion }}" != "success" ]`
- "Available platforms" (cache-image job, line 95): `run: echo ${{ steps.qemu.outputs.platforms }}`
- "Available platforms" (reset job, line 120): `run: echo ${{ steps.qemu.outputs.platforms }}`

Fix: move the values into `env:` variables and reference them as `"$ENV_VAR"` in the shell script.

Locations:

- `.github/workflows/ci.yml:34`
- `.github/workflows/ci.yml:55`
- `.github/workflows/ci.yml:71`
- `.github/workflows/ci.yml:95`
- `.github/workflows/ci.yml:120`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all 5 script injection instances in .github/workflows/ci.yml by moving ${{ }} expressions out of run: shell strings and into step env: blocks. Affected steps: 'Available platforms' in default, main, cache-image, and reset jobs (each using steps.qemu.outputs.platforms → $PLATFORMS), and 'Check' in the error job (toJson(steps.qemu) → $QEMU_JSON, steps.qemu.outcome → $QEMU_OUTCOME, steps.qemu.conclusion → $QEMU_CONCLUSION). All shell references now use properly double-quoted environment variables.

