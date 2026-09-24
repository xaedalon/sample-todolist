# Memory

Quick facts about this repository, for the next session.

- Factory project: **Todo List**, id `a3b1cccd-bbb5-4ebc-9c82-b8ab0456e03a`, root this directory,
  default branch `main`. `usesWorktrees` and `usesEnvironments` are both **off**.
- Project workflows (`.xaedalon/.factory/workflows/`): `analysis`, `design`, `implement`,
  `validate`, `verify`. One phase each, all `scheduling: sequential`, and no `needs:` between them yet.
- Phases (`.xaedalon/.factory/phases/`) were added from the Factory app. `analysis` has
  `approval: after`, and all of them use agent `developer`.
- Agent `developer` is user-level (`~/.xaedalon/.factory/agents/developer.agent.yaml`, provider
  copilot, model claude-sonnet-5). There is no `developer-haiku` yet.
- `PROJECT.md` holds the ten tasks, their dependency table, and every shared decision: signatures,
  class names, token values, markup.
- The ten tasks exist in Factory, each running analysis → design → implement → validate → verify, named "T<n>: <title>", and ticket ids T1–T10 match the
  PROJECT.md numbering. Each description says which tasks it depends on. The dependency links
  themselves have to be set in the Factory app, because the MCP cannot set them.
- Dev server port is pinned to **5180** (`strictPort`), both for `pnpm dev` and for `vite preview`. Decided in T1 of PROJECT.md.
- Supervised run, 2026-09-23. **T1 done and checked by hand**: the redo task `a56db0c9…` fixed the
  first attempt. `pnpm check` passes, the page serves on 5180, a second server on the same port is
  refused, and the contract test catches a hex colour added to `app.css`. The `developer-haiku` agent
  now carries `args: ['--allowedTools', 'Bash(pnpm *)']`.
- T2 and T3 were checked and are done. T4 was checked and accepted, although it had already drawn
  T6's checkbox and created all three icons. PROJECT.md now tells agents to build only the parts
  marked with their own task number, and T6 says to wire up the existing checkbox.
- T5 and T6 were checked and are done, including in a real browser (add, tick, delete, reload). T7 and T8 are queued as a chain.
- T7 is done after 2 reruns on the same task: the curly-quote labels, then focus now stays put on a click-away commit. All five rename paths were checked in Chrome.
- T8 was checked and is done: filters, hash and reload all verified in Chrome. It drew the item count early, and PROJECT.md now tells T9 to route it through itemsLeftLabel.
- T9 was checked and is done: count, clear completed, focus and the hidden footer all verified in Chrome. 74 tests pass.
- T10: the first run passes 81 tests, but in Chrome ticking under "All" moved focus to the next row. PROJECT.md T10 now has the rule plus a scenario. Waiting for the user to re-enable implement/validate/verify on T10. Everything else in the final keyboard/360px/light-mode check passed.
- **All ten tasks are done and checked (2026-09-24).** T10's rerun fixed tick focus: under "All"
  focus stays on the ticked checkbox, and under "Active" it moves to the next row. Both were checked
  in Chrome. The final `pnpm check` passes with 82 tests. Nothing is committed (AGENTS.md).
