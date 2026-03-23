# Dependencies

External systems this project depends on to create, read, and validate
component contracts.

---

## Figma desktop application

**Required for:** Extracting component data via the MCP server.

The Figma desktop app must be running and have the target design file open as
the active tab. The MCP server only serves data for the currently active file.

- Download: https://www.figma.com/downloads/
- Minimum version: any version with Dev Mode enabled
- Dev Mode must be active in the file being inspected

---

## Figma Dev Mode MCP server

**Required for:** All agent-based contract extraction.

The MCP server runs as a local HTTP server embedded in the Figma desktop app.

| Property | Value |
|---|---|
| Protocol | HTTP with Server-Sent Events (SSE) |
| Base URL | `http://127.0.0.1:3845/mcp` |
| Transport | JSON-RPC 2.0 over HTTP POST |
| Required headers | `Content-Type: application/json`, `Accept: application/json, text/event-stream` |
| Session management | `mcp-session-id` response header; must be included in all subsequent requests |

The server is unavailable when:
- Figma desktop is not running
- No file is open in the active tab
- Dev Mode is not enabled in the current file

Reference: https://developers.figma.com/docs/figma-mcp-server/

---

## BaseCore design system

**Required for:** Understanding the component vocabulary and naming conventions
this project's contracts are drawn from.

- Design file key: `UZ1o5wMJ98CHMBMiwvBJlc`
- Component canvas: node `0:2` ("Internal Only Canvas")
- Switch component set: node `101:78`

Components documented so far:

| Component | Node ID | Contract |
|---|---|---|
| Switch | `101:78` | `contract.json` / `contract.yml` |

---

## Node.js (optional, for tooling)

**Required for:** Future schema validation and contract linting.

- Minimum version: 18.x
- Package manager: npm or pnpm
- No runtime dependencies currently installed

---

## JSON Schema Draft 7

**Required for:** `schema.json` validation (not yet implemented).

The schema file in this directory will follow JSON Schema Draft 7:
`http://json-schema.org/draft-07/schema#`

Validation tools that support this draft:
- `ajv` (Node.js)
- `jsonschema` (Python)
- VS Code built-in JSON validation

---

## Not required

| Tool | Why not required |
|---|---|
| React / any framework | Contracts are framework-agnostic |
| Storybook | Contracts are not stories |
| TypeScript | Contracts are data files, not code |
| Any Figma plugin | The MCP server replaces plugin-based extraction |
