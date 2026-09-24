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
todo web app, specified as ten tasks with a dependency graph, for coding agents to build one task at
a time under Factory.

**Neither the app nor the pipeline is in here, and that is the point.** This repository holds only
the specification. Building the Factory pipeline that runs it (phases, workflows, an agent) is the
first half of the exercise. What the agents produce from it (the HTML, the stylesheets, the
TypeScript, the tests) is the second.

## What to read

[**`PROJECT.md`**](PROJECT.md) is the source of truth: the stack, the architecture, the
look-and-feel contract, the final markup, and ten tasks. Each task has a list of the files it may
touch, Gherkin scenarios that become its tests, and a "Done when" list. It is written so that a
*small* model can do any one task without guessing: every decision two tasks have to agree on is
made there, once.

The ten form a graph rather than a chain: tasks 3 and 4 can run at the same time, and so can 7 and
8. The dependency table is in PROJECT.md.

## Building the pipeline

Add the repository as a project in the Factory app, then build the pipeline yourself. Factory keeps
it under `.xaedalon/.factory/`, which this repository ignores so that each person builds their own.
A shape that is known to work on this spec:

| Workflow | One phase that… | Artifact |
|---|---|---|
| `analysis` | reads the task's section of PROJECT.md (a good place for `approval: after`) | `analysis.md` |
| `design` | plans the change | `design.md` |
| `implement` | writes the code and its tests | `implement.md` |
| `validate` | runs the tests | `validate.md` |
| `verify` | checks the result against "Done when" | `verify.md` |

One agent can run all five. With `session: task`, it keeps one conversation across a task's phases,
which is what you want when a later phase has to act on an earlier one. A small model such as
`provider: claude`, `model: haiku` is enough, provided the spec is as precise as this one.

Things worth knowing before you start, all learned the hard way:

- **Make sure the agent can run `pnpm`.** Factory's Default security profile starts Claude with
  `--restricted --permission-prompts none`. From Claude Code 2.1.281 on, that denies `pnpm`, so the
  agent can neither install nor test, and it reports success anyway. Give the agent
  `args: ['--allowedTools', 'Bash(pnpm *)']`, which lets `pnpm` through and nothing else: a write
  outside the workspace is still refused. Two things that look equivalent do not work. A
  `.claude/settings.json` allow rule is ignored in that mode, and a bare `--allowedTools Bash` lets
  writes escape the workspace.
- **Write prompts that ask for evidence.** A one-line prompt (`Implement Task: "{{ task.name }}"`)
  works when the task description points at PROJECT.md. But a validate or verify phase that is not
  told to paste real `pnpm check` output will happily write "✅ will pass". Point each phase at the
  previous phase's artifact (`{{ task.artifacts }}/<phase>/<phase>.md`) too.
- **Chain the workflows or list them all.** Without `needs:` between the workflows, a task has to
  list all five, in order.
- **Keep run output out of git.** Factory writes each task's artifacts under
  `.xaedalon/.factory/tasks/`. This repository ignores the whole of `.xaedalon/`. If you share your
  pipeline in a fork, ignore at least `.factory/tasks/`, `.factory/state/` and `.factory/.trash/`.

## Running the tasks

1. Create the ten tasks from PROJECT.md, named `T<n>: <title>`, each with your workflows.
2. Declare the dependencies from PROJECT.md's table in the app. Factory's MCP cannot set them.
3. Queue T1.

**Without worktrees, every task works in the checkout itself.** Run the tasks one at a time then,
because two tasks running together would edit the same files. Turn worktrees on before you let 3 and
4, or 7 and 8, run in parallel.

### Supervising a run

Supervising is what takes a run from "every phase passed" to an app that works. What that means in
practice:

- **Check the result, not the ticks.** An agent that cannot run a command tends to write "✅ will
  pass". Take a task as done when `pnpm check` passes in the working tree, not when `verify` says
  so. For anything a person uses, also try it in a browser. In the reference run, a rename bug and a
  keyboard-focus bug got past every test and were found there.
- **Fix the spec, then re-run the same task.** When a result is wrong, put the fix in PROJECT.md
  (or the task's description), then re-run implement → validate → verify on **that** task. Do not
  create a copy.
  - A task that is **`done`** has every workflow switched off, and the MCP cannot switch one back
    on. Re-enable the ones you need in the app, then queue the task.
  - A task **cancelled** mid-run keeps its unrun workflows enabled, and can be queued again at once.
- **Watch for stalls.** A run's log is empty until it finishes, so a looping agent looks the same
  as a busy one. If an implement step runs much longer than usual, run the tests in the working tree
  to see where it is stuck.

### Known Factory issues

- **A phase can run zero steps.** In the reference run, `design` completed in a few milliseconds
  with no steps and no artifact on every task, although `factory phase get design` showed its one
  step. Check that each phase actually wrote its artifact.
- **The MCP can create tasks and workflows, but not projects, phases, agents or task
  dependencies.** Those come from the app, or from YAML under `.xaedalon/.factory/`.
## For agents working here

[`AGENTS.md`](AGENTS.md) carries the directives; `CLAUDE.md` points at it.
