You merge the current PR, clean up branches, and ensure the CHANGELOG is up to date.

## Inputs

- **PR number**: Use `$ARGUMENTS` as the PR number. If empty, detect from the current branch using `gh pr view --json number`.

## Steps

### 1. Identify the PR

Determine the PR number from `$ARGUMENTS` or the current branch. Run `gh pr view <number> --json title,body,number,headRefName,labels` to get PR details including labels.

### 2. Check CI status

Run `gh pr checks <number>` and verify all checks pass. If any are failing or pending, report and stop.

### 3. Verify CHANGELOG is updated

If the project maintains a `CHANGELOG.md`, check whether the PR's changes are already documented under `## [Unreleased]`. The CHANGELOG entry should have been added during `/review-pr`. Skip this step if the project does not maintain a CHANGELOG.

- If present, verify the entry accurately reflects the current diff and PR summary. If the changes have evolved since the entry was written, update it.
- If missing, stop and report — run `/review-pr` first to add the CHANGELOG entry.
- If the entry needs minor corrections, update it, commit to the PR branch, and push.

### 4. Verify QC checklist is complete

Read the PR body and check that all QC checkboxes are marked as done (`- [x]`). If any are unchecked (`- [ ]`), report which items are incomplete and stop — do not merge. Run `/review-pr` first to review and mark the automated QC items; the human review item must be checked by a human.

### 5. Merge the PR

Run `gh pr merge <number> --squash`.

### 6. Update local main

Run `git checkout main && git pull`.

### 7. Delete branches

Delete the local branch with `git branch -d <branch-name>` and the remote branch with `git push origin --delete <branch-name>`.

### 8. Check if docs need updating

If `CLAUDE.md` documents a separate documentation repository (e.g., a `docs/` sister repo on GitHub), analyze the PR's changes and determine if they affect anything that should be documented there:
- Public API changes (new/renamed/removed classes, methods, parameters, return types)
- Changed behavior or semantics
- New features or capabilities users would need to know about
- Compiler/runtime support or build system changes
- Removed or deprecated functionality

If any docs impact is found, create an issue in the docs repo with:
- Title describing what changed
- Body linking to the PR and explaining what docs need updating
- Cross-references to related existing docs issues if applicable

If the changes are purely internal (refactoring, performance, CI fixes, test additions) with no user-facing impact, or if the project does not document a separate docs repo in CLAUDE.md, skip this step.

### 9. Report

Confirm the merge is complete: PR number, branch cleaned up, CHANGELOG status (if applicable), and whether a docs issue was created.