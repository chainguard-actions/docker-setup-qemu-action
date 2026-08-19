<!-- markdownlint-disable -->

# Hardening Report: docker--setup-qemu-action/v4.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **docker--setup-qemu-action/v4.1.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in ci.yml directly interpolate GitHub Actions expressions into shell commands (rule a). The expressions ${{ steps.qemu.outputs.platforms }}, ${{ toJson(steps.qemu) }}, ${{ steps.qemu.outcome }}, and ${{ steps.qemu.conclusion }} are substituted by the YAML template engine before the shell executes the command, allowing an attacker who can influence step outputs to inject arbitrary shell commands. Affected steps: 'Available platforms' in jobs default, main, cache-image, and reset (run: echo ${{ steps.qemu.outputs.platforms }}), and 'Check' in job error (echo "${{ toJson(steps.qemu) }}" and if [ "${{ steps.qemu.outcome }}" ... || [ "${{ steps.qemu.conclusion }}" ...]). These should be moved to env: variables and referenced as quoted shell variables (e.g., echo "$PLATFORMS").

Locations:

- `.github/workflows/ci.yml:30`
- `.github/workflows/ci.yml:52`
- `.github/workflows/ci.yml:66`
- `.github/workflows/ci.yml:67`
- `.github/workflows/ci.yml:88`
- `.github/workflows/ci.yml:113`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all 6 script injection locations in .github/workflows/ci.yml:
1. jobs.default 'Available platforms': moved `${{ steps.qemu.outputs.platforms }}` to env PLATFORMS, changed run to `echo "$PLATFORMS"`
2. jobs.main 'Available platforms': same fix as above
3. jobs.error 'Check': moved `${{ toJson(steps.qemu) }}`, `${{ steps.qemu.outcome }}`, and `${{ steps.qemu.conclusion }}` to env vars QEMU_JSON, QEMU_OUTCOME, QEMU_CONCLUSION; updated shell script to reference these env vars
4. jobs.cache-image 'Available platforms': same fix as #1
5. jobs.reset 'Available platforms': same fix as #1
All expressions are now safely passed through the env: block and referenced as quoted shell variables, preventing shell injection attacks.

