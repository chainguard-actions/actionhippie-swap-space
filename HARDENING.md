<!-- markdownlint-disable -->

# Hardening Report: actionhippie--swap-space/v1.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actionhippie--swap-space/v1.0.2** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation of user-controlled inputs inside run: shell commands. In the 'Create or resize swap space' step, ${{ inputs.path }} is assigned directly to a shell variable (export SWAP_FILE=${{ inputs.path }}) and ${{ inputs.size }} is passed directly to sudo fallocate (sudo fallocate -l ${{ inputs.size }} ...). An attacker calling this composite action can supply values containing shell metacharacters (e.g. semicolons, backticks, $(...)) to achieve arbitrary command execution.

Locations:

- `action.yml:38`
- `action.yml:43`

### unpinned-uses (severity: high)

All uses: references in the workflow files use mutable version tags instead of full 40-character SHA commit digests, making the workflows vulnerable to supply-chain attacks if those tags are moved. Failing references: actions/checkout@v4, cycjimmy/semantic-release-action@v3, EndBug/add-and-commit@v9 (release.yml); actions/checkout@v4, actionhippie/yamllint@v1, nowactions/update-majorver@v1 (testing.yml).

Locations:

- `.github/workflows/release.yml:13`
- `.github/workflows/release.yml:20`
- `.github/workflows/release.yml:28`
- `.github/workflows/testing.yml:13`
- `.github/workflows/testing.yml:17`
- `.github/workflows/testing.yml:22`

### missing-permissions (severity: medium)

Neither release.yml nor testing.yml defines a top-level permissions: block, and no job-level permissions: blocks are present either. Without explicit permissions, workflows run with the default (potentially write-all) token permissions, violating the principle of least privilege.

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

1. action.yml: Fixed script injection by moving ${{ inputs.path }} and ${{ inputs.size }} into an env: block (INPUT_PATH, INPUT_SIZE) and referencing them as plain shell variables in the run: block. 2. release.yml: Pinned actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262, cycjimmy/semantic-release-action@v3 → @8e58d20d0f6c8773181f43eb74d6a05e3099571d, EndBug/add-and-commit@v9 → @a94899bca583c204427a224a7af87c02f9b325d5; added top-level permissions: {} and job-level permissions: contents: write. 3. testing.yml: Pinned actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262, actionhippie/yamllint@v1 → @42cde5141b45bc5832d61ed19007b5ca0f354753, nowactions/update-majorver@v1 → @f2014bbbba95b635e990ce512c5653bd0f4753fb; added top-level permissions: {} and job-level permissions: contents: write.

