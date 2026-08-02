---
name: Manage Uniform entries and content types
description: >-
  Model content types and create/update entries in a Uniform project via the
  Platform API, with locale and publishing-state awareness.
api: openapi/uniform-platform-api-openapi.json
operations:
  - GET /api/v1/content-types
  - PUT /api/v1/content-types
  - GET /api/v1/entries
  - PUT /api/v1/entries
  - DELETE /api/v1/entries
  - GET /api/v1/entries-history
  - GET /api/v1/locales
generated: '2026-07-21'
method: generated
---

# Manage Uniform entries and content types

Operations are method + path (the spec declares no operationIds). Base URL
`https://uniform.app` (US) or `https://eu.uniform.app` (EU); auth via
`x-api-key` header or bearer token with the Developer role.

## Steps

1. **Read the content model first**: `GET /api/v1/content-types?projectId=…`.
   Re-read before making changes if edits may have happened in the dashboard.
2. **Check configured locales** with `GET /api/v1/locales` before writing
   localized fields; filter entry reads with the `locale` parameter.
3. **List/search entries** with `GET /api/v1/entries` — supports
   `offset`/`limit`, `orderBy`, `keyword`/`search`, `type` (content type id),
   `state` (0 = draft, 64 = published), and `slug` lookup.
4. **Upsert entries** with `PUT /api/v1/entries` (idempotent by entry id).
   Match the field shape of the entry's content type exactly.
5. **Track history** with `GET /api/v1/entries-history`; use `versionId` on
   `GET /api/v1/entries` to fetch a historical version.
6. **Delete** with `DELETE /api/v1/entries` only on explicit instruction.

## Error rules

- `400` payload does not match the content type — re-read the type and fix.
- `403` role lacks access to the project; `429` back off with jitter.
- No Idempotency-Key contract exists; PUT upserts are the safe retry path.
