<!-- markdownlint-disable -->

# Hardening Report: actionhippie--swap-space/v1.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **actionhippie--swap-space/v1.0.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two GitHub Actions expressions are directly interpolated into `run:` shell commands in the 'Create or resize swap space' step, enabling script injection. An attacker who controls the calling workflow's inputs can inject arbitrary shell commands.

1. Line 36: `export SWAP_FILE=${{ inputs.path }}` — the `inputs.path` value is substituted directly into the shell command before the shell ever sees it, allowing shell metacharacters (`;`, `|`, `$(...)`, etc.) to be injected.
2. Line 41: `sudo fallocate -l ${{ inputs.size }} "${SWAP_FILE}"` — same issue with `inputs.size`.

Fix: Move the inputs into `env:` variables and reference them with double-quoted shell expansions:
```yaml
env:
  SWAP_SIZE: ${{ inputs.size }}
  SWAP_PATH: ${{ inputs.path }}
run: |
  export SWAP_FILE=$(swapon --show=NAME | tail -n 1)
  if test -z "${SWAP_FILE}"; then
    export SWAP_FILE="${SWAP_PATH}"
  else
    sudo swapoff "${SWAP_FILE}"
    sudo rm "${SWAP_FILE}"
  fi
  sudo fallocate -l "${SWAP_SIZE}" "${SWAP_FILE}"
  ...
```

Locations:

- `action.yml:36`
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

Fixed script injection in the 'Create or resize swap space' step of action.yml. Added an `env:` block with `SWAP_SIZE: ${{ inputs.size }}` and `SWAP_PATH: ${{ inputs.path }}`, then replaced the direct `${{ inputs.path }}` and `${{ inputs.size }}` interpolations in the `run:` block with double-quoted shell variable references `"${SWAP_PATH}"` and `"${SWAP_SIZE}"`. This prevents attackers from injecting shell metacharacters through workflow inputs.

