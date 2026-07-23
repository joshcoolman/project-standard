# project-standard

General guidelines for Next.js apps.

---

## The `app/` folder

Next.js App Router.

- `name/` — a route (has a `page.tsx`).
- `_name/` — not a route; supporting code. Next ignores underscore folders, so `_actions/ _queries/ _components/ _providers/` are never URLs. Every non-route folder takes the underscore.
- `(name)/` — a route group; organizes without adding to the URL.

**Depth = scope.** Inside a route = that route only. `app/_components/` = shared by 2+ routes. `src/features/` = shared across the app.

**Every route is the same shape:** `page.tsx` (entry) · `_queries/` (reads) · `_actions/` (writes) · `_components/` (renders).

**Rules:**

- Concerns are always folders, never a bare file: `_actions/x.ts`, not `_actions.ts`.
- Named by subject; don't repeat the parent — `activity/_components/counter.tsx`, not `activity-counter.tsx`. Exception: qualify to avoid a naming conflict.
- No `index.ts` barrels inside `app/`; import the named file.
- kebab-case; import alias `#/` → `src/`.
- `src/features/` holds domain code shared beyond one route.

A repo states its own copy in `docs/CODE-STANDARDS.md`, following this.

---

## Components

Base UI is the primitive vocabulary. Reuse before you wrap, wrap before you build.

**Where a component lives** — litmus: does it import `#/features`?

- **Primitive** → `src/components/` — no `#/features`, no `next/*`; drops into any React + Base UI stack. May be a thin Base UI wrapper or a compound built from them. Test: domain-agnostic + stack-portable, not atomicity.
- **App-shared** → `app/_components/` — imports `#/features`, shared across 2+ routes.
- **Route-local** → that route's `_components/`.

**One folder per component — everywhere, no exceptions.** Every component is its own
folder holding `name.tsx` + its co-located `name.module.css` (+ any private hook),
in `src/components/`, `app/_components/`, and every route's own `_components/` alike.
The folder listing *is* the catalogue. No grouping-by-area folders (`nav/`, `forms/`)
adding a dig — the component name already carries the area. One shape, so an agent
placing a new component copies its neighbors and can't land anywhere else.

**One barrel per library, at its root — never per-component.** A published library
(`src/components/`) exposes a single root `index.ts` (`#/components`); the folders
below carry no `index.ts` of their own. App-internal glue (`app/_components/`, route
`_components/`) has no barrel at all — import the named file
(`#/app/_components/rerun-modal/rerun-modal`). A nested component reaches its route's
sibling concerns one level up (`../../_actions/x`).

**Order of search when a view needs a component:**

1. Shop the shelf — build it from `src/components/` if you can.
2. Base UI for the gaps — if Base UI has the primitive, build from it.
3. shadcn as scout, never source — if Base UI lacks it, adopt the library shadcn builds on (e.g. `react-day-picker` for calendars), not shadcn's code.
4. Hand-build only for a true outlier (a gallery).

**Rules:**

- Promote broadly-usable components to `src/components/`; the next use reuses them.
- Build on need. No speculative primitives. Bar to promote: properly shareable, not used twice.
- Wrap Base UI, don't scatter it — one wrapper owns the prop API. Headless only; style with tokens; one visual language.
- Styling is CSS-only via a co-located `*.module.css` — see **Styling** below.

---

## Styling

**Color, type, and scale are central tokens. Components style via CSS Modules. Bare
semantic tags carry a baseline. CSS only — no Tailwind.** One obvious way to express
a style, so an agent never flips a coin between utilities and modules and different
sessions never diverge. (Settled by migrating a real app off Tailwind end-to-end.)

**Why CSS, not Tailwind.** CSS is a frozen platform standard; Tailwind is a
fast-moving third-party layer (v4 was a breaking rewrite), and a model's CSS
knowledge is deeper and less version-fragile. Tailwind's ergonomics solve *human*
pains (no naming, no file-switching) an AI author doesn't feel. Keeping both
available is the ambiguity we're removing.

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

**Color format — HSL.** Colors are `hsl(H S L)` / `hsl(H S L / A)` — one notation for
every value, opaque or translucent. `tokens.css` is the reskin-by-eye surface, and
the edits that come up map to channels: drop the saturation, reduce the contrast
(lightness), drop the opacity 10% (`/ 70%` → `/ 60%`). OKLCH is more capable but only
pays off when you *derive* a palette in code; for a hand-authored palette it costs
legibility for a benefit you don't cash in. Hex is fine to paste, not to nudge.
(Revisit only if a repo generates its scale from a seed color — then that repo goes
OKLCH.)

