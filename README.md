# project-standard

These are general guidelines for naming that make a Next.js codebase easy to understand at a high level. We have found them useful. They are general principles that could be applied in other tech stacks if desired.

---

## The `app/` folder

Next.js App Router.

- `name/` — a route (has a `page.tsx`).
- `_name/` — not a route; supporting code. Next ignores underscore folders, so `_actions/ _queries/ _components/ _providers/` are never URLs. Every non-route folder takes the underscore.
- `(name)/` — a route group; organizes without adding to the URL.

**Every route is the same shape** — the reason this is important or useful is that if we have consistent naming and folder and file structures, it's easy to derive a general set of rules to apply to other new features. See **Route shape** below.

**Scope runs from portable to local.** Components at the source level have the widest application — usable anywhere in this app, and potentially in another one entirely, with no dependency on this repo. `app/_components/` sits between: not owned by any single route. A route's own `_components/` are purpose-built for that route. Naming gets shorter as the scope narrows, because the location supplies the meaning — an `activity` folder doesn't need an `activity-stepper`, just a `stepper`.

**Rules:**

- Concerns are always folders, never a bare file: `_actions/x.ts`, not `_actions.ts`.
- Named by subject; don't repeat the parent — `activity/_components/counter.tsx`, not `activity-counter.tsx`. Exception: qualify to avoid a naming conflict.
- No `index.ts` barrels inside `app/`; import the named file.
- kebab-case; import alias `#/` → `src/`.
- `src/features/` holds domain code shared beyond one route.

Follow this as closely as the repo can. Exceptions are expected: where a note, a quirk or a peculiarity of the particular app or framework is concerned, that is the repo's business — keep it in `docs/DELTAS.md`, along with the gotchas that can interfere with a well-running app.

---

## Route shape

```
app/<route>/
├── page.tsx              renders <View />, nothing else
├── view.tsx              composes components; no className, no module
├── use-view.ts           the state view.tsx renders -- omit if there is none
├── _actions/             this route's writes  (_queries/ for reads)
├── _hooks/               every other hook -- appears only once there is one
│   └── use-<subject>.ts
└── _components/
    └── <subject>/        <subject>.tsx + <subject>.module.css
```

`page.tsx`, `view.tsx` and `use-view.ts` sit bare in the route folder. They are
**route files**, the category Next.js already puts there (`page`, `layout`,
`loading`, `error`), not components — so "one folder per component, everywhere,
no exceptions" does not reach them. State it as the category, never as an
exception, or the next reader argues their component is special too.

**`view.tsx` carries no styles.** No `className`, no module of its own. It is a
`Stack` of components. Where the frame itself is the design — a full-height
centred login column, an infinite canvas surface — that frame becomes a named
component (`centered-panel`, `canvas-surface`), not a module on the view.

Not tidiness. It converts markup into named things: a styled `<div>` stays
anonymous forever, a component has to be called something, and naming it is what
reveals it already exists elsewhere. The cost is that it needs a layout
primitive to exist first, or the view has nowhere to put spacing.

**`use-view.ts` stays bare; a second hook creates `_hooks/`** and takes that one
and every one after it. `view.tsx` and `use-view.ts` are one thing cut in two,
which is why they share a base name — filing one away hides the pair. Every
other hook is a part, and parts live in folders, exactly as components do. The
folder is earned, so its presence carries information: this route has state
beyond its view.

**Naming follows the component rule:** one role-named wrapper (`view`,
`use-view`), subject-named parts (`_hooks/use-generate.ts`, not
`use-canvas-generate.ts`). A bare shape is not a subject — `row`, `card`,
`panel`, `list` need one; `totals`, `filters` already are one.

### The server/client seam

**`page.tsx` fetches. The view is seeded, not empty.** It is a server component
that runs the route's read and hands the result to a `'use client'` view as
`initial`; `use-view.ts` seeds its state from that prop and owns every read
after it.

```tsx
export default async function Trash() {
  const initial = await listTrashedImages();
  return <View initial={initial} />;
}
```

- **The loading state stops existing.** A spinner covering a query the server
  already ran is work the page invented for itself.
