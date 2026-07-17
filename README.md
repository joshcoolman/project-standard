# project-standard

How a project repo is organized inside — the layout, docs, and conventions every
project shares, so that once you know one, you know them all.

Third of a trio of canonical repos:

- **[dotfiles](https://github.com/joshcoolman/dotfiles)** — what makes a *machine* mine (shell + Claude config).
- **[bootstrap](https://github.com/joshcoolman/bootstrap)** — how a *project is born* (scaffolding recipes).
- **project-standard** (this repo) — how a *project is laid out inside*.

Extracted from what `eve-canvas-lab` and `genzen` already grew on their own, not
designed up front. It is a **set of slots, not a gate** — nothing here is
required, nothing errors if it's missing, and a repo with none of it is fine.
Filling a slot buys you something; leaving it empty costs nothing.

This is the whole standard: a person or an agent reads this file and applies it by
hand. No plugin, no linter, no scaffolder. Machinery gets added only when
repetition proves it's needed, and only after it's been run for real.

---

## Three surfaces, three homes

"The standard" is really three kinds of thing, and knowing which is which is how
you know where to look:

| Surface | What it covers | Home |
|---|---|---|
| **Layout** | files, folders, naming | **project-standard** (this repo) |
| **Behavior** | git workflow, session orientation, commit ritual | **global `CLAUDE.md`** (dotfiles) |
| **Architecture** | stack, how features are built | **bootstrap** |

Plus a fourth that is *not* canon: **project knowledge** — repo-specific taste or
domain (e.g. eve-canvas-lab's Eve docs, genzen's character/prompt references).
Knowledge lives in **`docs/reference/`** (a domain-named subfolder is fine), kept
*off* the top level so the slots stay legible, and never tries to become canon.
This is why leaning hard on one framework is correct *and* not part of the
standard.

This repo owns **layout**. It points at the other two homes so the whole standard
is discoverable from one place — the git workflow doesn't move, project-standard
just names where it lives.

---

## The docs

Each doc answers one question, and they line up on a time axis:

| Doc | Question it answers | When |
|---|---|---|
| **`docs/VISION.md`** | why are we doing this? | timeless |
| **`docs/ARCHITECTURE.md`** | how is it built / shaped? | emergent · optional |
| **`README.md`** + `## Status` | where are we *now*? | present |
| **`docs/plans/PLAN-<issue>-*.md`** | what's the path forward? | next · optional |
| **`docs/LOG.md`** | what did we learn? | past · optional |
| **`CLAUDE.md`** | how do I (agent) work in here? | orientation |

```
README.md                        front door. ## Status at the bottom.
CLAUDE.md                        agent orientation for the repo
docs/
├── VISION.md                    why / bet / direction
├── ARCHITECTURE.md              stack + how features are built   [optional]
├── LOG.md                       dated findings, newest first     [optional]
├── plans/
│   └── PLAN-<issue>-<slug>.md   per-unit build order             [optional]
└── reference/                   project knowledge, off the top   [optional]
<core-folder>/CLAUDE.md          orient an agent working in here  [optional]
```

Root stays README-only; everything else lives under `docs/`.

### README.md — where we are now

The front door, and a `## Status` block at the bottom: **Last shipped** (a few
bullets) + **Up next** (a pointer to issues). A snapshot, not a log — overwrite it
freely. This plus open issues is the whole session-orientation surface; there is
no continuation file. Keep it current at natural beats; don't ask, don't announce.

### CLAUDE.md — how to work here

Agent orientation: what this repo is, how to work in it, what's weird about it.
Not a duplicate of the README — the README is for humans arriving, this is for an
agent about to touch code. Also where deviations from this standard are declared
(see Baseline + divergence).

### docs/VISION.md — why

The why, the bet, what it explicitly is **not**. Durable; rarely rewritten. When
the vision moves, a "Reorientation" section on top beats a rewrite — the history
is worth keeping.

### docs/ARCHITECTURE.md — how it's shaped (optional, emergent)

The stack and how features are organized. **Descriptive, not prescriptive** — the
standard does not hand architecture down; the collaboration discovers it per repo,
and this file records what got decided. Two ways it gets born:

- **Scaffolded** (from a bootstrap skill) — the skill *made* the stack/architecture
  decisions, so it emits the baseline. The repo records only its divergences.
- **Greenfield** (hand-built) — starts blank ("undecided") and accretes as
  decisions crystallize. An early repo legitimately has no architecture doc yet.

### docs/plans/PLAN-&lt;issue&gt;-&lt;slug&gt;.md — the path forward (optional)

One per unit of work, named for the issue it executes (`PLAN-20-persistence.md`).
The handoff membrane: vetted before execution, then execution follows it and
nothing else. Stays afterward as the record — including what went wrong, which is
usually the part worth keeping. The folder appears when the second plan does.

`PLAN` is only ever this — a plan *for a unit of work*, inside `plans/`. It is not
a root doc. The overall roadmap is README `## Status` + open issues, not a
monolithic plan file.

### docs/LOG.md — what we learned (optional)

Dated findings, newest first, `YYYY-MM-DD` headings. This is the self-improvement
surface — the record you learn from. What the log is *for* is project-specific and
belongs in its own `#` heading (a tire-kick builds toward a verdict; a product
records decisions). The slot is standard; the contents are free.

Not a devlog. It has an inclusion criterion, and the criterion is the point:
entries earn their place by bearing on something the project needs to decide. Say
what that criterion is at the top.

### docs/reference/ — project knowledge (optional)

Where **project knowledge** lives — the repo-specific taste, domain material, and
framework notes that aren't standard slots (eve-canvas-lab's Eve coverage map,
genzen's character/prompt references). It sits in its own folder so it never
intermingles with the slots at the `docs/` top level, which would make the slots
harder to pick out. A domain-named subfolder (`reference/eve/`) is fine once it
grows. This is the fourth surface — knowledge, not canon — given a home.

### &lt;core-folder&gt;/CLAUDE.md — local orientation (optional)

A short file in folders an agent works in repeatedly — `components/`, `api/`,
`canvas/`, `features/*`. Claude Code auto-loads these when touching files in that
tree, so it's a real mechanism, not just a convention. Keep them short: what this
layer owns, what it must not do, the one thing that isn't obvious from the code.
(genzen runs this at scale — a `CLAUDE.md` per feature — and it's exemplary.)

---

## Naming

- **ALL-CAPS for the big-idea docs**: `README`, `CLAUDE`, `VISION`, `ARCHITECTURE`,
  `LOG`. The shout signals "load-bearing, read me."
- **`PLAN` prefix** on per-unit plans in `docs/plans/`. The name states the purpose
  in a file list. (PLAN / plans echoes Plan-Mode / plans, on purpose.)
- **lowercase for folders** and supporting/reference docs.
- **Slot name fixed, heading free.** `docs/LOG.md` is always `docs/LOG.md`; its `#`
  heading says what it is in *this* project. That's what stops every repo inventing
  a clever filename for the same job.

## The growth rule

**A folder appears when its second file does.** Until then the file sits flat in
`docs/`. A fresh repo is `README.md` + `CLAUDE.md` + `docs/VISION.md` and nothing
else. Keeps small repos small and makes "does this need a folder" a rule instead of
a per-project judgment call.

## Baseline + divergence

The recurring grammar of the whole system: **inherit a baseline, then record only
where you left it.** Same move in three places —

- `CLAUDE.md`: "follows project-standard, deviations: …"
- `ARCHITECTURE.md` (scaffolded): "bootstrap next-app, deviations: …"
- machine reconciliation (dotfiles): canonical / cut / **novel**

Declared deviations go in the repo's root `CLAUDE.md`, so a later pass doesn't
"fix" a deliberate choice:

```markdown
## Standard

Follows project-standard. Deviations:
- `docs/LOG.md` is a coherence log — this repo is a tire-kick.
- No `docs/ARCHITECTURE.md` — the shape is still in `docs/VISION.md`.
```

## Divergence: purge or promote

For undeclared divergence, the same three buckets as the dotfiles machine ritual,
applied to repos:

1. **Canonical** — in this doc. The repo adopts it.
2. **Cut** — retired here. The repo sheds it.
3. **Novel** — in the repo, not in this doc, not cut → **stop and ask: purge it, or
   promote it into this doc?**

Bucket 3 is the point. A repo that grew something the canon lacks is either drift
to purge or a finding to harvest — the only way to tell is to look. eve-canvas-lab
grew `docs/plans/` and a coherence log because the flat pattern didn't fit an agent
lab; those were promotions. When the same novel thing shows up in a third repo, it
graduates into the slots above. **The canon only stays canonical if the harvest
actually happens.**

## What's deliberately not here

- **No frontmatter, no schema.** A handful of hand-written files don't need
  metadata. Revisit if docs are ever generated at volume.
- **No linter, no init skill, no migrator.** Scaffolding a few files by hand is a
  20-second job; a skill written before it's been run for real is a prompt, not a
  skill. Codify only when repetition proves it hurts.
- **No tooling coupled to these slots.** The fleet-status tools work today with
  zero repos compliant, and must keep working. If a tool ever reads a slot, it
  shows what it finds and stays silent about what it doesn't.
- **No conformance.** There is no such thing as a non-conformant repo.

## Prior art worth naming

- **[OKF](https://github.com/GoogleCloudPlatform/knowledge-catalog/tree/main/okf)**
  — reserved filenames, and permissive consumption (never reject on missing files).
  Both taken. Its `type:` frontmatter and concept model are built for
  agent-generated catalogs at volume; not this.
- **EVE / Rails** — the filesystem *is* the API. Convention is what makes tooling
  possible without configuration; that's the payoff, if it's ever collected.
- **bootstrap** — the mule discipline: build it for real, log the friction, then
  codify. That's how this doc gets to change.

---

## Status

**Larval, first fold.** The slot set, the six-doc time-axis model, naming, the
growth rule, and the baseline+divergence grammar are in. Extracted from
`eve-canvas-lab` and `genzen`; proven as a deliberate standard on neither yet.

**Up next:** first real mule run — retrofit `eve-canvas-lab` against these slots by
hand (`FRICTION.md` → `docs/LOG.md`, `PLAN.md` → `docs/VISION.md`, add root +
per-folder `CLAUDE.md`, declare deviations) and log whatever fights. Whatever
survives that contact earns a **v0.1** tag. Then retrofit the `bootstrap`
scaffolders so a fresh `vite-app`/`next-app` is born project-standard-shaped.
