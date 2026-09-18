# sample-todolist

The sample project for [Xaedalon Factory](https://github.com/xaedalon/factory-community): a small
todo web app, specified as ten tasks with a dependency graph, built one task at a time by coding
agents.

**The app is not in here, and that is the point.** This repository holds the specification. What an
agent produces from it — the HTML, the stylesheets, the TypeScript, the tests — is the exercise.

## What to read

[**`PROJECT.md`**](PROJECT.md) is the source of truth: the stack, the look-and-feel contract, and ten
tasks. Each task's name is what you type into Factory's **New task**, and its "done when" list is
what you check at the approval gate.

The ten form a graph rather than a chain — tasks 3 and 4 can run at the same time, and so can 7 and
8 — which is what makes it a useful sample: it exercises task dependencies rather than a queue.

## Using it with Factory

```bash
git clone https://github.com/xaedalon/sample-todolist.git
factory project add sample-todolist /full/path/to/sample-todolist
```

Then, on the board: create the ten tasks from `PROJECT.md`, give each one a workflow, declare the
dependencies from the table, and press **Queue all**. Factory holds each task until the one it waits
for is done, so the graph runs itself.

If you have no workflows yet, Factory's [quickstart](https://github.com/xaedalon/factory-community/blob/main/docs/quickstart.md)
builds one, and [`docs/task-dependencies.md`](https://github.com/xaedalon/factory-community/blob/main/docs/task-dependencies.md)
covers declaring the graph and the two whole-project buttons.

## For agents working here

[`AGENTS.md`](AGENTS.md) carries the directives; `CLAUDE.md` points at it.
