# Todolist

A small todo web app, built one task at a time by coding agents under
[Xaedalon Factory](https://github.com/xaedalon/factory-community). It is deliberately ordinary: everybody knows what a todo list should
do, which makes it easy to tell whether an agent did the job.

This file is the source of truth. Each task below is one Factory task — the name is what to type
into **New task**, and "done when" is what a reviewer checks at the approval gate.

It is written so that a small model can do any one task without guessing: every decision that two
tasks have to agree on — file names, function signatures, class names, token values, markup — is
made here, once, before anyone starts. If a task seems to need a decision this file does not make,
make the smallest one that fits and write it down in the task's artifact.

## How to work a task

Do these in order. Do not skip ahead.

1. Read **Stack**, **Conventions**, **Architecture**, **Look and feel** and **The page** below, then
   your task. They are short and every task depends on them.
2. Touch only the files your task lists under **Files**. Another task may be running in parallel
   and owns the rest.
   **The page** and **Architecture** show the *finished* app, and every part is marked with the
   number of the task that adds it. Build only the parts marked with your task's number (or an
   earlier one that is missing). A later task's part is not yours, however easy it looks: a control
   drawn before its behaviour exists is a control that does nothing when clicked.
3. Turn each Gherkin scenario in your task into one vitest test, *before* writing the code, and
   watch it fail. The scenario name is the test name (see **Tests are the specification**).
4. Write the code until those tests pass.
5. Run `pnpm check`. It must end with no errors. From task 1 onwards it runs the type-checker, every
   test, and the production build.
6. Tick every line of **Done when** yourself. If one is not true, the task is not done.

Things never to do, in any task:

- Do not add a dependency that is not listed in **Stack**.
- Do not change a token value in `src/theme.css`, a signature in **Architecture**, or a class name
  in **The page**. Other tasks rely on them. If one is wrong, say so in the artifact.
- Do not delete or weaken another task's test to make yours pass. A test that has to change because
  the behaviour it describes changed on purpose is fine; say which and why.
- Do not build markup from strings (`innerHTML`, `insertAdjacentHTML`, template literals of HTML)
  in application code. Use `document.createElement` and `textContent`.
- Do not write dependency versions into `package.json` by hand. Only `pnpm add` writes them, and it
  also writes `pnpm-lock.yaml`. If you cannot run `pnpm`, stop and report that the task is
  **blocked**. Never report success for commands you did not run: a task is done when you have *seen*
  `pnpm check` pass, not when you expect it to.
- `noUncheckedIndexedAccess` is on, so `list[0]`, `match[1]` and `matches[0][0]` are possibly
  `undefined`, and `tsc` rejects using them as if they were not. Check them (`if (!m) throw …`) or
  use `?? ''`, and never silence the check with `!`. Run `pnpm typecheck` early.

## The app, when it is finished

One page, no backend. You can add a todo, tick it off, rename it, delete it, filter by what is left,
and clear the ones you have finished. It remembers everything across a reload. It works with a
keyboard alone.

And it looks like something somebody chose: one palette, one type pairing, one set of icons, in both
light and dark. Not decorated — *deliberate*. An agent that produces working behaviour on an
unstyled page has done half the task.

Picture it: a centred column, about 36rem wide, on a warm off-white page (near-black in dark mode).
A large serif "Todolist" heading. Under it, a wide input — "What needs doing?". Under that, one
raised white card holding the rows, each with a round tick on the left, the title, and two quiet
icon buttons on the right: rename and delete. Below the card, a small muted footer: "2 items left",
three pill-shaped filters, and "Clear completed".

## Stack

- **TypeScript**, strict, no `any`.
- **Vite** to serve and build, **vitest** to test, **pnpm** to install.
- **Dev dependencies, and only these:** `typescript`, `vite`, `vitest`, `jsdom`, `@types/node`.
  No runtime dependencies at all.
- **No UI framework.** Plain DOM. The app is small enough that a framework would be the largest
  thing in it, and the point is to read the code and see what it does.
- **No backend.** State lives in `localStorage`.
- **Three files, three jobs.** `index.html` is structure, `src/theme.css` and `src/app.css` are
  appearance, the TypeScript modules are behaviour. No `style=` attributes, no colour values outside
  `src/theme.css`, and no markup built by string concatenation.
- **Nothing the page loads comes from the internet.** No CSS framework, no icon font, no CDN, no
  Google Fonts request. Everything ships from the repository, so the app works offline and on a
  first load nobody has warmed.

## Conventions

- `pnpm check` must pass before a task is done. Every task that changes behaviour brings tests with
  it; a task whose tests all pass on the first run has probably not tested anything.
- Logic and DOM stay apart: what a todo *is* (`src/todos.ts`, `src/filter.ts`, `src/storage.ts`)
  has no idea a browser exists, and is tested without one. Rendering (`src/render.ts`) reads state
  and writes elements, and nothing else. Wiring (`src/app.ts`) listens to events, computes the next
  state, and asks rendering to draw it.
- Functions over classes. Plain objects and arrays, never mutated: every change returns a new one.
- Named exports only; no default exports.
- Small commits, present tense, explaining why rather than what.

### Tests are the specification

There are no spec files. The Gherkin scenarios in this file are the contracts, and the tests are
where they are enforced. Each scenario becomes one vitest test, named after it, with its steps as
comments:

```ts
describe('Feature: Add a todo', () => {
  it('Scenario: Submitting adds what is typed and clears the field', () => {
    // Given the app is open with no todos
    // When I type "Buy milk" into the field labelled "New todo" and submit the form
    // Then the list shows "Buy milk"
    // And the field is empty
  });
});
```

- Import what you use: `import { describe, it, expect, vi, afterEach } from 'vitest'`. There are
  no globals.
- Pure modules are tested in vitest's default `node` environment.
- Anything that touches the DOM starts with the comment `// @vitest-environment jsdom` on line 1.
- Behaviour tests drive the **real** `index.html`: `loadPage()` from `src/test/page.ts` puts its
  body into the jsdom document, and the test calls `mountApp` on it. Find elements the way a person
  would: by label, role or visible text where you can, by the class names in **The page** where you
  cannot.
- Simulate a key press with
  `el.dispatchEvent(new KeyboardEvent('keydown', { key: 'Enter', bubbles: true }))`, and a form
  submission with `form.requestSubmit()`.
- Every behaviour test calls the cleanup function `mountApp` returns in `afterEach`, and resets
  `window.location.hash = ''`.

## Architecture

These are the modules the finished app has, and the exact shape of each. The task that creates a
module is in brackets. Write the signatures exactly as shown, so tasks running in parallel agree.