- **Auth resolves once, on the server.** The client hook stops taking a `userId`
  prop it only used to decide whether to fetch.
- **`initial` is a seed, not the source of truth.** A route that reads the prop
  on every render instead of `useState(initial)` snaps back to server state
  mid-interaction.

### Converting an existing route

Do **one job per commit** — convert, then rename, then move, then extract. A
diff cannot distinguish a styling regression from a renamed import, and reuse
converges values a faithful conversion is trying to preserve.

Prove it renders identically with a pixel diff rather than by eye. Set a tall
viewport instead of using full-page screenshots (those resize and re-render, so
lazy images land mid-paint and the diff is nondeterministic), and park the
pointer off _content_ — a stray hover repaints a row border and reads as
thousands of changed pixels. When a region is genuinely unstable, mask it and
say so: "excluding the thumbnail column, max delta 0" is a stronger claim than
one number over the whole image.

---

## Components

Base UI is the primitive vocabulary. Reuse before you wrap, wrap before you build.

**Where a component lives** — the widest scope that still owns it. No dependency on this app at all (no `#/features`, no `next/*`) makes it a primitive: `src/components/`, where another app could take it as-is. Needs the app but isn't owned by any single route: `app/_components/`. Otherwise it lives with the route that uses it. The test is domain-agnostic and stack-portable, not atomicity — a compound built from three Base UI parts is still a primitive if nothing in it knows about this app.

**One folder per component — everywhere, no exceptions.** Every component is its own
folder holding `name.tsx` + its co-located `name.module.css` (+ any private hook),
in `src/components/`, `app/_components/`, and every route's own `_components/` alike.
The folder listing _is_ the catalogue. No grouping-by-area folders (`nav/`, `forms/`)
adding a dig — the component name already carries the area. One shape, so an agent
placing a new component copies its neighbors and can't land anywhere else.

**One barrel per library, at its root — never per-component.** A published library
(`src/components/`) exposes a single root `index.ts` (`#/components`); the folders
below carry no `index.ts` of their own. App-internal glue (`app/_components/`, route
`_components/`) has no barrel at all — import the named file
(`#/app/_components/rerun-modal/rerun-modal`). A nested component reaches its route's
sibling concerns **two** levels up (`../../_actions/x`, not `../_actions/x`) — the
component's own folder is the first hop.

**Order of search when a view needs a component:**

1. Shop the shelf — build it from `src/components/` if you can.
2. Base UI for the gaps — if Base UI has the primitive, build from it.
3. shadcn as scout, never source — if Base UI lacks it, adopt the library shadcn builds on (e.g. `react-day-picker` for calendars), not shadcn's code.
4. Hand-build only for a true outlier (a gallery).

**Rules:**

- Promote broadly-usable components to `src/components/`; the next use reuses them.
- Build on need. No speculative primitives. Bar to promote: properly shareable, not used twice.
- Wrap Base UI, don't scatter it — one wrapper owns the prop API. Headless only; style with tokens; one visual language.
- **Icons come from `lucide-react`.** Hand-roll an SVG only when nothing in the library fits — a rectangle drawn at the aspect ratio you picked, not a trash can. A copied glyph also opts out of the sizing a button applies to its `svg` children.
- Styling is CSS-only via a co-located `*.module.css` — see **Styling** below.

---

## Styling

**Color, type, and scale are central tokens. Components style via CSS Modules. Bare
semantic tags carry a baseline. CSS only — no Tailwind.** No layers of abstraction,
CSS co-located with the component it styles, and a standard that is widely supported
with no compile requirement. Straight CSS gives the most flexibility and control
without conforming to a framework's decisions and conventions, or being boxed into an
aesthetic defined by a library. Tailwind's ergonomics solve _human_ pains — no naming,
no file-switching — that an AI author doesn't feel, while its churn (v4 was a breaking
rewrite) is a cost that lands on everyone.

**The layered model** — values flow up from L0; override power increases up:

```
 L3  scope override   .compact { --h2-size: 1.5rem }   redefine a token per subtree
 L2  component module  .card { … }  .card > h2 { … }    contextual; beats L1
 L1  base elements     h1,h2,p { … var(--…) }           the reset + a bare-tag floor
 L0  tokens            :root { --surface --ink --space-* }   the values, one source
```