**Markup shape — leaf vs. layout.** "One wrapper class, bare tags styled through it"
is the **leaf** case (a button, a field, a prose block): one wrapper, a couple of
bare children. **Layout, form, overlay, and portalled components legitimately need a
flat set of named element classes** — one per semantic region — and that is correct,
not a smell. Repeated element types in distinct roles (three sibling buttons; a
label vs. a caption) can't be disambiguated by tag; portalled overlays have no single
wrapper (backdrop / viewport / popup are separate roots). Two idioms recur and are
the house style:

- **Base + modifier via template string** — `className={`${styles.thumb} ${error ?
  styles.thumbError : ''}`}`; the modifier sets only the delta.
- **Active/inactive conditional class** — base class always on, exactly one of
  `.xActive` / `.xInactive` added.

**Guardrail.** Styling children through the wrapper uses descendant selectors, which
reach nested child components too. Prefer the **direct-child** combinator (`.card >
h2`); reserve wrapper-styles-the-tags for **leaf/content** components that own all
their markup; keep descendant rules **shallow** (one level).

**Composes with Base UI.** Base UI stays the headless vocabulary; "wrap each primitive
once and style the wrapper with its module" *is* L2. A portalled primitive just means
the wrapper owns several portal-root classes.

### Migrating an existing Tailwind repo

Tailwind stays installed until the last step — convert files to modules first, remove
the framework last (the L1 reset can't land while the framework's reset is still
active). Gotchas worth pre-loading:

- **The framework reset is load-bearing.** Removing Tailwind removes Preflight; L1
  must own the replacement reset.
- **Tokens leave `@theme`** for plain `:root { --… }`, consumed via `var(--…)`.
- **The spacing-scale trap.** Tailwind spacing is `N × 4px` by default *unless the
  repo remapped it*. Map by **px value**, not index (`gap-6` = 24px, not `--space-6`).
  Getting it wrong drifts every gap 4–8px and files diverge.
- **`dark:`** becomes `:global([data-theme='dark']) .x { }` (or a media query).
- **Keyframe utilities / `line-clamp` / `truncate`** have no token — reproduce the
  keyframes or the `-webkit-box` idiom locally.
- **Tokens the framework gave implicitly** surface as gaps to add explicitly: status
  colors (`--danger`/`--success`/`--warning`), overlay scrims, on-dark text, shadows.

---

## The `docs/` folder

Root-level `docs/` holds what the project is and why — things that matter but don't belong in the codebase. Root stays README-only; everything else goes under `docs/`. Docs we tend to use:

```
docs/
├── OVERVIEW.md                  what the project is and why
├── SPEC.md                      what the app does and must do
└── reference/                   domain material, framework notes
```

Plus, at the repo root: `README.md` (front door; the `## Status` block = Last shipped / Up next) and `CLAUDE.md` (agent orientation — also dropped into folders an agent works in repeatedly, auto-loaded when touching that tree).

Group related docs into a folder once there are enough to warrant it — several notes on one framework become `reference/<framework>/`.

A finding or note worth keeping that doesn't belong to a task or a boundary goes in `reference/` — no required name or structure. A doc earns a spot at the `docs/` top level only if it's core to the whole application (like `OVERVIEW` and `SPEC`); short of that bar, it goes in `reference/`. That keeps the top level to `OVERVIEW`, `SPEC`, and `reference/`, so a docs menu stays short and browsable.

Plans, tasks, and bugs are **not** docs — they live as GitHub issues (see Git & issues). `docs/` never holds a plan file.

---

## Naming

- ALL-CAPS for big-idea docs: `README`, `CLAUDE`, `OVERVIEW`, `SPEC`.
- lowercase for folders and everything under `reference/`.
- Filename fixed, heading free: `docs/SPEC.md` is always `docs/SPEC.md`; its `#` heading names it for the project.

---

## Git & issues

Work happens on feature branches and lands through PRs. Plans, tasks, and bugs are GitHub issues — never markdown files in `docs/`.

- Never commit to `main` directly. Cut a feature branch (kebab-case) per unit of work.
- Do the work on the branch; open a PR against `main` when it's coherent and green.
- Merge when confident (squash), delete the branch, cut the next. Hotfix exception: a small, verified fix for broken prod can go straight to `main`.
- Capture work as issues, not docs. Planning produces an issue; the branch executes it; the PR closes it. If no issue exists for the work, create one.
- Orientation = README `## Status` + open issues. No plan files, no continuation file.

Assumes an authenticated `gh` CLI the agent drives on the repo's behalf.

---

## CLAUDE.md files

