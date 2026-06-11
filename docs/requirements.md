# Section Policy — What a Contract Must (and Need Not) Contain

This is the **semi-strict** checklist for a contract. There is no machine validator — the
consumer is an LLM, and your judgment plus this policy are the enforcement. The rule:
**a stable required core is always present; everything else is included only when the
component earns it.** Padding a lean component with sections it does not need is as wrong as
omitting one it does.

See `docs/glossary.md` for definitions and `templates/contract.template.json` for shapes.

---

## Always required

Present in every contract, no exceptions:

- [ ] `kind` — present (usually `"component"`)
- [ ] `name` — matches the Figma component/component-set name
- [ ] `version` — semantic version (`X.Y.Z`)
- [ ] `figma` — at least `nodeId` + `URL`
- [ ] `description` — behavioral, no appearance
- [ ] `props` — every behavioral input, **plus an `Extra Props` entry** for data inputs not
      modeled as Figma variants
- [ ] `composition` — the single structural section for every component (charts included);
      structural parts go under `nested_elements`
- [ ] `constraints` — at least `conditional_behaviors`
- [ ] `scenarios` — always present as a key; fill it with flows (may be `{}` only for the
      most trivial display-only components)
- [ ] `notes` — closing summary that restates **authority over Figma** and that visuals are
      out of scope

---

## Recommended

Include unless clearly not applicable (present in most contracts):

- [ ] `intent.use cases` — for product/feature components
- [ ] `responsive` — whenever layout/behavior varies with size

---

## Optional — include only when the trigger applies

| Section | Include when… |
|---|---|
| `category` | the component belongs to a named family (e.g. `"chart"`) |
| `figma_design_notes[]` | Figma is a static/poor mock — note that screenshots win |
| `modes` | an external/parent-driven mode changes behavior (distinct from `states`) |
| `variants` | one component renders several major variants |
| `states` (+ entry/exit conditions) | it has distinct interactive UI states |
| `state_machine` | it moves between states/variants — the single home for all transitions (incl. variant conversions) and named semantic operations |
| `triggers` | it is interactive (chart hover/scrub/select included, with a `figma_limitation` note when mocked statically) |
| `emits` | it fires outward events to its consumer |
| `accessibility` (or `aria-*` props) | a11y matters AND the prompt calls for it |

---

## Cross-cutting rules (always enforced)

- [ ] **Behavioral, not visual.** No colors, sizes, typography, spacing, radii, shadows,
      z-index, durations, or CSS/Tailwind classes anywhere. (See `docs/limitations.md`.)
- [ ] **Contract over Figma.** Conflicts with the Figma demo state are resolved in the
      behavioral prompt's favor and recorded in a `figma_note`.
- [ ] **Cross-reference, don't duplicate.** Shared/nested components point to their own
      contracts ("see the <name> contract").
- [ ] **`Extra Props`** captures real data inputs (sources, URLs/handlers, collections,
      data series) that Figma variants cannot express.
- [ ] **JSON only**, one file per component at `components/<category>/<name>.json`.

---

## Reference profiles (what "right-sized" looks like)

Right-sized section sets by component profile:

| Profile | Sections beyond the required core |
|---|---|
| **Display-only card** | `intent`, `responsive` |
| **Variant-driven card** | `intent`, `variants`, `state_machine` (variant conversions), `triggers`, `responsive` |
| **Stateful card** | `intent`/`modes`, `states`, `triggers`, `responsive` |
| **Controlled micro-component** | `modes`, `states`, `triggers`, `emits` (often "none/controlled"), `responsive` |
| **Chart** | `category`, `figma_design_notes`, `intent`, `triggers` (hover/scrub, with `figma_limitation`), `responsive` — structural parts go in `composition.nested_elements` |

(Required core is always present: `kind`, `name`, `version`, `figma`, `description`,
`props`, `composition`, `constraints`, `scenarios`, `notes`.) If your contract's section set
looks nothing like the closest profile, re-check whether you over- or under-included.

> **Canonical vocabulary.** Use `composition` (not `anatomy`), `state_machine` for all
> transitions and named operations (not `transitions`/`interactions`), and `triggers` for all
> runtime interactions (not `interactivity`). One concept, one section.

---

## Anti-patterns

| Anti-pattern | Why it's wrong |
|---|---|
| A CSS/visual value anywhere | The contract is no longer implementation-independent |
| Treating the Figma demo state as the spec | The contract is authoritative; record divergences |
| Missing `Extra Props` on a data-driven component | The implementer can't wire real data |
| A lean display component padded with `states`/`emits`/`accessibility` | False precision; describes behavior that doesn't exist |
| A nested component duplicated instead of cross-referenced | Drift between the two definitions |
| Omitting `notes`/authority statement | Loses the contract's standing vs. Figma |
