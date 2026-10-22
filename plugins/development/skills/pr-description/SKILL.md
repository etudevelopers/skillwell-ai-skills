---
name: pr-description
description: Write a clear pull request title and description from the current branch's changes. Use when the user asks for a PR description, PR summary or to prepare a pull request.
---

# PR description

Write a pull request description from the actual diff, not from memory.

## Steps

1. Run `git status`, `git log <base>..HEAD --oneline` and `git diff <base>...HEAD --stat` (base is usually `main`; ask if unclear). Read the diff for anything non-obvious.
2. Draft the description below.
3. Show it to the user. Do not create or edit the PR unless asked.

## Template

```
## Summary
One or two sentences: what changed and why.

## Changes
- Bullet per meaningful change (group by area, skip noise like formatting)

## Testing
How this was verified (tests run, manual steps). Say plainly if something was not tested.

## Notes for reviewers
Risky areas, follow-ups, migrations, config or env changes.
```

## Rules

- Title: imperative mood, under 72 characters.
- Mention the Jira key if it appears in the branch name or commits.
- Do not claim tests passed unless you ran them.
