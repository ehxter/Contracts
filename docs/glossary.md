# Glossary — Contract Sections

Every section that can appear in a component contract, defined, with its **tier**:

- **Required** — every contract has it.
- **Recommended** — include unless clearly not applicable.
- **Optional** — include only when the stated trigger applies.

For the exact JSON shape of each, see `templates/contract.template.json`. For the rule book
that decides which sections to include, see `.claude/skills/component-contract/SKILL.md`.
Real examples live in `components/`.

---

## Identity & source

### `kind` · *Required*
Almost always `"component"`. Reserved for future kinds.

### `category` · *Optional*
**When:** the component belongs to a named family that affects how it is read (e.g.
`"chart"`). Omit for plain components.

### `name` · *Required*
Canonical human name; matches the Figma component/component-set name. Note any Figma
misspelling via `figma.figmaName`.

### `version` · *Required*
Semantic version of the **behavioral** contract. Bump **major** on behavior/transition/emit
changes, **minor** on new states/props, **patch** on clarifications. Visual-only Figma
changes do not bump it.

### `figma` · *Required*
Where the component lives in Figma. Minimum: `nodeId` + `URL`. Optional sub-keys:
`figmaName`, `pageId`, and `documentationPage` (`nodeId`/`URL`/`note`) when a separate Figma
page documents the variant matrix or anatomy. Nested components may carry their own `figma`
block inside `composition`.

### `figma_design_notes[]` · *Optional*
**When:** Figma represents the component poorly — typically a static mock of a dynamic,
data-driven component (charts). A list of caveats stating that **screenshots are the source
of truth** for look/proportion, that code/measurements are approximate, and what is baked
vs. data-driven.

---

## Description & intent

### `description` · *Required*
One short paragraph: what the component is and does, in behavioral terms. No appearance.

### `intent` · *Recommended*
`use cases`: concrete situations where a designer/developer reaches for this component.

---

## Inputs

### `props` · *Required*
Behavioral inputs the consumer controls. Each entry: `type` (`enum`/`boolean`/`string`/…),
`default`, optional `options` (enums), optional `figma_property` (exact Figma variant
property name, or `null`), optional `required`, and a behavioral `description`.

- **`Extra Props`** (required sub-entry): the catch-all describing real data inputs needed
  for implementation that are **not** behavioral toggles or Figma variants — image sources,
  URLs/handlers, item collections, data series. May carry a `figma_note` stating where the
  contract overrides the Figma demo state.
- **`_note`**: use when the whole prop set is provisional (e.g. mirrors current Figma
  properties and will change when wired to real data — common for charts).

---

## Behavioral model (optional, by context)

### `modes` · *Optional*
**When:** an external/parent-driven mode (not an internal state) changes behavior (e.g.
`default` vs `selectable`). Describe each mode and how the same triggers behave differently.

### `variants` · *Optional*
**When:** one component renders several major variants differing in content/sections (e.g.
Video / Live / Empty State). Per variant: `description`, optional `header`, and `sections`.
Pair with `constraints.conditional_behaviors`.

### `states` · *Optional*
**When:** the component has distinct interactive UI states. Per state: `figma_variant` (the
Figma label, or `null` if inferred), `description`, `entry_conditions[]`,
`exit_conditions[]`. Display-only components omit this.

### `state_machine` · *Optional*
**When:** the component moves between states or variants and it is worth formalizing. This is
the **single home for all transitions** — including variant-to-variant conversions (e.g.
Empty State → Video) — and for **named semantic operations** (toggle/activate/select)
reachable from multiple triggers. `initial` + `transitions[]`
(`from`/`to`/`trigger`/optional `guard`/optional `description`) + named actions (e.g.
`activate_action`) with `triggers`/`guard`/`effects`/`emits`.

### `triggers` · *Optional*
**When:** the component is interactive — covers **all** runtime interactions, including chart
hover/scrub/select. Either input categories (`pointer`, `keyboard`, `touch`, `resize`,
`programmatic`) or a flat set of named triggers. Each: `description` (what fires it) +
`effect` (what the component does) + optional `figma_limitation` (how Figma mocks it
statically — for charts; record what is data-driven in `figma_design_notes`).

### `emits` · *Optional*
**When:** the component fires events outward. Each: `description`, `payload` shape,
`not_emitted_when[]` suppression rules.

---

## Structure & accessibility

### `accessibility` · *Optional*
**When:** a11y behavior matters AND the prompt calls for it. `role`, `aria_attributes`,
`keyboard_navigation`, `focus_management`, `screen_reader` — behavioral, not visual.
(ARIA may alternatively be modeled as `aria-*` props in `props`.)

### `responsive` · *Recommended*
Behavioral/layout changes driven by size — **not** visual reflow. `resize` with
`description`, `width`, `height`, and `breakpoint_behaviors` (`{}` means no size-driven
behavior).

### `composition` · *Required*
The single structural section for **every** component (charts included). What the component
is built from and asks of its parent:
- `nested_components` — child components; mark `shared: true` for reused ones, `used_when`
  for conditional ones, an optional `figma` block, and cross-reference a child's own
  contract when it has one.
- `nested_elements` — internal/structural parts (not consumer-facing). For charts these are
  the structural parts (axes, grid, bars/curve, tooltip): describe each behaviorally, use a
  `parts` map for sub-structure and `used_when` for prop-gated parts.
- `slots` — consumer-fillable slots (`{}` or a "None" note when there are none).
- `parent_requirements` — layout/context/form/label obligations on the parent.

---

## Rules & flows

### `constraints` · *Required*
The rules that bound behavior:
- `conditional_behaviors[]` — the workhorse: `{ if, then }`, optionally with `and-if`.
- `mutual_exclusions[]` — `{ rule, reason }` for things that can't co-exist.
- `required_combinations[]` — `{ rule, reason }` for things that must co-exist.

### `scenarios` · *Required key, content recommended*
Always present as a key (every existing contract has it). Named end-to-end behavioral flows
in plain present-tense prose; each traces one path from a starting condition to an outcome
(and what is emitted, or that nothing is). May be `{}` for the most trivial display-only
components, but prefer at least one or two.

### `notes` · *Required*
Closing prose summary tying the contract together: the component's essence, key conditional
behaviors, relationships to shared/nested components, and an explicit statement that visual
specifics live in Figma and that **this contract is authoritative over Figma** where they
differ. A string or an object with a `description` key.
