---
name: create-scheduled-dev-task
description: This skill should be used when the user asks to "create a scheduled development task", "set up an autonomous dev agent", "schedule a development loop", "create a dev agent for this project", "automate development on a schedule", "set up a recurring dev task", or wants to configure a scheduled task that autonomously works through a project's development roadmap. It handles project discovery, documentation scaffolding (via the project-design-and-development skill), prompt assembly, and scheduled task installation.
version: 0.1.0
---

# Create Scheduled Development Task

Set up a fully configured scheduled task that autonomously develops a project by working through its development roadmap one item at a time. Each run of the task picks up the next incomplete item, implements it, tests, commits, and updates the tracking table.

This skill orchestrates three concerns:
1. **Project readiness** — ensure the project has design docs, AGENTS.md files, and a development roadmap (delegating to the `project-design-and-development` skill if needed).
2. **Prompt assembly** — build a self-contained autonomous agent prompt from the project's structure.
3. **Task installation** — create and configure the scheduled task via the `mcp__scheduled-tasks__create_scheduled_task` tool.

Execute the four steps below in order.

---

## Step 1 — Project Discovery

Gather all information needed to populate the scheduled task prompt. Read `references/project-discovery-checklist.md` for the full checklist and discovery commands.

### 1.1 Identify the Project

Determine the project name, absolute path, and a one-sentence description. If the user has not specified the project, ask which project to set up.

```bash
# Verify project path exists
ls <project_path>
```

### 1.2 Check Documentation Readiness

Scan the project for required documentation artifacts:

```bash
# Check for roadmap
ls <project_path>/dev_roadmap.md 2>/dev/null || ls <project_path>/docs/ROADMAP.md 2>/dev/null

# Find AGENTS.md files
find <project_path> -name "AGENTS.md" -not -path "*node_modules*" -not -path "*.git*"

# Check for design docs and ADRs
ls <project_path>/docs/adr/ 2>/dev/null
ls <project_path>/docs/design/ 2>/dev/null
```

**If any of these are missing** — the project is not ready for autonomous development. Invoke the `project-design-and-development` skill to create the missing artifacts:

```
Skill("project-design-and-development")
```

This will create design docs, AGENTS.md files, README, and a development roadmap. After it completes, re-run the discovery checks above.

**If all artifacts exist** — proceed to extract the information needed for the prompt.

### 1.3 Extract Prompt Data

Gather the following from the project:

1. **Git configuration** — current branch, remote status, branch setup commands.
2. **Roadmap path and contents** — read the roadmap to verify it has phase tables with status, dependencies, and target files. The agent will read this at runtime to determine what to work on.
3. **Reference documents table** — build a markdown table from all AGENTS.md files, ADRs, design docs, handoff specs, and CLAUDE.md. These are the files the agent will read for implementation details.
4. **Build/test/lint commands** — read `package.json` scripts, `pyproject.toml` tools, `Makefile` targets, or `Cargo.toml` to identify the exact commands.
5. **Roadmap and reference file quality** — verify that the roadmap and reference files are structured well enough for the agent to derive implementation guidance at runtime. See `references/project-discovery-checklist.md` Section 6 for the verification checklist. If files are inadequate, invoke `Skill("project-design-and-development")` to improve them before proceeding.

### 1.4 Confirm Schedule

Ask the user for the desired run frequency. Default: hourly. Common options:

| Frequency | Cron Expression |
|-----------|----------------|
| Hourly | `17 * * * *` |
| Every 2 hours | `23 */2 * * *` |
| Every 4 hours | `11 */4 * * *` |
| Twice daily (9am, 5pm) | `17 9,17 * * *` |
| Weekdays hourly | `17 * * * 1-5` |

Always use an offset minute (not :00 or :30) to spread load across the fleet.

---

## Step 2 — Assemble the Prompt

Read `references/scheduled-task-prompt-template.md` for the full template with all placeholders.

Build the prompt by replacing each placeholder with project-specific values gathered in Step 1:

| Placeholder | Source |
|-------------|--------|
| `{{PROJECT_NAME}}` | Step 1.1 — project name |
| `{{PROJECT_DESCRIPTION_SUFFIX}}` | Step 1.1 — ` — ` + one-sentence description |
| `{{BRANCH_NAME}}` | Step 1.3 — git branch |
| `{{BRANCH_SETUP_COMMANDS}}` | Step 1.3 — cd + checkout + pull commands |
| `{{ROADMAP_PATH}}` | Step 1.3 — relative path to roadmap |
| `{{REFERENCE_DOCUMENTS_TABLE}}` | Step 1.3 — markdown table of all reference docs |
| `{{TEST_COMMANDS}}` | Step 1.3 — bulleted test/lint/build commands |
| `{{QUALITY_COMMANDS}}` | Step 1.3 — commands that must pass before commit |

