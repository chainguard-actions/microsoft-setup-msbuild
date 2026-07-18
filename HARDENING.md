<!-- markdownlint-disable -->

# Hardening Report: microsoft--setup-msbuild/v1.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **microsoft--setup-msbuild/v1.0.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow uses 'actions/checkout@v2', which is pinned to a mutable tag rather than an immutable 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit, enabling supply-chain attacks. It should be replaced with a full SHA pin, e.g. 'actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v2'.

Locations:

- `.github/workflows/test.yml:16`

### script-injection (severity: high)

Sub-rule (a): Three ${{ ... }} expressions from the 'steps.*.outputs.*' context are interpolated directly inside a 'run:' shell command string. GitHub Actions performs template substitution before the shell parses the command, so a malicious value in any of these outputs could inject arbitrary shell commands. Offending lines:
  echo "vswhere-path: ${{ steps.setup_msbuild_explicit.outputs.msbuildPath }}"
  echo "PATH: ${{ steps.setup_msbuild_path.outputs.msbuildPath }}"
  echo "Fallback: ${{ steps.setup_msbuild_fallback.outputs.msbuildPath }}"
Fix: move each value into an env: variable and reference it as a quoted shell variable, e.g. echo "vswhere-path: $MSBUILD_PATH".

Locations:

- `.github/workflows/test.yml:38`

### missing-permissions (severity: medium)

The workflow file '.github/workflows/test.yml' has no top-level 'permissions:' key and no job-level 'permissions:' key on any job. Without explicit permissions, the workflow inherits the repository's default token permissions (which may be 'write-all' for older repositories). A minimal explicit permissions block (e.g. 'permissions: {}' or 'permissions: contents: read') should be added.

Locations:

- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings in .github/workflows/test.yml: (1) Pinned actions/checkout@v2 to its full commit SHA ee0669bd1cc54295c223e0bb666b733df41de1c5 with a # v2 comment for readability. (2) Added a top-level `permissions: {}` block to restrict the GITHUB_TOKEN to no permissions by default. (3) Moved all three ${{ steps.*.outputs.msbuildPath }} template expressions out of the run: shell string and into an env: block (MSBUILD_PATH_EXPLICIT, MSBUILD_PATH_PATH, MSBUILD_PATH_FALLBACK), then referenced them as plain quoted shell variables in the run: script.