- **L0 — tokens** (`src/styles/tokens.css`, `:root`): the single source of values.
  Dark mode is a remap under `:root[data-theme="dark"]` — nothing hardcodes a color,
  so everything follows. Never a raw color above L0; raw numbers only for one-off
  structural values with no sensible token (a `z-index`, a 20px icon box).
- **L1 — reset + bare-tag baseline** (`base.css`): owns what a framework reset used
  to (box-sizing, `margin:0`, list/img/form resets) plus a light token-driven floor
  on bare tags. In a leaf-styled system this floor carries less than you'd think —
  keep it minimal; the real work is L2.
- **L2 — component module** (`component/component.module.css`): the workhorse. Named
  element classes, every value a token.
- **L3 — scope override**: because custom properties cascade, a context bends a token
  for everything beneath it — central default + contextual override, one mechanism.

**Color format.** Prefer HSL — it is easier to reason about. Avoid pure white and
pure black.

**Markup shape.** Prefer a wrapper class on one element and let the cascade style what
is inside — a class on every element is visual noise, and keeping them few makes both
the markup and the CSS easier to reason about. Use CSS nesting, as few class names as
possible, and avoid pseudo-selector gymnastics. Direct children of the wrapper are
fine; depth is where it turns fragile and starts reaching into nested components.

```css
.card {
  padding: var(--space-16);

  > h2 { font-size: var(--text-lg); }
  &:hover { border-color: var(--accent); }

  .meta { color: var(--text-muted); }

  /* not this — brittle, and it reaches past this component's own markup */
  div span h2 { font-weight: 600; }
}
```

**Composes with Base UI.** Base UI stays the headless vocabulary; "wrap each primitive
once and style the wrapper with its module" _is_ L2. A portalled primitive just means
the wrapper owns several portal-root classes.

### Shape of the token set

The layers say where a value lives. This says what belongs in the set, and how much
of it — nothing above stops `tokens.css` from sprawling.

- **Small enough to scan.** `tokens.css` is the reskin-by-eye surface, so it has to
  stay a size you can take in. Roughly four surfaces, two inks, two borders, one
  accent with two or three companions, three status hues. Not a 50–900 ramp — that is
  a framework's answer for an app it cannot see, and inheriting it means owning fifty
  decisions you never made.
- **One name per value, one value per name.** Two names pointing at the same value
  look correct right up until one of them moves. If they mean different things, give
  them different values now; if they don't, delete one.
- **Name the role, not the place or the vendor.** `--surface` gets reused.
  `--sidebar-bg` multiplies once per feature. `--primary` and `--card-foreground` are
  a library's vocabulary, and they outlive the components that needed them.
- **A derived token says what it derives from.** If raised-surface is base plus a
  lightness step, put that in a comment or compute it. Otherwise the two are kept in
  agreement by hand, and nothing tells you when that stops.
- **Use shadows sparingly.** A border and a slightly different background is enough to
  separate a panel from what is behind it. Save shadow for what genuinely floats —
  dialog, popover, menu. We are not building an elevation system.
- **Keep raw colors out above L0, and check it.** Unenforced, this one decays fast —
  a quarter of files during a conversion. genzen fails the build on a `.module.css`
  that writes one, and a genuine exception carries a comment saying why, so `grep` is
  the list of every place it is bent.

### Migrating an existing Tailwind repo

Tailwind stays installed until the last step — convert files to modules first, remove
the framework last (the L1 reset can't land while the framework's reset is still
active). Gotchas worth pre-loading:

- **The framework reset is load-bearing.** Removing Tailwind removes Preflight; L1
  must own the replacement reset.
- **Tokens leave `@theme`** for plain `:root { --… }`, consumed via `var(--…)`.
- **The spacing-scale trap.** Tailwind spacing is `N × 4px` by default _unless the
  repo remapped it_. Map by **px value**, not index (`gap-6` = 24px, not `--space-6`).
  Getting it wrong drifts every gap 4–8px and files diverge.
