You create a well-structured PR from the current changes: commit, push, label, and open the PR.

## Inputs

- **Arguments**: Use `$ARGUMENTS` for optional overrides (e.g., a specific label or PR title). If empty, auto-detect everything.

## Steps

### 1. Gather context

Run in parallel:
- `git status` — see all changed/untracked files
- `git diff HEAD` — see staged and unstaged changes
- `git branch --show-current` — get current branch
- `git log --oneline -10` — recent commit style

### 2. Classify the change

Based on the diff, assign exactly one label from:

| Label | Prefix | Use when |
|-------|--------|----------|
| `feature` | `feat/` | New functionality |
| `enhancement` | `enhance/` | Improving existing functionality |
| `bug` | `fix/` | Fixing something broken |
| `hotfix` | `hotfix/` | Urgent production fix |
| `refactor` | `refactor/` | Code restructuring, no behavior change |
| `chore` | `chore/` | Maintenance, dependencies, tooling |
| `docs` | `docs/` | Documentation only |
| `test` | `test/` | Adding or fixing tests |
| `revert` | `revert/` | Reverting a previous change |

If `$ARGUMENTS` specifies a label, use that instead of auto-detecting.

### 3. Create branch if on main

If on `main`, create a new branch using the format `<prefix><short-description>-<issue-number>` (e.g., `refactor/dedup-helper-185`). Derive the issue number from the changes or `$ARGUMENTS` if available. If no issue is being addressed, drop the trailing `-<issue-number>`.

### 4. Stage and commit

- Stage relevant files (prefer specific files over `git add -A`)
- Do NOT stage files that likely contain secrets (`.env`, credentials, etc.)
- Write a concise commit message (1-2 sentences) focusing on the "why"
- End with `Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>`

### 5. Push and create PR

Push the branch with `-u` flag, then create the PR:

```
gh pr create --title "<title>" --label "<label>" --body "$(cat <<'EOF'
## Summary
### <CHANGELOG section>
- <1-3 bullet points>

Closes #<issue-number> (if applicable)

## QC
- [ ] I, as a human being, have checked each line of code in this pull request
- [ ] <testable assertions based on what changed>
EOF
)"
```

The `### <CHANGELOG section>` header under `## Summary` must match the label:

| Label | CHANGELOG section |
|-------|-------------------|
| `feature`, `test` | `### Added` |
| `enhancement`, `chore`, `docs` | `### Changed` |
| `bug`, `hotfix` | `### Fixed` |
| `refactor` | `### Refactored` |
| `revert` | `### Removed` |

This allows `merge-pr` to use the Summary section directly when updating the CHANGELOG.

- PR title: short (under 70 characters), no prefix — the label carries the classification
- Add the `--label` flag with the label from step 2. If the label doesn't exist yet, create it first with `gh label create <name>`
- Include `Closes #<number>` in the body if an issue is being addressed
- Test plan items should be specific and verifiable (e.g., "All tests in foo_test.cpp pass" not "Tests pass")

### 6. Report

Return the PR URL and the label applied.