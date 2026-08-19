<!-- markdownlint-disable -->

# Hardening Report: microsoft--setup-msbuild/v3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **microsoft--setup-msbuild/v3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow uses `actions/checkout@v6`, which is pinned to a mutable tag (`v6`) rather than an immutable 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, enabling supply-chain attacks. It should be pinned to a full SHA, e.g. `actions/checkout@<40-char-sha> # v6`.

Locations:

- `.github/workflows/test.yml:24`

### script-injection (severity: high)

The 'Write MSBuild summary' step's `run:` block (PowerShell here-string) directly interpolates multiple `${{ }}` expressions inside the shell command string — violating sub-rule (a). The interpolated contexts include: `${{ matrix.runner }}` (line 59), `${{ runner.arch }}` (line 60), `${{ steps.setup_msbuild_explicit.outputs.msbuildPath || 'N/A' }}` (line 64), `${{ steps.setup_msbuild_path.outputs.msbuildPath || 'N/A' }}` (line 65), `${{ steps.setup_msbuild_fallback.outputs.msbuildPath || 'N/A' }}` (line 66), `${{ steps.setup_msbuild_path_x64.outputs.msbuildPath || 'N/A' }}` (line 67), and `${{ steps.setup_msbuild_path_arm64.outputs.msbuildPath || 'N/A' }}` (line 68). Any `${{ }}` expression interpolated directly into a `run:` block is evaluated by the template engine before the shell sees the string, allowing an attacker who can influence those values to inject arbitrary shell commands. These values should be passed via environment variables and referenced as `$ENV_VAR` (double-quoted) inside the script instead.

Locations:

- `.github/workflows/test.yml:55`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key, and the `build` job also has no job-level `permissions:` key. Without explicit permissions, the job inherits the repository's default token permissions, which may be overly broad (e.g. `write` access to contents). A minimal explicit `permissions:` block (e.g. `permissions: read-all` or specific scopes) should be added at the top level or on every job.

Locations:

- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings in .github/workflows/test.yml:
1. unpinned-uses: Pinned `actions/checkout@v6` to full SHA `d23441a48e516b6c34aea4fa41551a30e30af803` with `# v6` comment.
2. missing-permissions: Added top-level `permissions: contents: read` block (minimum needed for checkout-only workflow).
3. script-injection: Moved all 7 `${{ }}` expressions (matrix.runner, runner.arch, and 5 step outputs) from the PowerShell here-string `run:` block into the step's `env:` block, and replaced them with `$env:VAR_NAME` references in the PowerShell script.

