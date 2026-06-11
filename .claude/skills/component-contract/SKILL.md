---
name: component-contract
description: >-
  Author or update a component behavioral contract from a Figma design plus a
  behavioral prompt, producing one unified JSON contract under components/ that a
  coding agent (Claude Code) can implement from. Use whenever the user wants to
  create, extract, recheck, update, or validate a component contract; map a Figma
  component to a contract; or add a component to this repo. Reads Figma via the
  figma-console MCP (Desktop Bridge) and/or the official Figma MCP (URL-based).
---

# Component Contract — Authoring Rule Book

You are authoring a **behavioral contract**: the spec that sits between a Figma design
and a coding agent that implements the component. It captures every event-, interaction-,
and resize-driven behavior — and nothing visual. A coding agent reads it before writing a
line, so ambiguity here becomes hallucinated logic there.

This skill is **semi-strict**: the section policy below is the rule book, but you adapt
which optional sections appear to the component in front of you. The consumer is an LLM, so
there is no machine validator — your judgment and this rule book are the enforcement.

## The three inputs

A contract is the merge of three sources, each with a fixed role:

1. **Behavioral prompt (the user)** — the PRIMARY authority on intent and behavior. What
   the component is *for*, how it should behave, what is required vs optional. When the
   prompt and Figma disagree, the prompt wins and the contract records it.
2. **Figma Console MCP (`figma-console`, Desktop Bridge over WebSocket)** — the richest
   source of live structure when the desktop plugin is connected: component property
   definitions, the variant matrix, instance overrides, reactions, bound variables, and
   plugin-fresh screenshots. **Prefer it when connected.**
3. **Figma MCP (official, URL-based)** — reads a node straight from a Figma URL without the
   desktop plugin. Use it as the fallback, for headless/shared-link work, or to cross-check
   the console MCP.

### Picking and using the Figma source
- If the user gave a **URL**, you can start immediately with the **Figma MCP**
  (`get_metadata` → `get_design_context` → `get_screenshot` → `get_variable_defs`).
- Probe `figma-console` with `figma_get_status` (`probe: true`). If it reports a connected
  file, prefer it: `figma_get_component_for_development` (deep tree + property defs +
  reactions), `figma_get_selection` (what the user is pointing at),
  `figma_capture_screenshot` (verify after any change). If the Desktop Bridge isn't
  connected, say so and fall back to the Figma MCP rather than blocking.
- `get_component_for_development` output can exceed the token limit; when it does, read the
  saved file in slices and pull just the property definitions, the variant tree, and the
  instance `componentProperties`.
- When **both** are available, cross-check structure between them.

### Reconciliation rules
- **Prompt → behavior. Figma → structure** (variants, props, nesting, slots) and **visual
  reference only**.
- Figma is **never** a source of pixel values (colors, sizes, type, spacing, radii). Those
  stay in Figma/design tokens and are out of the contract.
- **This contract is authoritative over Figma** wherever they differ. Encode the behavior
  the prompt describes and record the divergence in a `figma_note` (and/or in `notes`).
- For **static-mock components** (charts especially): screenshots are the source of truth
  for look/proportion; code and measurements are approximate; capture the caveats in
  `figma_design_notes`.

## Extract workflow

1. **Gather inputs.** Read the behavioral prompt. Resolve the Figma source (URL or live
   selection). Choose the MCP (console if connected, else Figma MCP).
2. **Map structure.** Get the variant matrix, component **property definitions**, nested
   components, and instance overrides. These become `props`, `states`/`variants`, and
   `composition.nested_components`.
3. **Capture a screenshot** for confirmation — to read layout/anatomy, never to extract
   pixel values.
4. **Merge with the prompt.** The prompt defines what it *does* and what is required vs
   optional; Figma confirms what exists. Resolve conflicts in the prompt's favor.
5. **Choose sections by context** using the policy below — lean core first, optional
   sections only as earned.
6. **Fill the template.** Start from `templates/contract.template.json`, strip the
   underscore-annotation keys, keep only the sections you need.
7. **Mark authority & data.** Add a `figma_note` for any divergence; put real data inputs
   not modeled as Figma variants under **`Extra Props`**.
8. **Cross-reference** shared/nested components ("see the <name> contract") instead of
   duplicating them.
9. **Write** `components/<category>/<name>.json` (single JSON file).
10. **Self-review** against the checklist at the end.

When *updating* an existing contract (a "recheck"), re-pull the Figma node, diff it and the
prompt against the current contract, and report what was missing/wrong before editing.

