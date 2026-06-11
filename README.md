# Component Contract System

A **behavioral contract** captures every event-, interaction-, and resize-driven behavior a
component can exhibit — and nothing visual. It sits between a Figma design and a code
implementation: both humans and AI agents read it before writing a single line, so that
ambiguity in the design does not become hallucinated logic in the code.

A contract is produced by merging **three inputs** — a behavioral prompt, the Figma Console
MCP, and the Figma MCP — into one **unified JSON file** per component that a coding agent
(such as Claude Code) can implement from.

---

## How a contract is authored

```
Behavioral prompt (intent, authority)
        +
Figma Console MCP  (live structure: property defs, variants, overrides, reactions)
Figma MCP (URL)    (structure + reference from a shared link)
        │   merge — the prompt wins conflicts; Figma supplies structure, never pixels
        ▼
components/<category>/<name>.json   ← unified behavioral contract (JSON only)
        │   implementation
        ▼
Component code, using the target codebase's own design tokens
```

The work is driven by a **skill** — a semi-strict rule book the agent loads when creating or
updating a contract. The skill, a hierarchical template, and a section policy replace any
machine validation: the consumer is an LLM, so contracts are authored, not validated.

---

## Project structure

```
.
├── agent.manifest.yml                      Entry point — read first on a fresh clone
├── README.md                               This file
├── config.yml                              Runtime config — MCP servers, codebase path, output
├── .env.example                            Secret template — copy to .env
├── schema.json                             Non-enforcing stub — the template + skill are the real guideline
│
├── .claude/skills/
│   ├── component-contract/SKILL.md         ★ The authoring rule book / pipeline driver
│   └── component-contract-builder/         ★ Portable, self-contained variant (SKILL.md + BLOCKS.md)
│
├── templates/
│   └── contract.template.json              ★ Hierarchical, annotated JSON template (copy → strip _keys → fill)
│
├── components/                             Your contracts live here — one JSON per component, by category
│   └── <category>/<name>.json
│
└── docs/
    ├── agents.md                           The dual-MCP pipeline + extract workflow
    ├── context.md                          Philosophy and design decisions
    ├── glossary.md                         Every contract section defined, with its tier
    ├── requirements.md                     The section policy (required / recommended / optional)
    ├── limitations.md                      What never belongs in a contract
    └── dependencies.md                     The MCP tooling this project uses
```

---

## The contract format (adaptive)

A stable **required core** is always present; everything else is included **only when the
component earns it**.

- **Always required:** `kind`, `name`, `version`, `figma`, `description`, `props` (incl.
  `Extra Props`), `composition` (the single structural section), `constraints`,
  `scenarios`, `notes`.
- **Recommended:** `intent`, `responsive`.
- **Optional, by context:** `category`, `figma_design_notes`, `modes`, `variants`, `states`,
  `state_machine`, `triggers`, `emits`, `accessibility`.

One concept, one section — no redundant pairs: `composition` is the only structural section
(no `anatomy`); `state_machine` holds all transitions and named operations (no separate
`transitions`/`interactions`); `triggers` covers all runtime interactions (no
`interactivity`).

A display-only card stays lean; a stateful or interactive component pulls in the richer
sections. See `docs/requirements.md` for the full policy and `docs/glossary.md` for
definitions.

---

## Getting started

```bash
git clone <this-repo>
cd Contracts

# 1. Secrets and local paths
cp .env.example .env          # FIGMA_PAT (if using the URL-based Figma MCP), CODEBASE_PATH

# 2. Connect Figma
#    - Figma Console MCP: open the file in the Figma desktop app + run the Desktop Bridge plugin
#    - Figma MCP (official): just share/paste a node URL — no desktop plugin needed

# 3. Author a contract
#    In Claude Code, ask to create/recheck a contract — the `component-contract` skill drives it.
#    Or copy templates/contract.template.json, strip the _annotation keys, and fill it in.
```

---

## Writing or updating a contract

1. Resolve the Figma source (a node URL, or a live selection via the Desktop Bridge).
2. Pull structure: variant matrix, component property definitions, nested components,
   overrides (Console MCP preferred; Figma MCP as fallback/cross-check).
3. Merge with the behavioral prompt — the prompt defines behavior and what is required vs
   optional; conflicts resolve in the prompt's favor.
4. Choose sections per the policy; fill `templates/contract.template.json`.
5. Record divergences from Figma in a `figma_note`; put non-Figma data inputs under
   `Extra Props`.
6. Write `components/<category>/<name>.json` and confirm it parses.

Full protocol: `docs/agents.md`. Rule book: `.claude/skills/component-contract/SKILL.md`.

---

## Design principles

- **Behavioral, not visual** — no colors, sizes, or typography anywhere.
- **Contract over Figma** — the contract is authoritative; Figma is reference.
- **Lean by default** — optional sections only when the behavior demands them.
- **Explicit over implicit** — if it is not in the contract, it does not exist.
- **LLM-first** — authored for an AI consumer; no hidden logic, no machine validator.
- **JSON only** — one contract file per component.
