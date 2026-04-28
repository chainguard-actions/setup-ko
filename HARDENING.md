# Hardening Report: ko-build--setup-ko/v0.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

Action **ko-build--setup-ko/v0.8** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The run: block in action.yml directly interpolates attacker-controlled expressions inside shell command strings without first assigning them to environment variables. Specifically: `${{ inputs.version }}` is used directly in a `case` statement (line 22) and as a tag value (line 31); `${{ github.token }}` is interpolated into a curl -u argument (line 28) and echoed into a pipe (line 52); `${{ runner.os }}` and `${{ runner.arch }}` are interpolated directly (lines 34–35); and `${{ github.repository }}` is interpolated into a subshell (line 55). All of these should be passed via env: variables and referenced as $ENV_VAR in the shell.

Locations:

- `action.yml:22`
- `action.yml:28`
- `action.yml:31`
- `action.yml:34`
- `action.yml:35`
- `action.yml:52`
- `action.yml:55`

### github-env-injection (severity: high)

The run: block writes a value derived from the attacker-controlled expression `${{ github.repository }}` to $GITHUB_ENV without sanitization. On line 55, `${{ github.repository }}` is interpolated into a shell variable `repo` via subshell, and on line 57 that variable is written to $GITHUB_ENV with `echo "KO_DOCKER_REPO=ghcr.io/${repo}" >> $GITHUB_ENV`. A repository name containing newline characters could inject arbitrary environment variables into subsequent steps. The required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`) is absent.

Locations:

- `action.yml:55`
- `action.yml:57`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:22`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:31`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Rewrote action.yml to move all ${{ }} expressions out of the run: block into an env: block. Specifically: inputs.version → KO_VERSION, github.token → GITHUB_TOKEN, runner.os → RUNNER_OS, runner.arch → RUNNER_ARCH, github.repository → GITHUB_REPOSITORY. All shell references updated to use the corresponding $ENV_VAR names. For the github-env-injection finding, the repo variable derivation now uses `printf '%s' "$GITHUB_REPOSITORY" | tr '[:upper:]' '[:lower:]' | tr -d '\n\r'` to strip newlines before writing to $GITHUB_ENV.

