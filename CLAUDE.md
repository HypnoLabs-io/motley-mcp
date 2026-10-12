# CLAUDE.md

Guidance for Claude Code and other AI agents working in this repository.

@AGENTS.md

## What this repo is

`motley-mcp` is the **public catalog-metadata home** for the Motley MCP server. Motley
exposes live NYC family-data (neighborhoods, schools, admissions, safety, resources,
kids' activities) as MCP tools; this repository is the metadata that lets Motley be
listed in MCP registries and catalogs (the Official MCP Registry, Smithery, MCP.so, etc.).

It is **metadata only** — there is no server code here. The running server lives at
`https://mcp.motley.nyc/mcp` (Streamable HTTP, no auth) and is deployed separately.

## Files (this is the whole repo)

- `server.json` — Official MCP Registry record. Registry name `nyc.motley/mcp`. Keep it identical to the published record, and bump `version` when the server's tool surface changes.
- `.well-known/mcp/server-card.json` — the server card: server info, every tool's input and output schema, transports, and auth. Also intended to be served at `https://motley.nyc/.well-known/mcp/server-card.json`. Keep it in sync with the live server's `tools/list`.
- `mcp-config.json` — copy-paste client config (stdio bridge via `mcp-remote`).
- `README.md` — the public description, tool list, and connection instructions.

## Hard rules — this repository is PUBLIC

Everything committed here is world-readable and persists in git history. Do **not** add:

- **Personal data / PII** — no personal emails, names, addresses, or phone numbers. Any contact info must be a role or support address, never a personal one.
- **Internal working notes** — no submission trackers, ops logs, "attempted X / pending" status files, or anything describing a local environment or workflow. Those belong in a private note, not here.
- **Local machine details** — no absolute home paths (`/home/<user>/...`), usernames, or machine-specific commands. Invoke tools resolved from `PATH`.
- **Secrets** — no tokens, API keys, `.env` values, or credentials, ever.

Commit with your GitHub **noreply** email, not a personal address (this clone is already
configured that way). If PII or a secret ever lands in history, a normal edit is not
enough — history must be rewritten and force-pushed.

## Editing

- Changing the tool surface: update `.well-known/mcp/server-card.json` to match the live server, keep the README tool list in sync, and bump `version` in `server.json`.
- Publishing to the Official MCP Registry uses `mcp-publisher` (`validate` → `login` → `publish server.json`), invoked from a binary on your `PATH` — do not hardcode its location. The `nyc.motley` namespace authenticates by domain (`login http --domain motley.nyc`), never with a key committed here.
- Keep the README accurate to what the server actually exposes; do not invent tools or fields.
