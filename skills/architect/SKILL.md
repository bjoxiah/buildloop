---
name: architect
description: Turn the PRD into a technical architecture and coding standards by asking technical questions, then write architecture.md and standards.md, and optionally scaffold the initial project structure. Use after /start, before or alongside /features and /design.
---

# Architect: Decide the stack, document it, optionally scaffold

## Preconditions

Look for `docs/PRD.md`. If it doesn't exist, tell the user this works best
after `/start`, but you can proceed with a short version: ask what the
product is and who it's for before moving on to technical questions.

## 0. Determine greenfield vs. existing codebase

Check the actual repo, don't just ask: is there already substantial
application code (a `src/`/`app/` with real logic, a populated
`package.json`, etc.), as opposed to only the artifacts this toolkit
creates (`AGENTS.md`, `CLAUDE.md`, `docs/`, `.agent/`, `.claude/`)?

- **Existing codebase:** this is a documentation pass, not a stack
  decision. Read the actual code, infer the stack/structure/conventions
  already in use, and write `docs/ARCHITECTURE.md` describing what's
  really there. Also reverse-engineer `docs/STANDARDS.md` from what's
  actually configured — existing lint/format config, the test framework
  in use, how errors are handled — rather than inventing new rules; ask
  the user only to fill gaps nothing in the repo answers. Do not offer to
  scaffold — skip straight to the "Update AGENTS.md" section below once
  both docs are written. If something about the existing setup looks like
  a real problem, note it, but changing it is out of scope for this skill.
- **Greenfield:** proceed with the process below, including the scaffold
  offer.

## Process

1. **Read `docs/PRD.md`** for scope, constraints, and any tech preferences
   already stated.

2. **Ask technical questions**, batched, only for what isn't already known:
   - Language/framework preference, or should you recommend one based on
     the PRD?
   - Monorepo vs. single app; web/mobile/API split if relevant.
   - Data needs: what kind of data, roughly how much, relational vs.
     document vs. none for MVP.
   - Auth needs: none, simple, third-party (OAuth), enterprise SSO.
   - Hosting/deployment target, if the user has one in mind.
   - Any required integrations (payments, email, analytics, etc.).
   - Non-functional constraints worth calling out: expected scale,
     offline support, compliance/regulatory needs.
   - Coding standards & testing: preferred lint/format tooling (or a
     sensible default for the stack), what must be tested and how
     (unit/integration/e2e, any coverage expectation), error-handling
     conventions, security non-negotiables, commit/PR conventions if the
     user wants one enforced.

3. **Decide, don't just transcribe.** Where the user has no preference,
   pick a sensible default for the project's scale and say why in the doc.

4. **Log real trade-offs to `docs/DECISIONS.md`.** Any choice made between
   genuine alternatives (stack, database, hosting, auth approach) gets an
   entry there, with `Feature: n/a` — see the format in `/start`. Routine
   details that had no real alternative don't need an entry.

## Output: `docs/ARCHITECTURE.md`

```
# Architecture

## Stack
Languages, frameworks, key libraries, and why each was chosen.

## System Overview
A short diagram (mermaid is fine) or description of the major pieces and
how they talk to each other.

## Data Model
Key entities and relationships, at the level of a sketch, not a full schema.

## Folder / Module Structure
The intended top-level layout and what lives where.

## Feature Code Convention
Where a single feature's code lives, as one explicit pattern, e.g.
`src/features/<slug>/`. The `<slug>` must match the slug used in
`docs/features/NN-<slug>.md` (from `/features`) so a feature's doc and its
code can always be found from each other. Default to organizing
user-facing application code by feature (vertical slices: a feature's
components/logic/tests live together) rather than by technical layer —
ask only if the project type doesn't fit that (e.g. a library, a CLI with
few commands, an infra-only repo), and pick whatever unit does the same
job for that project type.

## Integrations & Services
Third-party services, APIs, and why.

## Non-functional Notes
Performance, security, scaling, compliance considerations that shaped
decisions above — only include what's actually relevant.
```

