# Directives

## SAMPLE PROJECT

This is a sample project in order to learn how Xaedalon Factory works, do not commit code, push code, create issues or interact outside of this repository unless explicitly asked for it. 

## What this repository versions

This repository is a **scaffold**: the specification agents build the app from, plus one Factory
pipeline that runs it. It is not the app, because building that is the exercise. The specification
describes no particular tool's configuration: tools change, and a spec tied to one goes stale with
it. The pipeline's own files are where Factory's configuration lives, and nowhere else. Only these files are
versioned:

| Path | What it is |
|---|---|
| `AGENTS.md`, `CLAUDE.md` | directives for agents |
| `PROJECT.md` | the specification: the source of truth for the build |
| `README.md` | what the sample is, and what any run of it needs |
| `.xaedalon/` | a Factory pipeline for this sample: `.factory/` config, workflows, phases, agents, profiles |
| `.gitignore` | keeps build output and dependencies out (`.xaedalon/.gitignore` keeps run output out) |

Everything else stays **unversioned**, even when it lives in this directory:

- **Run output:** what a run writes, such as Factory's `.xaedalon/.factory/tasks/`, `state/` and
  `.trash/`, or whatever another tool keeps in the checkout.
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
