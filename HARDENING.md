<!-- markdownlint-disable -->

# Hardening Report: actionhippie--swap-space/v1.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **actionhippie--swap-space/v1.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `inputs.*` expressions are interpolated directly inside a `run:` shell command string, enabling script injection. An attacker who controls the `path` or `size` inputs can inject arbitrary shell commands.

1. Line 35: `export SWAP_FILE=$(mktemp ${{ inputs.path }}/swap.XXXXXXXXXX)` — `${{ inputs.path }}` is substituted directly into the shell command before the shell ever sees it, allowing shell metacharacter injection.
2. Line 41: `sudo fallocate -l ${{ inputs.size }} "${SWAP_FILE}"` — `${{ inputs.size }}` is substituted directly into the shell command, allowing shell metacharacter injection.

Fix: Move the values into `env:` variables and reference them with double-quoted shell expansions, e.g.:
```yaml
env:
  SWAP_PATH: ${{ inputs.path }}
  SWAP_SIZE: ${{ inputs.size }}
run: |
  export SWAP_FILE=$(mktemp "${SWAP_PATH}/swap.XXXXXXXXXX")
  ...
  sudo fallocate -l "${SWAP_SIZE}" "${SWAP_FILE}"
```

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

Fixed all three findings in action.yml by adding an `env:` block to the 'Create or resize swap space' step with SWAP_PATH=${{ inputs.path }} and SWAP_SIZE=${{ inputs.size }}. Replaced the direct template interpolations in the run: block with double-quoted shell variable references: $(mktemp "${SWAP_PATH}/swap.XXXXXXXXXX") and sudo fallocate -l "${SWAP_SIZE}" "${SWAP_FILE}". This eliminates the script injection risk from attacker-controlled inputs.