```
index.html              structure only                                   [1, grows in 5, 8]
src/theme.css           design tokens, the only file with colour values  [1]
src/app.css             every rule, all values from tokens               [1, grows in 4–10]
src/main.ts             entry point: loads, mounts, saves                [1, rewritten in 4, 5]
src/icons.ts            the icon set as SVG nodes                        [4, grows in 6, 7]
src/todos.ts            the model: pure functions                        [2, grows in 9]
src/storage.ts          load and save, versioned                         [3]
src/state.ts            AppState, Filter, initialState                   [4]
src/filter.ts           filters and the URL hash                         [8]
src/render.ts           state → DOM                                      [4, grows in 6–9]
src/app.ts              events → next state → render                     [4, grows in 5–10]
src/test/page.ts        loadPage() test helper                           [4]
src/test/fake-storage.ts  fakeStorage() test helper                      [3]
src/contract.test.ts    the Look-and-feel contract, enforced             [1]
src/*.test.ts           one test file per module or feature              [each task]
```

**`src/todos.ts`** — no imports from the rest of the app.

```ts
export type Todo = { readonly id: string; readonly title: string; readonly done: boolean };

export function addTodo(list: readonly Todo[], title: string, id: string): readonly Todo[];
export function toggleTodo(list: readonly Todo[], id: string): readonly Todo[];
export function renameTodo(list: readonly Todo[], id: string, title: string): readonly Todo[];
export function removeTodo(list: readonly Todo[], id: string): readonly Todo[];
export function clearCompleted(list: readonly Todo[]): readonly Todo[];
export function countActive(list: readonly Todo[]): number;
export function itemsLeftLabel(count: number): string;          // task 9
```

- `id` is passed in, so the model stays pure and tests are deterministic. The app makes ids with
  `crypto.randomUUID()`.
- A refusal (empty title, unknown id) returns **the same list object** it was given, so
  `result === list` tells a caller nothing happened.
- `addTodo` appends to the end, trims the title, and starts the todo as not done.
- `renameTodo` trims too, and refuses an empty title. Deleting on an empty rename is the app's
  decision (task 7), not the model's.

**`src/storage.ts`** — imports only `Todo`.

```ts
export const STORAGE_KEY = 'todolist';
export const STORAGE_VERSION = 1;
export type StorageLike = Pick<Storage, 'getItem' | 'setItem'>;

export function loadTodos(storage: StorageLike): readonly Todo[];
export function saveTodos(storage: StorageLike, todos: readonly Todo[]): void;
```

The saved value is `JSON.stringify({ version: 1, todos })`. `loadTodos` returns `[]` for anything
else — missing, not JSON, another version, not an object, `todos` not an array, or any item without
a string `id`, a string `title` and a boolean `done`. It never throws and never writes.

**`src/state.ts`**

```ts
export type Filter = 'all' | 'active' | 'completed';
export type AppState = {
  readonly todos: readonly Todo[];
  readonly filter: Filter;
  readonly editingId: string | null;
};
export function initialState(todos: readonly Todo[]): AppState;   // filter 'all', editingId null
```

**`src/filter.ts`** (task 8)

```ts
export function applyFilter(list: readonly Todo[], filter: Filter): readonly Todo[];
export function parseFilter(hash: string): Filter;   // '#/active' → 'active', anything unknown → 'all'
export function filterHash(filter: Filter): string;  // 'all' → '#/', 'active' → '#/active', …
```

**`src/icons.ts`**

```ts
export type IconName = 'check' | 'trash' | 'pencil';
export function icon(name: IconName): SVGSVGElement;
```

Built with `document.createElementNS('http://www.w3.org/2000/svg', …)`. Every icon has
`viewBox="0 0 24 24"`, `fill="none"`, `stroke="currentColor"`, `stroke-width="2"`,
`stroke-linecap="round"`, `stroke-linejoin="round"`, `aria-hidden="true"` and
`focusable="false"`, and is made of `<path>` elements with exactly this data:

| name | path `d` values |
|---|---|
| `check` | `M20 6 9 17l-5-5` |
| `trash` | `M3 6h18` · `M8 6V4h8v2` · `M19 6l-1 14H6L5 6` · `M10 11v6` · `M14 11v6` |
| `pencil` | `M17 3l4 4L8 20H4v-4Z` · `M14 6l4 4` |

**`src/render.ts`**

```ts
export function renderList(section: HTMLElement, state: AppState): void;    // task 4
export function renderFooter(footer: HTMLElement, state: AppState): void;   // task 8
```

Each one rebuilds its container's contents from scratch with `container.replaceChildren(...)`.
Neither reads the DOM to decide what to draw, adds event listeners, or moves focus.

**`src/app.ts`**

```ts
export type MountOptions = {
  readonly todos?: readonly Todo[];                    // what to start with; default []
  readonly save?: (todos: readonly Todo[]) => void;    // called after every change to the todos
  readonly newId?: () => string;                       // default () => crypto.randomUUID()
};
export function mountApp(root: HTMLElement, options?: MountOptions): () => void;
```

`root` is the element containing the page's `<main class="app">`: `document.body` in the app and
in tests. `mountApp` holds the one `AppState` in a local variable, listens for events with **one
listener per container** (event delegation: read `data-id` from `event.target.closest('.todo')`),
and every change goes through one function:

```ts
function update(next: AppState): void {
  const todosChanged = next.todos !== state.todos;
  state = next;
  if (todosChanged) save(state.todos);
  draw();            // calls renderList, and renderFooter from task 8 on
}
```

Moving focus happens *after* `draw()`, by querying for the element to focus. It returns a function
that removes every listener it added to `window`.

**`src/main.ts`** — the only file that touches `localStorage` (the only other global the app uses is
`crypto.randomUUID`, as the default `newId` in `src/app.ts`):

```ts
import { mountApp } from './app';
import { loadTodos, saveTodos } from './storage';

mountApp(document.body, {
  todos: loadTodos(localStorage),
  save: (todos) => saveTodos(localStorage, todos),
});
```

## Look and feel

This is a contract, not a task. Every task that touches the page obeys it, because a theme applied
at the end is how an app ends up with the same colour written in nine places.
`src/contract.test.ts` (task 1) enforces what a machine can check.

**Colour.** One palette, defined once in `src/theme.css` as custom properties on `:root`, and
redefined for dark under `@media (prefers-color-scheme: dark)`. Nothing outside that file contains a
colour value — no hex, `rgb()`, `hsl()` or named colour. The values below meet WCAG AA in both
schemes (lowest text ratio 5.6:1; focus ring at least 5.8:1 against both the page and the card).
They are measured, so do not change them.

**Type.** Two roles: a serif heading face and a system body face, each a token ending in a generic
family. No webfont: system stacks cost nothing to load and look native on every platform. That is
also the choice, so nothing needs a `woff2`. Four sizes and no more.

**Icons.** Inline SVG, three of them, from `src/icons.ts` (see **Architecture**), drawn in
`currentColor` so an icon takes the colour of the button it sits in. Every icon-only control carries
an `aria-label`, because an icon is not a label.

