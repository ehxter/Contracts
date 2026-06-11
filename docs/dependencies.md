# Dependencies

External systems used to author and consume component contracts. There is **no build step
and no validation layer** — contracts are plain JSON read by humans and LLMs.

---

## Behavioral prompt (the user)

The primary input. The user's description of what the component is for and how it behaves is
the authority on intent. No tooling required; it drives every other source.

---

## Figma Console MCP — `figma-console` (Desktop Bridge)

**Used for:** rich, live extraction when the Figma desktop app is open.

A WebSocket bridge to the Figma desktop app via the **Desktop Bridge plugin**. It exposes
the live plugin runtime, so it can read component property definitions, the variant matrix,
instance overrides, reactions, and bound variables — and capture plugin-fresh screenshots.

| Property | Value |
|---|---|
| Transport | WebSocket to the desktop app (local) |
| Requires | Figma desktop running; Desktop Bridge plugin open in the target file |
| Connect check | `figma_get_status` with `probe: true` |
| Recover | `figma_reconnect`; or open Plugins → Development → Figma Desktop Bridge |

Key tools: `figma_get_status`, `figma_get_selection`, `figma_list_open_files`,
`figma_navigate`, `figma_get_component_for_development`, `figma_capture_screenshot`,
`figma_get_variables` / `figma_get_token_values`.

**Prefer this when connected.** If it isn't connected, fall back to the Figma MCP.

---

## Figma MCP — official (URL-based)

**Used for:** reading a node directly from a Figma URL, with no desktop plugin — shared
links, headless work, or cross-checking the Console MCP.

| Property | Value |
|---|---|
| Input | Figma file URL (`fileKey` + `node-id`) |
| Requires | Access to the file; no desktop app |
| URL parsing | `node-id=1-2` → `nodeId: 1:2`; `fileKey` is the segment after `/design/` |

Key tools: `get_metadata` (structure overview), `get_design_context` (reference code +
tokens + reactions), `get_screenshot` (visual confirmation), `get_variable_defs` (token
names).

---

## The authoring skill

**Used for:** driving the pipeline. `.claude/skills/component-contract/SKILL.md` is the
semi-strict rule book a Claude Code agent loads when creating/updating a contract. It
references the template and the docs in this folder.

---

## Target codebase (for the consuming agent)

**Used for:** implementing a component from a contract (downstream of this repo). The
coding agent reads the target repo to discover framework, styling system, and design tokens,
then implements the contract's behavior using **the codebase's** tokens — never values from
the contract (which has none). Set `CODEBASE_PATH` in `.env`; see `config.yml`.

---

## Not required

| Tool | Why not |
|---|---|
| JSON Schema validator | No machine validation — the LLM consumes unvalidated JSON; the template + skill are the guideline |
| YAML | Contracts are JSON only |
| The deprecated Figma Dev Mode MCP (`:3845`) / REST API | Replaced by the two MCP servers above |
| Any framework / Storybook / TypeScript | Contracts are framework-agnostic data files |
