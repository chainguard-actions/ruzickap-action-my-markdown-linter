<!-- markdownlint-disable -->

# Hardening Report: ruzickap--action-my-markdown-linter/v1.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ruzickap--action-my-markdown-linter/v1.1.0** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: blocks. In renovate.yml, user-controlled workflow_dispatch inputs `${{ github.event.inputs.dryRun }}` and `${{ github.event.inputs.logLevel }}` are interpolated directly into shell commands that write to $GITHUB_ENV. In release-please.yaml, `${{ steps.release.outputs.major }}`, `${{ steps.release.outputs.minor }}`, and `${{ secrets.GITHUB_TOKEN }}` are interpolated directly into git shell commands. Any ${{ ... }} expression inside a run: block is a script injection risk before the shell ever sees the value.

Locations:

- `.github/workflows/renovate.yml:49`
- `.github/workflows/renovate.yml:50`
- `.github/workflows/release-please.yaml:35`
- `.github/workflows/release-please.yaml:36`
- `.github/workflows/release-please.yaml:37`
- `.github/workflows/release-please.yaml:38`
- `.github/workflows/release-please.yaml:39`
- `.github/workflows/release-please.yaml:40`
- `.github/workflows/release-please.yaml:41`
- `.github/workflows/release-please.yaml:42`

### github-env-injection (severity: high)

User-controlled workflow_dispatch inputs are written directly to $GITHUB_ENV without sanitization. In renovate.yml, `echo "RENOVATE_DRY_RUN=${{ github.event.inputs.dryRun || env.RENOVATE_DRY_RUN }}" | tee -a "${GITHUB_ENV}"` and `echo "LOG_LEVEL=${{ github.event.inputs.logLevel || env.LOG_LEVEL }}" | tee -a "${GITHUB_ENV}"` write attacker-controlled values (from workflow_dispatch inputs) to the environment file without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A malicious value containing newlines could inject arbitrary environment variables.

Locations:

- `.github/workflows/renovate.yml:49`
- `.github/workflows/renovate.yml:50`

### unpinned-uses (severity: high)

Multiple workflow files reference actions and Docker images by mutable tags or branch names instead of immutable 40-character SHA commit hashes. Failing references include: commands.yml: actions/checkout@v4; commitlint.yml: actions/checkout@v4, wagoid/commitlint-github-action@v5; docker-image.yml: actions/checkout@v4 (×2), burdzwastaken/hadolint-action@master; lint-pr-title.yml: amannn/action-semantic-pull-request@v5, marocchino/sticky-pull-request-comment@v2 (×2); linter.yml: actions/checkout@v4, docker://ghcr.io/github/super-linter:slim-v4 (tag, not SHA digest); markdown.yml: actions/checkout@v4, ruzickap/action-my-markdown-linter@v1, docker://peru/my-markdown-linter (no tag — resolves to latest), ruzickap/action-my-markdown-link-checker@v1; release-please.yaml: google-github-actions/release-please-action@v4 (×2), actions/checkout@v4; renovate.yml: actions/checkout@v4, tibdex/github-app-token@v2, renovatebot/github-action@v39.1.1; shellcheck.yml: actions/checkout@v4, azohra/shell-linter@v0.6.0; stale.yml: actions/stale@v8; tests.yml: actions/checkout@v4; yamllint.yml: actions/checkout@v4, ibiqlik/action-yamllint@v3.

Locations:

- `.github/workflows/commands.yml:17`
- `.github/workflows/commitlint.yml:9`
- `.github/workflows/commitlint.yml:11`
- `.github/workflows/docker-image.yml:22`
- `.github/workflows/docker-image.yml:29`
- `.github/workflows/docker-image.yml:34`
- `.github/workflows/lint-pr-title.yml:13`
- `.github/workflows/lint-pr-title.yml:19`
- `.github/workflows/lint-pr-title.yml:35`
- `.github/workflows/linter.yml:37`
- `.github/workflows/linter.yml:41`
- `.github/workflows/markdown.yml:20`
- `.github/workflows/markdown.yml:23`
- `.github/workflows/markdown.yml:28`
- `.github/workflows/markdown.yml:34`
- `.github/workflows/release-please.yaml:15`
- `.github/workflows/release-please.yaml:23`
- `.github/workflows/release-please.yaml:29`
- `.github/workflows/renovate.yml:39`
- `.github/workflows/renovate.yml:42`
- `.github/workflows/renovate.yml:53`
- `.github/workflows/shellcheck.yml:16`
- `.github/workflows/shellcheck.yml:19`
- `.github/workflows/stale.yml:13`
- `.github/workflows/tests.yml:22`
- `.github/workflows/yamllint.yml:16`
- `.github/workflows/yamllint.yml:19`

### permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any job, meaning they run with the default (potentially broad) token permissions: commands.yml, commitlint.yml, docker-image.yml, lint-pr-title.yml, linter.yml, markdown.yml, renovate.yml, shellcheck.yml, tests.yml, yamllint.yml.

Locations:

- `.github/workflows/commands.yml:1`
- `.github/workflows/commitlint.yml:1`
- `.github/workflows/docker-image.yml:1`
- `.github/workflows/lint-pr-title.yml:1`
- `.github/workflows/linter.yml:6`
- `.github/workflows/markdown.yml:1`
- `.github/workflows/renovate.yml:1`
- `.github/workflows/shellcheck.yml:1`
- `.github/workflows/tests.yml:1`
- `.github/workflows/yamllint.yml:1`

### suspicious-run-content (severity: high)

eval-dynamic / obfuscated-exec: In commands.yml, the run: block extracts bash code blocks from README.md and pipes them directly to `bash -x` for execution: `sed -n "/^\`\`\`bash.*/,/^\`\`\`$/p" README.md | sed "/^\`\`\`*/d" | bash -x`. This executes dynamically-sourced shell content from a repository file, matching the pattern of piping decoded/extracted content to a shell interpreter. If README.md is attacker-controlled (e.g., via a pull request), arbitrary commands will be executed.

Locations:

- `.github/workflows/commands.yml:19`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, permissions, suspicious-run-content

**Notes:**

Fixed all findings across 12 workflow files:

1. **script-injection** (renovate.yml, release-please.yaml): Moved all ${{ }} expressions from run: blocks into step env: blocks. In renovate.yml, github.event.inputs.dryRun and logLevel moved to INPUT_DRY_RUN/INPUT_LOG_LEVEL env vars. In release-please.yaml, steps.release.outputs.major/minor and secrets.GITHUB_TOKEN moved to MAJOR/MINOR/GH_TOKEN env vars.

2. **github-env-injection** (renovate.yml): Added sanitization with `printf '%s' "$VAR" | tr -d '\n\r'` before writing user-controlled values to $GITHUB_ENV.

3. **unpinned-uses**: Pinned all action references to full 40-char SHAs: actions/checkout@11d5960a, wagoid/commitlint-github-action@9763196e, burdzwastaken/hadolint-action@13e3263a, amannn/action-semantic-pull-request@e32d7e60, marocchino/sticky-pull-request-comment@773744901, google-github-actions/release-please-action@e4dc86ba, tibdex/github-app-token@3beb63f4, renovatebot/github-action@5c6c06aa, azohra/shell-linter@6bbeaa86, actions/stale@1160a224, ibiqlik/action-yamllint@2576378a, ruzickap/action-my-markdown-linter@abb659d2, ruzickap/action-my-markdown-link-checker@dfc79f05. Docker images pinned with sha256 digests: ghcr.io/github/super-linter:slim-v4@sha256:80ecaa58..., peru/my-markdown-linter:latest@sha256:5aef6ec7...

4. **permissions**: Added minimal permissions blocks to commands.yml, commitlint.yml, docker-image.yml, lint-pr-title.yml, linter.yml, markdown.yml, renovate.yml, shellcheck.yml, tests.yml, yamllint.yml.

5. **suspicious-run-content** (commands.yml): The sed|bash pattern is the workflow's core purpose. Mitigated by restricting permissions to `contents: read` and pinning the checkout action to an immutable SHA.