**Spacing and shape.** A spacing scale and one border radius, both tokens. In `src/app.css` every
colour, font family, font size, space, radius, shadow and duration is a `var(--…)`. The only
literals allowed there are `0`, `1px`/`2px` border widths, `50%`, `100%`, unitless line-heights,
`flex`/`grid` numbers, and `100vh`.

**Controls are always visible.** Rename and delete sit on every row, all the time, in the muted
colour — not revealed on hover, which hides them from touch and keyboard users.

`src/theme.css`, in full — task 1 writes exactly this:

```css
/* Design tokens. The only file in the project that contains a colour value. */
:root {
  color-scheme: light dark;

  --color-bg: #f5f3ee;
  --color-surface: #ffffff;
  --color-border: #d9d5cc;
  --color-text: #1c1e21;
  --color-muted: #5c6169;
  --color-accent: #2b55c7;
  --color-on-accent: #ffffff;
  --color-danger: #b3261e;
  --color-focus: #2b55c7;
  --shadow-raised: 0 1px 2px rgb(0 0 0 / 0.06), 0 8px 24px rgb(0 0 0 / 0.06);

  --font-heading: "Iowan Old Style", "Palatino Linotype", Palatino, "Book Antiqua", Georgia, serif;
  --font-body: system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;

  --text-sm: 0.875rem;
  --text-md: 1rem;
  --text-lg: 1.125rem;
  --text-xl: 2.5rem;

  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-3: 0.75rem;
  --space-4: 1rem;
  --space-6: 1.5rem;
  --space-8: 2rem;

  --radius: 0.5rem;
  --control-size: 2.75rem;
  --icon-size: 1.25rem;
  --icon-size-sm: 0.875rem;
  --content-width: 36rem;
  --focus-offset: 2px;
  --duration: 150ms;
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-bg: #141619;
    --color-surface: #1e2125;
    --color-border: #363a40;
    --color-text: #ebe8e2;
    --color-muted: #a6abb3;
    --color-accent: #8fb3ff;
    --color-on-accent: #10131a;
    --color-danger: #ff8b82;
    --color-focus: #8fb3ff;
    --shadow-raised: 0 1px 2px rgb(0 0 0 / 0.4), 0 8px 24px rgb(0 0 0 / 0.3);
  }
}

@media (prefers-reduced-motion: reduce) {
  :root {
    --duration: 0ms;
  }
}
```

## The page

**`index.html` when all ten tasks are done.** The task that adds each part is in the comment. Keep
the comments out of the real file.

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="color-scheme" content="light dark" />
    <title>Todolist</title>
    <link rel="icon" href="data:," />
    <link rel="stylesheet" href="/src/app.css" />
  </head>
  <body>
    <main class="app">
      <header class="app-header">
        <h1 class="app-title">Todolist</h1>
      </header>
      <!-- 5 -->
      <form class="new-todo" id="new-todo" autocomplete="off">
        <label class="visually-hidden" for="new-todo-input">New todo</label>
        <input class="new-todo-input" id="new-todo-input" name="title" type="text"
               placeholder="What needs doing?" autofocus />
      </form>
      <!-- 4 -->
      <section class="todos" id="todos" aria-label="Todos"></section>
      <!-- 8 -->
      <footer class="footer" id="footer" hidden></footer>
    </main>
    <script type="module" src="/src/main.ts"></script>
  </body>
</html>
```

`<link rel="icon" href="data:," />` is there so the browser does not ask the server for a
`favicon.ico` that does not exist. It keeps the network tab clean. Element ids must not be spelt
only with the letters a–f (`#add`, `#bed`), because the contract test would read them as colours.

**What `renderList` draws inside `#todos`.** Each part names the task that adds it.

```html
<!-- when the visible list is empty (4; the filtered wording is 8) -->
<p class="empty">Nothing to do yet — add something above.</p>

<!-- otherwise -->
<ul class="todo-list">
  <li class="todo todo--done" data-id="{id}">              <!-- 4; todo--done only when done -->
    <label class="todo-check">                             <!-- 6 (in 4: just the span) -->
      <input class="todo-toggle visually-hidden" type="checkbox"
             aria-labelledby="title-{id}" checked />       <!-- 6; checked only when done -->
      <span class="todo-check-box" aria-hidden="true">{check icon}</span>   <!-- 4 -->
    </label>
    <span class="todo-title" id="title-{id}">{title}</span>                 <!-- 4 -->
    <button class="icon-button todo-edit" type="button"
            aria-label="Rename “{title}”">{pencil icon}</button>            <!-- 7 -->
    <button class="icon-button todo-delete" type="button"
            aria-label="Delete “{title}”">{trash icon}</button>             <!-- 6 -->
  </li>

  <!-- the row being renamed (7) replaces its contents with a single field -->
  <li class="todo todo--editing" data-id="{id}">
    <input class="todo-edit-input" type="text" value="{title}" aria-label="Rename “{title}”" />
  </li>
</ul>
```

**What `renderFooter` draws inside `#footer`.** It sets `footer.hidden = state.todos.length === 0`.

```html
<span class="todo-count">2 items left</span>                           <!-- 9 -->
<nav class="filters" aria-label="Filter todos">                        <!-- 8 -->
  <a class="filter-link" href="#/" aria-current="page">All</a>
  <a class="filter-link" href="#/active">Active</a>
  <a class="filter-link" href="#/completed">Completed</a>
</nav>
<button class="text-button clear-completed" type="button">Clear completed</button>  <!-- 9 -->
```

**The quotes in accessible names are curly: `“` and `”`, not `"`.** Write them in TypeScript as
escapes, so no editor or model can turn them into straight quotes: `` `Delete \u201C${todo.title}\u201D` ``
and `` `Rename \u201C${todo.title}\u201D` ``. Tests compare with the same escapes. A test that expects
`Delete "Buy milk"` with straight quotes is wrong.

`aria-current="page"` goes on the current filter's link only. There is no attribute at all on the
others, not `aria-current="false"`. `id` values from `crypto.randomUUID()` are safe to use in
`id="title-{id}"`.

## The ten tasks

| # | Task | Depends on |
|---|---|---|
| 1 | Scaffold the project | — |
| 2 | The todo model | 1 |
| 3 | Remember todos across a reload | 2 |
| 4 | Show the list | 2 |
| 5 | Add a todo | 3, 4 |
| 6 | Tick a todo off, and delete it | 5 |
| 7 | Rename a todo in place | 6 |
| 8 | Filter by what is left | 6 |
| 9 | Count what is left, and clear what is done | 8 |
| 10 | Make it usable with a keyboard alone | 7, 9 |

Tasks 3 and 4 can run at the same time, and so can 7 and 8. Everything else follows the order above.
Tasks 7 and 8 both add to `src/render.ts`, `src/app.ts` and `src/app.css`, so whichever merges
second keeps both sets of changes. Neither replaces the other's.

