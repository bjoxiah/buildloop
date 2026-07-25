# buildloop

Portable [Agent Skills](https://agentskills.io) (`SKILL.md` files) covering
the full loop of building a product with an AI coding agent: scaffold, plan,
design, break into features, and review. Works with any tool that reads
`SKILL.md`-style skills (Claude Code, and others as support lands).

## Install

```bash
npx skills add bjoxiah/buildloop
```

This installs all 5 skills into your project (`.claude/skills/`,
`.agents/skills/`, etc., depending on your agent), using the open
[Agent Skills](https://agentskills.io) format. See
[vercel-labs/skills](https://github.com/vercel-labs/skills) for install
options — global install, picking a single skill, targeting a specific
agent, and so on.

## Skills

| Skill | Invoke | What it does |
|---|---|---|
| Start | `/start` | Asks clarifying questions, then creates `docs/` and writes `docs/PRD.md`, `docs/ROADMAP.md`, and `docs/DECISIONS.md`. |
| Architect | `/architect` | Detects greenfield vs. an existing codebase. Existing: documents the real stack and standards as-is. Greenfield: asks technical questions, writes `docs/ARCHITECTURE.md` (including a feature-code convention) and `docs/STANDARDS.md`, records build/dev/test commands in `AGENTS.md`, and can scaffold the project — installing the actual lint/format/test tooling `docs/STANDARDS.md` calls for, not just documenting it — with a git safety net and reconciliation of anything the scaffolder generates. |
| Features | `/features` | Breaks the roadmap into one file per feature under `docs/features/` (plus a `README.md` index) — each with a `code_path` linking it to where its code will live — optionally creating matching GitHub issues via `gh`. |
| Design | `/design` | Two modes. System (once): researches comparable products, writes `docs/DESIGN.md` and `design/tokens.json`. Per-feature (as each feature is about to be built): designs that feature's screens/states within the system, writing a companion `docs/features/NN-slug.design.md`. |
| Review | `/review` | Reviews implemented code (located via the feature's `code_path`) against its acceptance criteria, architecture, design, and coding-standards docs — including whether required tests actually exist, not just that the suite passes; runs available tests/verification; sweeps for undocumented decisions; updates the feature's status. |

Suggested order: `/start` → `/architect` → `/features` & `/design` Mode 1
(either order) → per feature: `/design` Mode 2 (if it has UI) → implement
→ `/review`. Each skill can also be run standalone.

## Artifacts produced

```
AGENTS.md              # entry point; sections filled in progressively, plus a
                        # standing "log decisions as you go" rule (see below)
CLAUDE.md               # stub pointing to AGENTS.md, only if .claude/ is present
docs/
  PRD.md                 # /start
  ROADMAP.md              # /start
  DECISIONS.md             # /start creates it; appended to continuously (see below)
  ARCHITECTURE.md         # /architect — includes the feature-code convention
  STANDARDS.md             # /architect — style, testing, error handling, security
  DESIGN.md                # /design
  features/
    README.md               # /features — index over the files below
    01-<slug>.md             # /features — one file per feature: code_path, design_doc, github_issue
    01-<slug>.design.md      # /design (Mode 2) — only for features with screens
    02-<slug>.md
design/
  tokens.json             # /design (Mode 1)
src/features/<slug>/     # example only — the real path is whatever /architect
                          # decided and recorded as the Feature Code Convention
```

### When AGENTS.md / CLAUDE.md come in

- **`AGENTS.md`** is created by `/start` the first time it's needed, with
  marked sections plus one fixed, always-present section: a standing
  instruction telling *any* session — not just these five skills — to
  append to `docs/DECISIONS.md` the moment a real technical decision gets
  made. This matters because most decisions happen during ordinary
  implementation conversations, not inside a named skill. Each skill also
  edits its own marked section as it produces new docs: `/start` fills in
  the docs index, `/architect` fills in commands and the feature-code
  convention, `/design` and `/features` add their docs to the index. It's
  never regenerated from scratch — always edited in place, so manual notes
  added between runs survive.
- **`CLAUDE.md`** is only created if `.claude/` exists, and only ever as a
  stub pointing at `AGENTS.md` — there's deliberately one source of truth,
  not two files to keep in sync. If a scaffolding tool run by `/architect`
  generates its own `CLAUDE.md` (or `.cursorrules`, etc.), that gets
  reconciled back into `AGENTS.md` rather than left to conflict with it.

### When docs/DECISIONS.md gets written

Created by `/start` with the first entries (scope/priority trade-offs from
the initial Q&A, `Feature: n/a`). After that it's a continuous log, not a
per-skill one:

- Any session appends to it in the moment, per the standing `AGENTS.md`
  rule — this is the primary path, since most decisions happen during
  implementation.
- `/architect`, `/design`, and `/features` also append when *they*
  specifically make a call between genuine alternatives.
- `/review` sweeps the diff for anything that should have been logged and
  wasn't, as a backstop, and logs any deviation the user explicitly
  accepts instead of fixing.

Every entry that relates to a specific feature includes a `Feature:` line
pointing at its `docs/features/NN-slug.md` file. Routine decisions with no
real alternative don't get an entry — this is a log of judgment calls, not
a changelog.

### Feature code convention

`/architect` decides and records one explicit path pattern for where a
feature's code lives (default: organize application code by feature —
vertical slices — e.g. `src/features/<slug>/`; falls back to whatever unit
fits the project type for libraries/CLIs/infra repos). Every
`docs/features/NN-<slug>.md` file carries a matching `code_path`, so
`/review` can go straight to a feature's code instead of guessing from a
diff, and so the doc and the implementation can always be found from each
other.

### Design: system once, screens per feature

The design system (palette, type, spacing, principles, base components)
is a global invariant, decided once by `/design` in Mode 1 — the same
relationship `/architect`'s stack decision has to the whole project. A
feature's actual screens/states/flow can't all be known before
`/features` breaks the roadmap down, and don't need designing until
that feature is about to be built — so `/design` Mode 2 runs per feature,
scoped to one `docs/features/NN-slug.md`, constrained by the Mode 1
system, writing a companion `NN-slug.design.md`. `/review` checks a
feature's implementation against both: the global tokens and, if present,
that feature's own design doc.

Neither mode assumes a specific research or design-file product is
installed. Both search available tools/MCP servers by capability —
pattern-research tool, design-file tool — and use whatever's actually
configured (a Mobbin MCP server and a Penpot or Figma MCP server are the
common examples, not requirements). With neither available, research
falls back to asking the user for references, and design-file creation is
skipped in favor of the markdown docs, said plainly rather than implied.

### Coding standards

Defined once, in `docs/STANDARDS.md`, by `/architect` — style/lint/format
tooling, what must be tested and how, error handling, security
non-negotiables, commit/PR conventions. For an existing codebase it's
reverse-engineered from what's actually configured rather than invented
from scratch. Two things make this more than a document nobody reads:

- `/architect`'s scaffold step installs and configures the actual
  tooling, so standards are enforced by the toolchain, not just written
  down.
- `/review` checks the diff against it explicitly, including verifying
  that required tests were actually added for the new logic (in the
  location the standards specify) — not just that some test suite
  somewhere still passes.
