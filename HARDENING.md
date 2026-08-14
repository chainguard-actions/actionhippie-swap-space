<!-- markdownlint-disable -->

# Hardening Report: actionhippie--swap-space/v1.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actionhippie--swap-space/v1.1.0** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in run: blocks. In action.yml, the composite action step 'Create or resize swap space' directly interpolates ${{ inputs.path }} and ${{ inputs.size }} inside shell commands. These user-controlled inputs are substituted into the shell script before execution, allowing an attacker to inject arbitrary shell commands via crafted input values (e.g., inputs.size = '1G; malicious-command'). The expressions are not routed through env: vars and are completely unquoted.

Locations:

- `action.yml:36`
- `action.yml:42`

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in run: blocks. In automerge.yml, two steps ('Approve request' and 'Enable automerge') directly interpolate ${{github.event.pull_request.html_url}} inside shell commands passed to the gh CLI. This is an attacker-controlled value (pull request URL can be crafted) injected directly into the shell string, enabling command injection.

Locations:

- `.github/workflows/automerge.yml:37`
- `.github/workflows/automerge.yml:42`

### unpinned-uses (severity: high)

All uses: references across all three workflow files use mutable version tags instead of immutable 40-character SHA commit digests, making the workflows vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved. Unpinned references found:
- automerge.yml: tibdex/github-app-token@v2 (line 22), dependabot/fetch-metadata@v2 (line 31)
- release.yml: actions/checkout@v4 (line 15), cycjimmy/semantic-release-action@v3 (line 22), EndBug/add-and-commit@v9 (line 30)
- testing.yml: actions/checkout@v4 (line 21), actionhippie/yamllint@v1 (line 24), nowactions/update-majorver@v1 (line 28)

Locations:

- `.github/workflows/automerge.yml:22`
- `.github/workflows/automerge.yml:31`
- `.github/workflows/release.yml:15`
- `.github/workflows/release.yml:22`
- `.github/workflows/release.yml:30`
- `.github/workflows/testing.yml:21`
- `.github/workflows/testing.yml:24`
- `.github/workflows/testing.yml:28`

### missing-permissions (severity: medium)

release.yml and testing.yml have no top-level permissions: key and no job-level permissions: key on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) permissions, violating the principle of least privilege. Each workflow should declare the minimal permissions required.

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

Fixed all findings across 4 files:

1. action.yml (script-injection/static-inline-injection): Moved ${{ inputs.path }} and ${{ inputs.size }} from the run: block into an env: block as INPUT_PATH and INPUT_SIZE. Shell script now references them as "${INPUT_PATH}" and "${INPUT_SIZE}" — properly quoted, no expression interpolation in shell.

2. automerge.yml (script-injection): Moved ${{ github.event.pull_request.html_url }} from both run: blocks into env: blocks as PR_URL. Also pinned tibdex/github-app-token@v2 → SHA 3beb63f4bd073e61482598c45c71c1019b59b73a and dependabot/fetch-metadata@v2 → SHA 21025c705c08248db411dc16f3619e6b5f9ea21a.

3. release.yml (unpinned-uses + missing-permissions): Pinned actions/checkout@v4 → SHA 11d5960a326750d5838078e36cf38b85af677262, cycjimmy/semantic-release-action@v3 → SHA 8e58d20d0f6c8773181f43eb74d6a05e3099571d, EndBug/add-and-commit@v9 → SHA a94899bca583c204427a224a7af87c02f9b325d5. Added top-level permissions: contents: write.

4. testing.yml (unpinned-uses + missing-permissions): Pinned actions/checkout@v4 → SHA 11d5960a326750d5838078e36cf38b85af677262, actionhippie/yamllint@v1 → SHA 42cde5141b45bc5832d61ed19007b5ca0f354753, nowactions/update-majorver@v1 → SHA f2014bbbba95b635e990ce512c5653bd0f4753fb. Added top-level permissions: contents: write.

