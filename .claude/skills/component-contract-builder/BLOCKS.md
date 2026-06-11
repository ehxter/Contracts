# BLOCKS — The Building-Block Catalog

Every contract section is a **block**. Pick the **required core**, add **recommended** blocks
unless clearly N/A, and add **optional** blocks only when their *when* applies. Assemble the
chosen blocks into one JSON object and fill each from the prompt + Figma.

**How to read an entry**

```
### <block> · <tier>
when:   the condition under which the block earns its place
shape:  the JSON skeleton — placeholder tokens only, never sample data
rules:  loose guidance on how to fill it; gotchas; reminders
```

All shapes use `<placeholder>` tokens. Replace them; never copy them literally. **No visual
values anywhere** (see the golden rules in SKILL.md). Tiers: **required** · **recommended** ·
**optional**.

---

# Required core

These ten blocks appear in every contract.

### kind · required
when: always.
shape:
```json
"kind": "component"
```
rules: almost always `"component"`. Reserved for future kinds.

### name · required
when: always.
shape:
```json
"name": "<Canonical Component Name>"
```
rules: the human name; should match the Figma component / component-set name. If Figma
misspells it, keep the correct name here and note the Figma spelling in `figma.figmaName`.

### version · required
when: always.
shape:
```json
"version": "<major.minor.patch>"
```
rules: semantic version of the **behavior**. Bump major on behavior/transition/emit changes,
minor on new states/props, patch on clarifications. Visual-only Figma changes do not bump it.

### figma · required
when: always.
shape:
```json
"figma": {
  "nodeId": "<n:m>",
  "URL": "https://www.figma.com/design/<fileKey>/<file>?node-id=<n-m>",
  "figmaName": "<exact Figma layer name, only if it differs or is misspelled>",
  "pageId": "<canvas/page node id, optional>"
}
```
rules: `nodeId` + `URL` are the minimum; drop the optional keys when unused. A nested child
may carry its own `figma` block inside `composition`.

### description · required
when: always.
shape:
```json
"description": "<2–5 sentences: what the component is and does, behaviorally. No appearance.>"
```
rules: purpose + core behavior. Mention authority over Figma only if relevant.

### props · required
when: always.
shape:
```json
"props": {
  "<Enum Prop>": {
    "type": "enum",
    "default": "<default option>",
    "options": ["<option>", "<option>"],
    "figma_property": "<exact Figma variant property name, or null>",
    "description": "<what this prop changes about behavior>"
  },
  "<Boolean Prop>": {
    "type": "boolean",
    "default": false,
    "description": "<what toggling it does; if it gates a nested element, say so>"
  },
  "<String Prop>": {
    "type": "string",
    "default": null,
    "required": false,
    "description": "<what text it supplies and where it is shown>"
  },
  "Extra Props": {
    "description": "<real data inputs needed to implement, not behavioral toggles or Figma variants: sources, URLs/handlers, item collections, data series>",
    "figma_note": "<optional: where this contract overrides the Figma demo state>"
  }
}
```
rules: one entry per behavioral input. **Always include `Extra Props`** when the component
has data inputs beyond Figma variants (nearly always). Use a `_note` on the whole `props`
object when the prop set is provisional (e.g. a chart mirroring current Figma properties).

### composition · required
when: always — the single structural block.
shape:
```json
"composition": {
  "nested_components": {
    "<Child Component>": {
      "shared": false,
      "used_when": "<the prop/state/variant that makes this child appear, optional>",
      "figma": { "nodeId": "<n:m>", "URL": "<optional, if the child has its own source>" },
      "description": "<what the child contributes; if it has its own contract, say 'see the <name> contract'>",
      "props": { "<inline the child's key props when it has no standalone contract>": "<…>" },
      "behaviors": [
        "<in-context behavioral rule for this child, e.g. which state is interactive, what activating it does, what it never decides for itself>"
      ]
    }
  },
  "nested_elements": {
    "<Internal Element>": { "type": "container | element", "description": "<internal/structural part, not a consumer slot>" }
  },
  "slots": {
    "<slot>": { "required": false, "cardinality": "single | multiple | none", "allowed_types": "<component/content types>" }
  },
  "parent_requirements": {
    "description": "<what the parent/consumer must provide beyond props: layout, context, form/label association>"
  }
}
```
rules: `nested_components` = child components (mark `shared: true` for reused ones,
`used_when` for conditional ones); add `behaviors` when a child has no standalone contract or
its behavior in this parent is non-obvious. `nested_elements` = internal/structural parts
(for charts: axes, grid, bars/curve, tooltip — describe each behaviorally; use a `parts` map
for sub-structure). `slots` = consumer-fillable slots (`{}` or a "None" note when none).
`parent_requirements` only when the parent owes the component something.

