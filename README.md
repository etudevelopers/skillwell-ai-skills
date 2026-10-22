# Skillwell AI Skills

A [Claude Code](https://claude.com/claude-code) plugin marketplace with shared skills for Skillwell teams. Install only the plugins relevant to your role. No cloning needed.

## Contents

- [Quick install](#quick-install)
- [Available plugins](#available-plugins)
- [Starter guide: using the plugins](#starter-guide-using-the-plugins)
- [Using it in your own repo and branch (developers)](#using-it-in-your-own-repo-and-branch-developers)
- [Starter guide: creating your own skill](#starter-guide-creating-your-own-skill)
- [Sharing your skill with the team](#sharing-your-skill-with-the-team)
- [Troubleshooting](#troubleshooting)
- [Repo layout](#repo-layout)

## Quick install

In Claude Code:

```
/plugin marketplace add etudevelopers/skillwell-ai-skills
/plugin install <plugin-name>@skillwell-ai-skills
```

Then run `/reload-plugins` (or restart Claude Code).

## Available plugins

| Plugin | For | Skills |
|--------|-----|--------|
| `testing` | QA / testers | `pr-test-insights`: reads a pull request and writes a tester briefing: what changed, what is touched, what could break, how to test it<br>`pr-test-cases`: turns a Jira ticket and its PR into data-driven and state-driven UI and API test cases (Given/When/Then) in a CSV for import |

---

## Starter guide: using the plugins

You need Claude Code installed and signed in. Everything below happens inside Claude Code. You do not clone this repo, download files or edit config by hand.

### Step 1. Add the marketplace (once per machine)

```
/plugin marketplace add etudevelopers/skillwell-ai-skills
```

This registers the catalogue. It installs nothing yet. `etudevelopers/skillwell-ai-skills` is the GitHub `owner/repo`.

### Step 2. Install the plugin

```
/plugin install testing@skillwell-ai-skills
```

You can also run `/plugin`, open the **Discover** tab, and pick plugins from a list. As more plugins are added, install only the ones relevant to your role.

### Step 3. Reload

```
/reload-plugins
```

### Step 4. Use a skill

Two ways:

**Just ask.** Claude loads the matching skill automatically when your request fits its description.

> "What do I need to test in PR 42?"
> "Give me a test briefing for this branch"
> "Write test cases as a CSV for ticket ABC-123"

**Or call it by name.** Skills are namespaced as `/<plugin>:<skill>`:

```
/testing:pr-test-insights 42
/testing:pr-test-insights https://github.com/<owner>/<repo>/pull/42
/testing:pr-test-insights            (uses the current branch)
/testing:pr-test-cases ABC-123       (test cases as a CSV, from a Jira ticket)
```

### Step 5. Know what you can do next

| Task | Command |
|------|---------|
| See installed plugins and their skills | `/plugin` (Installed tab) |
| Get new and updated skills | `/plugin marketplace update skillwell-ai-skills`, then `/reload-plugins` |
| Turn a plugin off without removing it | `/plugin disable <plugin-name>@skillwell-ai-skills` |
| Remove a plugin | `/plugin uninstall <plugin-name>@skillwell-ai-skills` |

### Example: first run for QA

1. `/plugin marketplace add etudevelopers/skillwell-ai-skills`
2. `/plugin install testing@skillwell-ai-skills`
3. `/reload-plugins`
4. In the repo being tested, run `/testing:pr-test-insights 42` (use your PR number). The `gh` command-line tool must be installed and signed in so Claude can read the PR.
5. Read the briefing (summary, areas touched, test scenarios, setup, questions for the developer), and ask for changes, for example "focus on the API changes" or "add edge cases for the new field".
6. Then run `/testing:pr-test-cases ABC-123` (use the ticket key) to get the test cases as a CSV file. It asks you whenever the ticket does not say what the expected result is.

---

## Using it in your own repo and branch (developers)

The skill reads whatever repository you run Claude Code in. The skills repo and your application repo are separate: you install the plugin once, then use it in any repo. Nothing from this repo needs to be copied into yours.

### Option 1: just for you (recommended to start)

Install once. It applies to every repo you open.

```
/plugin marketplace add etudevelopers/skillwell-ai-skills
/plugin install testing@skillwell-ai-skills
/reload-plugins
```

Then, in a terminal inside your application repo, on your working branch:

```bash
cd <your-app-repo>
git switch <your-branch>        # for example the branch named after your ticket
claude
```

Inside Claude Code:

```
/testing:pr-test-insights
```

With no argument it compares your current branch with the base branch (`master` or `main`) and includes uncommitted changes, so you can run it **before** you open the PR. After the PR exists, anyone can run it by number:

```
/testing:pr-test-insights 42
```

### Option 2: for the whole team (declare it in the app repo)

So that every developer and tester who opens the app repo gets the plugin without installing it by hand, add the marketplace to the app repo's committed Claude settings. From inside the app repo:

```bash
claude plugin marketplace add etudevelopers/skillwell-ai-skills --scope project
claude plugin install testing@skillwell-ai-skills --scope project
```

This writes to `.claude/settings.json` in that repo. If the file already exists, the commands merge into it. The result looks like this (keep your existing keys such as `enabledPlugins`):

```json
{
  "extraKnownMarketplaces": {
    "skillwell-ai-skills": {
      "source": { "source": "github", "repo": "etudevelopers/skillwell-ai-skills" }
    }
  },
  "enabledPlugins": {
    "testing@skillwell-ai-skills": true
  }
}
```

Commit it through a normal pull request to the app repo. After it merges, team members are prompted to trust the repo and install the marketplace the next time they start Claude Code there. The app repo can stay private; only this skills repo is public, and the plugin does not send your code anywhere except to Claude in your own session.

### Tester workflow (give it the Jira ticket)

A tester does not need the PR number. Open Claude Code in a checkout of the app repo and give it the ticket URL or key:

```
/testing:pr-test-insights https://<your-site>.atlassian.net/browse/ABC-123
/testing:pr-test-insights ABC-123
```

It reads the ticket, finds the PR(s) whose title or branch name contains that key, reads the code changes and writes the briefing. If your Claude Code has no Jira connection, paste the ticket text after the command instead. To put the result on the ticket, ask it: "add this to the ticket as a comment" (or "add it to the description"). It shows you the text first and writes only after you say yes; it never changes existing description text.

### Test cases for import (CSV)

The usual order for a tester is: run `pr-test-insights` for the briefing, then `pr-test-cases` for the cases. After the briefing, the tester can have the test cases written to a CSV file:

```
/testing:pr-test-cases ABC-123
```

- It writes `ABC-123-test-cases.csv` in the current folder: UI and API cases in Given/When/Then, with the data to create under **Pre-requisites**. UI-level cases (checked on the screen) and API-level cases (checked by calling an endpoint) are kept apart: separate ID series (`ABC-123-UI-01`, `ABC-123-API-01`), a `Type` and a `Target` column (screen, or method and endpoint), UI rows first. If the change has no server side, it says so instead of inventing API cases. Data-driven cases get one row per data combination; state-driven cases name the starting state and the event.
- **Expected results come only from the Jira ticket** (description, acceptance criteria, comments, the developer's QA changelist). The code is used to find what to test, never what the answer should be, so a bug cannot become an expected result.
- If the ticket does not say what should happen in a scenario, Claude **asks you** in one batch. A scenario you cannot answer is left out of the file and listed as "Not written".
- If your import tool needs different column names, give Claude the header row and it will match it.
- It only writes the CSV file. It does not post to Jira or commit anything.

### Developer workflow

(Test cases from `pr-test-cases` are normally written by the tester from the Jira ticket. A developer can run it too, to check that the ticket states enough expected behaviour to test.)

1. Work on your branch as usual and commit.
2. Run `/testing:pr-test-insights` and read the briefing: what changed, what else is affected, how to test, what the tests in your branch do not cover.
3. Fix gaps or answer the "questions for the developer" before asking for review.
4. Open the PR. Paste the briefing into the PR description under "Testing notes", or ask Claude to "post this as a comment on the PR".
5. The tester runs `/testing:pr-test-insights <PR number>` against the same PR and gets the same briefing from the actual diff.

### What the skills need

| Needs | Why | If missing |
|-------|-----|------------|
| `git` and a checkout of your branch | Branch mode | Required |
| GitHub CLI (`gh`), signed in with `gh auth login` | Reads a PR by number | Use branch mode, or a GitHub tool in your Claude Code setup |
| A Jira/Atlassian connection in your Claude Code (optional for the briefing, effectively needed for test cases) | Fetches the ticket, finds PRs from a ticket URL, and can add the briefing to the ticket. `pr-test-cases` takes every expected result from the ticket | Paste the ticket text and give a PR number or branch; adding to Jira is then manual |
| Permission to write a file in the current folder (`pr-test-cases` only) | It writes `<KEY>-test-cases.csv` | Give it another folder |
| A code-graph or impact-analysis tool (optional) | More accurate "also affected" list | It falls back to searching the code |

### Running it automatically on every PR

A skill runs when someone invokes it; it does not trigger by itself when a PR is pushed. The supported way is manual: the developer runs it before opening the PR, or the tester runs it with the ticket URL. Posting a briefing to every PR or ticket automatically needs a CI step that runs a model (for example a GitHub Action with an API key stored as a repository secret) plus a Jira service account. That is separate setup with a cost, and this repo does not include it.

---

## Starter guide: creating your own skill

A **skill** is a folder containing one `SKILL.md` file: a short header that says when to use it, and instructions Claude follows. A **plugin** is a bundle of skills you can install and share. A **marketplace** (this repo) is a catalogue of plugins.

You can build and test a skill entirely on your own machine, without cloning this repo, then share it when it works.

### Path A: personal skill (fastest, 2 minutes, just for you)

No plugin, no repo. Create one file and it is available in every project.

1. Create the folder:
   ```bash
   mkdir -p ~/.claude/skills/my-skill
   ```
2. Create `~/.claude/skills/my-skill/SKILL.md`:
   ```markdown
   ---
   name: my-skill
   description: Summarise a meeting transcript into decisions and action items. Use when the user pastes meeting notes or a transcript.
   ---

   # Meeting summary

   ## Steps
   1. Read the transcript.
   2. List decisions made.
   3. List action items with an owner and due date; write "unassigned" if none is stated.
   4. List open questions.

   ## Output format
   ### Decisions
   - ...
   ### Action items
   | Owner | Action | Due |
   |-------|--------|-----|
   ### Open questions
   - ...

   ## Rules
   - Do not invent owners or dates.
   ```
3. Start a new Claude Code session (or restart) and try it: paste some notes and ask for a summary, or run `/my-skill`.
4. Edit the file, start a new session, and try again until the output is what you want.

Use Path A to experiment. When the skill is good enough to share, move to Path B or C.

### Path B: build a local plugin (to test exactly what teammates will install)

This mirrors the structure used in this repo, in a throwaway folder, with no clone and no GitHub.

1. Create the plugin folder anywhere, for example in your home directory:
   ```bash
   mkdir -p ~/my-plugin/.claude-plugin ~/my-plugin/skills/meeting-summary
   ```
2. Create `~/my-plugin/.claude-plugin/plugin.json`:
   ```json
   {
     "name": "my-plugin",
     "description": "My experimental skills.",
     "version": "0.1.0",
     "author": { "name": "Your Name" }
   }
   ```
3. Create `~/my-plugin/skills/meeting-summary/SKILL.md` (same format as Path A).
4. Launch Claude Code and load the plugin from that folder for this session:
   ```bash
   claude --plugin-dir ~/my-plugin
   ```
5. Run it by its namespaced name: `/my-plugin:meeting-summary`.
6. Edit `SKILL.md`, then run `/reload-plugins` inside Claude Code to pick up your changes without restarting.

If the skill does not show up, see [Troubleshooting](#troubleshooting).

### Writing a good SKILL.md

The file has two parts: frontmatter and instructions.

```markdown
---
name: skill-name
description: What it does. Use when <the phrases a user would say>.
---
```

| Part | Guidance |
|------|----------|
| `name` | Lowercase, hyphenated, same as the folder name. |
| `description` | The most important line. Claude reads it to decide when to load the skill. Say what it does and when to use it, with the words people actually use ("test cases", "release notes", "Jira ticket"). |
| Inputs | What the user must provide, and what to do if it is missing (ask, never guess). |
| Steps | Numbered, concrete actions. Say where to stop and ask the user a question. |
| Output format | Show the exact structure of the result (table columns, headings) so results are consistent. |
| Rules | Hard limits: what never to invent, what never to include. |

Tips:

- **One job per skill.** If it does two unrelated things, make two skills.
- **Show, don't describe.** An example output table beats a paragraph about tables.
- **Test with a real example** (a real ticket, a real PRD), not a toy one, and read the output critically.
- **Keep it short.** Under about 300 lines. Move long reference material into a separate file next to `SKILL.md` and link to it.
- **No secrets.** This repo is public. Never put credentials, customer data, internal URLs or confidential process details in a skill.

A blank template is in [CONTRIBUTING.md](CONTRIBUTING.md#skill-template).

---

## Sharing your skill with the team

When your skill works locally, add it to this marketplace so everyone can install it. You can do this entirely in the browser, with no clone.

### Option 1: GitHub web editor (no git needed)

1. Open the repo: https://github.com/etudevelopers/skillwell-ai-skills
2. Go to the folder for your team, for example `plugins/testing/skills/`.
3. Click **Add file → Create new file**.
4. In the filename box type `your-skill-name/SKILL.md` (typing the `/` creates the folder).
5. Paste your `SKILL.md` content.
6. Scroll down, choose **Create a new branch for this commit and start a pull request**, then **Propose changes** and **Create pull request**.
7. In the PR, also bump `version` in your plugin's `plugin.json` (open the file, click the pencil icon, edit the version, commit to the same branch).
8. Ask a teammate from your team to review. Once merged, everyone gets it after `/plugin marketplace update skillwell-ai-skills`.

### Option 2: Command line

If you are comfortable with git:

```bash
git clone https://github.com/etudevelopers/skillwell-ai-skills.git
cd skillwell-ai-skills
git switch -c <team>/<skill-name>
# add plugins/<team>/skills/<skill-name>/SKILL.md, bump the plugin version
git add -A && git commit -m "Add <skill-name> skill"
git push -u origin HEAD
```

Then open a pull request on GitHub.

### Adding a whole new plugin (for a new team)

You need three things, all described in [CONTRIBUTING.md](CONTRIBUTING.md#add-a-new-plugin): a `plugin.json`, at least one skill, and an entry in `.claude-plugin/marketplace.json`.

### Review and merge rules

The default branch is `master`. No change reaches it except through a pull request, and a PR cannot be merged until **at least 1 approving review** from someone other than the author. This is enforced two ways:

- A branch protection rule on `master` requires the approval, and dismisses approvals when new commits are pushed.
- The [Require approval](.github/workflows/require-approval.yml) workflow is a required status check. It fails until the approval count is met. To require more reviewers, change `REQUIRED_APPROVALS` in that file and the review count in the branch rule.

### Before you open the PR

- [ ] Tried it in Claude Code on a realistic example
- [ ] `name` matches the folder name
- [ ] Output format is defined
- [ ] No secrets or confidential data
- [ ] Plugin `version` bumped
- [ ] Skill listed in the plugin table in this README

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `/plugin` command not found | Update Claude Code to the latest version. |
| Marketplace not found | Check the spelling: `etudevelopers/skillwell-ai-skills`. If the repo has been made private, you need GitHub access and must be signed in to GitHub on your machine (for example with `gh auth login`). |
| Plugin installed but skill does not appear | Run `/reload-plugins`, or restart Claude Code. Check `/plugin` lists it as enabled. |
| Skill does not trigger on its own | Its `description` is too vague. Add the phrases people actually use, or call it by name: `/<plugin>:<skill>`. |
| New skills from teammates are missing | `/plugin marketplace update skillwell-ai-skills`, then `/reload-plugins`. |
| Local plugin not loading | Confirm `.claude-plugin/plugin.json` is valid JSON and the folder is named `skills/<skill-name>/SKILL.md`. Run `claude --plugin-dir <path>` again. |
| `pr-test-insights` cannot read the PR | It needs the GitHub CLI. Install `gh` and run `gh auth login`, then run the skill from inside the repo that owns the PR. A linked Jira ticket is optional: if Claude has no Jira connection it works from the PR alone, or you can paste the ticket text. |
| `pr-test-cases` has no ticket text to work from | Expected results come only from the ticket, never from the code. Connect Jira, or paste the ticket (description, acceptance criteria, comments) after the command. If the ticket has no expected results, Claude asks you for them before writing. |
| `pr-test-cases` leaves scenarios out | Those are listed under "Not written": the ticket did not say what should happen and you did not answer. Answer the questions and run it again. |
| The CSV does not import cleanly | Give Claude the header row of your import tool's template so it can match the column names and order. UI and API cases are separated by the `Type` column and the `UI`/`API` part of the Test ID. |

## Repo layout

```
.claude-plugin/marketplace.json            lists every plugin
plugins/<team>/.claude-plugin/plugin.json  one per plugin
plugins/<team>/skills/<skill-name>/SKILL.md
README.md                                  this guide
CONTRIBUTING.md                            skill template and review checklist
```

Today: one plugin, `testing`, with two skills (`pr-test-insights`, `pr-test-cases`).

## Visibility note

This repo is public. Do not put internal processes, Jira project details, credentials, customer data or other company-confidential content in skills. If a skill needs that, the repo must be made private first.
