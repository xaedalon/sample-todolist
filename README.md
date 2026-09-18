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

That is the whole setup — **the pipeline comes with the clone.** This repository carries its own
Factory definitions at [`.xaedalon/.factory/`](.xaedalon/.factory), and adding the repository as a
project is what loads them. There is nothing to import.

Then, on the board: create the ten tasks from `PROJECT.md`, pick the `analysis` workflow for each,
declare the dependencies from the table, and press **Queue all**. Factory holds each task until the
one it waits for is done, so the graph runs itself.

[`docs/task-dependencies.md`](https://github.com/xaedalon/factory-community/blob/main/docs/task-dependencies.md)
covers declaring the graph and the two whole-project buttons;
[quickstart](https://github.com/xaedalon/factory-community/blob/main/docs/quickstart.md) covers the
first task if you have never run one.

## What ships in `.xaedalon/.factory/`

| | |
|---|---|
| `config.yaml` | one line that matters — `scope: project` |
| `workflows/` | `analysis`, `design`, `implement`, `validate`, `verify` |
| `phases/` | one of each name, each a single agent step that writes an artifact |
| `agents/developer.agent.yaml` | who does the work |

The five workflows are a chain rather than a queue: each declares `needs:` on the one before, so
`design` will not start until `analysis` is done for that task. Every phase writes an artifact and
the next phase reads it — design reads the analysis, implement reads the design, verify reads all
three. Run `analysis` on a task and the other four follow.

The agent **names no provider on purpose**, so the sample runs on whichever coding agent you already
have: Factory falls back to the configured default, then to the only one installed. Its `model:
balanced` is a role rather than a model id, and each provider maps it to its own middle model. Pin
either in that file if you want a particular one.

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
  repository does not get a vote.
- **Plugins and provider paths** can be declared in `config.yaml`, but they are loaded once when the
  daemon starts and apply installation-wide. A project may define its own workflows, phases and
  agents; it may not define its own plugins.
- **Artifacts stay out of git.** `.xaedalon/.gitignore` ignores `.factory/tasks/`, which is where
  each task's artifacts are written.

One thing to expect rather than be surprised by: adding this repository with worktrees on writes four
more definitions here — `worktree-create`, `worktree-delete`, `worktree-add`, `worktree-remove`. They
are editable copies of Factory's built-ins, which cannot know how a particular repository wants its
worktrees made. They are deliberately not committed: until you change one, the built-in is the better
default.

### Taking the pipeline somewhere else

To use these workflows on a different project, export and import them rather than copying files:

```bash
factory workflow export analysis > analysis.bundle.yaml    # the workflow, its phases and its agent
factory bundle import analysis.bundle.yaml --scope user    # --dry-run first if you like
```

One workflow per export — it follows phases and agents but not the `needs` chain — so the full
pipeline is five of them. An import keeps whatever it replaces in `<scope>/.trash/`.

## For agents working here

[`AGENTS.md`](AGENTS.md) carries the directives; `CLAUDE.md` points at it.
