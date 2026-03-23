# Limitations — What Does Not Belong in a Contract

Contracts are behavioral. The items below belong in other files.
Including them in a contract pollutes it and breaks tool consumers that
rely on contracts for logic generation, not styling.

---

## Absolute exclusions

These never appear anywhere in a contract file:

| Category | Examples | Where it belongs |
|---|---|---|
| Dimensions | width, height, min/max sizes | Design tokens / Figma |
| Colors | hex values, rgba, CSS variables for color | Design tokens |
| Typography | font family, font size, font weight, line height, letter spacing | Design tokens |
| Spacing | padding, margin, gap values | Design tokens |
| Border radii | `rounded-full`, `border-radius: 12px` | Design tokens / CSS |
| Shadows | box-shadow values | Design tokens / CSS |
| Opacity values | `opacity: 0.02`, `rgba(…, 0.5)` | Design tokens / CSS |
| CSS class names | Tailwind utilities, BEM classes | Implementation files |
| Animation durations | `transition: 200ms` | Implementation / motion tokens |
| Z-index values | `z-index: 100` | Implementation / layering system |
| Asset URLs | `http://localhost:3845/assets/…` | Asset manifest |
| Breakpoint pixel values | `768px`, `1024px` | Design tokens / theme |

---

## Behavioral-looking details that are still excluded

These feel behavioral but belong elsewhere:

| Item | Reason for exclusion |
|---|---|
| "thumb is 16px circle" | That is structural, not behavioral. The behavior is "thumb moves to indicate value." |
| "track is pill-shaped" | Visual description. The behavior is "container holds the thumb." |
| "hover changes background to eggplant/600" | Color change is visual. The behavior is "hovered state is distinct from rest." |
| "disabled reduces opacity" | Opacity is visual. The behavior is "disabled state suppresses all interaction." |
| "focus ring appears on keyboard focus" | The *requirement* that a focus ring exists is in accessibility. Its style is not. |
| "transition easing curve" | Motion design. The behavior is "thumb moves when value changes." |
| "drop shadow on thumb when hovered" | Visual. No behavioral consequence. |

---

## What is borderline but acceptable

These are acceptable when they describe *behavioral* distinctions, not
visual details:

| Item | Why it is acceptable |
|---|---|
| "thumb position communicates value" | Position as a behavioral signal, not a pixel coordinate |
| "border appears on hover in the off state" | If the border change is the *only* way to distinguish a state, it is a behavioral boundary marker. Write it as "a visual affordance distinguishes hovered-off from rest-off" without specifying the border property. |
| "items-end alignment in disabled state" | Acceptable *only* if the misalignment is intentional and communicates something. Otherwise exclude. |

When in doubt: if removing the detail would make a developer unable to determine
*what the component does*, keep it. If removing it only affects how the component
looks, drop it.

---

## A useful test

Ask: "Could this sentence appear unchanged in a unit test assertion about
behavior?"

- "onChange fires with `checked: true` when the user clicks" → YES, keep it
- "The track background is `#515269` on hover" → NO, remove it
- "The thumb moves to the right when checked is true" → YES, keep it
- "The thumb is 16×16 pixels" → NO, remove it
