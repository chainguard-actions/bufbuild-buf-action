<!-- markdownlint-disable -->

# Hardening Report: bufbuild--buf-action/v1.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **bufbuild--buf-action/v1.4.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `run:` block in dependabot.yaml directly interpolates GitHub Actions expressions inside shell commands. Specifically, `${{ secrets.GITHUB_TOKEN }}` is embedded in a git remote URL and `${{ github.event.pull_request.head.ref }}` (an attacker-controlled PR branch name) is passed directly to `git push`. An attacker could craft a branch name containing shell metacharacters to achieve command injection. These values must be passed via `env:` variables and then double-quoted in the shell script.

Locations:

- `.github/workflows/dependabot.yaml:17`

### unpinned-uses (severity: high)

All `uses:` references across all workflow files use mutable tags or branch names instead of immutable 40-character SHA commit digests, making the workflows vulnerable to supply-chain attacks if the referenced actions are compromised or their tags are moved. Failing references include: `actions/checkout@v4`, `actions/setup-node@v4`, `actions/github-script@v7`, `bufbuild/base-workflows/.github/workflows/add-to-project.yaml@main`, `bufbuild/base-workflows/.github/workflows/emergency-review-bypass.yaml@main`, `bufbuild/base-workflows/.github/workflows/notify-approval-bypass.yaml@main`, `bufbuild/base-workflows/.github/workflows/pr-title.yaml@main`.

Locations:

- `.github/workflows/ci.yaml:17`
- `.github/workflows/ci.yaml:19`
- `.github/workflows/dependabot.yaml:13`
- `.github/workflows/add-to-project.yaml:16`
- `.github/workflows/emergency-review-bypass.yaml:10`
- `.github/workflows/notify-approval-bypass.yaml:9`
- `.github/workflows/pr-title.yaml:18`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed script-injection in dependabot.yaml by moving ${{ secrets.GITHUB_TOKEN }} and ${{ github.event.pull_request.head.ref }} into the step's env: block (as GITHUB_TOKEN and PR_HEAD_REF) and referencing them as double-quoted shell variables. Fixed unpinned-uses across all 6 workflow files: pinned actions/checkout@v4, actions/setup-node@v4, actions/github-script@v7, and all three bufbuild/base-workflows reusable workflow references (@main) to their full 40-character commit SHAs, preserving original tags as comments.

