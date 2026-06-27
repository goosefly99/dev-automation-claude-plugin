# dev-automation — agent guidance

This plugin gives you a set of senior-engineering workflows: architecture decisions, system
design, code review, debugging, deploy checklists, technical documentation, incident response,
standup updates, tech-debt triage, testing strategy, and autonomous scheduled development. Each
workflow is a skill under `skills/`. This file is the agent-native version of those skills so you
can run them without Claude Code's slash commands — trigger on the user's intent, not a command
name.

How to use this file: when a request matches one of the trigger descriptions below, follow that
section's steps and produce the described output. The deliverable is almost always a single
structured Markdown document you write for the user (or a file in the repo when they ask you to
save it). Stay concrete: ground every recommendation in the actual code, requirements, and
constraints in front of you, and make trade-offs explicit rather than asserting one right answer.

Tool note: several workflows are richer when you have access to source control, monitoring, a
ticket tracker, or chat history. Use whatever read and search tools you have (file reads, repo
grep, git log, web fetch, the host's CLI). When such a source is not available, do the
standalone version from the information the user gives you — never block waiting for a connector
that does not exist here.

---

## Architecture decision records and design evaluation

Trigger: the user is choosing between technologies (for example Kafka versus SQS), wants a design
decision written down with trade-offs and consequences, asks you to review a system-design
proposal, or wants a new component designed from requirements and constraints.

What to do: pull the constraints into the open first — deadline, scale target, team familiarity,
existing stack, cost ceiling. Name at least two real options even when the user is leaning one
way; a balanced comparison is the value. Use available read and search tools to find prior
decision records and related design docs in the repo so you do not contradict an earlier choice.
For deeper requirements gathering, scalability analysis, and trade-off framing, lean on the
system-design workflow below.

Output — an Architecture Decision Record:

- Title, status (Proposed / Accepted / Deprecated / Superseded), date, and who signs off.
- Context: the situation and the forces in play.
- Decision: the change being proposed.
- Options considered: each option with a small table scoring complexity, cost, scalability, and
  team familiarity, plus its pros and cons.
- Trade-off analysis: the key tensions between options, reasoned explicitly.
- Consequences: what gets easier, what gets harder, what you will need to revisit.
- Action items: a short checklist of implementation and follow-up steps.

## System design

Trigger: "design a system for", "how should we architect", "what's the right architecture for",
or any request touching API design, data modeling, or service boundaries.

What to do: work the framework in order. (1) Requirements — separate functional from
non-functional (scale, latency, availability, cost) and capture hard constraints. (2) High-level
design — components, data flow, API contracts, storage choices. (3) Deep dive — data model, API
endpoint design across REST, GraphQL, or gRPC, caching, queue and event design, error handling
and retries. (4) Scale and reliability — load estimate, horizontal versus vertical scaling,
failover and redundancy, monitoring and alerting. (5) Trade-offs — make every decision's cost
explicit across complexity, cost, familiarity, time to market, and maintainability.

Output: a structured design document with diagrams (ASCII or described in words), stated
assumptions, explicit trade-off analysis, and a clear note on what you would revisit as the
system grows.

## Code review

Trigger: a pull-request URL, a diff, a file path, "review this before I merge", "is this code
safe?", or a question about N+1 queries, injection risk, missing edge cases, or error handling.
If nothing concrete is supplied, ask what to review.

What to do: pull the change in with whatever access you have (read the files, fetch the diff via
git or the host CLI) and review across four lenses. Security — injection (SQL, XSS, CSRF), auth
flaws, secrets in code, insecure deserialization, path traversal, SSRF. Performance — N+1
queries, needless allocations, hot-path complexity, missing indexes, unbounded loops or queries,
resource leaks. Correctness — empty/null/overflow edge cases, races and concurrency, error
propagation, off-by-one, type safety. Maintainability — naming, single responsibility,
duplication, test coverage, docs for non-obvious logic. Tie findings to a file and line, and give
an actionable fix with a code example, not just a complaint.

Output: a review with a one-or-two-sentence summary, a table of critical issues (file, line,
issue, severity), a table of non-blocking suggestions (file, line, suggestion, category), a short
"what looks good" list, and a verdict of Approve, Request Changes, or Needs Discussion.

## Debugging

Trigger: an error message or stack trace, "works in staging but not prod", "broke after the
deploy", or behavior diverging from expectation where the cause is not obvious.

What to do: run the four phases. Reproduce — pin down expected versus actual, the exact repro
steps, and the scope (when it started, who is affected). Isolate — narrow to a component or code
path; check recent changes (deploys, config, dependency bumps) since those are the top suspects;
read the logs and exact error text. Diagnose — form hypotheses and test them, trace the code
path, and find the root cause rather than the symptom. Fix — propose the change with reasoning,
weigh side effects and edge cases, and name a regression test or guard to add. Treat exact error
strings as load-bearing; do not paraphrase them.

Output: a debug report with reproduction (expected, actual, steps), the root cause, the fix
(code or config), and prevention (the test to add and the guard to put in place).

## Deploy checklist

Trigger: about to ship a release, deploying a change with database migrations or feature flags,
verifying CI status and approvals before production, or wanting rollback triggers documented up
front.

What to do: generate a checklist tailored to the deploy. Adapt it to what the user tells you —
feature flags add flag-verification steps, a database migration adds migration-specific checks, a
breaking API change adds consumer-notification steps. Where you can read CI status, the release
diff, or current monitoring baselines, pre-fill those instead of leaving placeholders.

Output: a checklist in three phases plus triggers. Pre-deploy — tests green in CI, code reviewed
and approved, no known critical bugs, migrations tested, flags configured, rollback plan written,
on-call notified. Deploy — ship to staging and verify, smoke tests, production rollout (canary if
available), watch error rate and latency for the first minutes, verify key user flows.
Post-deploy — confirm metrics nominal, update the changelog, notify stakeholders, close tickets.
Rollback triggers — concrete thresholds (error rate over X percent, P50 latency over X ms, a
named critical flow failing).

## Technical documentation

Trigger: "write docs for", "document this", "create a README", "write a runbook", "onboarding
guide", or any technical-writing request including API docs, architecture docs, or runbooks.

What to do: identify the reader and what they need before writing a word, then pick the right
shape. A README covers what this is and why, a quick start that reaches first success in under
five minutes, configuration and usage, and how to contribute. API docs cover endpoint reference
with request and response examples, auth, error codes, rate limits, pagination, and SDK examples.
A runbook covers when to use it, prerequisites and access, the step-by-step procedure, rollback
steps, and the escalation path. An architecture doc covers context and goals, high-level design
with diagrams, key decisions and trade-offs, and data flow. An onboarding guide covers
environment setup, how the key systems connect, common tasks with walkthroughs, and who to ask
for what.

Output: the finished document. Lead with the most useful information, show with examples and
commands rather than telling, link instead of duplicating, and keep it current — stale docs are
worse than none.

## Incident response

Trigger: "we have an incident", "production is down", an alert needing a severity call, a mid-
incident status update, or a postmortem after resolution. If the phase is unclear, ask whether
this is a new incident, an update, or a postmortem.

What to do: work the phase you are in. Triage — assess severity (SEV1 service down all users;
SEV2 major feature degraded, many users; SEV3 minor feature, some users; SEV4 cosmetic),
identify affected systems and users, and assign incident-commander, comms, and responder roles.
Communicate — draft factual internal and, if needed, customer updates on a fixed cadence: what is
happening, who is affected, what you are doing, when the next update lands. Mitigate — record
mitigation steps and a running timeline, and confirm resolution. Postmortem — write it blameless,
focused on systems and process, not people.

Output: for a live incident, a status update (severity, status of Investigating / Identified /
Monitoring / Resolved, impact, current status, actions taken, next steps with ETA, and a
timeline table). For the wrap-up, a postmortem with summary, impact, timeline, root cause, a
five-whys chain, what went well and poorly, action items with owners and due dates, and lessons
learned.

## Standup updates

Trigger: preparing for daily standup, summarizing recent commits, PRs, and ticket moves, or
turning a few rough notes into a shareable update.

What to do: gather recent activity from whatever you can read — git log for commits and PRs,
ticket status changes, relevant chat decisions — or take the user's plain description of what
they did. Keep it concise and action-oriented, and reference tickets where they exist.

Output: a standup note in three sections — Yesterday (completed items), Today (planned items),
Blockers (each with context and who can unblock it). Offer to reformat for Slack, email, or the
team's tool.

## Tech debt

Trigger: "tech debt", "technical debt audit", "what should we refactor", "code health", or any
question about code quality, refactoring priorities, or the maintenance backlog.

What to do: identify debt and sort it into categories — code debt (duplication, poor
abstractions, magic numbers), architecture debt (a monolith that should split, the wrong data
store), test debt (low coverage, flaky or missing tests), dependency debt (outdated or
unmaintained libraries, security exposure), documentation debt (missing runbooks, stale READMEs,
tribal knowledge), and infrastructure debt (manual deploys, no monitoring, no IaC). Score each
item on impact (how much it slows the team), risk (what happens if ignored), and effort, then
rank by roughly (impact plus risk) weighted against effort so low-effort high-value items rise.

Output: a prioritized list, each item with estimated effort, a business justification, and a
phased remediation plan that can run alongside feature work.

## Testing strategy

Trigger: "how should we test", "test strategy for", "write tests for", "test plan", "what tests
do we need", or any question about testing approach, coverage, or test architecture.

What to do: shape the strategy around the testing pyramid — many fast focused unit tests, some
medium integration tests, few slow high-confidence end-to-end tests. Tailor by component: API
endpoints get unit tests for logic, integration tests for the HTTP layer, and contract tests for
consumers; data pipelines get input validation, transformation correctness, and idempotency
checks; frontends get component, interaction, visual-regression, and accessibility tests;
infrastructure gets smoke, chaos, and load tests. Focus coverage on business-critical paths,
error handling, edge cases, security boundaries, and data integrity; skip trivial
getters/setters, framework code, and one-off scripts.

Output: a test plan listing what to test, the test type for each area, coverage targets, example
test cases, and the gaps in existing coverage.

## Scheduled development tasks (Claude-only automation — run manually under Codex)

Trigger: "create a scheduled development task", "set up an autonomous dev agent", "schedule a
development loop", "automate development on a schedule", or any request to configure a recurring
task that autonomously works through a project's roadmap one item per run.

Important runtime note: the install step of this workflow depends on Claude Code's
`scheduled-tasks` MCP server (the tool that registers and schedules the task). That MCP is
Claude-only and is not available under Codex. So under Codex you perform every preparation step
manually and then hand the operator a ready-to-install prompt rather than installing it yourself.

What to do:

1. Project discovery. Confirm the project name, absolute path, and a one-sentence description.
   Check the project is ready for autonomous development: it must have a development roadmap (a
   roadmap or tracking file with phase tables carrying status, dependencies, and target files),
   per-area AGENTS.md guidance, and design docs or ADRs. If any are missing, the project is not
   ready — scaffold the missing design docs, AGENTS.md files, README, and roadmap first (this is
   the job of the companion project-design-and-development workflow; under Codex, do that
   authoring directly), then re-check.

2. Extract the data the recurring prompt needs: the git branch and the branch-setup commands
   (cd into the project, checkout, pull), the roadmap's relative path, a table of every reference
   document the agent will read (AGENTS.md files, ADRs, design docs, handoff specs, CLAUDE.md),
   and the exact build, test, and lint commands from the project's package manifest or Makefile.

3. Assemble the recurring prompt. It must be fully self-contained — the scheduled run is a fresh
   session with no access to this guidance, so every instruction is inline. Resolve all
   placeholders. Do not bake in phase-specific progress or item descriptions; the agent reads the
   roadmap at runtime to decide what to work on next, which keeps the prompt durable. Always
   instruct the recurring run to spawn a fresh subagent each time for isolation. Keep the prompt
   roughly 100 to 250 lines. Validate it: no unresolved placeholders, every referenced path
   exists, the roadmap path is right, the branch exists or has creation commands, and the
   build/test/lint commands are valid for the toolchain.

4. Pick a schedule with the operator (default hourly) and always use an offset minute rather than
   :00 or :30 to spread load. Choose a task id of the form project-name-dev.

5. Report, do not install. Present the operator the assembled prompt, the chosen cron expression,
   the task id, the target project path and branch, the roadmap summary (items, phases, current
   completion), and the commands the agent will run. State plainly that installing the schedule
   requires Claude Code's `scheduled-tasks` MCP, so they (or a Claude session) must register it
   there; everything else is done and verified.

Output: the complete, validated, self-contained recurring prompt plus its cron expression, task
id, and an install note for the operator — the deliverable an operator can paste straight into
the Claude-side scheduler.

---

## Conventions across every workflow

- Understand the problem before producing the artifact: read the task and the code it touches,
  trace the real flow, and only then write the document. A confident answer built on a wrong
  mental model is worse than asking one clarifying question.
- Ground recommendations in the actual repo and constraints, not generic best practice. State
  assumptions you had to make.
- Make trade-offs explicit. There is rarely one right answer; show the alternatives and why you
  chose one.
- Be concise and action-oriented. Tables and checklists over prose where they carry the
  information better.
- Degrade gracefully when a source control, monitoring, tracker, or chat integration is not
  available — do the standalone version from what the user gave you.