---

### 1. Scaffold the project

A repository someone can clone and run, with the theme already in place so that no later task has
to retrofit one onto markup that hardcoded its colours.

**Files:** `package.json`, `pnpm-lock.yaml`, `tsconfig.json`, `vite.config.ts`, `.gitignore`,
`index.html`, `src/theme.css`, `src/app.css`, `src/main.ts`, `src/contract.test.ts`.

**Build it like this**

- `pnpm add -D typescript vite vitest jsdom @types/node`. Commit the lockfile.
- `package.json` has `"private": true`, `"type": "module"` and these scripts:
  ```json
  {
    "dev": "vite",
    "build": "tsc --noEmit && vite build",
    "test": "vitest run",
    "typecheck": "tsc --noEmit",
    "check": "tsc --noEmit && vitest run && vite build"
  }
  ```
- `tsconfig.json`: `target` and `lib` `ES2022` (plus `DOM`, `DOM.Iterable` in `lib`),
  `module: "ESNext"`, `moduleResolution: "bundler"`, `strict`, `noUncheckedIndexedAccess`,
  `exactOptionalPropertyTypes`, `noEmit`, `isolatedModules`, `skipLibCheck`,
  `types: ["vite/client", "node"]`, `include: ["src", "vite.config.ts"]`.
- `vite.config.ts` — the app has its own port, **5180**, and `strictPort` makes a busy port an
  error instead of a silent move to the next one:
  ```ts
  import { defineConfig } from 'vitest/config';

  export default defineConfig({
    server: { port: 5180, strictPort: true },
    preview: { port: 5180, strictPort: true },
    test: { include: ['src/**/*.test.ts'] },
  });
  ```
- `.gitignore`: `node_modules/`, `dist/`, `.DS_Store`, `.vscode/`, `.idea/`, `*.log`, `coverage/`.
  Keep whatever entries it already has.
- `index.html`: the page from **The page**, *without* the form, the section and the footer. Only
  the header and the script.
- `src/theme.css`: exactly the file in **Look and feel**.
- `src/main.ts`: for now just `export {};` and a one-line comment saying it becomes the entry point
  in task 4.
- `src/app.css`, the base every later task builds on:
  ```css
  @import "./theme.css";

  *, *::before, *::after { box-sizing: border-box; }
  [hidden] { display: none !important; }

  html {
    font-family: var(--font-body);
    font-size: 100%;
    line-height: 1.5;
    color: var(--color-text);
    background: var(--color-bg);
  }
  body { margin: 0; min-height: 100vh; }

  .app {
    max-width: var(--content-width);
    margin: 0 auto;
    padding: var(--space-8) var(--space-4);
  }
  .app-title {
    font-family: var(--font-heading);
    font-size: var(--text-xl);
    font-weight: 600;
    line-height: 1.1;
    margin: 0 0 var(--space-6);
  }

  :focus-visible {
    outline: 2px solid var(--color-focus);
    outline-offset: var(--focus-offset);
  }

  .visually-hidden {
    position: absolute;
    width: 1px;
    height: 1px;
    margin: -1px;
    padding: 0;
    overflow: hidden;
    clip-path: inset(50%);
    white-space: nowrap;
    border: 0;
  }
  ```
- Take the versions pnpm installs, and do not pin older ones. Checked on 2026-09-23 with pnpm 12 and
  Node 24, this setup installs TypeScript 7, Vite 8, vitest 5 and jsdom 30 with no build scripts to
  approve, and passes `pnpm check`.

**Scenarios** — in `src/contract.test.ts`, node environment, reading files with `node:fs`:

```gherkin
Feature: The look-and-feel contract

  Scenario: Every colour token is redefined for dark mode
    Given src/theme.css
    Then every "--color-" property declared in :root is declared again inside
      "@media (prefers-color-scheme: dark)"

  Scenario: No colour value outside the theme
    Given every .css, .ts and .html file under src/ and index.html,
      except src/theme.css, *.test.ts files and anything under src/test/
    Then none contains a hex colour, "rgb(", "rgba(", "hsl(" or "hsla("

  Scenario: No inline styles
    Given index.html and every .ts file under src/, except *.test.ts files and src/test/
    Then none contains "style=" or ".style."

  Scenario: One stylesheet
    Given index.html
    Then it links exactly one stylesheet, and it is /src/app.css
```

Match a colour with `/#[0-9a-fA-F]{3,8}\b|\b(rgba?|hsla?)\(/`. Find files with
`readdirSync(new URL('src/', root), { recursive: true }).map(String)`, where
`root = new URL('../', import.meta.url)`.

**Done when**
- `pnpm install && pnpm dev` serves, at `http://localhost:5180`, a page showing only the
  "Todolist" heading, in the serif face, on the off-white page — or near-black in dark mode.
- With something else already listening on 5180, `pnpm dev` stops with an error rather than
  moving to 5181.
- `pnpm check` passes, and `pnpm build` produces `dist/`.
- The four contract scenarios pass. Try adding `color: red` to `app.css`: the red is visible, which
  proves the stylesheet loads. Then remove it.
- Nothing else is created: no `src/todos.ts`, no components, no sample code.

---

### 2. The todo model

What a todo is, and every change that can happen to one — as pure functions, with no DOM anywhere
near them.

**Files:** `src/todos.ts`, `src/todos.test.ts`.

Implement everything in **Architecture → `src/todos.ts`** except `itemsLeftLabel`, which comes in
task 9.

**Scenarios**

```gherkin
Feature: The todo model

  Scenario: Adding appends a todo that is not done
    Given an empty list
    When I add "Buy milk" with id "a"
    Then the list is [{ id: "a", title: "Buy milk", done: false }]

  Scenario: Adding trims the title
    When I add "  Buy milk  " with id "a"
    Then the todo's title is "Buy milk"

  Scenario: Adding an empty or whitespace-only title is refused
    Given a list with one todo
    When I add "   " with id "b"
    Then I get back the very same list object

  Scenario: Adding does not mutate the list it was given
    Given a list with one todo
    When I add another
    Then the original list still has one todo

  Scenario: Toggling flips done, and flips it back
    Given a list with todo "a" not done
    When I toggle "a"
    Then "a" is done
    When I toggle "a" again
    Then "a" is not done

  Scenario: Toggling an unknown id changes nothing
    Then toggling "zzz" returns the very same list object

  Scenario: Renaming changes only the title, trimmed
    Given todo "a" titled "Buy milk" and done
    When I rename "a" to "  Buy oat milk "
    Then "a" is titled "Buy oat milk" and still done

  Scenario: Renaming to an empty title is refused
    Then renaming "a" to "  " returns the very same list object

  Scenario: Renaming an unknown id changes nothing and does not throw
    Then renaming "zzz" returns the very same list object

  Scenario: Removing takes the todo out and keeps the order of the rest
    Given todos "a", "b", "c"
    When I remove "b"
    Then the list is "a", "c"

  Scenario: Removing an unknown id changes nothing
    Then removing "zzz" returns the very same list object

  Scenario: Clearing completed keeps only what is not done
    Given "a" done, "b" not done, "c" done
    Then clearing completed leaves only "b"

  Scenario: Counting what is left
    Given "a" done, "b" not done, "c" not done
    Then countActive is 2
```

