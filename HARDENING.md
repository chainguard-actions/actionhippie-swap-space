<!-- markdownlint-disable -->

# Hardening Report: actionhippie--swap-space/v1.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **actionhippie--swap-space/v1.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Create or resize swap space' step in action.yml directly interpolates ${{ inputs.path }} and ${{ inputs.size }} inside run: shell commands. GitHub Actions performs YAML template substitution before the shell ever sees the string, so a caller can supply a value like `; malicious_command #` to execute arbitrary shell commands. Both inputs are declared `required: false` with defaults, meaning any calling workflow can override them with attacker-controlled values. The offending lines are:
  Line 35: `export SWAP_FILE=${{ inputs.path }}`
  Line 41: `sudo fallocate -l ${{ inputs.size }} "${SWAP_FILE}"`
Fix: route the inputs through env: variables and reference them as quoted shell variables (e.g., `"$INPUT_PATH"`, `"$INPUT_SIZE"`) instead of using ${{ }} directly inside the run: block.

Locations:

- `action.yml:35`
- `action.yml:41`

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

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all three findings in action.yml's 'Create or resize swap space' step. Added an env: block with INPUT_PATH: ${{ inputs.path }} and INPUT_SIZE: ${{ inputs.size }}, then replaced the direct ${{ inputs.path }} and ${{ inputs.size }} interpolations in the run: block with quoted shell variable references "$INPUT_PATH" and "$INPUT_SIZE". This prevents shell injection by ensuring GitHub Actions template substitution only occurs in the safe env: context, not inside the shell command string.

