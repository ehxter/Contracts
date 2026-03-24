# Component Contract System

A behavioral contract captures every event-driven, interaction-driven, and
resize-driven behavior a component can exhibit — nothing more, nothing less.

It sits between a Figma design and a code implementation. Both humans and AI
agents read it before writing a single line. The goal is to reduce ambiguity,
prevent hallucinated logic, and constrain AI agents to implement components
within clearly defined behavioral boundaries.

---

## Getting started

```bash
git clone <this-repo>
cd Contracts

# 1. Configure secrets and local paths
cp .env.example .env
#    set FIGMA_PAT       — your Figma personal access token
#    set FIGMA_FILE_KEY  — key from your Figma file URL (overrides config.yml default)
#    set CODEBASE_PATH   — absolute path to the codebase where components will be built

# 2. Review project config
#    edit config.yml — Figma file key, codebase framework/styling hints, output paths

# 3. Read the agent manifest
#    cat agent.manifest.yml — full operating guide for any AI agent starting cold

# 4. To extract contracts from Figma: open the target file in the Figma desktop app
#    The MCP server starts automatically at http://127.0.0.1:3845/mcp

# 5. To build components from contracts: point CODEBASE_PATH at your target repo
```

---

## Project structure

```
.
├── agent.manifest.yml      Agent entry point — read this first on a fresh clone
├── figma-registry.yml      Component → Figma mapping (paste URLs here, name links to components/)
├── config.yml              Runtime config — MCP URL, API base URL, codebase path
├── .env.example            Secret template — copy to .env and fill in values
├── schema.json             JSON Schema for contract validation (in progress)
│
├── components/             One subdirectory per component
│   ├── switch/
│   │   ├── contract.json   Behavioral contract in JSON
│   │   └── contract.yml    Same contract in YAML
│   └── button/
│       ├── contract.json
│       └── contract.yml
│
└── docs/                   Reference material for humans and agents
    ├── agents.md           MCP protocol, extraction steps, common mistakes
    ├── context.md          Project philosophy and design decisions
    ├── glossary.md         Every contract key defined
    ├── limitations.md      What does NOT belong in a contract
    ├── requirements.md     Validation checklist for a finished contract
    └── dependencies.md     External systems this project depends on
```

---

## What is a component contract?

A contract formally specifies:

- **Props** — behavioral inputs the consumer provides
- **States** — every named UI state the component can occupy
- **State machine** — which transitions between states are valid, and what triggers them
- **Triggers** — every input source: pointer, keyboard, touch, resize, programmatic
- **Interactions** — higher-level named operations (toggle, submit, expand…)
- **Emits** — events fired outward and their payloads
- **Accessibility** — ARIA role, keyboard nav, focus management, screen reader requirements
- **Responsive** — behavioral (not visual) changes at different container sizes
- **Composition** — what the parent must provide; named child slots
- **Constraints** — mutual exclusions, conditional rules, required combinations
- **Scenarios** — end-to-end named behavioral flows

It is not documentation. It is an execution boundary — behavior is undefined
if anything outside the contract is assumed.

---

## Reading a contract

Open `components/<name>/contract.json` or `contract.yml`. Work through the
top-level keys in order: `figma` → `description` → `props` → `states` →
`state_machine` → `triggers` → `interactions` → `emits` → `accessibility` →
`responsive` → `composition` → `constraints` → `scenarios`.

Each key is defined in full in `docs/glossary.md`.

---

## Writing a new contract

1. Open the Figma file in the desktop app with Dev Mode enabled.
2. The MCP server starts automatically at `http://127.0.0.1:3845/mcp`.
3. Call `get_metadata` on the component set node to map all variants.
4. Call `get_design_context` on each variant to extract conditional logic.
5. Create `components/<name>/` and fill `contract.json` key by key.
6. Translate to `contract.yml` — identical data, YAML format.
7. Run through the checklist in `docs/requirements.md` before committing.

For AI-assisted extraction, see `docs/agents.md` for the MCP protocol details.
For building components from contracts, see `agent.manifest.yml` for the full workflow.

---

## Design principles

- Behavioral, not visual — no colors, sizes, or typography anywhere
- Explicit over implicit — every state named, every transition guarded
- Machine-readable first — structured for agent consumption
- No hidden logic — if it is not in the contract, it does not exist
- State clarity over convenience — ambiguous states are always split

---

## Contract sections in depth

| Section | Purpose |
|---|---|
| `props` | What the consumer controls. Only props with behavioral consequences. |
| `states` | Named modes. Always includes `focused` and `pressed` even without Figma variants. |
| `state_machine` | The formal transition graph. Every state must be reachable and escapable. |
| `triggers` | Categorized by input type. All five categories present even if empty. |
| `interactions` | Semantic operations that one or more triggers invoke. |
| `emits` | Outward events with payloads and explicit suppression rules. |
| `accessibility` | WAI-ARIA role, managed attributes, keyboard nav, focus, screen reader. |
| `responsive` | Behavioral changes driven by size — not visual reflow. |
| `composition` | Parent obligations and named child slots. |
| `constraints` | Mutual exclusions, conditional if-then rules, required co-presence. |
| `scenarios` | Prose flows. Minimum four: happy path, keyboard, disabled, controlled update. |

---

## Future extensions

- Schema validation (`schema.json` fully defined)
- Contract linting CLI
- Automated test generation from `state_machine` and `scenarios`
- Contract diffing across versions
- Multi-component composition contracts
