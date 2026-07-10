<!-- markdownlint-disable -->

# Hardening Report: microsoft--setup-msbuild/v1.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **microsoft--setup-msbuild/v1.0.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow file uses actions/checkout@v2, which is pinned to a mutable tag rather than an immutable 40-character commit SHA. This allows supply-chain attacks if the tag is moved. Replace with a full SHA pin, e.g. actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v2.

Locations:

- `.github/workflows/test.yml:16`

### script-injection (severity: high)

Sub-rule (a): The 'echo msbuild path' step directly interpolates GitHub Actions expressions (${{ steps.setup_msbuild_explicit.outputs.msbuildPath }}, ${{ steps.setup_msbuild_path.outputs.msbuildPath }}, ${{ steps.setup_msbuild_fallback.outputs.msbuildPath }}) inside a run: shell command string. These values flow through YAML template substitution before the shell sees them, enabling script injection if the values contain shell metacharacters. Use environment variables instead: set the values in an env: block and reference them as quoted shell variables.

Locations:

- `.github/workflows/test.yml:34`

### missing-permissions (severity: medium)

The workflow file has no top-level permissions: key and the single job ('build') also has no job-level permissions: key. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially write) permissions. Add a top-level permissions: block with minimal required scopes (e.g. contents: read).

Locations:

- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings in .github/workflows/test.yml: (1) Pinned actions/checkout@v2 to full SHA ee0669bd1cc54295c223e0bb666b733df41de1c5; (2) Added top-level 'permissions: contents: read' block; (3) Moved all three ${{ steps.*.outputs.msbuildPath }} expressions out of the run: shell string into an env: block, referencing them as $env:MSBUILD_PATH_EXPLICIT, $env:MSBUILD_PATH_PATH, and $env:MSBUILD_PATH_FALLBACK in the PowerShell script (appropriate for the windows-latest runner).