- **`dark:`** becomes `:global([data-theme='dark']) .x { }` (or a media query).
- **Keyframe utilities / `line-clamp` / `truncate`** have no token — reproduce the
  keyframes or the `-webkit-box` idiom locally.
- **Tokens the framework gave implicitly** surface as gaps to add explicitly: status
  colors (`--danger`/`--success`/`--warning`), overlay scrims, on-dark text, shadows.
  Name the **role**, then pick **one** value for it and accept the pixel shift at
  conversion time. A framework offers several values per role, and carrying two
  forward so that nothing shifts defers a decision that only gets more expensive — by
  the time a design pass reaches it, both values have call sites and a comment
  defending them.

---

## The `docs/` folder

Root-level `docs/` holds what the project is and why — things that matter but don't belong in the codebase. Root stays README-only; everything else goes under `docs/`. Docs we tend to use:

```
docs/
├── OVERVIEW.md                  what the project is and why
├── SPEC.md                      what the app does and must do
└── reference/                   domain material, framework notes
```

Plus, at the repo root: `README.md` (front door; the `## Status` block) and `CLAUDE.md` (agent orientation, also placed in folders that are a real boundary — auto-loaded when touching that tree). Both are governed by **Writing the docs** below.

Group related docs into a folder once there are enough to warrant it — several notes on one framework become `reference/<framework>/`.

A finding or note worth keeping that doesn't belong to a task or a boundary goes in `reference/` — no required name or structure. A doc earns a spot at the `docs/` top level only if it's core to the whole application (like `OVERVIEW` and `SPEC`); short of that bar, it goes in `reference/`. That keeps the top level to `OVERVIEW`, `SPEC`, and `reference/`, so a docs menu stays short and browsable.

Plans, tasks, and bugs are **not** docs — they live as GitHub issues (see Git & issues). `docs/` never holds a plan file.

---

## Naming

- **Don't repeat an ancestor folder in the name.** Location already said it. Inside `app/activity/_components/`, the component is `onboarding-form/`, not `activity-onboarding-form/` — the path reads `activity/…/onboarding-form` either way, and the prefix only pushes the one distinguishing word further right, where the eye reaches it last. The test: strip every word the ancestors already supply, and if the name still identifies the thing, those words were noise. This is the concrete form of Legibility's *a name carries only what its location doesn't*, and it is what makes placement do the work naming would otherwise duplicate. It bends where the bare name would be read far from its folder and lose its meaning there — a widely imported symbol, not a component sitting in the folder that names it.
- ALL-CAPS for big-idea docs: `README`, `CLAUDE`, `OVERVIEW`, `SPEC`.
- lowercase for folders and everything under `reference/`.
- Filename fixed, heading free: `docs/SPEC.md` is always `docs/SPEC.md`; its `#` heading names it for the project.

---

## Git & issues

Work happens on feature branches and lands through PRs. Plans, tasks, and bugs are GitHub issues — never markdown files in `docs/`.

- Never commit to `main` directly. Cut a feature branch (kebab-case) per unit of work.
- Do the work on the branch; open a PR against `main` when it's coherent and green.
- **Squash-merge, delete the branch.** _When_ to merge is a Behavior question, not a Layout one — `dotfiles` owns it, and this standard defers. Hotfix exception: a small, verified fix for broken prod can go straight to `main`.
- Capture work as issues, not docs. Planning produces an issue; the branch executes it; the PR closes it. If no issue exists for the work, create one.
- Orientation = README `## Status` + open issues. No plan files, no continuation file.
- **Bring the README up to date as part of the PR.** It is the first place either of you looks for what to do next, so work that moves where the project stands leaves `## Status` reflecting the work done and the work still to do. Not exhaustive — just where you are.
- After a squash merge, `git diff origin/main` is the "am I up to date" check — **not** `git log origin/main..HEAD`, which reports a fully-landed branch as unmerged.

Assumes an authenticated `gh` CLI the agent drives on the repo's behalf.

---

## Writing the docs

Primary reader is a model booting a session; a human is second. Brief, bulleted, no prose.

