# Skillwell AI Skills

A [Claude Code](https://claude.com/claude-code) plugin marketplace with shared skills for Skillwell teams. Install only the plugins relevant to your role.

## Install

In Claude Code:

```
/plugin marketplace add etudevelopers/skillwell-ai-skills
/plugin install <plugin-name>@skillwell-ai-skills
```

Restart Claude Code (or run `/reload-plugins`) after installing.

To pick up new skills later:

```
/plugin marketplace update skillwell-ai-skills
```

## Plugins

| Plugin | For | Skills |
|--------|-----|--------|
| `product` | Product owners | `prd-to-docs`: PRD to stakeholder brief, release notes or feature guide |
| `development` | Developers | `pr-description`: PR title and description from your branch's diff |
| `testing` | QA | `jira-test-cases`: Jira ticket to traceable test cases<br>`prd-test-plan`: PRD to test plan |

## Using a skill

Skills are invoked automatically when your request matches, or explicitly by name:

```
/testing:jira-test-cases ABC-123
/testing:prd-test-plan   (then paste or point to the PRD)
/product:prd-to-docs
/development:pr-description
```

## Repo layout

```
.claude-plugin/marketplace.json   lists every plugin
plugins/<team>/.claude-plugin/plugin.json
plugins/<team>/skills/<skill-name>/SKILL.md
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) to add or change a skill.

## Visibility note

This repo is public. Do not put internal processes, Jira project details, credentials, customer data or other company-confidential content in skills. If a skill needs that, the repo must be made private first.
