# Limitations — What Does Not Belong in a Contract

Contracts are **behavioral**. The items below belong elsewhere. Including them pollutes the
contract and misleads the coding agent that consumes it.

---

## Absolute exclusions

These never appear anywhere in a contract:

| Category | Examples | Where it belongs |
|---|---|---|
| Dimensions | width, height, min/max sizes | Design tokens / Figma |
| Colors | hex, rgba, color CSS variables | Design tokens |
| Typography | font family, size, weight, line height, letter spacing | Design tokens |
| Spacing | padding, margin, gap values | Design tokens |
| Border radii | `rounded-full`, `border-radius: 12px` | Design tokens / CSS |
| Shadows / opacity | box-shadow, `opacity: 0.3` | Design tokens / CSS |
| CSS class names | Tailwind utilities, BEM classes | Implementation files |
| Animation durations / easing | `transition: 200ms` | Motion tokens / implementation |
| Z-index | `z-index: 100` | Layering system |
| Asset URLs | `https://…/asset/…` | Asset manifest / Extra Props (as a data input, not a value) |
| Breakpoint pixel values | `768px`, `1024px` | Design tokens / theme |

The exception is a **token name** (e.g. an icon token) when it is behaviorally relevant —
the name, never the resolved value.

---

## Behavioral-looking details that are still excluded

| Item | Why excluded |
|---|---|
| "thumb is a 16px circle" | Structural/visual. The behavior is "thumb position communicates value." |
| "hover changes background to eggplant/600" | Color is visual. The behavior is "hovered is a distinct state." |
| "disabled reduces opacity" | Opacity is visual. The behavior is "disabled suppresses all interaction." |
| "focus ring appears on keyboard focus" | That a focus indicator is required goes in `accessibility`; its style does not. |
| "blur radius 100px on the glow" | Visual. The behavior is "a decorative glow whose color derives from the thumbnail's dominant color." |

**The test:** could the sentence appear unchanged in a behavioral unit-test assertion?
"onChange fires with `checked: true` on click" → keep. "The track is `#515269` on hover"
→ drop.

---

## The contract is authoritative over Figma

When the behavioral prompt and the Figma file disagree, **the contract wins** — it describes
the intended behavior, not Figma's current demo state.

- Encode the prompt's behavior, and record the divergence in a `figma_note` (on the relevant
  prop / `Extra Props`) and in `notes`.
- Example: a Figma demo always shows an image and toggles only a button, but the prompt makes
  **both** the image and the button independent toggles. The contract reflects the prompt and
  flags the difference. Figma is reference, not law.

---

## Static-mock components (charts)

Some components — charts especially — are represented in Figma as a **static approximation**
of a live, data-driven component. For these:

- **Screenshots are the source of truth** for look and proportion.
- Baked sample values (bar heights, curve shapes, axis ticks, segment widths) are
  **illustrative**; the real values are data-driven.
- Treat dimensions/measurements as approximate; prefer screenshot proportions over literal
  pixel numbers when they conflict.
- Capture all of this in `figma_design_notes`, and mark provisional prop sets with a `_note`.

Even here, the contract still carries **no visual values** — `figma_design_notes` describes
*what is unreliable in Figma*, not colors or sizes to implement.
