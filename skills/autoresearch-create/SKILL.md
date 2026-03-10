---
name: autoresearch-create
description: Create a new autoresearch skill for any optimization target. Generates a domain-specific SKILL.md that drives the autoresearch extension's experiment loop. Use when asked to "create an autoresearch skill", "set up autoresearch for X", or "make a new experiment loop".
---

# Create an Autoresearch Skill

Generate a domain-specific skill that drives the autoresearch extension (`run_experiment` + `log_experiment` tools, widget, `/autoresearch` dashboard).

## Gather Inputs

Ask the user for these (propose smart defaults from context):

1. **Name** — kebab-case skill name, e.g. `autoresearch-vitest-speed`
2. **Goal** — one sentence: what are we optimizing?
3. **Metric** — what number do we measure? (e.g. wall-clock seconds, val_bpb, bundle size KB)
4. **Direction** — lower is better or higher is better?
5. **Command** — the shell command to run per experiment (e.g. `pnpm test:vitest`, `uv run train.py`)
6. **Files in scope** — what can the agent modify?
7. **Files read-only** — what must NOT be modified?
8. **Constraints** — hard rules (e.g. "all tests must pass", "don't delete files")
9. **Ideas** — 5-10 things to try, roughly ordered by expected impact
10. **Working directory** — where to run from

## Generate the Skill

Create the skill at `~/.pi/agent-shopify/skills/<name>/SKILL.md` following this template:

```markdown
---
name: {{name}}
description: {{one-line description including trigger phrases}}
---

# autoresearch — {{Goal}}

Autonomous experiment loop to {{goal description}}.

## Tools

You have two custom tools from the autoresearch extension. **Always use these instead of raw bash:**

- **`run_experiment`** — pass it a `command` to run. It times execution, captures output, detects pass/fail. Returns structured results.
- **`log_experiment`** — records each experiment's `commit`, `metric` ({{metric unit}}), `status` (keep/discard/crash), and `description`. Persists state, updates widget and dashboard.

The user can view all results anytime with `/autoresearch`.

## Setup

1. **Agree on a run tag** with the user (e.g. `{{example-tag}}`).
2. **Create a branch**: `git checkout -b autoresearch/<tag>` from HEAD.
3. **Read the codebase** to understand what you're working with:
   {{list of files to read with brief descriptions}}
4. **Confirm and go.**

## The Metric

**{{Metric name}}** — {{direction}} is better.

## Constraints

{{bullet list of hard constraints}}

## What's In Scope

You CAN modify:
{{bullet list of editable files/patterns}}

You CANNOT modify:
{{bullet list of read-only files/patterns}}

## What to Try

Ideas roughly ordered by expected impact:

{{numbered list of 5-10 experiment ideas with brief rationale}}

## The Experiment Loop

LOOP FOREVER:

1. Edit files with an experimental idea
2. `GIT_EDITOR=true git add -A && git commit -m "description"`
3. Use `run_experiment` with command `{{command}}`
4. Use `log_experiment` to record the result
5. If {{metric}} improved → keep (status: `keep`)
6. If {{metric}} worse OR broken → revert with `git reset --hard HEAD~1` (status: `discard` or `crash`)
7. Repeat

**NEVER STOP.** Loop indefinitely. Do not ask for permission to continue.

**Crashes**: If it's a trivial fix, fix and retry. If fundamentally broken, discard and move on.
```

## After Generating

1. Write the file to `~/.pi/agent-shopify/skills/<name>/SKILL.md`
2. Tell the user to `/reload` to pick it up
3. Tell the user they can now run it with `/skill:{{name}}`
4. Show a preview of the generated skill

## Examples of Good Skills

**Test speed**: metric=seconds, direction=lower, command=`pnpm test`, scope=vitest configs, constraint=all tests must pass

**Bundle size**: metric=KB, direction=lower, command=`pnpm build && du -sb dist`, scope=webpack/vite config + imports, constraint=app must still work

**LLM training**: metric=val_bpb, direction=lower, command=`uv run train.py`, scope=train.py only, constraint=5-min time budget

**Build speed**: metric=seconds, direction=lower, command=`pnpm build`, scope=tsconfig + bundler config, constraint=output must be identical

**Lighthouse score**: metric=performance score, direction=higher, command=`lighthouse http://localhost:3000 --output=json`, scope=components + loading strategy, constraint=no feature removal