### Prompt Assembly Rules

- The assembled prompt must be **fully self-contained**. The scheduled task runs in a separate session with no access to this skill or any other skill. Every instruction must be inline.
- Do not include `{{...}}` placeholders in the final prompt — all must be resolved.
- **Do not hardcode phase-specific progress, item descriptions, or completion status into the prompt.** The agent reads the roadmap file at runtime to determine what to work on. This keeps the prompt durable as the project evolves.
- The reference documents table should list all files the agent needs for implementation details (AGENTS.md files, design docs, handoff specs, ADRs).
- Include the absolute project path in the branch setup commands (`cd /path/to/project`).
- The prompt should be between 100-250 lines. Without hardcoded phase guidance, prompts are shorter and more maintainable.

### Validation

Before proceeding, verify the assembled prompt:

1. No unresolved `{{...}}` placeholders remain.
2. All file paths referenced in the prompt exist in the project.
3. The roadmap path is correct.
4. Git branch name matches an existing branch (or creation commands are included).
5. Test/lint/build commands are valid for the project's toolchain.

---

## Step 3 — Create the Scheduled Task

Use `mcp__scheduled-tasks__create_scheduled_task` to install the task:
Always specify in the cron prompt: spawn a fresh Agent subagent each run for scheduling jobs. 

```
mcp__scheduled-tasks__create_scheduled_task(
  taskId: "<project-name>-dev",
  description: "Autonomous development agent for the <Project Name> project — picks up the next roadmap item, implements it, tests, commits, and updates tracking.",
  prompt: "<assembled prompt from Step 2>",
  cronExpression: "<cron from Step 1.4>"
)
```

### Task ID Convention

Use `<project-name>-dev` as the task ID. Examples: `librefrets-dev`, `taskflow-dev`, `foxdot-dev-agent`. Check against existing tasks to avoid conflicts:

```
mcp__scheduled-tasks__list_scheduled_tasks()
```

If a task with that ID already exists, ask the user whether to update the existing task or create a new one with a different ID.

---

## Step 4 — Verify and Configure

### 4.1 Verify Installation

After creation, list scheduled tasks to confirm the new task appears:

```
mcp__scheduled-tasks__list_scheduled_tasks()
```

Verify:
- The task ID matches what was specified.
- The cron expression is correct.
- The task is enabled.
- The `nextRunAt` time is reasonable.

### 4.2 Create the Task's SKILL.md

The scheduled task needs a SKILL.md file at `~/.claude/scheduled-tasks/<taskId>/SKILL.md`. This file was auto-created by the MCP tool with the prompt as its body. Verify it exists:

```bash
cat ~/.claude/scheduled-tasks/<taskId>/SKILL.md
```

### 4.3 Present Summary to User

Report to the user:

- **Task ID** and description
- **Schedule** — human-readable frequency and next run time
- **Project path** targeted
- **Roadmap** — total items, phases, current completion status
- **Branch** the agent will work on
- **Commands** the agent will run (test, lint, build)

### 4.4 Optional: Trigger First Run

Offer to trigger the first run immediately for verification:

```
mcp__scheduled-tasks__update_scheduled_task(
  taskId: "<taskId>"
)
```

Or suggest the user wait for the next scheduled run and check results afterward.

---

## Handling Edge Cases

### Project Has No Roadmap or AGENTS.md

Invoke `project-design-and-development` skill first:

```
Skill("project-design-and-development")
```

This creates all required documentation artifacts. After completion, restart from Step 1.2.

### Project Uses Non-Standard Roadmap Location

Search for roadmap-like files:

```bash
find <project_path> -maxdepth 3 -iname "*roadmap*" -o -iname "*tracking*" | head -10
```

If found at a non-standard path, use that path. If not found, flag that the `project-design-and-development` skill should be invoked.

### Task Already Exists for This Project

If a scheduled task with the same or similar ID exists:

1. Show the user the existing task details.
2. Ask whether to **update** the existing task (preserving task ID and history) or **create a new one** with a different ID.
3. To update, use `mcp__scheduled-tasks__update_scheduled_task` with the new prompt and/or cron expression.

### Multiple Projects in One Repository

If the repo contains multiple independent projects (monorepo), create a separate scheduled task for each project. Each task should have its own `cd` path, roadmap reference, and AGENTS.md scope.

---

## Reference Files

- **`references/scheduled-task-prompt-template.md`** — full prompt template with all 7 sections, placeholder definitions, and examples for each ecosystem
- **`references/project-discovery-checklist.md`** — step-by-step checklist for gathering all information needed to assemble the prompt, with exact discovery commands
