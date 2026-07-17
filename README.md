# project-standard

How a project repo is organized inside — the layout, docs, and conventions every
project shares, so that once you know one, you know them all.

Third of a trio of canonical repos:

- **[dotfiles](https://github.com/joshcoolman/dotfiles)** — what makes a *machine* mine (shell + Claude config).
- **[bootstrap](https://github.com/joshcoolman/bootstrap)** — how a *project is born* (scaffolding recipes).
- **project-standard** (this repo) — how a *project is laid out inside*.

Extracted from what `eve-canvas-lab` and `bootstrap` already grew on their own,
not designed up front. It is a **set of slots, not a gate** — nothing here is
required, nothing errors if it's missing, and a repo with none of it is fine.
Filling a slot buys you something; leaving it empty costs nothing.

This is the whole standard. There is no tooling, no plugin, no linter, no
scaffolder — a person or an agent reads this file and applies it by hand.
Machinery gets added only when repetition proves it's needed, and only after it's
been run for real (bootstrap's admission bar, applied to this repo too).

---

## The slots

```
README.md                        front door. ## Status at the bottom.
CLAUDE.md                        agent orientation for the repo
docs/
├── PLAN.md                      the why, the bet, the direction
├── LOG.md                       dated findings, newest first          [optional]
└── plans/
    └── PLAN-<issue>-<slug>.md   per-unit execution spec               [optional]
<core-folder>/CLAUDE.md          orient an agent working in here       [optional]
```

Root stays README-only; everything else lives under `docs/`.

### README.md

The front door, and a `## Status` block at the bottom: **Last shipped** (a few
bullets) + **Up next** (a pointer to issues). A snapshot, not a log — overwrite
it freely. This plus open issues is the whole session-orientation surface; there
is no continuation file.

Keep it current at natural beats — a unit of work done, a direction change,
before a long-running operation. Don't ask, don't announce.

### CLAUDE.md (root)

Agent orientation: what this repo is, how to work in it, what's weird about it.
Not a duplicate of the README — the README is for humans arriving, this is for an
agent about to touch code. Also the place to declare deviations from this
standard (see below).

### docs/PLAN.md

The durable why. What we're building, the bet we're making, what it explicitly is
**not**. Includes direction/roadmap. Rarely rewritten; when the vision moves, a
"Reorientation" section on top beats a rewrite — the history is worth keeping.

### docs/LOG.md

Dated findings, newest first, `YYYY-MM-DD` headings. What this log is *for* is
project-specific and belongs in its own `#` heading — a tire-kick's log builds
toward a verdict; a product's log records decisions. The slot is standard; the
contents are free.

Not a devlog. It has an inclusion criterion, and the criterion is the point:
entries earn their place by bearing on something the project needs to decide. Say
what that criterion is at the top.

### docs/plans/PLAN-&lt;issue&gt;-&lt;slug&gt;.md

One per unit of work, named for the issue it executes (`PLAN-20-persistence.md`).
The handoff membrane: vetted before execution, then execution follows it and
nothing else. Stays afterward as the record — including what went wrong, which is
usually the part worth keeping.

The folder appears when the second plan does. One plan sits flat in `docs/`.

### &lt;core-folder&gt;/CLAUDE.md

A short orientation file in folders an agent works in repeatedly — `components/`,
`api/`, `canvas/`. Claude Code auto-loads these when touching files in that tree,
so it's a real mechanism, not just a convention. Keep them short: what this layer
owns, what it must not do, the one thing that isn't obvious from the code.

---

## Naming

- `PLAN` prefix on plan docs — the name says the purpose, at a glance, in a list.
- Shouty root docs (`README`, `CLAUDE`, `PLAN`, `LOG`); lowercase folders.
- The **slot name is fixed; the heading inside is free.** `docs/LOG.md` is always
  `docs/LOG.md`; its `#` heading says what it is in *this* project. That's what
  stops every repo inventing a clever filename for the same job.

## The growth rule

**A folder appears when its second file does.** Until then the file sits flat in
`docs/`. A fresh repo is `README.md` + `CLAUDE.md` + `docs/PLAN.md` and nothing
else. This keeps small repos small and makes "does this need a folder" a rule
instead of a judgment call re-litigated per project.

## Divergence: purge or promote

Same three buckets as the machine-reconciliation loop in
[dotfiles/WORKFLOW-OVERVIEW.md](https://github.com/joshcoolman/dotfiles/blob/main/WORKFLOW-OVERVIEW.md),
applied to repos:

1. **Canonical** — in this doc. The repo adopts it.
2. **Cut** — retired here. The repo sheds it.
3. **Novel** — in the repo, not in this doc, not cut → **stop and ask: purge it,
   or promote it into this doc?**

Bucket 3 is the point. A repo that grew something the canon doesn't have is either
drift to purge or a finding to harvest — and the only way to tell is to look.
`eve-canvas-lab` grew `docs/plans/` and a coherence log because the four-file
pattern didn't fit an agent lab; those were promotions, not drift.

When the same novel thing shows up in a third repo, it graduates into the slots
above. **The canon only stays canonical if the harvest actually happens.**

Deviations worth keeping get declared in the repo's root `CLAUDE.md`, so a later
pass doesn't "fix" a deliberate choice:

```markdown
## Standard

Follows project-standard. Deviations:
- `docs/LOG.md` is a coherence log — this repo is a tire-kick.
- No `docs/plans/` yet — nothing has needed a spec.
```

## What's deliberately not here

- **No frontmatter, no `type:` field, no schema.** A handful of hand-written files
  don't need metadata. Revisit if docs are ever generated at volume.
- **No linter, no init skill, no migrator.** Scaffolding a few files by hand is a
  20-second job; a skill written before it's been run for real is a prompt, not a
  skill. Codify only when repetition proves it hurts.
- **No tooling coupled to these slots.** The fleet-status tools work today with
  zero repos compliant, and must keep working. If a tool ever reads a slot, it
  shows what it finds and stays silent about what it doesn't.
- **No conformance.** There is no such thing as a non-conformant repo.

## Prior art worth naming

- **[OKF](https://github.com/GoogleCloudPlatform/knowledge-catalog/tree/main/okf)** —
  reserved filenames, and permissive consumption (never reject on missing files).
  Both taken. Its `type:` frontmatter and concept model are built for
  agent-generated catalogs at volume; not this.
- **EVE / Rails** — the filesystem *is* the API. Convention is what makes tooling
  possible without configuration; that's the payoff, if it's ever collected.
- **bootstrap** — the mule discipline: build it for real, log the friction, then
  codify. That's how this doc gets to change.

---

## Status

**Larval.** Extracted from two repos (`eve-canvas-lab`, `bootstrap`), proven on
neither as a deliberate standard yet. Expect churn.

**Up next:** first real mule run — retrofit `eve-canvas-lab` against these slots
by hand (rename its coherence log to `docs/LOG.md`, add root + folder
`CLAUDE.md`s, declare its deviations), and log whatever fights. Whatever survives
that contact is what earns a v0.1 tag. Then retrofit `bootstrap` as a second,
maximally-different consumer (a plugin repo with no app) — and resolve the one
known collision: `PLAN.md` means "build order" in bootstrap and "vision + roadmap"
in eve-canvas-lab. One contract, or the standard ships with a landmine in its
most-used file.
