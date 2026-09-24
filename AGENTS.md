# Directives

## SAMPLE PROJECT

This is a sample project in order to learn how Xaedalon Factory works, do not commit code, push code, create issues or interact outside of this repository unless explicitly asked for it. 

## What this repository versions

This repository is a **scaffold**: the specification and the Factory pipeline that a Factory
instance uses to build the app. It is not the app. Only these files are versioned:

| Path | What it is |
|---|---|
| `AGENTS.md`, `CLAUDE.md` | directives for agents |
| `PROJECT.md` | the specification: the source of truth for the build |
| `README.md` | what the sample is and how to run it with Factory |
| `.gitignore` | keeps build output and dependencies out |
| `.xaedalon/` | the Factory pipeline: `config.yaml`, `phases/`, `workflows/`, `agents/`, and `.xaedalon/.gitignore`. Not `.factory/tasks/`, `.factory/state/` or `.factory/.trash/`, which that file already ignores |

Everything a development run produces stays **unversioned**, even when it lives in this directory:

- **The app itself:** `src/`, `index.html`, `package.json`, `pnpm-lock.yaml`, `tsconfig.json`,
  `vite.config.ts`, and any other code, tests or config a task creates.
- **Build output and dependencies:** `node_modules/`, `dist/`, `coverage/` (already in `.gitignore`).
- **Development notes:** `learnings.md`, `memory.md`, `improvements.md` (see below).

Before any commit here, check that `git status` stages nothing outside the table above. A change to
the scaffold is committed on its own, never mixed with code a run produced.

## Best Guess

Give me your best guess and reasons, whenever there is a decision to make.

## TDD/BDD as SDD

Use `TDD/BDD` to work as `SDD` where specs are covered by using `TDD/BDD` as contracts only. There
should not be spec files, just contracts covered by tests.

### BDD

Use `Gherkin` for BDD.

## Learnings and Memory

As we work and before compacting the context session save important information, decisions, indexes,
quick access or data of project in `learnings.md` and `memory.md`. These files belong to a
development session, not to the scaffold: create them while developing, and never commit them here.
Anything in them that the scaffold itself should keep goes into `PROJECT.md` or `README.md`.

## Project

`PROJECT.md` is the document that should be the source of truth of the project. Build it as you
build.

## Improvements

Create an `improvements.md` file where you add the improvements that you notice and you think we
should do, so we will tackle them into a specific session. Like `learnings.md` and `memory.md`, it is
a development file: never commit it here.
