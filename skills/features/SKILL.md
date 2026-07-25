---
name: features
description: Break the roadmap into concrete user stories and features, one file per feature under docs/features/, and optionally create matching GitHub issues. Use after /start (and ideally /architect), before implementation begins.
---

# Features: Roadmap to backlog

## Preconditions

Read `docs/ROADMAP.md` (required) and `docs/ARCHITECTURE.md` (if it exists,
for technical grounding). If the roadmap doesn't exist yet, tell the user
to run `/start` first.

## Process

1. For each phase in the roadmap, break the outcome-level goal into
   concrete features. Keep each one small enough to be a single reviewable
   unit of work — split anything that bundles multiple unrelated outcomes.

2. Order features within each phase by priority/dependency, not just the
   order they occurred to you.

3. Log to `docs/DECISIONS.md` (format in `/start`) only if a real
   trade-off was made while breaking the backlog down — e.g. deliberately
   deferring a feature the PRD implied was in-scope. Routine breakdown
   doesn't need an entry.

## Output: one file per feature under `docs/features/`

Create `docs/features/` if it doesn't exist. Each feature gets its own
file, numbered for stable ordering: `docs/features/<NN>-<slug>.md`, where
`NN` is a global sequence across all phases (`01`, `02`, ...) and `<slug>`
is a short kebab-case name.

```
---
phase: <Phase name from the roadmap>
status: todo
github_issue: null
code_path: <path from docs/ARCHITECTURE.md's Feature Code Convention, e.g. src/features/<slug>/, or null>
design_doc: null
---

### <Feature name>
As a <user type>, I want <capability>, so that <benefit>.

Acceptance criteria:
- [ ] <specific, testable condition>
- [ ] <specific, testable condition>
```

`status` moves through `todo` -> `in-progress` -> `done` as work
progresses (this is what `/review` checks and updates). `github_issue`
gets filled in below if an issue is created for this feature — leave it
`null` otherwise.

`code_path` is where this feature's code will live. If `docs/ARCHITECTURE.md`
already exists, read its "Feature Code Convention" section and derive the
path from the `<slug>` used in this file's own filename — the two must
match. If `/architect` hasn't run yet, leave `code_path` as `null`;
`/architect` backfills it later once the convention is decided, and
`/review` will also resolve it from the real repo layout if it's still
null by then.

`design_doc` stays `null` here — it's filled in by `/design` when it's run
for this specific feature (Mode 2), pointing at the companion
`docs/features/NN-slug.design.md` it creates. Not every feature needs one
(e.g. a pure backend/API feature has no screens).

### `docs/features/README.md`

A generated index/rollup so the backlog is still readable as a whole —
regenerate this whenever features are added, statuses change, or issues
are linked:

```
# Features

## Phase 0 — Foundation
- [ ] [01-project-scaffold](01-project-scaffold.md) — todo
- [x] [02-ci-pipeline](02-ci-pipeline.md) — done (#12)

## Phase 1 — MVP
- [ ] [03-user-signup](03-user-signup.md) — todo
```

The individual files under `docs/features/` are the backlog of record;
this README is just a table of contents over them.

## Optional: GitHub issues as backlog

If the user wants issues created (ask first — this is visible to
collaborators and not trivially reversible in bulk):

1. Check availability before attempting anything:
   - `gh --version` — is the CLI installed?
   - `gh auth status` — is it authenticated?
   - `gh repo view` — is the current directory inside a repo with a
     GitHub remote?
2. If any check fails, tell the user what's missing and fall back to the
   `docs/features/*.md` files as the backlog — this is a fully valid
   standalone backlog with or without GitHub.
3. If all checks pass, confirm scope with the user (all phases, or just
   the next one?), then create one issue per feature via `gh issue create`,
   using the feature file's title and acceptance criteria as the body.
   Apply labels for phase and priority if the repo has them, or offer to
   create those labels first via `gh label create`.
4. After each issue is created, write the issue number back into that
   feature file's `github_issue` frontmatter field and update the
   `docs/features/README.md` line for it (as in the example above) — the
   file and the issue should always cross-reference each other.
5. If a GitHub Project board is wanted, use `gh project` to add created
   issues to it — confirm the project exists or should be created first.

## Update AGENTS.md

Add `docs/features/README.md` to the `## Project docs` section of the
root `AGENTS.md` (between the `docs-index` markers), pointing readers
there for the current backlog.

## Next step

Tell the user: "Backlog is written under `docs/features/`. For a UI-facing
feature, run `/design` naming it before building, to work out its screens
against the design system. Once implemented, run `/review` to check it
against that file's acceptance criteria."
