<!-- markdownlint-disable -->

# Hardening Report: bufbuild--buf-action/v1.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **bufbuild--buf-action/v1.5.0** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `dependabot.yaml` workflow uses `pull_request_target` and directly interpolates GitHub Actions expressions into a `run:` shell command. Specifically, `${{ github.event.pull_request.head.ref }}` (the PR branch name, fully attacker-controlled) is interpolated into a `git push` command, and `${{ secrets.GITHUB_TOKEN }}` is interpolated into a `git remote set-url` command. An attacker could craft a branch name containing shell metacharacters to achieve arbitrary command execution with write access to the repository. Sub-rule (a): direct expression interpolation inside a run: block.

Offending lines:
  `git remote set-url origin https://x-access-token:${{ secrets.GITHUB_TOKEN }}@github.com/$GITHUB_REPOSITORY`
  `git push -u origin HEAD:${{ github.event.pull_request.head.ref }}`

Locations:

- `.github/workflows/dependabot.yaml:19`
- `.github/workflows/dependabot.yaml:25`

### unpinned-uses (severity: high)

Multiple workflow files reference external actions and reusable workflows using mutable tag or branch refs instead of pinned 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the referenced tag or branch is moved to a malicious commit.

Failing references in ci.yaml: `actions/checkout@v4`, `actions/setup-node@v4`, `actions/github-script@v7`.
Failing reference in add-to-project.yaml: `bufbuild/base-workflows/.github/workflows/add-to-project.yaml@main`.
Failing reference in dependabot.yaml: `actions/checkout@v4`.
Failing reference in emergency-review-bypass.yaml: `bufbuild/base-workflows/.github/workflows/emergency-review-bypass.yaml@main`.
Failing reference in notify-approval-bypass.yaml: `bufbuild/base-workflows/.github/workflows/notify-approval-bypass.yaml@main`.
Failing reference in pr-title.yaml: `bufbuild/base-workflows/.github/workflows/pr-title.yaml@main`.

Locations:

- `.github/workflows/ci.yaml:14`
- `.github/workflows/add-to-project.yaml:16`
- `.github/workflows/dependabot.yaml:14`
- `.github/workflows/emergency-review-bypass.yaml:11`
- `.github/workflows/notify-approval-bypass.yaml:10`
- `.github/workflows/pr-title.yaml:14`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed script-injection in dependabot.yaml by moving ${{ secrets.GITHUB_TOKEN }} and ${{ github.event.pull_request.head.ref }} from the run: block into the step's env: block, referencing them as ${GITHUB_TOKEN} and ${HEAD_REF} in the shell script. Fixed unpinned-uses across all 6 workflow files: pinned actions/checkout@v4 to SHA 11d5960a326750d5838078e36cf38b85af677262, actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020, actions/github-script@v7 to SHA f28e40c7f34bde8b3046d885e986cb6290c5673b, and all four bufbuild/base-workflows reusable workflow references (@main) to SHA 3b9e85361ddfb81422eaf6f58b88377a08891953. All SHAs were resolved via lookup_action_sha.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings:
1. hardened/action/.github/workflows/dependabot.yaml line 27: Added double quotes around `HEAD:${HEAD_REF}` in the git push command → `git push -u origin "HEAD:${HEAD_REF}"`.
2. hardened/action/.github/workflows/ci.yaml (6 locations): Moved `${BUF_MODULE}` out of printf format strings and passed it as a properly double-quoted separate argument using `%s` as the format specifier. For example: `printf "version: v2\nmodules:\n  - path: proto\n    name: %s\n" "${BUF_MODULE}" > buf.yaml`. This applies to test-push, test-push-token-only, test-push-unnamed, test-breaking-against-registry, test-archive, and test-archive-root jobs.

