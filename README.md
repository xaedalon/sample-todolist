# sample-todolist

The sample project for [Xaedalon Factory](https://github.com/xaedalon/factory-community): a small
todo web app, specified as ten tasks with a dependency graph, built by coding agents — several at
once, each in a worktree of its own, each merged back when somebody approves it.

**The app is not in here, and that is the point.** This repository holds the specification. What an
agent produces from it — the HTML, the stylesheets, the TypeScript, the tests — is the exercise.

## What to read

[**`PROJECT.md`**](PROJECT.md) is the source of truth: the stack, the look-and-feel contract, and ten
tasks. Each task's name is what you type into Factory's **New task**, and its "done when" list is
what the `verify` phase checks before the work is merged.

The ten form a graph rather than a chain — tasks 3 and 4 can run at the same time, and so can 7 and
8 — which is what makes it a useful sample: it exercises task dependencies rather than a queue, and
the pipeline below is built so that tasks which *can* run together actually do.

## Using it with Factory

```bash
git clone https://github.com/xaedalon/sample-todolist.git
factory project add sample-todolist /full/path/to/sample-todolist
```

That is the whole setup — **the pipeline comes with the clone.** This repository carries its own
Factory definitions at [`.xaedalon/.factory/`](.xaedalon/.factory), and adding the repository as a
project is what loads them. There is nothing to import.

Two project settings have to be on, because the pipeline is built out of both: **Worktrees** (on by
default for a git repository) and **Environments** (off by default — turn it on on the project's
page, or `PATCH /api/projects/<id> {"usesEnvironments": true}`). Without them the first workflow
waits for a flag nothing can provide, and Factory says so rather than running an agent in the
repository itself.

Then, on the board: create the ten tasks from `PROJECT.md`, declare the dependencies from the table,
pick **`merge`** as each task's workflow, and press **Queue all**. One workflow per task is enough —
`merge` needs `verify`, which needs `validate`, and so on back to the worktree, so Factory assembles
the whole chain and tells you what it added. Leave the Branch field empty; the workflows derive it.

**There is no approval gate**, on purpose: every task runs from an empty worktree to a merged branch
without stopping, so ten tasks and one **Queue all** is the whole interaction. Put it back by adding
`approval: before` to `phases/merge-integrate.phase.yaml` if you would rather see each one first.

[`docs/task-dependencies.md`](https://github.com/xaedalon/factory-community/blob/main/docs/task-dependencies.md)
covers declaring the graph and the two whole-project buttons;
[quickstart](https://github.com/xaedalon/factory-community/blob/main/docs/quickstart.md) covers the
first task if you have never run one.

## The pipeline

Picking `merge` on a task gets all eight of these, in this order:

| | | |
|---|---|---|
| 1 | `worktree-create` | a worktree at `<worktrees root>/<task directory>`, on `task/<task directory>`, branched from the default branch |
| 2 | `environment-create` | checks what the pipeline needs (a git identity, because step 8 commits), then installs this worktree's dependencies if it has a manifest |
| 3–7 | `analysis` → `design` → `implement` → `validate` → `verify` | the work: one agent step each, each writing an artifact the next one reads |
| 8 | `merge` | commit the work, **an agent** brings the default branch in and resolves any conflict, fast-forward the default branch, take the environment away, remove the worktree |

**Several tasks run at once.** That is the reason for the worktree: five agents in one working copy
overwrite each other, five agents in five worktrees do not. The five work workflows are
`scheduling: parallel` and Factory runs three tasks at a time by default.

**The merge is the part that has to be careful**, because it is the only part touching something
shared, and it is an *agent* that drives it rather than a `git merge` that either works or fails. The
order is what makes that safe:

1. a shell step commits what the task produced — no judgement in it, so no agent needed;
2. a shell step takes a lock (`<repo>/.git/factory-merge.lock`);
3. **the agent** merges the default branch *into the task's branch*, inside the worktree, and
   resolves whatever conflicts that turns up — it is the same conversation that wrote the code and
   read the review (`session: task`), and it writes a `merge` artifact recording each decision;
4. a shell step fast-forwards the default branch and releases the lock. A fast-forward either
   applies completely or does nothing at all, so the one write to shared state is the one operation
   with nothing to decide.

The lock spans steps 2 to 4 — including the agent — and that span was bought with evidence. With the
lock around the fast-forward alone, two tasks merging together both found the default branch where
they had left it, both agents said "already up to date", the first landed, and the second hit a
conflict in a merge it had already finished, with the agent long gone. It recovered, but it needed a
person to press Retry. Holding the lock across the decision means one task integrates at a time;
everything up to `verify` still runs in parallel, which is where the time actually goes.

If any of it fails anyway, `on_fail: merge-repair` runs an agent *before* the task is blocked: it gets
the failure reason and the task's own session, releases the lock if this task was holding it, puts the
branch back into a state that can be landed, and writes down what it did — so the one Retry that
follows usually just works. Measured: asked to recover from exactly the race above, it reconstructed
the timeline, resolved the conflict keeping both sides, and left the branch fast-forwardable.

**Nothing is thrown away unless it merged.** A failing phase stops its workflow, so a task that
cannot be merged keeps its worktree, its branch and everything in it. `worktree-delete` run on its
own — for work you are abandoning — refuses while the branch holds commits the default branch does
not, and prints the two commands that remove it anyway. Branches always survive; Factory never
deletes one.

`environment-update` and `environment-delete` are there for the same reason: to be run by hand on one
task, when a manifest changed underneath it or when you want its dependencies back off the disk.

## What ships in `.xaedalon/.factory/`

| | |
|---|---|
| `config.yaml` | one line that matters — `scope: project` |
| `workflows/` | the eight above, plus `environment-update`, `environment-delete` and `merge-repair` |
| `phases/` | five agent phases for the work, a sixth for repairing a failed merge, and five doing the worktree, environment and merge machinery |
| `agents/developer.agent.yaml` | who does the work |
| `.gitignore` | `node_modules/` and friends — the merge commits everything a task produced, so this is what keeps a dependency tree out of it |

The five work workflows chain with `needs:`, so `design` will not start until `analysis` is done for
that task, and each phase writes an artifact the next one reads — design reads the analysis,
implement reads the design, verify reads all three.

The agent **pins `provider: copilot`, and that is one line to change** to `claude` or `codex`. It has
to name one: Factory's documented fallback is "the configured default, then the only one installed",
but nothing a scope can write sets that default — only the CLI's `factory run --provider` — so on a
machine with two CLIs installed an unpinned agent step is refused at plan time, with a message saying
exactly that. `model: balanced` is a role rather than a model id, and each provider maps it to its
own middle model.

Five of the twelve workflows are **overrides**: Factory's built-in `worktree-create`, `worktree-delete`,
`environment-create`, `environment-update` and `environment-delete` all declare `override: required`,
which is them saying that where a worktree goes and what an environment is made of are not things a
built-in can know. The files here are that answer, and they are commented at length — they are the
part of this sample worth reading if you are writing your own.

### How Factory finds them

`.xaedalon/.factory/` is a Factory **project scope**, and definitions resolve through a chain:
project, then your user scope, then Factory's built-ins — first match wins, so a name defined here
shadows one of your own. Three things make that work from a clone:

- `config.yaml` declares `scope: project`. Without that declaration a directory found while walking
  up is still treated as a project scope, but the declaration is what keeps a *user* scope from being
  promoted by accident.
- The daemon resolves the chain **per project, from the path that project points at** — not from the
  directory it happened to be started in. So a definition committed to a repository is usable by
  that repository's tasks without launching anything inside it.
- `factory config path` prints the chain, and `factory workflow list` prints `project` beside each of
  the five. If they show as anything else, Factory is looking somewhere you did not mean.

### What a repository cannot carry

Definitions travel; **settings do not**, and the split is deliberate:

- **Worktrees, environments, the execution profile and the default branch** are per-installation
  choices, kept in Factory's own database and set when you add the project (changeable afterwards on
  its page). Two people cloning this repository can legitimately want different answers, so the
  repository does not get a vote — which is exactly why this README has to ask you to switch
  Environments on rather than shipping the setting.
- **Plugins and provider paths** can be declared in `config.yaml`, but they are loaded once when the
  daemon starts and apply installation-wide. A project may define its own workflows, phases and
  agents; it may not define its own plugins.
- **Artifacts stay out of git.** `.xaedalon/.gitignore` ignores `.factory/tasks/`, which is where
  each task's artifacts are written.

Nothing is scaffolded into your clone when you add it: Factory copies its built-ins into a project
that turns worktrees or environments on, and finds this project has already written its own.

### Taking the pipeline somewhere else

To use these workflows on a different project, export and import them rather than copying files:

```bash
factory workflow export analysis > analysis.bundle.yaml    # the workflow, its phases and its agent
factory bundle import analysis.bundle.yaml --scope user    # --dry-run first if you like
```

One workflow per export — it follows phases and agents but not the `needs` chain — so the whole
pipeline is ten of them. An import keeps whatever it replaces in `<scope>/.trash/`. Copying the
`.xaedalon/.factory/` directory into another repository works too, and for the worktree, environment
and merge files it is the better answer: they are written for *this* project, and reading them
before reusing them is the point.

## For agents working here

[`AGENTS.md`](AGENTS.md) carries the directives; `CLAUDE.md` points at it.
