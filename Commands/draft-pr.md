---
description: Create a draft pull request with an auto-generated summary of all changes
argument-hint: <base-branch>
---

Create a draft pull request targeting the base branch: $ARGUMENTS

Follow these steps:

1. Run `git status` and `git diff HEAD` (or `git diff <base-branch>...HEAD` if branch exists on remote) to understand all staged, unstaged, and committed changes on the current branch compared to `$ARGUMENTS`.
2. Run `git log $ARGUMENTS..HEAD --oneline` to see all commits being included.
3. If there are uncommitted changes, ask the user whether to commit them first before creating the PR.
4. Push the current branch to origin if not already pushed (`git push -u origin <branch>`). If the push is rejected due to remote changes, pull with rebase first.
5. Analyze all changes and generate:
   - A concise PR title (under 70 characters) summarizing the overall change
   - A brief markdown summary with bullet points grouped by area (e.g. infrastructure, backend, config) describing what changed and why
6. Create the draft PR with:
```
gh pr create --draft --base $ARGUMENTS --title "<title>" --body "<summary>"
```

The PR body must follow this format:
```
## Summary
- <bullet point per logical change group>

## Test plan
- [ ] <key thing to verify>
```

Return the PR URL when done.
