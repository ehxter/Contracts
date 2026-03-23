# Glossary

Definitions for every top-level and nested key used in component contracts.
Keys are listed in the order they appear in a contract file.

---

## Top-level keys

### `kind`
Always `"component"`. Reserved for future types (e.g., `"layout"`, `"pattern"`).

### `name`
The component's canonical name matching the Figma component set name exactly.

### `version`
Semantic version (`major.minor.patch`) of the *behavioral* contract.
Increment `major` when emitted events or state transitions change.
Increment `minor` when new states or props are added.
Increment `patch` for clarifications that do not change behavior.
Visual-only changes in Figma do not require a version bump.

---

## `figma`

Maps the contract to specific nodes in the Figma document.

| Key | Meaning |
|---|---|
| `nodeId` | The component set or parent frame node ID |
| `pageId` | The canvas/page node ID (`0:1`, `0:2`, …) |
| `pageName` | Human-readable page name from Figma |
| `componentSetId` | Same as `nodeId` for component sets; different if the contract covers a sub-tree |
| `variants[]` | All Figma component variants with their `nodeId` and `label` |

---

## `description`

One to three sentences. States the component's purpose in behavioral terms.
Does not mention appearance, color, or size.

---

## `props`

Behavioral inputs the consumer provides. Each prop entry:

| Key | Meaning |
|---|---|
| `type` | `boolean`, `string`, `number`, `function`, or a union of string literals |
| `default` | The value when the prop is not provided |
| `figma_property` | The Figma variant property name this prop maps to. `null` if no Figma mapping exists |
| `description` | What this prop does to the component's behavior |

**Exclude** props that only affect appearance with no behavioral consequence.

---

## `states`

Named UI states the component can occupy. A state is a distinct behavioral
mode — not a visual variation. Each state entry:

| Key | Meaning |
|---|---|
| `figma_variant` | The Figma variant label prefix that represents this state. `null` if the state has no Figma counterpart |
| `description` | What is true about the component while in this state |
| `entry_conditions[]` | The conditions that cause the component to enter this state |
| `exit_conditions[]` | The conditions that cause the component to leave this state |

Standard states to always consider (include even if some are empty or `null`-mapped):
`rest`, `hovered`, `focused`, `pressed`, `disabled`, `loading`, `error`,
`selected`, `expanded`, `checked`, `indeterminate`, `dragging`

---

## `state_machine`

The formal graph of valid state transitions.

| Key | Meaning |
|---|---|
| `initial` | The state the component starts in |
| `transitions[]` | Each valid state change |
| `transitions[].from` | Source state(s) — a string or array of strings |
| `transitions[].to` | Target state |
| `transitions[].trigger` | The event that initiates the transition (dot-notation: `pointer.enter`, `keyboard.focus`, `prop.disabled → true`) |
| `transitions[].guard` | Optional condition that must be true for the transition to occur |

**Named actions** (like `toggle_action`) are documented as separate objects
within `state_machine` when a single operation can be initiated from multiple
triggers and produces a side-effect (like emitting an event).

---

## `triggers`

Every input source that can cause the component to change state or emit an event.
Organized by input category:

| Category | Covers |
|---|---|
| `pointer` | `enter`, `leave`, `down`, `up`, `click` |
| `keyboard` | `tab`, `shift_tab`, `space`, `enter`, `escape`, `arrow_*` |
| `touch` | `tap`, `long_press`, `swipe_*` |
| `resize` | Container size changes; breakpoint crossings |
| `programmatic` | Prop changes, imperative method calls (`focus()`, `blur()`) |

Each trigger entry has:
- `description` — what causes this event to fire
- `effect` — what the component does in response

---

## `interactions`

Higher-level named operations that one or more triggers can initiate.
The distinction: triggers are raw input events; interactions are semantic
operations (toggle, submit, expand, clear, select).

Each interaction entry:

| Key | Meaning |
|---|---|
| `description` | What this operation does |
| `initiated_by[]` | Which triggers invoke it |
| `guard` | Condition that must hold |
| `effect` | Concrete outcome |
| `emits` | Which `emits` key fires as a result |

---

## `emits`

Events the component fires outward to its consumer. Each entry:

| Key | Meaning |
|---|---|
| `description` | When and why this event fires |
| `payload` | Object describing the event's data shape — key names and their behavioral types |
| `not_emitted_when[]` | Explicit list of conditions under which this event is suppressed |

---

## `accessibility`

### `role`
The WAI-ARIA role assigned to the root element.

### `aria_attributes`
ARIA attributes the component manages, with a description of what each reflects.

### `keyboard_navigation`
- `receives_focus`: whether the component can receive keyboard focus
- `tab_stop`: when it appears in the tab order
- Per-key behavior entries

### `focus_management`
- `focus_visible`: whether and when a visible focus ring must appear
- `focus_trap`: whether focus is constrained within the component
- `focus_on_disable`: what happens to focus when the component becomes disabled

### `screen_reader`
Announcements that must occur on key interactions.

---

## `responsive`

Behavioral (not visual) changes driven by container or viewport size.
If the component's behavior is identical at all sizes, `behaviors` is `{}`.

---

## `composition`

### `parent_requirements`
What the parent component or consumer must provide beyond props:
layout context, React context providers, label association obligations.

### `slots`
Named child slot definitions. For each slot:

| Key | Meaning |
|---|---|
| `required` | Whether the slot must be filled |
| `cardinality` | `single`, `multiple`, or `none` |
| `allowed_types` | Component types or `ReactNode` |
| `behaviors_inherited` | Behaviors this slot's content inherits from the parent |
| `behaviors_overridden` | Parent behaviors that the slot's content can override |

An empty `slots: {}` means the component has no consumer-facing child slots.

---

## `constraints`

### `mutual_exclusions[]`
States or props that cannot be active simultaneously. Each entry has `rule`
and `reason`.

### `conditional_behaviors[]`
If-then rules that express context-dependent behavior. Format:
`{ "if": "...", "then": "..." }`.

### `required_combinations[]`
Props or attributes that must co-exist. Each entry has `rule` and `reason`.

---

## `scenarios`

Named end-to-end behavioral flows. Written in plain prose, present tense.
Each scenario traces one path through the state machine from a starting
condition to a final outcome, including what the component emits.

Minimum required scenarios:
- Happy-path interaction (user successfully uses the component)
- Keyboard-only path
- Disabled component attempt
- Controlled-mode programmatic update
