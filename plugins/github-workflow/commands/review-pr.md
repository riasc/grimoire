You review a PR for quality, correctness, and adherence to project guidelines, and critically assess all existing comments on the PR.

## Inputs

- **PR number**: Use `$ARGUMENTS` as the PR number. If empty, detect from the current branch using `gh pr view --json number`.

## Steps

### 1. Identify the PR

Determine the PR number from `$ARGUMENTS` or the current branch. Run `gh pr view <number> --json title,body,number,headRefName,labels` to get PR details including labels.

### 2. Gather context

- Read the PR diff with `gh pr diff <number>`
- Read `CLAUDE.md` for project guidelines and conventions
- Read relevant memory files from the memory system for user preferences and past feedback
- If a language-specific skill is relevant (e.g., `cpp-coding-standards` for C++ changes), it auto-loads when the diff matches its description; rely on it as the standards baseline.

### 3. Review the code

Review the diff against project guidelines. Focus on:
- **Correctness**: Logic errors, off-by-one, missing error handling, use-after-move, etc.
- **CLAUDE.md compliance**: Does the code follow documented conventions?
- **Memory/feedback compliance**: Does the code respect past feedback and preferences stored in memory?
- **API consistency**: Do new/changed public interfaces match existing patterns?
- **Test coverage**: Are new code paths tested? Are edge cases covered?

Do NOT flag: style nitpicks, minor formatting, or things that are clearly intentional.

### 4. Assess existing PR comments

Fetch all comments on the PR:
- Review comments: `gh api repos/{owner}/{repo}/pulls/<number>/comments`
- Issue-level comments: `gh api repos/{owner}/{repo}/issues/<number>/comments`
- Review summaries: `gh api repos/{owner}/{repo}/pulls/<number>/reviews`

For each comment, critically assess whether it raises a valid concern:
- **Is it a real bug or correctness issue?** Flag for action.
- **Is it a meaningful improvement within the PR's scope?** Flag for action if warranted.
- **Is it a style/preference suggestion with no functional impact?** Skip unless it aligns with project conventions.
- **Is it already addressed by a subsequent commit?** Note as resolved.
- **Is it from an automated tool (e.g., CodeRabbit)?** Evaluate on merit — these can surface real issues but also produce false positives.

### 5. Post review on the PR

Post your review directly on the PR so there is a permanent record of the reasoning:

**Reply to each inline comment** — For each review comment (inline code comment), reply directly in that thread using `gh api repos/{owner}/{repo}/pulls/<number>/comments/<comment_id>/replies --method POST -f body="..."`. Include your verdict (**Address**, **Skip**, or **Discuss**) and the reasoning. Reference the summary comment for broader context.

**Post a summary comment** — Post a single issue-level comment (`gh pr comment <number> --body "..."`) with your own code review findings only, categorized as:
- **Bug**: Definite correctness issue
- **Guideline violation**: Contradicts CLAUDE.md or established feedback
- **Suggestion**: Improvement worth considering (clearly mark as optional)

Do not repeat the comment assessments here — those are already in the inline replies.

### 6. Report findings to the user

Present the same findings in the conversation so the user can decide what to act on.

### 7. Validate the PR label

Check the PR's current label against the actual diff. The label should match the nature of the changes:

| Label | Use when |
|-------|----------|
| `feature` | New functionality |
| `enhancement` | Improving existing functionality |
| `bug` | Fixing something broken |
| `hotfix` | Urgent production fix |
| `refactor` | Code restructuring, no behavior change |
| `chore` | Maintenance, dependencies, tooling |
| `docs` | Documentation only |
| `test` | Adding or fixing tests |
| `revert` | Reverting a previous change |

If the label no longer fits (e.g., a `refactor` PR gained a bug fix), update it with `gh pr edit <number> --remove-label "<old>" --add-label "<new>"`. If there is no label, add the appropriate one. Report any label changes to the user.

### 8. Update CHANGELOG

If the project has a `CHANGELOG.md`, check whether the PR's changes are already documented under `## [Unreleased]`. If the project does not maintain a CHANGELOG, skip this step.

If not present, add an entry:
- Determine the CHANGELOG section from the PR body's `### <CHANGELOG section>` header under `## Summary`. If the header is missing, fall back to the label mapping:

| Label | CHANGELOG section |
|-------|-------------------|
| `feature`, `test` | `### Added` |
| `enhancement`, `chore`, `docs` | `### Changed` |
| `bug`, `hotfix` | `### Fixed` |
| `refactor` | `### Refactored` |
| `revert` | `### Removed` |

Only these five CHANGELOG sections are used: `### Added`, `### Refactored`, `### Removed`, `### Changed`, `### Fixed`.

- Use the `## Summary` bullet points as the basis for the CHANGELOG entry content. Condense them into one entry following the existing format: bold feature name, concise description, then always append both the issue and PR number as links: `([#issue](...issues/issue), [#pr](...pull/pr))`

If already present, verify the entry accurately reflects the current diff and PR summary. Update if the changes have evolved since the entry was written.

Also verify that the PR body's `### <CHANGELOG section>` header matches the label. If they disagree, update the PR body header to match the label (since the label was validated in step 7).

Commit the CHANGELOG update to the PR branch, push, and note the update in the report.

### 9. Verify and mark the QC checklist

Run `gh pr checks <number>` to get CI status. Read the PR body and find all QC checkboxes (`- [ ]` items). For each item, verify it is satisfied based on CI results (e.g., all tests passing across platforms). Mark each verified item as checked by updating the PR body (`gh pr edit <number> --body "..."`).

**Important**: Never check the "I, as a human being, have checked each line of code" item — only a human can mark that. If any other items cannot be confirmed as passing, report which items failed.

### 10. Make changes if approved

Wait for user confirmation before making any code changes. If changes are made, push them to the PR branch.