If `docs/features/` already has files (from a prior `/features` run),
backfill each one's `code_path` frontmatter now using the convention just
decided, and update `docs/features/README.md` if it shows paths.

## Output: `docs/STANDARDS.md`

This is what `/review` checks implementation against, so be concrete
enough to actually verify — "write good code" isn't checkable, "each
feature has unit tests for its core logic, co-located under its
`code_path`" is.

```
# Coding Standards

## Style & Formatting
Linter/formatter and config for this stack (e.g. ESLint+Prettier, ruff,
rustfmt), and the command to run them.

## Testing
What must be tested and how (unit/integration/e2e), any coverage
expectation, and where tests live — default to co-located with the
feature under its `code_path` unless the stack's convention differs.

## Error Handling
How errors are handled, reported, and logged.

## Security
Non-negotiables relevant to this project — input validation at
boundaries, secrets handling, auth checks — only what's actually relevant.

## Commit / PR Conventions
Commit message format, branch naming, PR checklist — only if the user
wants one enforced.
```

## Optional: scaffold the project (greenfield only)

After the architecture and standards docs are written, offer to scaffold
the initial project structure to match them — e.g. running the framework's
official init command, creating the base folder layout (including the
feature-code convention above), initial config files, and package manager
setup. This includes actually installing and configuring the lint/format/
test tooling decided in `docs/STANDARDS.md` — standards that only exist as
prose don't get followed. If the user wants CI, a minimal workflow that
runs those same checks on push/PR is the most reliable enforcement; offer
it, don't assume it.

- **Confirm with the user before running install commands or generating
  many files** — this touches the working tree broadly and can be
  disruptive if the user wanted to review the plan first.
- Prefer official scaffolding tools (`create-*`, framework CLIs) over
  hand-rolled boilerplate when one exists for the chosen stack.
- **Safety net first:** many scaffolders refuse to run in a non-empty
  directory, and some overwrite `README.md`/`.gitignore`/`package.json`
  or generate their own `AGENTS.md`/`CLAUDE.md`/`.cursorrules`. Before
  running anything, make sure the repo is a git repo with everything
  committed (init + commit if it isn't) so nothing this toolkit has
  written so far — `AGENTS.md`, `CLAUDE.md`, `docs/` — can be
  unrecoverably lost. Don't rely on manual file copies for this; git is
  the safety net.
- Run the scaffold tool.
- **Reconcile afterward, don't let either side silently win:** if the
  scaffolder generated its own `AGENTS.md`/`CLAUDE.md`/`.cursorrules`/
  similar, fold anything genuinely useful from it (package-manager
  quirks, generated scripts) into our `AGENTS.md`'s `Commands` or
  `Conventions` sections, then remove the scaffolder's duplicate file so
  there is exactly one instructions file. If it touched `docs/` or
  `.agent/`/`.claude/`, restore ours from git — those are never the
  scaffolder's to own.
- After scaffolding, verify it actually builds/runs before declaring done.

## Update AGENTS.md

- Replace the content between `<!-- agent-skills:commands:start -->` and
  `<!-- agent-skills:commands:end -->` with the actual install/dev/build/
  test commands for the chosen (or discovered) stack.
- Replace the content between `<!-- agent-skills:conventions:start -->`
  and `<!-- agent-skills:conventions:end -->` with the feature code
  convention decided above (e.g. "Feature code lives at
  `src/features/<slug>/`, matching `docs/features/NN-<slug>.md`.") and a
  pointer to `docs/STANDARDS.md` for style/testing/error-handling rules.
- Add `docs/ARCHITECTURE.md` and `docs/STANDARDS.md` (and `docs/DECISIONS.md`
  if `/start` hasn't already listed it) to the `## Project docs` section.

## Next step

Tell the user: "Architecture is documented. Next, run `/features` to break
the roadmap into concrete work, and/or `/design` for UI research and design
tokens."
