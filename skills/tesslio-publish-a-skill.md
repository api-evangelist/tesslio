---
name: Author and publish a Tessl skill
description: Scaffold, lint, quality-review, and publish an Agent Skill to the Tessl registry, then verify it resolves.
api: openapi/tesslio-openapi-original.json
operations:
  - "POST /v1/tiles"
  - "GET /v1/tiles/{workspaceName}/{tileName}"
  - "GET /v1/tiles/{workspaceName}/{tileName}/versions/{version}"
---

# Author and publish a Tessl skill

Use this skill to take a new Agent Skill from scaffold to a published registry version.

## Prerequisites
- Tessl CLI installed (`curl -fsSL https://get.tessl.io | sh`).
- Authenticated: run `tessl login` (OAuth device flow) or export `TESSL_TOKEN` (a workspace API key) for CI.
- A workspace to publish under (`tessl workspace list`).

## Steps
1. **Scaffold** the skill: `tessl skill new --name my-skill --description "When to trigger it" --workspace <ws>`. This creates an Agent Skills-spec directory with a `SKILL.md`.
2. **Lint** structure and frontmatter: `tessl skill lint ./my-skill` (validates against agentskills.io/specification).
3. **Quality-review** and optionally auto-improve: `tessl review run ./my-skill --workspace <ws>` (server-side, polls to completion). Gate in CI with `--json --threshold 80`.
4. **Security-scan** (Snyk): `tessl review run security ./my-skill --workspace <ws> --fail-on high`.
5. **Publish**: `tessl skill publish --workspace <ws> [--public]`. This bundles the skill into a plugin and issues `POST /v1/tiles` to publish a tile/plugin version.
6. **Verify** it resolved: `tessl plugin info <ws>/my-skill`, or read it back via `GET /v1/tiles/{workspaceName}/{tileName}` and `GET /v1/tiles/{workspaceName}/{tileName}/versions/{version}`.

## Rules and conventions
- Auth: bearer token / `TESSL_TOKEN` in the `Authorization` header (see authentication/tesslio-authentication.yml).
- Publishing an existing version returns **409 Conflict** — use `--bump patch|minor|major`.
- Unpublish is only allowed within 2 days; otherwise `tessl plugin archive` to block new installs.
- Errors are plain JSON (not RFC 9457); handle 401/403/409/422 per errors/tesslio-problem-types.yml.
