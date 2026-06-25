# SUSTAIN Phase — Dispatcher

This phase manages tasks T11 (Technical Writer), T12 (Skill Maker), and T13 (Compound Learning + Final Assembly). Features PARALLEL #7.

## Visual Output

Print pipeline dashboard with SUSTAIN ● active on phase start, then:

```
  → Starting SUSTAIN phase (documentation + skills)
```

On PARALLEL #7 completion:
```
┌─ SUSTAIN COMPLETE ────────────────────────── ⏱ {time} ─┐
│                                                          │
│  ✓ Technical Writer    {N} docs (API ref, dev guide...)  │
│  ✓ Skill Maker         {N} project-specific skills       │
│                                                          │
│  → Final assembly and compound learning                  │
└──────────────────────────────────────────────────────────┘
```

After T13 completes, print the final summary template from the orchestrator.

## Re-Anchor

Before creating SUSTAIN agent tasks, re-read from disk:
- All receipts from `.orchestrator/receipts/` (complete pipeline history for compound learning)
- `infrastructure/` listing, `.github/workflows/` listing
- `docs/architecture/` listing

## PARALLEL #7: T11 + T12

Delegate the following to their subagents to run CONCURRENTLY (each is backgrounded + isolated in its own worktree per its definition):

```python
TaskUpdate(taskId=t11_id, status="in_progress")
TaskUpdate(taskId=t12_id, status="in_progress")
```

- **`technical-writer` (T11)** — Delegate to the `technical-writer` subagent (agents/technical-writer.md — runs backgrounded in its own worktree per its definition). Task context: read ALL workspace folders at `Drydock/` for full project context; read all project deliverables (`api/`, `services/`, `frontend/`, `infrastructure/`, `tests/`, `docs/`); read protocols from `Drydock/.protocols/`; read `.drydock.yaml` for paths and preferences. Produce: API reference (from OpenAPI specs), developer guides, operational guide, architecture guide, contributing guide; if `features.documentation_site` is true, scaffold a Docusaurus site. Write docs to project root `docs/` and workspace artifacts to `Drydock/technical-writer/`. When complete, write a receipt JSON to `Drydock/.orchestrator/receipts/T11-technical-writer.json` with task, agent, phase, status, artifacts, metrics, effort, verification, then mark its task complete.

- **`skill-maker` (T12)** — Delegate to the `skill-maker` subagent (agents/skill-maker.md — runs backgrounded in its own worktree per its definition). Task context: analyze the completed project for recurring patterns (API routes, DB queries, auth checks, deployment procedures, testing patterns, domain-specific workflows); read protocols from `Drydock/.protocols/`. Produce 3-5 project-specific skills as `SKILL.md` files; install them to `.claude/skills/` and write workspace artifacts to `Drydock/skill-maker/`. When complete, write a receipt JSON to `Drydock/.orchestrator/receipts/T12-skill-maker.json` with task, agent, phase, status, artifacts, metrics, effort, verification, then mark its task complete.

## Worktree Merge-Back

The SUSTAIN subagents (`technical-writer`, `skill-maker`) edit isolated `worktree` branches per their definitions. After the wave completes, merge those subagent worktree branches back:

```python
for branch in sustain_worktree_branches:
  Bash(f"git merge --no-ff {branch} -m 'drydock: merge {branch}'")
  Bash(f"git branch -d {branch}")
```

## T13: Compound Learning + Final Assembly

After T11 and T12 complete (and worktree branches are merged):

```python
TaskUpdate(taskId=t13_id, status="in_progress")
```

### Compound Learning

Write to `Drydock/.orchestrator/compound-learnings.md`:

```markdown
## Learning: [date] — [project name]

### What Worked
- [patterns, decisions, tools that worked well]

### What Failed
- [errors encountered, root causes, how they were fixed]

### Architecture Insights
- [patterns that emerged, tech stack fit/misfit]

### Time Sinks
- [phases that took longest, what slowed them down]

### Skip Next Time
- [unnecessary steps for this project type]

### Add Next Time
- [missing steps that should have been included]
```

### Drydock Native Directive (MANDATORY)

Write the following block to the project's `CLAUDE.md` (create if it doesn't exist, append if it does — but check for existing `# Drydock Native` header first to avoid duplicates):

```markdown
# Drydock Native

This project was built with the Drydock plugin. The `Drydock/` directory contains architecture decisions, security findings, test plans, and receipts from the build pipeline.

**At the start of every session, ask the user how they'd like to work.** Use AskUserQuestion:
- Header: "Drydock Native Project"
- Question: "This project was built with the Drydock pipeline. How would you like to work today?"
- Options:
  1. "Use Drydock (Recommended)" — "Route changes through specialized agents — architecture, security, and test baselines stay intact. Best for features, refactors, and anything that touches system behavior."
  2. "Work directly without the plugin" — "Make changes freely. Good for quick fixes, experiments, or when you know exactly what you're changing. You can always invoke /drydock later if needed."
  3. "Chat about this" — "Let's discuss what I'm planning and figure out the best approach together."

If the user chooses Drydock, invoke `/drydock` — it auto-routes to the right mode:
- Adding features → Feature mode
- Refactoring / architecture → Architect mode
- Code review → Review mode
- Adding tests → Test mode
- Security hardening → Harden mode
- Deployment changes → Ship mode
- Brainstorming → Explore mode
- Performance → Optimize mode

If the user chooses to work directly, respect that choice fully — no further reminders this session. They can always invoke `/drydock` manually if they change their mind.

**Why this exists:** This project has architecture decisions (ADRs), API contracts, security baselines, and test coverage established by the pipeline. The Drydock plugin ensures changes go through the right specialized agents — but it's always the user's call. The plugin won't run the full pipeline for a feature request; it adapts to the scope of work.
```

**Why this is mandatory:** Without this directive, new Claude Code sessions treat the project as a regular codebase and make ad-hoc changes — violating ADRs, skipping tests, ignoring security baselines. The directive gives the user an informed choice at every session start.

Optionally also append key project patterns (tech stack, conventions, common commands) to CLAUDE.md for cross-session persistence.

### Final Assembly

1. **Integration decision** — ask user via AskUserQuestion:
```python
AskUserQuestion(questions=[{
  "question": "Code is ready. Integrate into your project root?",
  "header": "Assembly",
  "options": [
    {"label": "Integrate all code (Recommended)", "description": "Copy services, frontend, infra to project root"},
    {"label": "Keep in workspace only", "description": "Leave everything in Drydock/"},
    {"label": "Let me choose what to copy", "description": "Select which components to integrate"},
    {"label": "Chat about this", "description": "Discuss integration strategy"}
  ],
  "multiSelect": false
}])
```

2. **Run final validation:** `docker-compose up`, `make test`, `terraform validate`, health checks.

3. **Present final summary** using the orchestrator's template.

4. **Finalize:**
```python
TaskUpdate(taskId=t13_id, status="completed")
```
Delegated subagents finish on their own and their worktrees auto-clean — no team teardown is needed.

## Pipeline Complete

Print the final summary template from the orchestrator. All tasks should show as completed in TaskList.
