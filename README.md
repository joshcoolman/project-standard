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

**Order of search when a view needs a component:**

1. Shop the shelf — build it from `src/components/` if you can.
2. Base UI for the gaps — if Base UI has the primitive, build from it.
3. shadcn as scout, never source — if Base UI lacks it, adopt the library shadcn builds on (e.g. `react-day-picker` for calendars), not shadcn's code.
4. Hand-build only for a true outlier (a gallery).

**Rules:**

- Promote broadly-usable components to `src/components/`; the next use reuses them.
- Build on need. No speculative primitives. Bar to promote: properly shareable, not used twice.
- Wrap Base UI, don't scatter it — one wrapper owns the prop API. Headless only; style with tokens; one visual language.
- Styling: co-locate a `*.module.css`; every value references a `tokens.css` variable, never a raw color or number. Dark mode comes free from the token remap. Skin third-party components the same way — pass module classes into their slot map.

---

## The `docs/` folder

Root-level `docs/` holds what the project is and why — things that matter but don't belong in the codebase. Root stays README-only; everything else goes under `docs/`. Docs we tend to use:

```
docs/
├── OVERVIEW.md                  what the project is and why
├── ARCHITECTURE.md              stack + how features are organized
├── LOG.md                       dated findings, newest first
└── reference/                   domain material, framework notes
```

Plus, at the repo root: `README.md` (front door; the `## Status` block = Last shipped / Up next) and `CLAUDE.md` (agent orientation — also dropped into folders an agent works in repeatedly, auto-loaded when touching that tree).

Group related docs into a folder once there are enough to warrant it — several notes on one framework become `reference/<framework>/`.

Plans, tasks, and bugs are **not** docs — they live as GitHub issues (see Git & issues). `docs/` never holds a plan file.

---

## Naming

- ALL-CAPS for big-idea docs: `README`, `CLAUDE`, `OVERVIEW`, `ARCHITECTURE`, `LOG`.
- lowercase for folders and reference docs.
- Filename fixed, heading free: `docs/LOG.md` is always `docs/LOG.md`; its `#` heading names it for the project.

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
