---
name: design
description: Define the project's UI design system (principles, tokens, base components) and, per feature, the screens/flows for that feature specifically. Two modes — run without a feature named for the system, or naming a feature for its screens. Use after /start; the system pass works alongside /architect and /features, the per-feature pass happens as each feature is about to be built.
---

# Design: System once, screens per feature

Design work splits into two things that don't belong on the same
schedule: the design *system* (palette, type, spacing, principles) is a
global invariant decided once, the same way `/architect` decides the
stack once. The screens/flows for an individual feature are feature-shaped
and can't all be known upfront — they get worked out per feature, as it's
about to be built, the same way `/features` breaks the roadmap down one
feature at a time. This skill has two modes for that reason.

## Which mode

- **Mode 1 — System.** No feature named, or `docs/DESIGN.md` doesn't
  exist yet. Run this once, early (after `/start`, alongside `/architect`).
- **Mode 2 — Feature screens.** A specific feature is named or implied
  ("design the login screen" while `docs/features/03-login.md` exists).
  Run this per feature, as it's about to be built — not all upfront.

If Mode 2 is requested but `docs/DESIGN.md` doesn't exist yet, run Mode 1
first so there's a system to stay consistent with.

## Tool discovery (applies to both modes)

Don't hardcode assumptions about what design tooling is installed — look
for it. Two distinct capabilities matter here, and either may or may not
be available depending on what's configured in this environment:

- **Pattern/competitor research.** A tool for browsing real product UI
  patterns — a Mobbin MCP server is a common example, but treat it as one
  example of the category, not a requirement. Search your available
  tools/MCP servers for something matching "pattern," "design research,"
  "UI reference," etc. before assuming none exists.
- **Design file creation/editing.** A tool for creating or editing an
  actual design file — Penpot and Figma MCP servers are common examples.
  Search for something matching "penpot," "figma," "design," etc.

Use whichever is actually configured. If neither is available, degrade
gracefully: for research, ask the user for reference links/screenshots/
named apps instead; for file creation, document the intended structure in
markdown and say plainly that no design file was created, rather than
implying one was. Never make the skill's usefulness depend on the user
having one specific product installed.

---

## Mode 1 — System

### Preconditions

Read `docs/PRD.md` for target users and product category. If it doesn't
exist, ask directly: what is this product, and who is it for?

### Process

1. **Research.** Using whatever research tool is available (see above),
   look at comparable products and UI patterns relevant to this product
   category and its target users. Note what works and what to
   deliberately avoid, not just a list of inspirations.

2. **Define the direction**, informed by the research and the PRD's
   target users:
   - Design principles: 3-5 words/phrases describing the intended feel
     (e.g. "minimal, dense, fast" vs. "warm, playful, spacious") and why
     that fits the users.
   - Color palette (primary, neutral scale, semantic colors for
     success/warning/error).
   - Typography scale (font choices, size/weight steps).
   - Spacing scale, corner radius, elevation/shadow conventions.
   - Base component inventory: the recurring UI pieces this product will
     need (buttons, inputs, cards, etc.) — not full screens, those are
     Mode 2's job.

3. Log to `docs/DECISIONS.md` (format in `/start`, `Feature: n/a`) if the
   direction involved a real trade-off — e.g. rejecting a heavier
   component library in favor of a lighter one. Skip it for routine token
   choices.

### Output

**`docs/DESIGN.md`** — principles, palette, typography, spacing, base
component inventory, and the research references that informed the
direction. Human-readable; this is the system every feature's screens
must stay consistent with.

**`design/tokens.json`** — machine-usable tokens:

```json
{
  "color": { "primary": "#...", "neutral": { "50": "#...", "900": "#..." }, "success": "#...", "warning": "#...", "error": "#..." },
  "typography": { "fontFamily": "...", "scale": { "sm": "...", "base": "...", "lg": "...", "xl": "..." } },
  "spacing": { "unit": 4, "scale": [0, 1, 2, 3, 4, 6, 8, 12, 16] },
  "radius": { "sm": "...", "md": "...", "lg": "..." }
}
```

Adjust the shape to whatever the chosen stack consumes most naturally
(CSS variables, Tailwind config, etc.) if `docs/ARCHITECTURE.md` already
specifies one.

**Design file (optional).** If a design-file tool is available and the
user wants an actual file, set up the base structure now: a
tokens/styles page or library matching the values above, plus a
components page for the base inventory. Screens come later, per feature,
in Mode 2.

### Update AGENTS.md

Add `docs/DESIGN.md` and `design/tokens.json` to the `## Project docs`
section of the root `AGENTS.md`.

### Next step

Tell the user: "Design system is written. Run `/design` again naming a
specific feature when you're about to build its screens — this doesn't
all need to happen upfront."

---

## Mode 2 — Feature screens

### Preconditions

Find the named feature's file under `docs/features/`. Read `docs/DESIGN.md`
and `design/tokens.json` — everything here must stay inside that system,
not reinvent it.

### Process

1. **Research**, scoped to this feature. Using whatever research tool is
   available (see above), look at how comparable products handle this
   specific screen or flow — not the whole product category again.

2. **Design the screens/states/flow** for this feature only, using the
   system's tokens and principles as fixed constraints: the specific
   screens needed, key states (empty, loading, error, success), and the
   path through them. Reuse base components from `docs/DESIGN.md`'s
   inventory; only introduce a new component if the feature genuinely
   needs one, and add it back to the system inventory if so.

3. Log to `docs/DECISIONS.md` (`Feature:` pointing at this feature's file)
   only for a real trade-off specific to this screen — not routine layout
   choices.

### Output

Write a companion file: `docs/features/NN-slug.design.md` (same number
and slug as the feature file), covering the screens/states/flow above.
Update the feature file's `design_doc` frontmatter field to point at it.

**Design file (optional).** If a design-file tool is available, create or
update this feature's screens there, inside the existing tokens/styles
structure from Mode 1 — don't redefine tokens per feature.

### Next step

Tell the user: "Screens for this feature are designed. Build it, then run
`/review` — it checks the implementation against this feature's design
doc as well as the system tokens."
