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

A sample project for [Xaedalon Factory](https://github.com/xaedalon/factory-community), and for any
other way of driving coding agents: a small todo web app, specified as ten tasks with a dependency
graph, for agents to build one task at a time.

**The app is not in here, and that is the point.** This repository holds the specification, and
one pipeline that runs it. How you run the tasks (which tool, which agents, which steps) is yours to
decide, and the tools change faster than this spec does. That is why the specification describes no
particular tool's configuration.

## What to read

[**`PROJECT.md`**](PROJECT.md) is the source of truth: the stack, the architecture, the
look-and-feel contract, the final markup, and ten tasks. Each task has a list of the files it may
touch, Gherkin scenarios that become its tests, and a "Done when" list. It is written so that a
*small* model can do any one task without guessing: every decision two tasks have to agree on is
made there, once.

## A pipeline for Factory

[`.xaedalon/.factory/`](.xaedalon/.factory/) holds the Factory definitions this sample has been run
with: five workflows (`analysis`, `design`, `implement`, `validate`, `verify`), their phases, the
agent they use and a security profile. Add the repository as a project in Factory and they are
there. Use them, change them or replace them. They describe the Factory version they were written
against, so check them against the one you have. Run output under `.xaedalon/.factory/tasks/`,
`state/` and `.trash/` is not versioned.

## What any run needs

These follow from the spec, whatever runs it:

- **Respect the dependency graph.** The table in PROJECT.md says which tasks come first. Tasks 3 and
  4 can run at the same time, and so can 7 and 8, but only if each works in its own copy of the
  repository. In a single checkout, run the tasks one at a time.
- **The agent must be able to run `pnpm`.** Every task installs, tests and builds with it. Check
  that your setup allows it before starting, and check again when the tool or the agent changes: an
  agent that cannot run a command may still report that it passed.
- **Keep run output out of git.** This repository versions only the scaffold (see `AGENTS.md`).
  Whatever your tool writes into the checkout, such as its configuration, its state or each task's
  notes, belongs in `.gitignore` or somewhere else.

## Checking the result

- **Check the result, not the report.** Take a task as done when `pnpm check` passes in the working
  tree and every line of its "Done when" is true, not when an agent says so. For anything a person
  uses, also try it in a browser: some bugs get past every test.
- **Fix the spec, then re-run the same task.** When a result is wrong, put the fix in PROJECT.md,
  then run that task again rather than starting a copy of it.

## For agents working here

[`AGENTS.md`](AGENTS.md) carries the directives; `CLAUDE.md` points at it.
