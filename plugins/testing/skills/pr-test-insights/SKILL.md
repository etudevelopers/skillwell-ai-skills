---
name: pr-test-insights
description: Read a pull request (or the current branch's changes) and write a tester-facing briefing: what changed, what areas it touches, what could break, and how to test it. Use when a PR is opened or updated, when a tester asks "what do I need to test for this PR / ticket", or when the user gives a PR number, PR URL or branch and wants test guidance.
---

# PR test insights

Turn a code change into a briefing a tester can act on without reading the diff. Everything must be grounded in the actual changes, not in the PR title alone.

## Inputs

- A PR number or URL, **or** a branch name, **or** nothing (use the current branch against its base).
- Optional: the linked ticket. Look for a ticket key (pattern `ABC-123`) in the PR title, branch name, commit messages and PR description. If a Jira/Atlassian tool is available in the session, fetch the ticket (description, acceptance criteria, comments) for the intended behaviour. If not, work from the change alone and say so; never guess ticket content.

## Steps

1. **Gather the change.**
   - PR: `gh pr view <n> --json title,body,author,labels,baseRefName,headRefName,files,commits` and `gh pr diff <n>`. If `gh` is not available, use a GitHub tool from the session if there is one, or ask the user for the base and head branches and use `git`.
   - Branch (the default when no argument is given, and the right mode for a developer checking their own work before opening a PR): `git log <base>..HEAD --oneline` and `git diff <base>...HEAD`. Use `origin/<base>` if the local base is stale. The base is usually `main` or `master`; check `git remote show origin` or ask if unclear. Include uncommitted changes (`git status`, `git diff`) and say so if there are any.
   - For very large diffs, read the file list first, then read the files that carry behaviour (skip lockfiles, generated code and pure formatting).
2. **Classify every changed file** into an area: UI / front end, API / back end, business logic, database or migration, configuration or feature flag, dependency, build or CI, tests, documentation.
3. **Separate behaviour changes from non-behaviour changes.** Refactors, renames, comments and formatting need regression attention at most. Say which is which.
4. **Find the blast radius.** For each changed function, endpoint, component, query or config key that carries behaviour, find its callers and consumers. If the session has a code-graph or impact-analysis tool (for example an MCP server offering callers, callees or impact queries), prefer it; otherwise use `grep`. List what else depends on it, even if it is not in the diff.
5. **Read the tests in the PR.** Note what the developer already covers and what is not covered (new branches, error paths, edge cases).
6. **Compare intent to change.** If the PR or ticket description says one thing and the diff does something different or more, flag it as a question for the developer.
7. **Write the briefing** in the format below.

## Output format

### Summary
Two or three sentences in plain language: what this change does and why, from the user's point of view.

### What changed
Bullets in behaviour terms ("the statement text now uses the learner's name"), not code terms. Mark each as **new**, **changed**, **fixed** or **internal only**.

### Areas touched
| Area | What is affected | Files | Risk |
|------|------------------|-------|------|
Risk is Low / Medium / High, with a few words on why.

### Also affected (not in the diff)
Features, screens, endpoints or jobs that use the changed code and could regress.

### How to test
Prioritised, numbered scenarios. For each: a title, the steps in one or two lines, and the expected result.
1. **Primary**: the change working as intended
2. **Negative and edge cases**: invalid input, empty state, boundaries, permissions or roles, error messages
3. **Regression**: the items under "Also affected"
4. **Data, state and timing**: concurrency, repeat actions, refresh or return flows, long or unusual values

### Setup needed
Test data, accounts or roles, feature flags, config values, migrations, environments, build or deploy steps.

### Not worth testing
Parts of the diff with no user-visible effect, so the tester can skip them.

### Gaps and questions for the developer
Anything unclear, any behaviour the diff changes that the description does not mention, and cases the PR's own tests do not cover.

## Rules

- Base every statement on the diff or the ticket. Mark inferences as *assumed*; never present a guess as fact.
- Do not invent screens, field names, messages or URLs. Use the exact names that appear in the code.
- No secrets in the output: credentials, tokens, customer data, internal hostnames.
- Keep it short enough to read in a few minutes. Prefer a prioritised list of scenarios over an exhaustive one.
- Do not post the briefing to the PR, a ticket or chat unless the user asks.
- This skill is for one PR or branch. For release-wide regression planning across many changes, use a dedicated release-analysis skill if the repo has one.
- This skill only reads. It does not build, run tests, commit or push.
