# project-standard

General guidelines for Next.js apps.

---

## The `app/` folder

Next.js App Router.

- `name/` — a route (has a `page.tsx`).
- `_name/` — not a route; supporting code. Next ignores underscore folders, so `_actions/ _queries/ _components/ _providers/` are never URLs. Every non-route folder takes the underscore.
- `(name)/` — a route group; organizes without adding to the URL.

**Depth = scope.** Inside a route = that route only. `app/_components/` = shared by 2+ routes. `src/features/` = shared across the app.

**Every route is the same shape** — see **Route shape** below.

**Rules:**

- Concerns are always folders, never a bare file: `_actions/x.ts`, not `_actions.ts`.
- Named by subject; don't repeat the parent — `activity/_components/counter.tsx`, not `activity-counter.tsx`. Exception: qualify to avoid a naming conflict.
- No `index.ts` barrels inside `app/`; import the named file.
- kebab-case; import alias `#/` → `src/`.
- `src/features/` holds domain code shared beyond one route.

A repo states its own copy in `docs/CODE-STANDARDS.md`, following this.

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

**Where a component lives** — litmus: does it import `#/features`?

- **Primitive** → `src/components/` — no `#/features`, no `next/*`; drops into any React + Base UI stack. May be a thin Base UI wrapper or a compound built from them. Test: domain-agnostic + stack-portable, not atomicity.
- **App-shared** → `app/_components/` — imports `#/features`, shared across 2+ routes.
- **Route-local** → that route's `_components/`.

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
semantic tags carry a baseline. CSS only — no Tailwind.** One obvious way to express
a style, so an agent never flips a coin between utilities and modules and different
sessions never diverge. (Settled by migrating a real app off Tailwind end-to-end.)

**Why CSS, not Tailwind.** CSS is a frozen platform standard; Tailwind is a
fast-moving third-party layer (v4 was a breaking rewrite), and a model's CSS
knowledge is deeper and less version-fragile. Tailwind's ergonomics solve _human_
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
pays off when you _derive_ a palette in code; for a hand-authored palette it costs
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
once and style the wrapper with its module" _is_ L2. A portalled primitive just means
the wrapper owns several portal-root classes.

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
- After a squash merge, `git diff origin/main` is the "am I up to date" check — **not** `git log origin/main..HEAD`, which reports a fully-landed branch as unmerged.

Assumes an authenticated `gh` CLI the agent drives on the repo's behalf.

---

## Writing the docs

Primary reader is a model booting a session; a human is second. Write for token efficiency — brief, bulleted, no prose.

- **The code is the documentation.** Structure and naming carry the convention. A doc that restates what the file tree already shows is dead weight that goes stale.
- **State a rule once, where it binds.** The same rule in three files drifts into three different rules and the reader has to pick a winner, burning judgment on a fork that shouldn't exist. A repo's `docs/CODE-STANDARDS.md` states its delta from this standard and links here; it does not re-copy it.
- **Prose earns its place only where the code cannot show it** — a past failure, a consequence, a reason. Uniformity teaches structure; only prose teaches consequence.
- **Where to stop cutting: when removing one more sentence would change a decision.** Not "when it stops being coherent" — terse text stays coherent while shedding the _why_, and the why is the only part a cold session can't re-derive. Be ruthless about how many ideas earn a place, then don't compress the survivors past their reason.
- **Length is a symptom.** A doc is long because it's doing a job it shouldn't — cataloguing the file tree, restating another file, logging history. Fix the job; the length follows. Optimizing length directly yields dense unusable rules.
- **A ritual belongs in a skill, not a `CLAUDE.md`.** Anything needed occasionally and expensive always (migration steps, a verification sequence, a lifecycle) loads on demand.

### `CLAUDE.md` files

Write a subfolder `CLAUDE.md` only where the folder is a boundary you could violate without reading it — an invariant, an ownership rule, a dependency direction, a "don't do X here." If the rules are obvious from the files or already stated above, skip it; a missing one is correct when there's nothing non-obvious to say. Frequency of work in a folder is not the test; a boundary is.

What it holds:

- One line: what this owns and its dependency direction (`pipeline` imports `models`, never the reverse).
- **Responsibilities** and **Does NOT own** — the boundary from both sides; each not-owned item names who owns it instead.
- Invariants that fail silently if broken, decisions worth not relitigating (with the _why_), and the gotchas the code can't tell you.