**Done when**
- Every scenario is a passing test, and `pnpm check` passes.
- `src/todos.ts` imports nothing, and never uses `push`, `splice`, `sort` in place, or assignment to
  a todo's property.

---

### 3. Remember todos across a reload

**Files:** `src/storage.ts`, `src/storage.test.ts`, `src/test/fake-storage.ts`.

Implement **Architecture → `src/storage.ts`**. `src/test/fake-storage.ts` exports
`fakeStorage(initial?: Record<string, string>): StorageLike & { readonly data: Map<string, string> }`,
backed by a `Map`, so tests can look at exactly what was written.

**Scenarios**

```gherkin
Feature: Remember todos across a reload

  Scenario: What is saved loads back the same
    Given an empty fake storage
    When I save [{ id: "a", title: "Buy milk", done: true }]
    And I load
    Then I get [{ id: "a", title: "Buy milk", done: true }]

  Scenario: The saved value is versioned under one key
    When I save any list
    Then storage has exactly one key, "todolist"
    And its value parses to an object whose "version" is 1 and whose "todos" is the list

  Scenario Outline: Anything unreadable loads as an empty list
    Given the key "todolist" holds <value>
    When I load
    Then I get an empty list, and nothing is thrown

    Examples:
      | value                                                       |
      | nothing at all                                              |
      | "{not json"                                                 |
      | "null"                                                      |
      | "[]"                                                        |
      | '{"version":2,"todos":[]}'                                  |
      | '{"version":1,"todos":"nope"}'                              |
      | '{"version":1,"todos":[{"id":"a","title":"x"}]}'            |
      | '{"version":1,"todos":[{"id":1,"title":"x","done":false}]}' |

  Scenario: Loading never writes
    Given the key "todolist" holds "{not json"
    When I load
    Then the key "todolist" still holds "{not json"
```

Use `it.each` for the outline, one row per example.

**Done when**
- Every scenario passes, and `pnpm check` passes.
- `src/storage.ts` never mentions `localStorage` or `window`. The storage is always passed in.

---

### 4. Show the list

The todos drawn as a card of rows, and the wiring every later task hangs its behaviour on.

**Files:** `index.html` (add the `#todos` section), `src/state.ts`, `src/icons.ts` (the `check`
icon only), `src/render.ts` (`renderList`), `src/app.ts` (`mountApp`, drawing only),
`src/main.ts`, `src/app.css`, `src/test/page.ts`, `src/render.test.ts`, `src/icons.test.ts`.

**Build it like this**

- `src/main.ts` for now: `mountApp(document.body);`. Task 5 changes it to the full version in
  **Architecture**, once storage exists.
- `mountApp` reads `options.todos ?? []`, makes the state with `initialState`, calls
  `renderList` on `root.querySelector<HTMLElement>('#todos')`, and returns `() => {}`. Throw a clear
  error if `#todos` is missing.
- Each row is the markup in **The page**, minus what later tasks add. In task 4 the tick is just
  `<span class="todo-check-box" aria-hidden="true">` holding the `check` icon.
- `src/test/page.ts` exports `loadPage(): HTMLElement`. It reads `index.html` with `node:fs`, puts
  what is between `<body>` and `</body>` into `document.body.innerHTML`, removes every `<script>`,
  and returns `document.body`. Tests may use `innerHTML`; application code may not.
- Add to `src/app.css`:
  ```css
  .todos {
    background: var(--color-surface);
    border: 1px solid var(--color-border);
    border-radius: var(--radius);
    box-shadow: var(--shadow-raised);
    overflow: hidden;
  }
  .todo-list { list-style: none; margin: 0; padding: 0; }
  .todo {
    display: flex;
    align-items: center;
    gap: var(--space-3);
    min-height: var(--control-size);
    padding: var(--space-2) var(--space-4);
    border-top: 1px solid var(--color-border);
  }
  .todo:first-child { border-top: 0; }
  .todo-check-box {
    flex: none;
    display: inline-grid;
    place-items: center;
    width: var(--icon-size);
    height: var(--icon-size);
    border: 2px solid var(--color-muted);
    border-radius: 50%;
    color: var(--color-on-accent);
    transition: background-color var(--duration), border-color var(--duration);
  }
  .todo-check-box svg {
    width: var(--icon-size-sm);
    height: var(--icon-size-sm);
    opacity: 0;
    transition: opacity var(--duration);
  }
  .todo--done .todo-check-box { background: var(--color-accent); border-color: var(--color-accent); }
  .todo--done .todo-check-box svg { opacity: 1; }
  .todo-title { flex: 1; min-width: 0; overflow-wrap: anywhere; }
  .todo--done .todo-title { color: var(--color-muted); text-decoration: line-through; }
  .empty {
    margin: 0;
    padding: var(--space-6) var(--space-4);
    color: var(--color-muted);
    text-align: center;
  }
  ```

**Scenarios** — `src/render.test.ts` and `src/icons.test.ts`, jsdom environment:

```gherkin
Feature: Show the list

  Scenario: An empty list says so
    Given no todos
    When the list is rendered
    Then it shows "Nothing to do yet — add something above."
    And there is no ".todo-list"

  Scenario: Each todo is a row with its title
    Given todos "Buy milk" and "Walk the dog"
    When the list is rendered
    Then there are two ".todo" rows, in that order, showing those titles
    And each row's "data-id" is its todo's id

  Scenario: A finished todo is visibly finished
    Given "Buy milk" is done and "Walk the dog" is not
    When the list is rendered
    Then only the "Buy milk" row has the class "todo--done"

  Scenario: A title is text, never markup
    Given a todo titled "<b>bold</b>"
    When the list is rendered
    Then the row shows the literal text "<b>bold</b>" and contains no <b> element

  Scenario: Rendering is repeatable
    Given any list
    When it is rendered twice into the same section
    Then the section's HTML is identical both times, and there is one ".todo-list", not two

  Scenario: Mounting draws the page it is given
    Given the real index.html loaded with loadPage()
    When I mount the app with todos "Buy milk"
    Then "#todos" shows a row "Buy milk"

Feature: Icons

  Scenario: An icon inherits its colour and hides from assistive technology
    When I make the "check" icon
    Then it is an <svg> with stroke "currentColor", fill "none" and aria-hidden "true"
```

