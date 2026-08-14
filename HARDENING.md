<!-- markdownlint-disable -->

# Hardening Report: actionhippie--swap-space/v1.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actionhippie--swap-space/v1.0.1** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `run:` steps in action.yml directly interpolate GitHub Actions expressions inside shell commands. `${{ inputs.path }}` (line 36) is passed unquoted to `sudo mktemp`, and `${{ inputs.size }}` (line 41) is passed unquoted to `sudo fallocate`. Because these values are substituted by the YAML template engine before the shell ever sees them, a caller can inject arbitrary shell metacharacters (e.g. a path like `/tmp/x; malicious-command`). Both inputs should be moved to `env:` variables and then double-quoted in the shell script.

Locations:

- `action.yml:36`
- `action.yml:41`

### unpinned-uses (severity: high)

All `uses:` references in the workflow files use mutable tag-based refs instead of full 40-character commit SHA digests, making the workflows vulnerable to supply-chain attacks if any referenced action's tag is moved or compromised.

In `.github/workflows/release.yml`:
- `actions/checkout@v4` (line 13)
- `cycjimmy/semantic-release-action@v3` (line 20)
- `EndBug/add-and-commit@v9` (line 28)

In `.github/workflows/testing.yml`:
- `actions/checkout@v4` (line 17)
- `actionhippie/yamllint@v1` (line 21)
- `nowactions/update-majorver@v1` (line 26)

Locations:

- `.github/workflows/release.yml:13`
- `.github/workflows/release.yml:20`
- `.github/workflows/release.yml:28`
- `.github/workflows/testing.yml:17`
- `.github/workflows/testing.yml:21`
- `.github/workflows/testing.yml:26`

### missing-permissions (severity: medium)

Neither `.github/workflows/release.yml` nor `.github/workflows/testing.yml` declares a top-level `permissions:` key, and no job within either file has a `permissions:` block. Without explicit permissions, workflows run with the default repository token permissions, which may be overly broad (e.g. write access to contents). Each workflow should declare the minimal required permissions at the top level or per job.

Locations:

- `.github/workflows/release.yml:1`
- `.github/workflows/testing.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.path }}" appears directly in run: block of step "Create or resize swap space"; move to env: map

Locations:

- `action.yml:37`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.size }}" appears directly in run: block of step "Create or resize swap space"; move to env: map

Locations:

- `action.yml:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 5 findings: (1) Moved ${{ inputs.path }} and ${{ inputs.size }} from run: shell blocks to env: variables (INPUT_PATH, INPUT_SIZE) and double-quoted them in the shell script to prevent injection. (2) Pinned all 6 uses: references to full 40-char commit SHAs: actions/checkout@11d5960a, cycjimmy/semantic-release-action@8e58d20d, EndBug/add-and-commit@a94899bc, actionhippie/yamllint@42cde514, nowactions/update-majorver@f2014bbb. (3) Added permissions: contents: write to release.yml and permissions: contents: read to testing.yml.