### constraints · required
when: always — at least `conditional_behaviors`.
shape:
```json
"constraints": {
  "conditional_behaviors": [
    { "if": "<a prop/state/variant condition>", "then": "<what must be true>" },
    { "if": "<condition>", "and-if": "<second condition>", "then": "<what must be true>" }
  ],
  "mutual_exclusions": [
    { "rule": "<things that cannot be active at once>", "reason": "<why>" }
  ],
  "required_combinations": [
    { "rule": "<things that must co-exist>", "reason": "<why>" }
  ]
}
```
rules: `conditional_behaviors` is the workhorse (if/then, optional `and-if`). Add
`mutual_exclusions` / `required_combinations` only when they apply. Algorithmic or derived
logic (e.g. a parent computing a child's state from data) lives here as prose `if/then`
rules plus, if helpful, a named sub-object with numbered steps and worked cases.

### scenarios · required (key); content recommended
when: always present as a key.
shape:
```json
"scenarios": {
  "<scenario_name>": "<present-tense prose walkthrough of one flow from a starting condition to an outcome, noting what is emitted or that nothing is>"
}
```
rules: named end-to-end behavioral flows. May be `{}` for the most trivial display-only
components, but prefer one or two.

### notes · required
when: always.
shape:
```json
"notes": "<closing prose: the component's essence, key conditional behaviors, relationships to nested/shared children, and an explicit statement that visual specifics live in Figma and this contract is authoritative over Figma where they differ>"
```
rules: a string, or an object with a `description` key. Always restate **authority over
Figma** and that visuals are out of scope.

---

# Recommended

Include unless clearly not applicable.

### intent · recommended
when: a product/feature component where use cases clarify reach-for situations.
shape:
```json
"intent": {
  "use cases": [ "<concrete situation where someone reaches for this component>" ]
}
```
rules: concrete, behavioral use cases — not a feature list.

### responsive · recommended
when: layout or behavior varies with available size.
shape:
```json
"responsive": {
  "resize": {
    "description": "<how it responds to space, often via a Size prop rather than fluid resizing>",
    "width": "determined by parent | fixed | <…>",
    "height": "determined by content | fixed | <…>",
    "breakpoint_behaviors": {
      "desktop": "<what is true on desktop>",
      "mobile": "<what is true on mobile>"
    }
  }
}
```
rules: behavioral/layout change driven by size — **not** visual reflow. `width`/`height` may
state *fixed vs fluid* (a behavioral fact) but never a pixel number. `{}` breakpoint_behaviors
means no size-driven behavior.

---

# Optional — include only when the *when* applies

### category · optional
when: the component belongs to a named family that affects how it is read (e.g. a chart).
shape:
```json
"category": "<family label>"
```
rules: omit for plain components.

### figma_design_notes · optional
when: Figma represents the component poorly — typically a static mock of a dynamic,
data-driven component (charts).
shape:
```json
"figma_design_notes": [
  "<caveat: screenshots are the source of truth for look/proportion; code/measurements are approximate>",
  "<what is baked vs data-driven: counts, shapes, labels, ticks are variable>"
]
```
rules: describes *what is unreliable in Figma* — never colors or sizes to implement.

### modes · optional
when: an external/parent-driven mode (not an internal state) changes how the same triggers
behave.
shape:
```json
"modes": {
  "description": "<how modes are applied, usually by the parent, and what they change>",
  "<mode-name>": { "description": "<what this mode does and how interactions differ in it>" }
}
```
rules: a mode is set from outside; a state is reached from inside. Keep them distinct.

### variants · optional
when: one component renders several major variants that differ in content/sections.
shape:
```json
"variants": {
  "<Variant>": {
    "description": "<what this variant represents>",
    "header": "<header title/icon used by this variant, optional>",
    "sections": { "<Section>": "<what this section shows in this variant>" }
  }
}
```
rules: pair with `constraints.conditional_behaviors`. Variant-to-variant conversions belong
in `state_machine`, not here.

### states · optional
when: the component has distinct interactive UI states (rest/hover/selected/disabled/…).
shape:
```json
"states": {
  "<state>": {
    "figma_variant": "<the Figma variant label, or null if inferred>",
    "description": "<what is true while in this state>",
    "entry_conditions": ["<what causes entry>"],
    "exit_conditions": ["<what causes exit>"]
  }
}
```
rules: one entry per state; always give entry/exit. Display-only components omit this block.

### state_machine · optional
when: it moves between states or variants and the moves are worth formalizing. The **single
home** for all transitions (including variant conversions) and for named semantic operations
reachable from multiple triggers.
shape:
```json
"state_machine": {
  "initial": "<state>",
  "transitions": [
    { "from": "<state>", "to": "<state>", "trigger": "<event>", "guard": "<condition, optional>", "description": "<optional>" }
  ],
  "<named_operation>": {
    "description": "<a semantic operation, e.g. toggle/activate/select, or a data-driven recompute>",
    "triggers": ["<event>"],
    "guard": "<condition, optional>",
    "effects": ["<what happens>"],
    "emits": "<event-name, optional>"
  }
}
```
rules: declare `initial` + every valid transition. Variant conversions are just transitions.
A named operation may be user-triggered (toggle/activate) or a derive/recompute that runs
when data changes.

### triggers · optional
when: the component is interactive. Covers **all** runtime interactions.
shape:
```json
"triggers": {
  "<named_trigger>": {
    "description": "<what the user does>",
    "effect": "<what the component does>",
    "figma_limitation": "<optional: how Figma mocks this statically>"
  }
}
```
rules: each trigger = description (what fires it) + effect (what happens). For rich
components, group by input category (`pointer`, `keyboard`, `touch`, `resize`,
`programmatic`). For interactions Figma mocks statically (charts), add `figma_limitation`
and record what is data-driven in `figma_design_notes`.

### emits · optional
when: the component fires events outward to its consumer.
shape:
```json
"emits": {
  "<event-name>": {
    "description": "<when it fires>",
    "payload": { "<key>": "<behavioral type>" },
    "not_emitted_when": ["<suppression condition>"]
  }
}
```
rules: describe payloads behaviorally (shape/meaning), not concrete values.

### accessibility · optional
when: a11y behavior matters **and** the prompt calls for it. (ARIA can instead be modeled as
`aria-*` props in `props`.)
shape:
```json
"accessibility": {
  "role": "<WAI-ARIA role of the root>",
  "aria_attributes": { "<aria-attr>": "<what it reflects>" },
  "keyboard_navigation": { "receives_focus": true, "tab_stop": "<when in the tab order>", "<key>": "<behavior>" },
  "focus_management": { "focus_visible": "<when a focus indicator is required>", "focus_trap": false },
  "screen_reader": ["<announcement that must occur on a key interaction>"]
}
```
rules: role, managed ARIA, keyboard nav, focus management, announcements — behavioral, not
visual. *That* a focus indicator is required goes here; its style does not.

---

# Assembly skeleton

Picked blocks compose into one object in roughly this order (structure only — placeholders,
no real component):

```json
{
  "kind": "component",
  "name": "<Component Name>",
  "version": "<major.minor.patch>",
  "figma": { "nodeId": "<n:m>", "URL": "<url>" },
  "description": "<behavioral summary>",

  "intent": { "use cases": ["<…>"] },

  "props": { "<Prop>": { "type": "<enum|boolean|string>", "description": "<…>" }, "Extra Props": { "description": "<…>" } },

  "states": { "<state>": { "description": "<…>", "entry_conditions": ["<…>"], "exit_conditions": ["<…>"] } },
  "state_machine": { "initial": "<state>", "transitions": [] },
  "triggers": { "<trigger>": { "description": "<…>", "effect": "<…>" } },

  "responsive": { "resize": { "description": "<…>", "breakpoint_behaviors": {} } },

  "composition": { "nested_components": {}, "nested_elements": {} },

  "constraints": { "conditional_behaviors": [ { "if": "<…>", "then": "<…>" } ] },

  "scenarios": { "<scenario>": "<…>" },

  "notes": "<closing summary; authoritative over Figma; visuals out of scope>"
}
```

Include only the blocks the component earns; drop the rest entirely.