**Done when**
- Every scenario passes, and `pnpm check` (with the contract test) passes.
- `pnpm dev` shows the card with the empty message. With a todo passed to `mountApp` by hand, it
  shows a row with a round tick, and a done one shows a filled accent circle, a white tick and a
  struck-through muted title. Undo that by-hand edit before finishing.
- Grepping `src/` for `innerHTML` finds it only in `src/test/`.

---

### 5. Add a todo

**Files:** `index.html` (add the form), `src/app.ts`, `src/main.ts` (the full version from
**Architecture**), `src/app.css`, `src/app.add.test.ts`.

**Build it like this**

- Listen for `submit` on `#new-todo`. Always call `event.preventDefault()`. Read the input's value
  and call `addTodo(state.todos, value, newId())`. If the result is the same list, do nothing more:
  leave the text where it is. Otherwise `update`, then clear the input and keep focus in it.
- `mountApp` now uses `options.save` (default: do nothing) and `options.newId` (default:
  `() => crypto.randomUUID()`).
- Add to `src/app.css`:
  ```css
  .new-todo { margin: 0 0 var(--space-4); }
  .new-todo-input {
    width: 100%;
    font: inherit;
    font-size: var(--text-lg);
    padding: var(--space-3) var(--space-4);
    color: var(--color-text);
    background: var(--color-surface);
    border: 1px solid var(--color-border);
    border-radius: var(--radius);
    box-shadow: var(--shadow-raised);
  }
  .new-todo-input::placeholder { color: var(--color-muted); opacity: 1; }
  ```

**Scenarios** — jsdom, using `loadPage()` and a `save` spy (`vi.fn()`):

```gherkin
Feature: Add a todo

  Scenario: Submitting adds what is typed and clears the field
    Given the app is open with no todos
    When I type "Buy milk" into the field labelled "New todo" and submit the form
    Then the list shows "Buy milk"
    And the field is empty

  Scenario: What is added is saved
    When I add "Buy milk"
    Then save was called once, with a list holding one todo titled "Buy milk"

  Scenario: Whitespace-only input adds nothing and keeps what was typed
    When I type "   " and submit
    Then the list still says "Nothing to do yet — add something above."
    And the field still holds "   "
    And save was not called

  Scenario: Several in a row
    When I add "one", then "two", then "three"
    Then the list shows "one", "two", "three" in that order
    And the field has focus

  Scenario: What was saved is there on the next start
    Given the app was mounted with todos loaded from a fake storage holding "Buy milk"
    Then the list shows "Buy milk"
```

**Done when**
- Every scenario passes, and `pnpm check` passes.
- In `pnpm dev`: type, press Enter, the row appears, and you can keep typing. Reload, and it is
  still there. `localStorage.todolist` in the devtools holds `{"version":1,"todos":[…]}`.

---

### 6. Tick a todo off, and delete it

**Files:** `src/icons.ts` (add `trash`), `src/render.ts`, `src/app.ts`, `src/app.css`,
`src/app.toggle-delete.test.ts`.

**Build it like this**

- Task 4 may already have drawn the label and checkbox below without wiring them up. If so, keep
  that markup: do not draw a second checkbox. Your job is then the listener and the tests.
- In each row, wrap the existing `.todo-check-box` in `<label class="todo-check">` and put the
  checkbox before it inside the label: `<input class="todo-toggle visually-hidden" type="checkbox">`,
  `checked` when done, `aria-labelledby="title-{id}"`. Give the title span `id="title-{id}"`.
- Add the delete button after the title: `<button class="icon-button todo-delete" type="button">`
  with `aria-label` set to `Delete “{title}”` (curly quotes) and the `trash` icon inside.
- One `change` listener on `#todos` handles `.todo-toggle`, and one `click` listener handles
  `.todo-delete`. Each finds the id with `target.closest('.todo')?.dataset.id`.
- No confirmation dialogue for deleting a single todo.
- Add to `src/app.css`:
  ```css
  .todo-check { position: relative; display: inline-grid; cursor: pointer; }
  .todo-toggle:focus-visible + .todo-check-box {
    outline: 2px solid var(--color-focus);
    outline-offset: var(--focus-offset);
  }
  .icon-button {
    flex: none;
    display: inline-grid;
    place-items: center;
    width: var(--control-size);
    height: var(--control-size);
    padding: 0;
    border: 0;
    border-radius: var(--radius);
    background: transparent;
    color: var(--color-muted);
    cursor: pointer;
    transition: color var(--duration);
  }
  .icon-button svg { width: var(--icon-size); height: var(--icon-size); }
  .icon-button:hover { color: var(--color-text); }
  .todo-delete:hover { color: var(--color-danger); }
  ```

**Scenarios**

```gherkin
Feature: Tick a todo off, and delete it

  Scenario: Ticking a todo marks it done and saves it
    Given the app is open with "Buy milk" not done
    When I click the checkbox labelled "Buy milk"
    Then the checkbox is checked and the row has the class "todo--done"
    And save was called with "Buy milk" done

  Scenario: Unticking marks it not done again
    Given "Buy milk" is done
    When I click its checkbox
    Then the row does not have the class "todo--done"

  Scenario: Deleting removes the row at once and saves
    Given "Buy milk" and "Walk the dog"
    When I click the button labelled "Delete “Buy milk”"
    Then only "Walk the dog" is left
    And save was called with only "Walk the dog"

  Scenario: The delete control names its todo
    Given "Buy milk"
    Then its delete button's accessible name is "Delete “Buy milk”", not just "Delete"
    And the button contains an <svg> and no text

  Scenario: Deleting the last todo shows the empty message
    Given only "Buy milk"
    When I delete it
    Then the list says "Nothing to do yet — add something above."
```

**Done when**
- Every scenario passes, and `pnpm check` passes.
- In `pnpm dev`: clicking the circle or the checkbox fills it, and the title strikes through. The
  trash icon turns red on hover. Both changes survive a reload.

---

### 7. Rename a todo in place

**Files:** `src/icons.ts` (add `pencil`), `src/render.ts`, `src/app.ts`, `src/app.css`,
`src/app.rename.test.ts`.

**Build it like this**

- Add the rename button between the title and delete: `<button class="icon-button todo-edit"
  type="button">`, `aria-label` `Rename “{title}”`, `pencil` icon. It exists so renaming works with
  no mouse. Double-clicking the title does the same thing.
- Starting a rename sets `state.editingId` to that id. `renderList` then draws that row as
  `<li class="todo todo--editing">` holding only
  `<input class="todo-edit-input" type="text" aria-label="Rename “{title}”">` with its value set to
  the title. After `draw()`, the app focuses that input and calls `.select()`.
- **Enter** commits, **Escape** cancels, and **leaving the field** (`focusout`) commits. The order
  matters: always set `editingId` to `null` *first*, then redraw. The redraw removes the field,
  which can fire `focusout`, so the `focusout` handler must do nothing when `state.editingId` no
  longer matches that row.
