# Todolist

A small todo web app, built one task at a time by coding agents under
[Xaedalon Factory](../factory). It is deliberately ordinary: everybody knows what a todo list should
do, which makes it easy to tell whether an agent did the job.

This file is the source of truth. Each task below is one Factory task — the name is what to type
into **New task**, and "done when" is what a reviewer checks at the approval gate.

## The app, when it is finished

One page, no backend. You can add a todo, tick it off, rename it, delete it, filter by what is left,
and clear the ones you have finished. It remembers everything across a reload. It works with a
keyboard alone.

## Stack

- **TypeScript**, strict, no `any`.
- **Vite** to serve and build, **vitest** to test, **pnpm** to install.
- **No UI framework.** Plain DOM. The app is small enough that a framework would be the largest
  thing in it, and the point is to read the code and see what it does.
- **No backend.** State lives in `localStorage`.

## Conventions

- `pnpm test` must pass before a task is done. Every task that changes behaviour brings tests with
  it; a task whose tests all pass on the first run has probably not tested anything.
- Logic and DOM stay apart: what a todo *is* (`src/todos.ts`) has no idea a browser exists, and is
  tested without one. Rendering reads state and writes elements, and nothing else.
- Small commits, present tense, explaining why rather than what.
- No dependencies beyond the toolchain without saying why in the commit message.

## The ten tasks

| # | Task | Depends on |
|---|---|---|
| 1 | Scaffold the project | — |
| 2 | The todo model | 1 |
| 3 | Remember todos across a reload | 2 |
| 4 | Show the list | 2 |
| 5 | Add a todo | 4 |
| 6 | Tick a todo off, and delete it | 4 |
| 7 | Rename a todo in place | 6 |
| 8 | Filter by what is left | 6 |
| 9 | Count what is left, and clear what is done | 8 |
| 10 | Make it usable with a keyboard alone | 9 |

Tasks 3 and 4 can run at the same time; so can 7 and 8. Everything else follows the order above.

---

### 1. Scaffold the project

A repository someone can clone and run.

**Done when**
- `pnpm install && pnpm dev` serves a page saying nothing more than "Todolist".
- `pnpm test` runs vitest and passes with at least one real assertion.
- `pnpm build` produces `dist/`.
- TypeScript is strict: `strict`, `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`.
- `.gitignore` covers `node_modules`, `dist` and editor droppings.

### 2. The todo model

What a todo is, and every change that can happen to one — as pure functions, with no DOM anywhere
near them.

**Done when**
- `src/todos.ts` exports a `Todo` type (`id`, `title`, `done`) and functions to add, toggle, rename
  and remove, each returning a new list rather than mutating the one it was given.
- Adding trims whitespace and refuses an empty title.
- Renaming an id that is not there leaves the list untouched rather than throwing.
- `src/todos.test.ts` covers each function, including the refusals.

### 3. Remember todos across a reload

**Done when**
- `src/storage.ts` saves and loads the list under one `localStorage` key.
- The saved shape carries a version number, so a later change can migrate rather than guess.
- Anything unreadable — absent, truncated, not JSON, the wrong shape — loads as an empty list
  instead of throwing, and does not overwrite what is there until the next real save.
- Tested against a fake storage object, not a real browser.

### 4. Show the list

**Done when**
- `src/render.ts` turns a list of todos into DOM inside a container it is handed.
- A finished todo is visibly finished, not merely different.
- An empty list says something useful rather than showing nothing at all.
- Rendering the same list twice produces the same DOM, and rendering is the only thing that writes
  to the page.

### 5. Add a todo

**Done when**
- There is a text field; Enter adds what is in it and clears it.
- Whitespace-only input adds nothing and does not clear the field.
- A new todo appears without a reload, and is still there after one.
- The field keeps focus after adding, so several can be typed in a row.

### 6. Tick a todo off, and delete it

**Done when**
- Each row has a checkbox that toggles it, and a control that deletes it.
- Both survive a reload.
- Deleting takes effect immediately; there is no confirmation dialogue for a single todo.

### 7. Rename a todo in place

**Done when**
- Double-clicking a title turns it into an editable field with the text selected.
- Enter commits, Escape cancels and restores the original, clicking elsewhere commits.
- Committing an empty title deletes the todo, which is what every todo app does and what people
  expect.

### 8. Filter by what is left

**Done when**
- Three filters: all, active, completed.
- The current filter is in the URL hash (`#/active`), so a reload keeps it and a link can share it.
- An unknown hash falls back to all rather than showing nothing.
- Switching filters does not lose or alter any todo.

### 9. Count what is left, and clear what is done

**Done when**
- A footer says how many are left, with correct pluralisation ("1 item left", "2 items left").
- A "clear completed" control removes every finished todo, and is absent when there are none.
- The footer disappears entirely when the list is empty.

### 10. Make it usable with a keyboard alone

**Done when**
- Every control is reachable by Tab, in an order that matches the page.
- Focus is visible on every control — a real outline, not a removed one.
- Deleting a todo moves focus somewhere sensible rather than to the top of the page.
- Each control has an accessible name; checkboxes are labelled by their todo's title.
- `README.md` says what the app is and how to run it.

## Running it

```bash
pnpm install
pnpm dev      # http://localhost:5173
pnpm test
pnpm build
```
