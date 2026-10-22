# Contributing

Each team owns its folder under `plugins/`. Add skills to your team's plugin, or propose a new plugin for a new team.

> **This repo is public.** Never commit secrets, credentials, customer data, internal URLs, or company-confidential processes.

## Add a skill to an existing plugin

1. Create a branch: `git switch -c <team>/<skill-name>`
2. Create `plugins/<team>/skills/<skill-name>/SKILL.md` using the template below.
3. Test it locally (see below).
4. Bump `version` in `plugins/<team>/.claude-plugin/plugin.json` (patch for fixes, minor for a new skill).
5. Add a row to the plugin table in `README.md`.
6. Open a pull request. A teammate from your team reviews it.

## Add a new plugin

1. Create `plugins/<name>/.claude-plugin/plugin.json`:
   ```json
   {
     "name": "<name>",
     "description": "One sentence on who it is for.",
     "version": "0.1.0",
     "author": { "name": "Skillwell <Team>" }
   }
   ```
2. Add `plugins/<name>/skills/<skill-name>/SKILL.md`.
3. Register it in `.claude-plugin/marketplace.json` under `plugins`:
   ```json
   { "name": "<name>", "source": "./plugins/<name>", "description": "..." }
   ```

## Skill template

Copy this into `SKILL.md`:

````markdown
---
name: skill-name
description: What it does, and when to use it. Include the phrases a user would actually say. Claude uses this line to decide when to load the skill.
---

# Skill title

One line on the outcome.

## Inputs

- What the user must provide (and what to do if it is missing)

## Steps

1. Numbered, concrete actions
2. Where to ask the user instead of guessing

## Output format

Show the exact structure of the result (table, headings, template).

## Rules

- Hard constraints, things never to do
````

## Conventions

- **Name**: lowercase, hyphenated, matches the folder name (`jira-test-cases`).
- **Description**: one or two sentences, starts with what it does, then "Use when...". This is the trigger, so make it specific.
- **One job per skill.** Split big workflows into separate skills.
- **Ask, don't assume.** If input is missing or ambiguous, the skill should ask or mark assumptions explicitly.
- **Specific outputs.** Define the output format so results are consistent across authors.
- **No company-specific data** in this repo while it is public.
- Keep `SKILL.md` under ~300 lines; put long reference material in a sibling file and link to it.

## Test locally

From a clone of this repo, in Claude Code:

```
/plugin marketplace add ./
/plugin install <plugin-name>@skillwell-ai-skills
```

Then run your skill on a realistic example. After editing, run `/plugin marketplace update skillwell-ai-skills` and reload.

## Review checklist

- [ ] Frontmatter has `name` and `description`; name matches folder
- [ ] Output format is defined
- [ ] No secrets or confidential data
- [ ] Plugin `version` bumped
- [ ] README table updated
- [ ] Tried on a real example
