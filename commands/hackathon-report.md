---
description: Run a multi-agent hackathon performance report — project-quality judging (hackathon-judge) plus, when a prior agent task is given, a 5-axis agent-performance scorecard (agent-evaluator) — merged into one saved Markdown report.
argument-hint: [path-to-project] [--task "description of what the agents were asked to build"]
---

# /hackathon-report

Surface for a combined hackathon report: how good is the resulting project,
and (optionally) how well did the agents that built it perform the task.

**Input**: $ARGUMENTS

---

## Mode Selection

| Input | Mode |
|---|---|
| A path (or blank for current directory) | **Project Mode** — judge the project only |
| Path + `--task "..."` | **Combined Mode** — judge the project AND score the agent run that produced it |

If the path doesn't exist or isn't a readable directory, stop and ask the user
for the correct project path.

## Phase 1 — GATHER

```bash
PROJECT_DIR="${ARG_PATH:-.}"
ls "$PROJECT_DIR"
cat "$PROJECT_DIR"/README* 2>/dev/null | head -100
git -C "$PROJECT_DIR" log --no-pager -5 --oneline 2>/dev/null
```

Confirm this looks like a hackathon submission (README/pitch present, or the
user has confirmed the path). If it's ambiguous, ask once before proceeding.

## Phase 2 — JUDGE THE PROJECT

Dispatch the `hackathon-judge` agent with:

- The project directory
- Any sponsor-specific rubric the user has supplied
- Instruction to verify claims (build/run/test), not just read the README

Collect its full scorecard (see `agents/hackathon-judge.md` for the exact
format) — this becomes the **Project Quality** section of the report.

## Phase 3 — SCORE AGENT PERFORMANCE (Combined Mode only)

Only run this phase when `--task` was given, or the user otherwise supplies
the original task + the agent transcript/final output that built the project.

Dispatch the `agent-evaluator` agent with:

- The original task description (`--task` value)
- The agents' final output/transcript (ask the user for it if not already in
  context — do not fabricate one)

Collect its full scorecard (see `agents/agent-evaluator.md`) — this becomes
the **Agent Performance** section of the report.

## Phase 4 — MERGE AND SAVE

Combine both scorecards (or just the project scorecard in Project Mode) into
a single Markdown report:

```markdown
# Hackathon Report — <project name>
Generated: <ISO date>

## Project Quality (hackathon-judge)
<scorecard from Phase 2, verbatim>

## Agent Performance (agent-evaluator)
<scorecard from Phase 3, verbatim — omit this section entirely in Project Mode>

## Combined Verdict
<1-3 sentences: is the project demo-ready, and if a task was scored, did the
agent run meet the task well — tie the two together only if both ran>
```

Save it to `<PROJECT_DIR>/.ecc/hackathon-reports/<slug>-<YYYY-MM-DD>.md`
(create the `.ecc/hackathon-reports/` directory if missing) and print the
full report to the user. Do not overwrite an existing report from the same
day for the same project without asking.

## Notes

- This command never modifies the project under review — both agents are
  read-only (see each agent's Bash Tool Constraints).
- For a `GITHUB_APP.md`-style recurring setup (e.g. the ECC GitHub App
  commenting this report automatically on each hackathon team's PR), this
  command is the piece that runs per-repo; wiring a GitHub Action/webhook to
  invoke it automatically is a separate, explicit setup step — do not assume
  CI access without the user configuring it.