## Section policy (required vs optional)

Use `templates/contract.template.json` for the exact shape of each section. Tiers:

**ALWAYS REQUIRED** — present in every contract (verified across all of `components/`):
- `kind` · `name` · `version`
- `figma` — source mapping (`nodeId` + `URL` minimum)
- `description` — behavioral, no appearance
- `props` — with an **`Extra Props`** entry whenever the component has data inputs beyond
  Figma variants (nearly always; a pure slot/config component may omit it)
- `composition` — the single structural section for every component (charts included);
  structural parts (chart axes, grid, bars/curve, tooltip) go under `nested_elements`; a
  nested component entry may also carry a `behaviors` array of its in-context behavioral
  rules when it has no standalone contract or its behavior is non-obvious
- `constraints` — at least `conditional_behaviors`
- `scenarios` — always present as a key; fill it with flows (may be `{}` only for the most
  trivial display-only components)
- `notes` — closing summary; restate that visuals live in Figma and that the contract is
  authoritative over Figma

**RECOMMENDED** — include unless clearly N/A (present in ~12/14 of `components/`):
- `intent.use cases` · `responsive`

**OPTIONAL — add only when the trigger applies:**
| Section | Add when |
|---|---|
| `category` | the component belongs to a named family (e.g. `"chart"`) |
| `figma_design_notes[]` | Figma is a static/poor mock (charts) — note screenshots win |
| `modes` | an external/parent-driven mode changes behavior (distinct from `states`) |
| `variants` | one component renders several major variants |
| `states` (+ entry/exit) | it has distinct interactive UI states |
| `state_machine` | it moves between states/variants — the single home for all transitions (incl. variant conversions) and named semantic operations (toggle/activate) |
| `triggers` | it is interactive (pointer/keyboard/touch/resize/programmatic, or named); chart hover/scrub/select live here, with a `figma_limitation` note when Figma mocks them statically |
| `emits` | it fires outward events to its consumer |
| `accessibility` (or `aria-*` props) | a11y matters AND the prompt calls for it |

**Lean by default.** A display-only card needs roughly: core + `intent` + `responsive` +
`composition` + `constraints` + `scenarios` + `notes`. Do **not** add `states`,
`state_machine`, `emits`, or `accessibility` to a component that has no such behavior just
to look complete.

## Do / Don't

**Do**
- Write behavior in plain, present-tense prose ("activating it navigates to…", "the badge
  is omitted when…").
- Make every optional inclusion earn its place against the trigger above.
- Keep one component = one JSON file under the right `components/<category>/` folder.
- Mirror the conventions already in `components/` (that directory is the ground truth).

**Don't**
- Don't write any visual value (color, size, font, spacing, radius, shadow, z-index,
  duration) or CSS/Tailwind class anywhere.
- Don't invent behavior, props, or states the prompt and Figma don't support.
- Don't treat Figma's demo state as the spec — the contract overrides it.
- Don't produce YAML; contracts are JSON only.
- Don't pad a lean component with rich sections it doesn't need.
- Don't introduce redundant sections — one concept, one home: use `composition` (never
  `anatomy`), `state_machine` for all transitions and named operations (never a separate
  `transitions` or `interactions`), and `triggers` for runtime interactions (never
  `interactivity`).

## Self-review checklist (before finishing)

- [ ] All ALWAYS-REQUIRED sections present; `props` has an `Extra Props` entry for data.
- [ ] No visual values or class names anywhere.
- [ ] Every optional section present is justified by its trigger; none is missing that the
      behavior demands.
- [ ] Conflicts with Figma are resolved in the prompt's favor and recorded in a `figma_note`.
- [ ] Shared/nested components are cross-referenced, not duplicated.
- [ ] `notes` restates authority-over-Figma and that visuals are out of scope.
- [ ] File written to `components/<category>/<name>.json` and parses as JSON
      (`python3 -m json.tool <file>`).
- [ ] If Figma was changed, a fresh `figma_capture_screenshot` confirms the result.

## References
- `templates/contract.template.json` — the annotated shape of every section.
- `docs/agents.md` — the dual-MCP pipeline and extract workflow in depth.
- `docs/glossary.md` — every section defined, with its tier.
- `docs/requirements.md` — the section policy as a checklist.
- `docs/limitations.md` — what never belongs in a contract (behavioral, not visual).
- `components/<category>/<name>.json` — where authored contracts are written (one JSON per
  component).