- Committing with **Enter**: if the trimmed value is empty, `removeTodo`, then move focus to
  `#new-todo-input`. Otherwise `renameTodo`, then focus that row's `.todo-edit` button. Escape also
  focuses that row's `.todo-edit` button, and leaves the title as it was.
- Committing by **leaving the field** (`focusout`) saves the same way but **never moves focus**.
  The person has just put focus somewhere else, for example by clicking into the new todo field,
  and taking it back would undo their click.
- Add to `src/app.css`:
  ```css
  .todo-edit-input {
    flex: 1;
    min-width: 0;
    font: inherit;
    padding: var(--space-1) var(--space-2);
    color: var(--color-text);
    background: var(--color-bg);
    border: 1px solid var(--color-accent);
    border-radius: var(--radius);
  }
  ```

**Scenarios**

```gherkin
Feature: Rename a todo in place

  Scenario: Double-clicking a title opens it for editing, text selected
    Given "Buy milk"
    When I double-click its title
    Then the row holds a field labelled "Rename “Buy milk”" with value "Buy milk"
    And that field has focus with all of its text selected

  Scenario: The rename button does the same
    When I click the button labelled "Rename “Buy milk”"
    Then the row holds the rename field, focused

  Scenario: Enter commits
    Given I am renaming "Buy milk"
    When I change it to "Buy oat milk" and press Enter
    Then the row shows "Buy oat milk" and save was called with it
    And the "Rename “Buy oat milk”" button has focus

  Scenario: Escape cancels
    Given I am renaming "Buy milk"
    When I change it to "Buy oat milk" and press Escape
    Then the row shows "Buy milk" and save was not called

  Scenario: Leaving the field commits
    Given I am renaming "Buy milk"
    When I change it to "Buy oat milk" and focus moves elsewhere
    Then the row shows "Buy oat milk"

  Scenario: Leaving the field leaves focus where it went
    Given I am renaming "Buy milk"
    When I change it to "Buy oat milk" and focus the new todo field
    Then the new todo field still has focus"

  Scenario: Committing an empty title deletes the todo
    Given I am renaming "Buy milk"
    When I clear the field and press Enter
    Then "Buy milk" is gone and the new todo field has focus

  Scenario: A done todo stays done when renamed
    Given "Buy milk" is done
    When I rename it to "Buy oat milk"
    Then "Buy oat milk" is still done
```

`element.focus()` in jsdom does move `document.activeElement`. Check selection with
`input.selectionStart === 0 && input.selectionEnd === input.value.length`. jsdom does **not** fire
`focusout` when a redraw removes the focused field, although some browsers do. The guard on
`state.editingId` is for the browsers, so it must be there even though no jsdom test can reach it.

**Done when**
- Every scenario passes, and `pnpm check` passes.
- In `pnpm dev`: double-click, type, Enter; double-click, Escape; tab to the pencil, press Enter or
  Space, type, Enter. All three behave as described.

---

### 8. Filter by what is left

**Files:** `index.html` (add the footer), `src/filter.ts`, `src/filter.test.ts`, `src/render.ts`
(`renderFooter`, and filtering in `renderList`), `src/app.ts`, `src/app.css`,
`src/app.filter.test.ts`.

**Build it like this**

- Implement **Architecture → `src/filter.ts`**. `parseFilter` accepts `'#/active'` and
  `'#/completed'`. Everything else — `''`, `'#/'`, `'#'`, `'#/Active'`, `'#/nope'` — is `'all'`.
- `renderList` draws `applyFilter(state.todos, state.filter)`. When that is empty but
  `state.todos` is not, the empty message depends on the filter: active →
  "Nothing left to do. Nice.", completed → "Nothing finished yet."
- `renderFooter` draws the `nav.filters` from **The page**, with `aria-current="page"` on the
  current one, and sets `footer.hidden` when there are no todos at all.
- `mountApp` reads the filter from `window.location.hash` at start-up, and listens for
  `hashchange` on `window` to update `state.filter` from `parseFilter(window.location.hash)`. The
  cleanup function it returns removes that listener. Changing filter never touches the todos, so it does not save.
- Add to `src/app.css`:
  ```css
  .footer {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    justify-content: space-between;
    gap: var(--space-3);
    margin-top: var(--space-4);
    padding: 0 var(--space-2);
    font-size: var(--text-sm);
    color: var(--color-muted);
  }
  .filters { display: flex; gap: var(--space-1); }
  .filter-link {
    padding: var(--space-1) var(--space-3);
    border-radius: var(--radius);
    color: var(--color-muted);
    text-decoration: none;
    transition: color var(--duration), background-color var(--duration);
  }
  .filter-link:hover { color: var(--color-text); }
  .filter-link[aria-current="page"] { background: var(--color-accent); color: var(--color-on-accent); }
  ```

**Scenarios** — `src/filter.test.ts` (node) and `src/app.filter.test.ts` (jsdom):

```gherkin
Feature: Filter by what is left

  Scenario Outline: The hash picks the filter
    Then parseFilter("<hash>") is "<filter>"

    Examples:
      | hash         | filter    |
      | #/active     | active    |
      | #/completed  | completed |
      | #/           | all       |
      |              | all       |
      | #/nope       | all       |
      | #/Active     | all       |

  Scenario: Each filter shows the right todos
    Given "a" done and "b" not done
    Then applyFilter for all is "a", "b"; for active is "b"; for completed is "a"

  Scenario: Following a filter link filters the list
    Given the app is open with "Buy milk" done and "Walk the dog" not done
    When the hash becomes "#/active"
    Then the list shows only "Walk the dog"
    And the "Active" link has aria-current "page" and the others have none

  Scenario: A reload keeps the filter
    Given the hash is "#/completed" before the app is mounted
    When the app is mounted
    Then the list shows only "Buy milk"

  Scenario: An unknown hash shows everything
    Given the hash is "#/nope"
    When the app is mounted
    Then both todos are shown and "All" is current

  Scenario: Switching filters does not change any todo
    When the hash goes to "#/active", then "#/completed", then "#/"
    Then both todos are shown, unchanged, and save was never called

  Scenario: An empty filter says why
    Given only "Buy milk", done
    When the hash becomes "#/active"
    Then the list says "Nothing left to do. Nice."

  Scenario: No todos, no footer
    Given the app is open with no todos
    Then the footer is hidden
```

To simulate a filter change in jsdom, set `window.location.hash`, then
`window.dispatchEvent(new HashChangeEvent('hashchange'))` so the change happens now. jsdom also fires
its own `hashchange` a moment later, so the handler runs twice. That is harmless as long as the
handler reads `window.location.hash` rather than the event's `newURL`, so write it that way.

**Done when**
- Every scenario passes, and `pnpm check` passes.
- In `pnpm dev`: the pills filter the list. The current one is filled with the accent colour. The
  URL shows `#/active`, and reloading or opening that URL in a new tab keeps the filter.

