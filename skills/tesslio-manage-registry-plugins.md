---
name: Search and install plugins from the Tessl registry
description: Discover, match, and install Tessl registry plugins into a repository for AI coding agents, via the MCP server or the API.
api: openapi/tesslio-openapi-original.json
operations:
  - "GET /v1/tiles"
  - "POST /v1/tiles/match"
  - "GET /v1/tiles/{workspaceName}/{tileName}"
  - "GET /v1/users/me"
---

# Search and install plugins from the Tessl registry

Use this skill to give a coding agent the right packaged context (plugins/skills) for a repository.

## Path A — via the Tessl MCP server (recommended for agents)
1. Ensure the registry MCP server is wired: `tessl init --agent <claude-code|cursor|...>` (configures `tessl mcp start`, stdio).
2. Call the `search` MCP tool with a `query` (plugin name, PURL, or npm/PyPI URL).
3. Call the `install` MCP tool with `packageName` = `workspace/plugin[@version]`; omit it to sync all plugins from `tessl.json`.
4. Call the `status` MCP tool to confirm plugins are up-to-date (returns auth + manifest summary).

## Path B — via the API directly
1. Confirm identity: `GET /v1/users/me` with the `Authorization` bearer token.
2. List available tiles/plugins: `GET /v1/tiles` (supports `cursor` + `count` paging and `filter[field][op]` query grammar).
3. Match candidates for a context need: `POST /v1/tiles/match`.
4. Read a specific one: `GET /v1/tiles/{workspaceName}/{tileName}` (and its `/versions` for a pinned version).

## Rules and conventions
- Auth: bearer token / `TESSL_TOKEN` in the `Authorization` header.
- Pagination is cursor-based (`cursor`, `count`); filtering uses `filter[field][op]=value` with ops eq/ne/gt/gte/lt/lte/like.
- Install policy warnings (e.g. high/critical Snyk findings) never hard-block installs but prompt; pre-accept with the CLI `--accept-warnings`.
- Handle 404 (unknown tile), 410 (archived/unpublished), 429 (rate limit) per errors/tesslio-problem-types.yml.
