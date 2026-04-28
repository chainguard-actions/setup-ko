# Hardening Report: ko-build--setup-ko/v0.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

Action **ko-build--setup-ko/v0.8** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The single `run:` step in action.yml directly interpolates attacker-controlled expressions into the shell command string without first assigning them to environment variables. Specifically: `${{ inputs.version }}` is used in a `case` statement (line 22) and as a tag value (line 31); `${{ github.token }}` is interpolated into a curl `-u` argument (line 28) and piped to `ko login` (line 53); and `${{ github.repository }}` is interpolated into a command substitution (line 57). An attacker who controls these values (e.g. via a crafted version input or a forked PR) can inject arbitrary shell commands.

Locations:

- `action.yml:22`
- `action.yml:28`
- `action.yml:31`
- `action.yml:53`
- `action.yml:57`

### github-env-injection (severity: high)

The `run:` step writes `repo` (derived from `${{ github.repository }}`, an attacker-controlled value) to `$GITHUB_ENV` at line 59 (`echo "KO_DOCKER_REPO=ghcr.io/${repo}" >> $GITHUB_ENV`) without first sanitizing the value with `printf '%s' ... | tr -d '\n\r'`. A malicious repository name containing newlines could inject arbitrary environment variables or override existing ones for subsequent steps.

Locations:

- `action.yml:59`

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

Moved all ${{ }} expressions (${{ inputs.version }}, ${{ github.token }}, ${{ github.repository }}, ${{ runner.os }}, ${{ runner.arch }}) from the run: block into an env: block (as KO_VERSION, GITHUB_TOKEN, GITHUB_REPOSITORY, RUNNER_OS, RUNNER_ARCH). Updated the shell script to reference these as plain environment variables. For the github-env-injection finding, sanitized the repo value using `printf '%s' "$GITHUB_REPOSITORY" | tr '[:upper:]' '[:lower:]' | tr -d '\n\r'` before writing to $GITHUB_ENV to prevent newline injection.

