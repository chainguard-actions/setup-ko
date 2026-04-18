# Hardening Report: ko-build--setup-ko/v0.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `c40cfe5fa14e08549b1b988e7e5a26da4816abf0`

**Test Policy SHA:** `f2e7d85641cde4267138117189b8eba7ba2bfbde`

Action **ko-build--setup-ko/v0.8** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The run: block in action.yml directly interpolates multiple GitHub Actions expressions into shell commands without first assigning them to environment variables. This allows an attacker to inject arbitrary shell commands via user-controlled inputs or context values.

Affected interpolations:
- Line 23: `${{ inputs.use-sudo }}` used directly in an `if` condition
- Line 31: `${{ inputs.version }}` used unquoted in a `case` statement (especially dangerous — allows word splitting and glob expansion)
- Line 36: `${{ github.token }}` embedded in a curl `-u` argument string
- Line 39: `${{ inputs.version }}` assigned directly to a shell variable without quoting
- Line 42: `${{ runner.os }}` assigned directly to a shell variable
- Line 43: `${{ runner.arch }}` used inside a command substitution
- Line 60: `${{ github.token }}` piped directly to `ko login`
- Line 64: `${{ github.repository }}` used inside a command substitution

Fix: assign each expression to an environment variable via `env:` at the step level, then reference the env var (e.g. `$INPUT_VERSION`) in the shell script.

Locations:

- `action.yml:23`
- `action.yml:31`
- `action.yml:36`
- `action.yml:39`
- `action.yml:42`
- `action.yml:43`
- `action.yml:60`
- `action.yml:64`

### github-env-injection (severity: high)

The run: block derives the variable `repo` from the attacker-controllable expression `${{ github.repository }}` (line 64) and then writes it to $GITHUB_ENV on line 65 (`echo "KO_DOCKER_REPO=ghcr.io/${repo}" >> $GITHUB_ENV`) without the required sanitization step (`printf '%s' "$repo" | tr -d '\n\r'`). An attacker who can control the repository name (e.g. via a fork with a crafted name) could inject newlines into GITHUB_ENV, causing arbitrary environment variables to be set for subsequent steps in the workflow.

Fix: sanitize the value before writing:
  safe_repo=$(printf '%s' "$repo" | tr -d '\n\r')
  echo "KO_DOCKER_REPO=ghcr.io/${safe_repo}" >> "$GITHUB_ENV"

Locations:

- `action.yml:64`
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

Rewrote action.yml to move all ${{ }} expressions (inputs.use-sudo, inputs.version, github.token, runner.os, runner.arch, github.repository) into an env: block on the step. In the shell script, all references now use the corresponding environment variables ($INPUT_USE_SUDO, $INPUT_VERSION, $GITHUB_TOKEN, $RUNNER_OS, $RUNNER_ARCH, $GITHUB_REPOSITORY). Additionally, the github.repository-derived value is sanitized with `printf '%s' "$repo" | tr -d '\n\r'` before being written to $GITHUB_ENV to prevent newline injection attacks.

