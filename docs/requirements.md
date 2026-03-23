# Requirements — What a Valid Contract Must Contain

A contract is valid when every item in this checklist is satisfied.
Use this as a pre-commit gate and as a review guide.

---

## Structure requirements

- [ ] `kind` is present and equals `"component"`
- [ ] `name` matches the Figma component set name exactly
- [ ] `version` follows semantic versioning (`X.Y.Z`)
- [ ] All top-level keys are present, even if their value is `{}`, `[]`, or `null`

No top-level key may be omitted. Omission means "not considered," which is
different from "not applicable."

---

## Figma mapping requirements

- [ ] `figma.nodeId` points to a valid, accessible node in the Figma document
- [ ] `figma.variants` lists every variant in the Figma component set
- [ ] Every Figma variant label is accounted for in either `props` or `states`

---

## Props requirements

- [ ] Every prop that affects behavior (not only appearance) is documented
- [ ] Each prop has `type`, `default`, `figma_property`, and `description`
- [ ] `figma_property` is `null` when no Figma mapping exists, not omitted
- [ ] No prop contains visual values (colors, sizes, weights)

---

## States requirements

- [ ] Every named state in the Figma variant matrix has a `states` entry
- [ ] `focused` is always documented, even if `figma_variant` is `null`
- [ ] `pressed` is documented for any component with pointer click interaction
- [ ] Each state has `entry_conditions` and `exit_conditions` — neither empty
- [ ] `figma_variant` is `null` (not omitted) for states with no Figma counterpart

---

## State machine requirements

- [ ] `initial` state is declared
- [ ] Every `states` entry has at least one transition leading to it
- [ ] Every `states` entry has at least one transition leading away from it
  (exception: terminal states, which must be explicitly noted)
- [ ] The `disabled` state has a transition from every interactive state
- [ ] Any multi-trigger operation (like toggle) is documented as a named action

---

## Triggers requirements

- [ ] `pointer`, `keyboard`, `touch`, `resize`, and `programmatic` sub-keys
  are all present
- [ ] Every trigger that appears in `state_machine.transitions` has an entry here
- [ ] `resize.breakpoint_behaviors` is `{}` if the component has no responsive
  behavior (not omitted)

---

## Interactions requirements

- [ ] Every named action referenced in `state_machine` has an entry in
  `interactions`
- [ ] Each interaction lists all triggers that can initiate it
- [ ] Each interaction with a side-effect lists the `emits` key it fires

---

## Emits requirements

- [ ] Every event referenced by `interactions` or `state_machine.toggle_action`
  has an entry in `emits`
- [ ] Every `emits` entry has a `payload` (even if `{}` for events with no data)
- [ ] Every `emits` entry has a `not_emitted_when` list with at least one entry

---

## Accessibility requirements

- [ ] `role` is a valid WAI-ARIA role
- [ ] `aria_attributes` documents every ARIA attribute the component manages
- [ ] `keyboard_navigation.receives_focus` is explicitly `true` or `false`
- [ ] `focus_management.focus_visible` describes the focus indicator requirement
- [ ] `focus_management.focus_on_disable` describes focus behavior if component
  becomes disabled while focused
- [ ] At least one `screen_reader` announcement is documented

---

## Constraints requirements

- [ ] If any two states are mutually exclusive, `mutual_exclusions` names them
- [ ] If `disabled` is a state, a `conditional_behaviors` entry covers it:
  `"if disabled is true, then all interaction triggers are suppressed"`
- [ ] If both controlled and uncontrolled modes exist, both are documented in
  `conditional_behaviors`
- [ ] If an ARIA label is required, `required_combinations` names this rule

---

## Scenarios requirements

- [ ] At minimum four scenarios are present:
  1. Successful user interaction (happy path)
  2. Keyboard-only path
  3. Disabled component interaction attempt
  4. Programmatic / controlled-mode update
- [ ] Each scenario is written in plain English, present tense
- [ ] Each scenario names what the component emits (or explicitly states it
  emits nothing)

---

## Anti-patterns that invalidate a contract

| Anti-pattern | Why it invalidates |
|---|---|
| A CSS value anywhere in the file | Contract is no longer implementation-independent |
| A `states` entry with no `entry_conditions` | State machine is non-deterministic |
| An `emits` entry with no `not_emitted_when` | Emission rules are undefined |
| `composition.slots` omitted entirely | Child composition is unknown |
| A scenario that never mentions emit or "nothing fires" | Observable output is unspecified |
