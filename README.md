# Skillwell AI Skills

A [Claude Code](https://claude.com/claude-code) plugin marketplace with shared skills for Skillwell teams. Install only the plugins relevant to your role. No cloning needed.

## Contents

- [Quick install](#quick-install)
- [Available plugins](#available-plugins)
- [Starter guide: using the plugins](#starter-guide-using-the-plugins)
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
| `product` | Product owners | `prd-to-docs`: PRD to stakeholder brief, release notes or feature guide |
| `development` | Developers | `pr-description`: PR title and description from your branch's diff |
| `testing` | QA | `jira-test-cases`: Jira ticket to traceable test cases<br>`prd-test-plan`: PRD to test plan |

---

## Starter guide: using the plugins

You need Claude Code installed and signed in. Everything below happens inside Claude Code. You do not clone this repo, download files or edit config by hand.

### Step 1. Add the marketplace (once per machine)

```
/plugin marketplace add etudevelopers/skillwell-ai-skills
```

This registers the catalogue. It installs nothing yet. `etudevelopers/skillwell-ai-skills` is the GitHub `owner/repo`.

### Step 2. Install the plugin for your role

| If you are... | Run |
|---------------|-----|
| Product owner | `/plugin install product@skillwell-ai-skills` |
| Developer | `/plugin install development@skillwell-ai-skills` |
| QA / tester | `/plugin install testing@skillwell-ai-skills` |

Install more than one if you wear several hats. You can also run `/plugin`, open the **Discover** tab, and pick plugins from a list.

### Step 3. Reload

```
/reload-plugins
```

### Step 4. Use a skill

Two ways:

**Just ask.** Claude loads the matching skill automatically when your request fits its description.

> "Write test cases for ABC-123"
> "Turn this PRD into release notes"

**Or call it by name.** Skills are namespaced as `/<plugin>:<skill>`:

```
/testing:jira-test-cases ABC-123
/testing:prd-test-plan
/product:prd-to-docs
/development:pr-description
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
4. `/testing:jira-test-cases ABC-123`, or paste the ticket text after the command if your Claude Code has no Jira connection.
5. Review the table of cases, answer any open questions it lists, and ask it to adjust (for example "add boundary cases for the amount field" or "export as CSV").

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
| Jira ticket cannot be fetched | Your Claude Code may not have a Jira connection. Paste the ticket text instead. |

## Repo layout

```
.claude-plugin/marketplace.json            lists every plugin
plugins/<team>/.claude-plugin/plugin.json  one per plugin
plugins/<team>/skills/<skill-name>/SKILL.md
README.md                                  this guide
CONTRIBUTING.md                            skill template and review checklist
```

## Visibility note

This repo is public. Do not put internal processes, Jira project details, credentials, customer data or other company-confidential content in skills. If a skill needs that, the repo must be made private first.
