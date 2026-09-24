# Learnings

- The Factory MCP can create **tasks and workflows**, but not projects, phases or agents. A
  project is added from the Factory app, and phases and agents are YAML files under
  `.xaedalon/.factory/`.
- `factory_workflow_create` with `from` copies `scheduling` but has no way to set `needs:`. A chain
  between workflows has to be written into the YAML by hand.
- To make a spec workable for a small model, decide everything two tasks must agree on up front:
  file ownership per task, exact signatures, class names, token values, markup. Then give each task
  Gherkin scenarios that map one-to-one onto vitest tests, and one command (`pnpm check`) to finish on.
- The palette in PROJECT.md was checked with a WCAG contrast script. The lowest text ratio is 5.6:1
  (muted text on the page, light mode).
- The Factory MCP cannot set **task dependencies** either: neither `task_create` nor `task_update`
  accepts them. They have to be declared in the app.
- PROJECT.md was checked on 2026-09-23 by building task 1 as written in a scratch directory. pnpm 12
  installed TypeScript 7, Vite 8, vitest 5 and jsdom 30 with no build scripts to approve, and
  `pnpm check` passed. With `strictPort`, a second `pnpm dev` exits with "Port 5180 is already in use".
- jsdom: `requestSubmit`, checkbox `click()` firing `change`, `focus()` with `select()`, and
  `dblclick` all behave like a browser. It does *not* fire `focusout` when the focused element is
  removed. Setting `location.hash` fires a native `hashchange` asynchronously, on top of any manual
  dispatch.
- **Factory's Default profile stopped the agent running pnpm.** Claude is started with
  `--restricted --permission-mode acceptEdits --permission-prompts none`. On Claude Code 2.1.281,
  `pnpm` needs approval, and with no prompts every approval becomes a denial. The T1 agent then
  wrote `package.json` by hand and reported success on "will pass" predictions. A project
  `.claude/settings.json` allow rule does **not** help in that mode. What works is agent
  `args: ['--allowedTools', 'Bash(pnpm *)']`: pnpm runs, and writes outside the workspace are still
  refused. `--allowedTools Bash` also works, but it lets writes escape the workspace.
- A task in state `done` offers only `archive`. To redo one, create a new task with only the
  workflows that need to run again.
- An agent that cannot run a command tends to report "✅ will pass". Read the artifacts for evidence
  (actual command output), not for ticks.
- **Rerunning a `done` task:** Factory disables every workflow that has run, and the MCP cannot
  switch one back on. A person re-enables the needed workflows in the app, then the MCP can `queue`
  the task. A task cancelled mid-run keeps its unrun workflows enabled, so it can be re-queued
  straight away.
- **Curly quotes:** Haiku wrote accessible names with straight quotes wherever the spec showed `“ ”`,
  which broke an earlier task's passing test, and it looped for 10+ minutes trying to fix that.
  PROJECT.md now spells them `“` and `”`, and the rerun got them right.
