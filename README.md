# dev-automation-claude-plugin

Project design, documentation scaffolding, and autonomous scheduled development tasks.

## What it does

This is a Claude Code skill plugin that bundles eleven development-focused skills into a single installable package. The skills cover the full software delivery lifecycle — from early-phase architecture and system design, through day-to-day code review, debugging, and documentation, out to deploy checklists, incident response, and postmortem-friendly workflows. A dedicated scheduler skill is also included for standing up autonomous, recurring development tasks that work through a project roadmap one item at a time.

Each skill is defined by a `SKILL.md` file and is invoked automatically by Claude Code when your request matches the skill's trigger description, or explicitly when you call it with its slash command.

## Skills included

| Skill | Purpose |
|-------|---------|
| `architecture` | Create or evaluate Architecture Decision Records (ADRs) with trade-offs and consequences. |
| `code-review` | Review code changes for security, performance, correctness, and maintainability. |
| `create-scheduled-dev-task` | Set up a fully configured scheduled task that autonomously develops a project against its roadmap. |
| `debug` | Run a structured debugging session — reproduce, isolate, diagnose, and fix. |
| `deploy-checklist` | Generate a pre-deployment verification checklist before shipping a release. |
| `documentation` | Write and maintain technical documentation — READMEs, runbooks, API docs, onboarding guides. |
| `incident-response` | Run an incident-response workflow covering triage, communication, and blameless postmortems. |
| `standup` | Generate a yesterday/today/blockers standup update from recent activity across your tools. |
| `system-design` | Design systems, services, and architectures with explicit framework for requirements and trade-offs. |
| `tech-debt` | Identify, categorize, and prioritize technical debt across code, architecture, and infrastructure. |
| `testing-strategy` | Design test strategies and test plans balancing coverage, speed, and maintenance cost. |

## Installation

Install directly from GitHub using the Claude Code plugin manager:

```
/plugin install goosefly99/dev-automation-claude-plugin
```

Or clone and install manually:

```bash
git clone https://github.com/goosefly99/dev-automation-claude-plugin.git
```

Then point Claude Code at the cloned directory per your local plugin configuration.

## Using skills

Once the plugin is installed, Claude Code will surface its skills automatically. You don't need to memorize slash commands:

- **Automatic invocation** — when your request matches a skill's trigger description (for example, "review this PR" or "we have an incident"), Claude Code invokes the relevant skill on its own.
- **Explicit invocation** — you can also call a skill directly by name using the Skill tool (for example, `architecture`, `debug`, `deploy-checklist`).

Skills that expect an argument will declare an `argument-hint` in their frontmatter — pass the relevant context (a PR URL, an error message, a decision to evaluate) alongside the invocation and the skill will handle the rest.

## Repository layout

```
.claude-plugin/
  plugin.json            # Plugin metadata and skill manifest
skills/
  architecture/SKILL.md
  code-review/SKILL.md
  create-scheduled-dev-task/
    SKILL.md
    references/          # Supporting templates and checklists
  debug/SKILL.md
  deploy-checklist/SKILL.md
  documentation/SKILL.md
  incident-response/SKILL.md
  standup/SKILL.md
  system-design/SKILL.md
  tech-debt/SKILL.md
  testing-strategy/SKILL.md
```

This plugin contains no runtime code, no MCP servers, and no Node dependencies — it is a pure collection of skill definitions.
