---
name: review
description: Review implemented code for a feature (or the current diff/branch) against the acceptance criteria, architecture, design, and coding standards docs, running available tests and verification. Use after implementing work defined by /features.
---

# Review: Verify implementation against stated goals

## Scope

Default to reviewing the current diff/branch. If the user names a specific
feature, find its file under `docs/features/` (by slug, number, or GitHub
issue). Use its `code_path` frontmatter to go straight to the relevant
code instead of guessing from the diff. If `code_path` is `null`, resolve
it from the real repo layout now (matching the convention in
`docs/ARCHITECTURE.md` if one exists) and write it back into the feature
file. If the diff touches code but you can't tell which feature file it
maps to, check `docs/features/README.md` for the likely match before
asking the user.

## Process

1. **Acceptance criteria.** For each matching feature file's acceptance
   criteria, check the implementation and mark it pass / fail / uncertain
   — don't skip uncertain ones silently, call them out for the user to
   confirm.

2. **Architecture consistency.** If `docs/ARCHITECTURE.md` exists, check
   that the implementation matches its stated stack, structure, and
   conventions. Flag meaningful deviations (wrong layer for logic, new
   dependency that contradicts a documented choice) — not nitpicks.

3. **Design consistency.** If this touches UI, check two things, not just
   one: `docs/DESIGN.md` / `design/tokens.json` (colors, spacing,
   typography, component patterns match the system rather than one-off
   values), and — if the feature file's `design_doc` is set — the
   feature-specific screens/states/flow in that companion file
   (`docs/features/NN-slug.design.md`) from `/design` Mode 2. The system
   tokens being followed doesn't mean the actual screens match what was
   designed for this feature; check both.

4. **Coding standards consistency.** If `docs/STANDARDS.md` exists, check
   the diff against it specifically — don't just eyeball general code
   quality:
   - **Tests exist, not just pass.** Confirm tests were actually added for
     the new logic, in the location `docs/STANDARDS.md` specifies
     (default: co-located under the feature's `code_path`). A green test
     suite that never ran new code proves nothing — check the new tests
     actually exercise the new logic.
   - **Style/lint/format** clean per the tooling `/architect` configured —
     run it, don't assume.
   - **Error handling, security, and commit conventions** match what's
     documented, where relevant to this diff.

5. **Run available verification** rather than eyeballing only:
   - Lint/typecheck/test/build scripts if the project defines them (check
     `AGENTS.md` for the commands `/architect` recorded).
   - For UI changes, start the dev server and exercise the actual feature
     (not just the happy path) — take screenshots of affected screens if
     tooling allows, and compare against the design intent.
   - Prefer this project's own `/verify` or `/run` skill if present for
     driving the app, rather than reinventing that here.

6. **Decision sweep.** Scan the diff for real technical decisions (library
   choice, data structure, notable trade-off) and check whether each one
   already has an entry in `docs/DECISIONS.md`. `AGENTS.md` tells every
   session to log these as they happen, but this is the backstop for
   anything that slipped through — log missed ones now, with a `Feature:`
   line pointing at this feature's file.

7. **Report, don't silently fix.** Produce a summary:
   - Acceptance criteria: pass/fail/uncertain per item.
   - Architecture/design/standards deviations found, if any.
   - Decisions found undocumented and now logged, if any.
   - A punch list of what's left or broken.

   Then offer to fix what's found — only make changes after the user
   confirms, since "review" implies reporting first.

## If it passes

Update the feature file's `status` frontmatter to `done`, and reflect that
in `docs/features/README.md`. If the user explicitly accepts a deviation
from the architecture, design, or standards docs rather than fixing it,
log that call to `docs/DECISIONS.md` (format in `/start`, `Feature:`
pointing at this file) so it isn't silently lost.

## Next step

If everything passes: tell the user the feature is verified and ready,
and point to `docs/features/README.md` for what's next in the backlog. If
issues were found: list them clearly and wait for direction before
changing code.
