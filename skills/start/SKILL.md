---
name: start
description: Kick off a new project by asking clarifying questions until there is enough clarity to write a PRD and a phased roadmap. Use this first, before architecture, design, or feature breakdown.
---

# Start: Discover, then write the PRD and Roadmap

You are helping a user turn a rough idea into a written Product Requirements
Document (PRD) and a phased roadmap. Do not skip straight to writing — you
almost never have enough information from the first message alone.

## Process

1. **Ask, in small batches (2-4 questions at a time), not one giant form.**
   Cover, in roughly this order, skipping anything the user has already
   answered:
   - The problem or opportunity: what's broken or missing today, for whom?
   - Target users: who specifically, and what do they currently do instead?
   - Core value proposition: the one thing this must do well.
   - Must-have (MVP) vs. nice-to-have vs. explicitly out of scope.
   - Constraints: timeline, team size/skills, budget, existing systems to
     integrate with, any hard tech preferences or bans.
   - Success metrics: how will you know this worked?
   - Platform(s): web, mobile, CLI, API-only, etc.

2. **Keep asking until the load-bearing decisions are actually answered.**
   If the user says "you decide" or "whatever you think," make a reasonable
   default explicit in the PRD as an assumption rather than silently
   guessing — assumptions must be visible and revisable.

3. **Do not ask about implementation details** (frameworks, databases,
   hosting) here — that belongs to `/architect`. Keep this phase about the
   product, not the stack.

## Output

Once you have enough clarity, create the `docs/` folder for this project
(if it doesn't already exist) and write three files into it:

### `docs/PRD.md`

```
# <Product Name> — PRD

## Overview
One paragraph: what this is and why it exists.

## Problem
What's broken/missing today, for whom.

## Target Users / Personas
Who this is for, in concrete terms.

## Goals & Success Metrics
What "working" looks like, ideally measurable.

## Scope
### MVP (must-have)
### Later (nice-to-have)
### Out of scope (explicitly not doing)

## Constraints & Assumptions
Timeline, team, budget, integrations, anything assumed rather than confirmed.

## Open Questions
Anything still unresolved that should be revisited.
```

### `docs/ROADMAP.md`

Derive this from the PRD's scope section. Phases are outcome-level
milestones, not task lists (tasks come later from `/features`):

```
# Roadmap

## Phase 0 — Foundation
Outcome: project scaffolded, architecture decided, dev environment works.

## Phase 1 — MVP
Outcome: <the smallest version that delivers the core value prop>.

## Phase 2 — <next increment>
Outcome: ...

## Later / Backlog
Ideas explicitly deferred past the current roadmap.
```

Keep phases small enough to be checkpoints, not seasons.

### `docs/DECISIONS.md`

This starts the project's decision log. It is **not** a planning-phase
artifact — it's a running, append-only record of every real technical
decision made for the life of the project, most of which happen during
implementation, outside any of these five skills. `AGENTS.md` carries a
standing instruction telling any session to append here as decisions get
made; `/review` also sweeps for anything that should have been logged and
wasn't. Create the file now with a header and one entry per notable
scope/priority trade-off made during this conversation (e.g. "excluded X
from MVP because Y"):

```
# Decisions

## <YYYY-MM-DD> — <short title>
**Feature:** <docs/features/NN-slug.md, or "n/a" for cross-cutting decisions>
**Decision:** what was chosen.
**Why:** the reasoning.
**Alternatives considered:** what else was on the table, if anything.
```

Never rewrite past entries — only append. `Feature:` will mostly be
`n/a` for entries logged here at the planning stage, since
`docs/features/` doesn't exist yet.

## Update AGENTS.md

Open `AGENTS.md` at the project root. If it doesn't exist yet, create it
with a `## Project docs` / `## Commands` / `## Conventions` section
structure. Replace the content between
`<!-- agent-skills:docs-index:start -->` and
`<!-- agent-skills:docs-index:end -->` with links to `docs/PRD.md`,
`docs/ROADMAP.md`, and `docs/DECISIONS.md`. Leave the `Commands` and
`Conventions` sections untouched — those are filled in by `/architect`.
If `CLAUDE.md` exists at the root, leave it alone; it should just point to
`AGENTS.md`.

## Next step

Tell the user: "PRD, roadmap, and the decision log are written. Next, run
`/architect` to decide the technical approach, or `/design` to start on UI
research — either can go first."