---

### 9. Count what is left, and clear what is done

**Files:** `src/todos.ts` (add `itemsLeftLabel`), `src/todos.test.ts`, `src/render.ts`,
`src/app.ts`, `src/app.css`, `src/app.footer.test.ts`.

**Build it like this**

- `itemsLeftLabel(1)` is `"1 item left"`, and any other number `n` gives `"{n} items left"`,
  including 0. `src/todos.ts` may already export it, because task 2 added it early. If so, keep that
  one: do not write a second one. Add the pluralisation scenarios to `src/todos.test.ts`, which does
  not test it yet.
- Task 8 already draws `<span class="todo-count">`, pluralising inline
  (`` `${count} item${count === 1 ? '' : 's'} left` ``). Keep the span, but make its text
  `itemsLeftLabel(countActive(state.todos))`, so the wording lives in one tested place.
- `renderFooter` puts `<span class="todo-count">` before the filters, and
  `<button class="text-button clear-completed" type="button">Clear completed</button>` after them,
  *only* when at least one todo is done. The count is of all active todos, whatever the filter.
- Clicking it runs `clearCompleted`, then moves focus to `#new-todo-input`, because the button it
  was on has gone.
- Add to `src/app.css`:
  ```css
  .text-button {
    font: inherit;
    padding: var(--space-1) var(--space-2);
    border: 0;
    border-radius: var(--radius);
    background: transparent;
    color: var(--color-muted);
    cursor: pointer;
    transition: color var(--duration);
  }
  .text-button:hover { color: var(--color-danger); }
  ```

**Scenarios**

```gherkin
Feature: Count what is left, and clear what is done

  Scenario Outline: The count is pluralised
    Then itemsLeftLabel(<n>) is "<label>"

    Examples:
      | n | label         |
      | 0 | 0 items left  |
      | 1 | 1 item left   |
      | 2 | 2 items left  |

  Scenario: The footer counts what is left, whatever the filter
    Given "a" done, "b" and "c" not done, and the hash "#/completed"
    Then the footer says "2 items left"

  Scenario: Ticking one off updates the count
    Given "b" and "c" not done
    When I tick "b"
    Then the footer says "1 item left"

  Scenario: Clear completed removes every finished todo
    Given "a" done, "b" not done, "c" done
    When I click "Clear completed"
    Then only "b" is left, save was called with only "b"
    And the new todo field has focus

  Scenario: No finished todos, no clear button
    Given "b" not done
    Then there is no "Clear completed" button

  Scenario: The footer disappears with the last todo
    Given only "b"
    When I delete it
    Then the footer is hidden
```

**Done when**
- Every scenario passes, and `pnpm check` passes.
- In `pnpm dev`: the count changes as you tick, "Clear completed" appears with the first ticked
  todo and turns red on hover, and the whole footer goes when the list is empty.

---

### 10. Make it usable with a keyboard alone

Everything already works with a mouse. This task makes sure all of it works without one, and
finishes the polish.

**Files:** `src/app.ts`, `src/app.css`, `src/app.keyboard.test.ts`, `README.md`.

**Build it like this**

- **Tab order** is the page order, and nothing gets a positive `tabindex`: new todo field, then per
  row the checkbox, rename and delete, then the three filters, then "Clear completed".
- **Focus after a row disappears.** When a delete, or a tick under the "active" or "completed"
  filter, takes the focused row out of view, focus moves to the same control in the **next**
  visible row. If there is none, to the previous row's. If there are no rows, to
  `#new-todo-input`. **Decide this after `update`, not before:** if the row is still in the list
  after the redraw, which is always the case when ticking under "All", focus the same control in
  that same row. Only a row that has actually gone moves focus to a neighbour. Work out the neighbour's id *before* calling `update`, then query for it
  after. Two exceptions keep their own rule and their tests: an empty rename (task 7) and "Clear
  completed" (task 9) both send focus to `#new-todo-input`.
- **Focus is never lost to `<body>`.** After any action, `document.activeElement` is a control on
  the page.
- **Big text.** At 200% zoom nothing overlaps and nothing scrolls sideways. `.footer` already wraps.
  Check that long titles wrap (`overflow-wrap: anywhere` is already there) and that no rule sets a
  fixed `height` on text.
- **Reduced motion.** The token already drops `--duration` to `0ms`. Make sure every `transition`
  in `app.css` uses `var(--duration)` and nothing else.
- **README.** Add a short **The app** section at the top of `README.md`: one sentence saying what
  it is, then `pnpm install`, `pnpm dev`, `pnpm test`, `pnpm build`. Keep everything the README
  already says about Factory below it.

**Scenarios**

```gherkin
Feature: Make it usable with a keyboard alone

  Scenario: Every control has an accessible name
    Given the app is open with "Buy milk" and "Walk the dog", one done
    Then every input, button and link on the page has a non-empty accessible name
      (its label, aria-label, aria-labelledby target text, or its own text)

  Scenario: No positive tabindex anywhere
    Then no element has a tabindex greater than 0

  Scenario: Deleting moves focus to the next row
    Given "a", "b", "c"
    When I delete "b" using its delete button
    Then the delete button of "c" has focus

  Scenario: Deleting the last row moves focus to the previous one
    Given "a", "b"
    When I delete "b"
    Then the delete button of "a" has focus

  Scenario: Deleting the only row moves focus to the new todo field
    Given only "a"
    When I delete it
    Then the new todo field has focus

  Scenario: Ticking under "All" keeps focus on the same checkbox
    Given "a" and "b" not done, and the hash "#/"
    When I tick "a"
    Then the checkbox of "a" has focus, and it is checked

  Scenario: Ticking a row out of the active filter keeps focus in the list
    Given "a" and "b" not done, and the hash "#/active"
    When I tick "a"
    Then the checkbox of "b" has focus

  Scenario: A whole session without a mouse
    Given the app is open with no todos
    When I add "one" and "two" by typing and submitting
    And I tick "one" with a change event on its checkbox
    And I rename "two" to "three" with the rename button and Enter
    And I clear completed
    Then the list shows only "three", and focus is on the new todo field
```

**Done when**
- Every scenario passes, and `pnpm check` passes.
- By hand in `pnpm dev`, with the mouse put away: add, tick, rename, delete, filter and clear
  everything using Tab, Shift+Tab, Space and Enter. The focus ring is visible on every control, in
  light and dark. Toggle dark mode in the OS or in devtools' rendering panel.
- At 200% browser zoom and at 360px wide, nothing overlaps and there is no horizontal scrollbar.
- With `prefers-reduced-motion: reduce` emulated in devtools, nothing animates.
- `README.md` has **The app** section.

## Running it

```bash
pnpm install
pnpm dev      # http://localhost:5180 (pinned; fails if the port is taken)
pnpm test
pnpm check    # types, tests and build: what every task must pass
pnpm build
```