- **The code is the documentation.** Structure and naming carry the convention. A doc that restates the file tree is dead weight that goes stale.
- **State a rule once, where it binds.** The same rule in three files becomes three rules. A repo's `docs/DELTAS.md` links here and states only its delta.
- **Prose earns its place where the code cannot show it** — a past failure, a consequence, a reason. Structure teaches shape; only prose teaches consequence.
- **Stop cutting when removing one more sentence would change a decision.** Terse text stays coherent while shedding the _why_, and the why is the part a cold session cannot re-derive.
- **Length is a symptom.** A long doc is doing a job it shouldn't — cataloguing the tree, restating another file, logging history. Fix the job.
- **A ritual belongs in a skill, not a `CLAUDE.md`** — migration steps, a verification sequence, a lifecycle. It loads when it is needed.

### `CLAUDE.md` files

Write one only where the folder is a boundary you could violate without reading it — an invariant, an ownership rule, a dependency direction, a "don't do X here." If the rules are obvious from the files, skip it. A boundary is the test, not how often you work there.

It holds what this owns and its dependency direction, what it does **not** own and who does, the invariants that fail silently, and the gotchas the code can't tell you. Not a file inventory, not a restatement of global rules. It is re-read every time an agent touches the folder, so keep it short.

### The README

The front door, and the first place either of you looks for what to do next. What the project is, how to run it, where things stand — not a changelog and not a manual.

**The `## Status` block.** A snapshot, not a log. It is read at every session boot, so it is the most expensive text in the repo.

- **Last shipped — 6 bullets max, one line each, most-recent first.** The cap is the rule; without one it becomes a changelog. (Measured once at 38KB, ~9.5k tokens every boot.)
- **Up next — a short pointer to the ordered issues.** Issue bodies are the durable record.
- Narrative goes in `git log` and PR descriptions. A lesson that changes how the next work is done goes in `docs/reference/`.
- Overwrite freely.

---

## Legibility

The point of all of the above: the repo reads top-down and is cheap to traverse. You orient from the tree's surface and the boundary contracts, and open code only for the thing you came to change.

- A shallow read of the tree says what the app does: `app/` names the routes (the surface), `features/` names the domains (what it's made of).
- Minimize jumps, not folders. Colocate by default; hoist when a second consumer earns it.
- Name for the domain, not the screen or the action.
- Traverse by contracts, not implementations — a page per folder, edge to edge.
- The structure is the documentation; nothing separate explains the layout, so nothing drifts.

Naming serves the same goal: the shortest name that still communicates. A name carries only what its location doesn't — every word earns its place given where the file sits (the `app/` rules make this concrete). Fewer words, a more scannable tree, lower load.

**One way to do a thing.** An agent writes new code by copying its neighbours, not by
reading a rulebook. Every choice we leave open is a coin it flips, and different
sessions flip it differently — that is how a codebase drifts. One shape deletes the
coin.

**Say which neighbour to copy.** Building a new feature should be answerable by
opening an existing one, so name the one to copy in the root `CLAUDE.md`, and name
anything that is a known exception. Copying a neighbour is only safe if you know which
neighbour is current.

**Exceptions are fine; unexplained ones aren't.** Write the reason next to the thing,
in a form you can grep — genzen requires one on every `sql-scope-exempt` and
`raw-color-exempt`. An exception with no reason is indistinguishable from drift.

**When the codebase doesn't answer it, ask.** If nothing beside you does this yet and
what you are about to write would be meaningfully different, that is a question, not a
judgment call. A new pattern introduced quietly gets copied by everything after it.

---

## Other stacks

Everything above assumes a Next.js app, and the `app/` half is Next machinery — route
folders, `_folders`, route groups. The rest is not: legibility, naming for the domain,
one folder per component, tokens and CSS Modules, docs as issues, `CLAUDE.md` as a
boundary contract. Where a rule here comes from something general rather than from
Next, apply it. This is not a second set of rules.

The one thing `app/` gives you for free is a surface that names itself — the route
folders are the outline of the app. Without it, let the domains be the surface:
`src/viewer/`, not `src/features/viewer/`. A wrapper folder only earns its place once
there are enough domains that fencing them off helps.