Write a subfolder `CLAUDE.md` only where the folder is a boundary you could violate without reading it — an invariant, an ownership rule, a dependency direction, a "don't do X here." If the rules are obvious from the files or already stated above, skip it; a missing one is correct when there's nothing non-obvious to say.

What it holds:

- One line: what this owns and its dependency direction (`pipeline` imports `models`, never the reverse).
- **Responsibilities** and **Does NOT own** — the boundary from both sides; each not-owned item names who owns it instead.
- Invariants that fail silently if broken, decisions worth not relitigating (with the *why*), and the gotchas the code can't tell you.

Not a file inventory, not a restatement of global rules. Keep it short — it's re-read every time an agent touches the folder, so length tracks the boundary's complexity, not the file count.

---

## Legibility

The point of all of the above: the repo reads top-down and is cheap to traverse. You orient from the tree's surface and the boundary contracts, and open code only for the thing you came to change.

- A shallow read of the tree says what the app does: `app/` names the routes (the surface), `features/` names the domains (what it's made of).
- Minimize jumps, not folders. Colocate by default; hoist when a second consumer earns it.
- Name for the domain, not the screen or the action.
- Traverse by contracts, not implementations — a page per folder, edge to edge.
- The structure is the documentation; nothing separate explains the layout, so nothing drifts.

Naming serves the same goal: the shortest name that still communicates. A name carries only what its location doesn't — every word earns its place given where the file sits (the `app/` rules make this concrete). Fewer words, a more scannable tree, lower load.

**Uniformity so pattern-matching can't miss.** Legibility is the human-facing half;
this is its agent-facing twin, and it's why the rules above are rigid rather than
"prefer." An agent writing new code pattern-matches off its neighbors, not off a
rulebook it goes and reads. So *every exception is a fork it resolves at write-time* —
and with nothing local to disambiguate, different sessions resolve the same fork
differently. That's how a codebase drifts: not one wrong decision, but a hundred small
coin-flips that each landed plausibly. One shape deletes the coin. When there is
exactly one way — one way to style (no Tailwind), one folder layout (one per
component, everywhere), one import form (one root barrel) — copying the surroundings
*always* yields the conforming answer, because there's only one thing next to it to
copy. Every degree of freedom removed is judgment that no longer varies. Prefer the
rigid rule over the flexible one wherever the flexibility buys nothing.

---

## Non-Next.js apps

The `app/` half of this standard is Next machinery — routes, `_folders`, route groups, Base UI sourcing. Strip it and the rest holds unchanged. The catch: `app/` gave you a surface that names itself. Without it, rebuild that legibility by hand.

**Transfers verbatim:** the Legibility goal; naming for the domain, not the mechanism; docs-as-issues and README `## Status`; `CLAUDE.md` as a boundary contract; the component-sourcing order and the primitive-vs-shared split.

**Rebuild the surface.** In Next, the route folders under `app/` *are* the surface, and `features/` reads as the domain layer beneath them. Without `app/`, a `features/` wrapper just hides the domains one hop down. So lift domains to the `src/` root and let them *be* the surface — `src/viewer/`, not `src/features/viewer/`. Keep a wrapper only once enough domains exist that fencing them off aids scanning; for a small or single-domain app it costs a jump and buys nothing.

**Amendments:**

- **Encode app-vs-infra with a prefix you own.** Nothing claims prefixes in a plain `src/`, so assign the meaning: `_`-prefix the non-app folders (`_components/`, `_styles/`). It reads as "infrastructure, skip me," sorts them out of the eye's path, and leaves the domain folder standing where attention lands. Verify it's inert in your toolchain first — then it's free.
- **Nesting expresses ownership.** A capability that belongs to one domain lives inside it (`viewer/search/`), not as a root-level peer. If two peers reference each other, nesting one converts the cycle into a clean parent→child dependency.
- **Colocate, then separate by concern within a domain.** Domain logic (model, store, pipeline) at the folder root; views in `components/`. The root then scans as *what the domain is*.
- **Shed scaffolding that stops earning its place.** Frameworks accrete; a hand-rolled app should drop what it no longer uses — a router once it's down to one view, a feature you stopped opening, test infra nothing imports. Audit question: carried weight, or working weight?

**Reorg checklist.** Does `src/` read as the app at a glance? Is every folder named for a domain, not a mechanism (`utils`, `core`, `helpers`)? Is infrastructure visually separated from the app? Does anything sit as a peer that's really a child? What's present that the app no longer needs?

Taste in service of legibility, not a second rulebook: the `_`-prefix and root-level domains are right *because* the app is small — a larger one may want `features/` back. Minimize jumps, not folders; a folder earns its place by a real boundary or a second consumer, never speculation.