Not a file inventory, not a restatement of global rules. Keep it short — it's re-read every time an agent touches the folder, so length tracks the boundary's complexity, not the file count.

### The README `## Status` block

A snapshot, not a log — it is read at every session boot, so it is the most expensive text in the repo.

- **`**Last shipped**` — 6 bullets max, one line each, most-recent first.** The cap is the rule; without a shape it drifts into a changelog. (Measured once at 38KB, ~9.5k tokens per boot, every bullet distilled back to one line by the consumer.)
- **`**Up next**` — a short pointer to the ordered issues.** Issue bodies are the durable record.
- Narrative belongs in `git log` and PR descriptions. A lesson that would change how the next work is done goes in `docs/reference/`, not here.
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

**Uniformity so pattern-matching can't miss.** Legibility is the human-facing half;
this is its agent-facing twin, and it's why the rules above are rigid rather than
"prefer." An agent writing new code pattern-matches off its neighbors, not off a
rulebook it goes and reads. So _every exception is a fork it resolves at write-time_ —
and with nothing local to disambiguate, different sessions resolve the same fork
differently. That's how a codebase drifts: not one wrong decision, but a hundred small
coin-flips that each landed plausibly. One shape deletes the coin. When there is
exactly one way — one way to style (no Tailwind), one folder layout (one per
component, everywhere), one import form (one root barrel) — copying the surroundings
_always_ yields the conforming answer, because there's only one thing next to it to
copy. Every degree of freedom removed is judgment that no longer varies. Prefer the
rigid rule over the flexible one wherever the flexibility buys nothing.

**These are guidelines, and exceptions are expected — but an exception carries its
reason, in place.** That is what keeps the two halves above from contradicting each
other: rigidity is the default because copying your neighbours should always yield the
conforming answer, and an exception is fine precisely because it announces itself
instead of quietly becoming a second pattern. Write the reason next to the thing, in a
form you can grep — genzen requires one on every `sql-scope-exempt` and
`raw-color-exempt`, so the list of places a rule is knowingly bent is a search, not a
memory. An unjustified exception is indistinguishable from drift; a justified one is
the rule working.

---

## Non-Next.js apps

The `app/` half of this standard is Next machinery — routes, `_folders`, route groups, Base UI sourcing. Strip it and the rest holds unchanged. The catch: `app/` gave you a surface that names itself. Without it, rebuild that legibility by hand.

**Transfers verbatim:** the Legibility goal; naming for the domain, not the mechanism; docs-as-issues and README `## Status`; `CLAUDE.md` as a boundary contract; the component-sourcing order and the primitive-vs-shared split.

**Rebuild the surface.** In Next, the route folders under `app/` _are_ the surface, and `features/` reads as the domain layer beneath them. Without `app/`, a `features/` wrapper just hides the domains one hop down. So lift domains to the `src/` root and let them _be_ the surface — `src/viewer/`, not `src/features/viewer/`. Keep a wrapper only once enough domains exist that fencing them off aids scanning; for a small or single-domain app it costs a jump and buys nothing.

**Amendments:**

- **Encode app-vs-infra with a prefix you own.** Nothing claims prefixes in a plain `src/`, so assign the meaning: `_`-prefix the non-app folders (`_components/`, `_styles/`). It reads as "infrastructure, skip me," sorts them out of the eye's path, and leaves the domain folder standing where attention lands. Verify it's inert in your toolchain first — then it's free.
- **Nesting expresses ownership.** A capability that belongs to one domain lives inside it (`viewer/search/`), not as a root-level peer. If two peers reference each other, nesting one converts the cycle into a clean parent→child dependency.
- **Colocate, then separate by concern within a domain.** Domain logic (model, store, pipeline) at the folder root; views in `components/`. The root then scans as _what the domain is_.
- **Shed scaffolding that stops earning its place.** Frameworks accrete; a hand-rolled app should drop what it no longer uses — a router once it's down to one view, a feature you stopped opening, test infra nothing imports. Audit question: carried weight, or working weight?

**Reorg checklist.** Does `src/` read as the app at a glance? Is every folder named for a domain, not a mechanism (`utils`, `core`, `helpers`)? Is infrastructure visually separated from the app? Does anything sit as a peer that's really a child? What's present that the app no longer needs?

Taste in service of legibility, not a second rulebook: the `_`-prefix and root-level domains are right _because_ the app is small — a larger one may want `features/` back. Minimize jumps, not folders; a folder earns its place by a real boundary or a second consumer, never speculation.
