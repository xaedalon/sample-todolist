# Improvements

- **Chain the workflows.** Add `needs:` so that `design` needs `analysis`, `implement` needs
  `design`, `validate` needs `implement` and `verify` needs `validate`. Then picking `verify` on a
  task plans the whole chain.
- **Add the `developer-haiku` agent** (`provider: claude`, `model: haiku`, `session: task`), and
  decide which phases use it. Analysis and verify are the cheapest to try it on.
- **README.md describes a pipeline that is no longer there.** It mentions a `merge` workflow,
  worktrees and environments, and none of them exist in this project any more. Either restore them
  or rewrite that part.
- **Phase prompts are one line** (for example `Design Task: "{{ task.name }}" "{{ task.description }}"`). They
  should point the agent at PROJECT.md and at the previous phase's artifact, the way the deleted
  versions did.
- **Worktrees are off**, so tasks 3 and 4, or 7 and 8, would share one working copy if they ran at
  the same time. Turn worktrees on before running tasks in parallel.
- **The design phase never runs.** Every task's `design` run finished in about 3ms with no steps and
  no artifact, so design is effectively skipped. Find out why Factory resolves it to zero steps.
- **Verify passes on predictions.** On the first T1 run, verify reported "✅ will pass" without
  running anything. Make the verify phase prompt require pasted `pnpm check` output as evidence.
- **Report Claude refusals in Factory.** Its docs say a refused action is recorded on the run, but
  the pnpm denials were not; the run completed as if nothing had happened. File this against Factory,
  along with the `--allowedTools` fix for the Default profile on Claude Code 2.1.281+.
- **Rerunning a done task needs the app.** The MCP cannot re-enable workflows on a `done` task.
  Add that to the MCP, or document it.
- **Checks not yet automated:** 200% zoom and `prefers-reduced-motion` were checked by reading the
  code, not in a browser.
- **Small leftovers in the code:** `todosSection as HTMLElement` casts inside closures in `app.ts`;
  in `filter.test.ts` the "Each filter shows the right todos" test is misnamed; the archived-worthy
  "T1: Scaffold the project (redo)" task is still on the board.
