# sample-todolist

A small todo web app: add, tick, rename, delete todos. Remembers everything across reloads. Works with keyboard alone.

## The app

```bash
pnpm install
pnpm dev      # http://localhost:5180
pnpm test
pnpm build
pnpm check    # types, tests and build
```

---

The sample project for [Xaedalon Factory](https://github.com/xaedalon/factory-community): a small
todo web app, specified as ten tasks with a dependency graph, and built one task at a time by a
coding agent under Factory's supervision.

**The app is not in here, and that is the point.** This repository holds the specification and the
pipeline. What an agent produces from them — the HTML, the stylesheets, the TypeScript, the tests —
is the exercise.

## What to read

[**`PROJECT.md`**](PROJECT.md) is the source of truth: the stack, the architecture, the
look-and-feel contract, the final markup, and ten tasks. Each task has a list of the files it may
touch, Gherkin scenarios that become its tests, and a "Done when" list. It is written so that a
*small* model can do any one task without guessing: every decision two tasks have to agree on is
made there, once.

The ten form a graph rather than a chain: tasks 3 and 4 can run at the same time, and so can 7 and
8. The dependency table is in PROJECT.md.

## The pipeline

Five workflows, one phase each, all run by one agent:

| Workflow | What it does | Artifact |
|---|---|---|
| `analysis` | reads the task's section of PROJECT.md. **Stops for approval** (`approval: after`) | `analysis.md` |
| `design` | plans the change | `design.md` |
| `implement` | writes the code and its tests | `implement.md` |
| `validate` | runs the tests | `validate.md` |
| `verify` | checks the result against "Done when" | `verify.md` |

A task lists all five. The workflows have no `needs:` between them, so picking one does not pull in
the rest.

The agent is [`developer-haiku`](.xaedalon/.factory/agents/developer-haiku.agent.yaml):
`provider: claude`, `model: haiku`, `session: task`, which keeps one conversation across a task's
phases. It carries `args: ['--allowedTools', 'Bash(pnpm *)']`, and that line matters. Factory's
Default security profile starts Claude with `--restricted --permission-prompts none`. From Claude
Code 2.1.281 on, that denies `pnpm`, so the agent can neither install nor test, and it reports
success anyway. The allow rule lets `pnpm` through and nothing else: a write outside the workspace
is still refused. Two things that look equivalent do not work. A `.claude/settings.json` allow rule
is ignored in that mode, and a bare `--allowedTools Bash` lets writes escape the workspace.

## Using it with Factory

1. Add the repository as a project in the Factory app. The pipeline comes with the clone: Factory
   loads the definitions in [`.xaedalon/.factory/`](.xaedalon/.factory), and there is nothing to
   import.
2. Create the ten tasks from PROJECT.md, named `T<n>: <title>`, each with the five workflows.
3. Declare the dependencies from PROJECT.md's table in the app. Factory's MCP cannot set them.
4. Queue T1. Each task stops after `analysis` for your approval.

**Worktrees and environments are off** for this project, so every task works in the checkout
itself. Run the tasks one at a time: two tasks running together would edit the same files. Turn
worktrees on before you let 3 and 4, or 7 and 8, run in parallel. No workflow commits, so the work
stays in the working tree for you to review.

### Supervising a run

Supervising is what took this sample from "every phase passed" to an app that works. What that
means in practice:

- **Check the result, not the ticks.** An agent that cannot run a command tends to write "✅ will
  pass". Take a task as done when `pnpm check` passes in the working tree, not when `verify` says
  so. For anything a person uses, also try it in a browser. The rename and keyboard-focus bugs got
  past every test and were found there.
- **Fix the spec, then re-run the same task.** When a result is wrong, put the fix in PROJECT.md
  (or the task's description), then re-run implement → validate → verify on **that** task. Do not
  create a copy.
  - A task that is **`done`** has every workflow switched off, and the MCP cannot switch one back
    on. Re-enable the ones you need in the app, then queue the task.
  - A task **cancelled** mid-run keeps its unrun workflows enabled, and can be queued again at once.
- **Watch for stalls.** A run's log is empty until it finishes, so a looping agent looks the same
  as a busy one. If an implement step runs much longer than usual, run the tests in the working tree
  to see where it is stuck.

### Known issues

- **`design` does nothing.** It completes in a few milliseconds with no steps and no artifact on
  every task, although `factory phase get design` shows its one step. The task's `progress.total`
  leaves it out as well.
- **The phase prompts are one line** (`Implement Task: "{{ task.name }}" "{{ task.description }}"`).
  They work here because every task description points at PROJECT.md. They do not point the agent
  at the previous phase's artifact.
- **The MCP can create tasks and workflows, but not projects, phases, agents or task
  dependencies.** Those come from the app, or from YAML under `.xaedalon/.factory/`.
- **Two checks from task 10 are not yet automated:** 200% zoom and `prefers-reduced-motion`. They
  were checked by reading the code, not in a browser.

## What ships in `.xaedalon/.factory/`

| | |
|---|---|
| `config.yaml` | `scope: project`, which is what makes these the project's own definitions |
| `workflows/` | the five above |
| `phases/` | one per workflow |
| `agents/developer-haiku.agent.yaml` | who does the work, and the `pnpm` allowance |

`.xaedalon/.gitignore` keeps what runs produce out of git: `.factory/tasks/` (each task's
artifacts), `.factory/state/` and `.factory/.trash/`. Everything else in the directory is committed,
so anyone who clones the repository gets the pipeline.

## For agents working here

[`AGENTS.md`](AGENTS.md) carries the directives; `CLAUDE.md` points at it.
