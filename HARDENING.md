# Hardening Report: ko-build--setup-ko/v0.9

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

Action **ko-build--setup-ko/v0.9** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The run: block in action.yml directly interpolates multiple attacker-controlled expressions inside shell commands without first assigning them to environment variables. Affected expressions include: `${{ inputs.use-sudo }}` (line 23), `${{ inputs.version }}` (lines 30 and 38), `${{ github.token }}` (lines 35 and 60), `${{ runner.os }}` (line 41), `${{ runner.arch }}` (line 42), and `${{ github.repository }}` (line 63). An attacker controlling `inputs.version` or `inputs.use-sudo` (e.g. via workflow_dispatch) could inject arbitrary shell commands. These values must be passed via `env:` variables and referenced as `$ENV_VAR` in the shell script.

Locations:

- `action.yml:23`
- `action.yml:30`
- `action.yml:35`
- `action.yml:38`
- `action.yml:41`
- `action.yml:42`
- `action.yml:60`
- `action.yml:63`

### github-env-injection (severity: high)

The run: block writes a value derived from the attacker-controlled expression `${{ github.repository }}` to `$GITHUB_ENV` (line 65) without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The value is first lowercased via `tr` but newlines are not stripped, allowing an attacker to inject additional environment variable definitions into the GitHub environment file by embedding newline characters in the repository name. The fix is to apply `printf '%s' "$repo" | tr -d '\n\r'` before writing to `$GITHUB_ENV`.

Locations:

- `action.yml:65`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.use-sudo }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:23`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:31`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:40`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Rewrote action.yml to move all ${{ }} expressions from the run: block into an env: block. Specifically: INPUT_USE_SUDO=${{ inputs.use-sudo }}, INPUT_VERSION=${{ inputs.version }}, GITHUB_TOKEN=${{ github.token }}, RUNNER_OS=${{ runner.os }}, RUNNER_ARCH=${{ runner.arch }}, GITHUB_REPOSITORY=${{ github.repository }}. All shell references updated to use the corresponding $ENV_VAR names. For the github-env-injection finding, the repository value is now sanitized with `printf '%s' "$GITHUB_REPOSITORY" | tr -d '\n\r'` before lowercasing, and the final value written to $GITHUB_ENV is also sanitized with `printf '%s' "$repo" | tr -d '\n\r'` to prevent newline injection attacks.

