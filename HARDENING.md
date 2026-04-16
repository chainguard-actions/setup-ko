# Hardening Report: imjasonh--setup-ko/v0.9

> This file was generated automatically by the hardening agent.

**Policy SHA:** `c40cfe5fa14e08549b1b988e7e5a26da4816abf0`

**Test Policy SHA:** `f2e7d85641cde4267138117189b8eba7ba2bfbde`

Action **imjasonh--setup-ko/v0.9** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple GitHub Actions expressions are interpolated directly into run: shell commands without first being assigned to environment variables. This allows an attacker to inject arbitrary shell commands via user-controlled inputs.

- Line 23: `${{ inputs.use-sudo }}` used directly inside an `if [[ "..." == "true" ]]` test.
- Line 30: `${{ inputs.version }}` used unquoted in a `case` statement — particularly dangerous as it is unquoted and can break out of the case construct.
- Line 38: `${{ inputs.version }}` used directly in `tag="${{ inputs.version }}"`.
- Line 35: `${{ github.token }}` used directly in a curl `-u` flag (though token is a secret, direct interpolation is still unsafe).
- Line 60: `${{ github.token }}` piped directly to `ko login`.
- Line 64: `${{ github.repository }}` used directly in a command substitution.

Fix: assign each expression to an env var (e.g., `env: VERSION: ${{ inputs.version }}`) and reference `$VERSION` in the run block.

Locations:

- `action.yml:23`
- `action.yml:30`
- `action.yml:35`
- `action.yml:38`
- `action.yml:60`
- `action.yml:64`

### github-env-injection (severity: high)

The value of `${{ github.repository }}` is interpolated into the shell variable `repo` (line 64) and then written to `$GITHUB_ENV` on line 65 without the required sanitization step (`printf '%s' "$repo" | tr -d '\n\r'`). An attacker who can influence the repository name (e.g., via a fork with a crafted name) could inject newlines into GITHUB_ENV, allowing them to set arbitrary environment variables for subsequent steps.

Failing pattern (line 64–65):
  repo=$(echo "${{ github.repository }}" | tr '[:upper:]' '[:lower:]')
  echo "KO_DOCKER_REPO=ghcr.io/${repo}" >> $GITHUB_ENV

Fix: sanitize before writing:
  safe_repo=$(printf '%s' "$repo" | tr -d '\n\r')
  echo "KO_DOCKER_REPO=ghcr.io/${safe_repo}" >> "$GITHUB_ENV"

Locations:

- `action.yml:64`

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

Fixed all findings in hardened/imjasonh--setup-ko/v0.9/action.yml:

1. script-injection / static-inline-injection: Moved all GitHub Actions expressions out of the run: shell block into an env: block on the step:
   - inputs.use-sudo → USE_SUDO env var, referenced as "$USE_SUDO" in the if condition
   - inputs.version → VERSION env var, referenced as $VERSION in case statement and tag assignment
   - github.token → GH_TOKEN env var, referenced as $GH_TOKEN in curl -u flag and ko login pipe
   - github.repository → GH_REPOSITORY env var, referenced as "$GH_REPOSITORY" in repo assignment
   - runner.os → RUNNER_OS_INPUT env var (bonus fix)
   - runner.arch → RUNNER_ARCH_INPUT env var (bonus fix)

2. github-env-injection: Added sanitization step before writing to GITHUB_ENV:
   - Added: safe_repo=$(printf '%s' "$repo" | tr -d '\n\r')
   - Changed echo to use $safe_repo instead of $repo when writing KO_DOCKER_REPO to $GITHUB_ENV
   - Also quoted $GITHUB_ENV in the redirection: >> "$GITHUB_ENV"

