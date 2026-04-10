# Project Discovery Checklist

Use this checklist during Step 1 (Project Discovery) to gather all information needed to generate the scheduled task prompt. Every placeholder in the prompt template must be resolved before task creation.

---

## 1. Project Identity

- [ ] **Project name** — short, kebab-case identifier (used as `taskId` and in the prompt). Example: `librefrets`, `taskflow-api`.
- [ ] **Project path** — absolute path on the local filesystem. Example: `C:\Users\olive\Documents\librefrets`.
- [ ] **Project description** — one sentence describing what the project is and its tech stack. Example: `an interactive guitar fretboard SPA (TypeScript + React + Vite + SVG + tonal)`.

## 2. Git Configuration

- [ ] **Branch name** — which branch the agent should work on. Default: `main`.
- [ ] **Branch exists?** — check if the branch exists. If not, include branch creation commands.
- [ ] **Remote configured?** — check if `origin` remote exists and the agent should push.
- [ ] **Branch setup commands** — the exact shell commands to `cd` to the project and ensure the correct branch.

### Discovery commands:
```bash
cd <project_path>
git branch --show-current          # Current branch
git remote -v                      # Check remotes
git branch -a                      # List all branches
```

## 3. Roadmap Location

- [ ] **Roadmap file path** — relative path from project root. Common: `dev_roadmap.md`, `docs/ROADMAP.md`, `tech_debt_roadmap.md`.
- [ ] **Roadmap exists?** — verify the file exists. If not, flag that `project-design-and-development` skill should be invoked first.
- [ ] **Total item count** — how many items are in the tracking table.
- [ ] **Phase count** — how many phases.
- [ ] **Current progress** — how many items are Complete, In Progress, Not Started.

### Discovery commands:
```bash
# Check for roadmap files
ls <project_path>/dev_roadmap.md <project_path>/docs/ROADMAP.md 2>/dev/null

# Count items (grep for table rows)
grep -c "| .* | .* |" <project_path>/dev_roadmap.md
```

## 4. Reference Documents Inventory

Scan for all documentation files that the agent should read:

- [ ] **AGENTS.md files** — list every AGENTS.md in the project tree.
- [ ] **ADR files** — list all files in `docs/adr/`.
- [ ] **Design docs** — list all files in `docs/design/`.
- [ ] **CLAUDE.md** — check if it exists at the project root.
- [ ] **Other docs** — any HANDOFF-*, SPEC-*, or other reference files.

### Discovery commands:
```bash
# Find all AGENTS.md files
find <project_path> -name "AGENTS.md" -not -path "*node_modules*" -not -path "*.git*"

# Find ADRs and design docs
ls <project_path>/docs/adr/ 2>/dev/null
ls <project_path>/docs/design/ 2>/dev/null

# Check for CLAUDE.md
ls <project_path>/CLAUDE.md 2>/dev/null
```

Build the reference documents table from the results:
```markdown
| File | Purpose |
|------|---------|
| `dev_roadmap.md` | Phased task tracking |
| `docs/design/project-design.md` | Full design specification |
| `docs/adr/001-tech-stack.md` | Technology choices |
| `AGENTS.md` | Root project structure and conventions |
| `src/AGENTS.md` | Source directory layout, module boundaries |
```

## 5. Build/Test/Lint Commands

Identify the project's toolchain and construct the test, lint, and build commands:

- [ ] **Package manager** — npm, yarn, pnpm, pip, cargo, go, make.
- [ ] **Test command** — exact command to run tests.
- [ ] **Lint command** — exact command to run linter.
- [ ] **Build command** — exact command to produce a production build (if applicable).
- [ ] **All commands verified** — run each command once to confirm it works.

### Discovery by ecosystem:

**Node.js** — read `package.json` scripts:
```bash
cat <project_path>/package.json | grep -A 20 '"scripts"'
```
Typical: `npm run test`, `npm run lint`, `npm run build`.

**Python** — read `pyproject.toml` or `setup.cfg`:
```bash
cat <project_path>/pyproject.toml | grep -A 10 '\[tool\.'
```
Typical: `pytest tests/ -v`, `ruff check .`, `mypy src/`.

**Rust** — standard toolchain:
Typical: `cargo test`, `cargo clippy -- -D warnings`, `cargo build --release`.

**Go** — standard toolchain:
Typical: `go test ./...`, `golangci-lint run`, `go build ./...`.

## 6. Roadmap and Reference File Verification

Verify that the roadmap and reference files are structured well enough for the agent to derive implementation guidance at runtime (without hardcoding phase details into the prompt):

- [ ] **Roadmap has phase tables** — each phase has a table with columns for item number, description, status, dependencies, and target files.
- [ ] **Roadmap has status overview** — a summary table showing counts of Not Started, In Progress, Done, and Blocked items.
- [ ] **Roadmap has build order** — phases or items include dependency ordering or documented build sequences.
- [ ] **AGENTS.md files cover key directories** — at minimum, root AGENTS.md and source directory AGENTS.md files exist with conventions and patterns.
- [ ] **Handoff specs or design docs exist** — for complex items, corresponding specs with code contracts, type signatures, and implementation details are available.
- [ ] **Tech debt or improvement items use scoring** — if the project has tech debt or improvement items, they should include priority scoring so the agent can select the highest-value work.

If any of these are missing or inadequate, invoke the `project-design-and-development` skill to improve them before creating the scheduled task:
```
Skill("project-design-and-development")
```

The goal is for the prompt to instruct the agent to **read the roadmap and reference files at runtime** rather than embedding phase-specific guidance directly into the prompt. This keeps the prompt durable as the project evolves.

## 7. Schedule Configuration

- [ ] **Cron expression** — how often the task should run. Default: hourly (`17 * * * *` — offset from :00 to spread load).
- [ ] **User preference** — did the user specify a frequency? ("every hour", "every 2 hours", "twice daily").
- [ ] **Notify on completion** — should this session receive notifications when the task finishes? Default: true.

### Common cron patterns:
| Frequency | Cron Expression | Notes |
|-----------|----------------|-------|
| Hourly | `17 * * * *` | Offset from :00 |
| Every 2 hours | `23 */2 * * *` | Offset from :00 |
| Every 4 hours | `11 */4 * * *` | |
| Twice daily | `17 9,17 * * *` | 9am and 5pm |
| Daily at 9am | `17 9 * * *` | |
| Weekdays hourly | `17 * * * 1-5` | Mon-Fri |

## 8. Pre-Creation Verification

Before creating the scheduled task, verify:

- [ ] All placeholders in the prompt template are resolved — no `{{...}}` remain.
- [ ] The roadmap file exists and has a valid tracking table.
- [ ] At least one AGENTS.md file exists in the project.
- [ ] The git branch exists or branch creation commands are included.
- [ ] Test/lint/build commands are valid for the project's ecosystem.
- [ ] The `taskId` is unique — check against existing scheduled tasks.
- [ ] The cron expression is valid and uses an offset minute (not :00 or :30).
