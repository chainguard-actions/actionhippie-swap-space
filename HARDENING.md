<!-- markdownlint-disable -->

# Hardening Report: actionhippie--swap-space/v1.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **actionhippie--swap-space/v1.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `${{ inputs.* }}` expressions are interpolated directly inside a `run:` shell command string in the 'Create or resize swap space' step. This allows an attacker (or any caller of this composite action) to inject arbitrary shell commands via the `inputs.path` and `inputs.size` values.

Offending lines:
  - `export SWAP_FILE=$(sudo mktemp ${{ inputs.path }}/swap.XXXXXXXXXX)` — inputs.path is injected unquoted into a shell command.
  - `sudo fallocate -l ${{ inputs.size }} "${SWAP_FILE}"` — inputs.size is injected unquoted into a shell command.

Fix: Move the inputs into `env:` variables and reference them as double-quoted shell variables, e.g.:
```yaml
env:
  SWAP_PATH: ${{ inputs.path }}
  SWAP_SIZE: ${{ inputs.size }}
run: |
  export SWAP_FILE=$(sudo mktemp "$SWAP_PATH"/swap.XXXXXXXXXX)
  sudo fallocate -l "$SWAP_SIZE" "${SWAP_FILE}"
```

Locations:

- `action.yml:34`
- `action.yml:40`

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

Fixed script injection in the 'Create or resize swap space' step of action.yml. Moved `${{ inputs.path }}` and `${{ inputs.size }}` expressions from the run: shell block into an `env:` block as `SWAP_PATH` and `SWAP_SIZE` respectively. Updated the shell commands to reference these as double-quoted environment variables: `"$SWAP_PATH"/swap.XXXXXXXXXX` and `"$SWAP_SIZE"`, preventing arbitrary shell command injection via caller-controlled input values.

