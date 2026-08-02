---
name: Stage and ship changes with Uniform releases
description: >-
  Group content changes into a Uniform release, inspect its contents, and
  schedule/ship it via the Platform API.
api: openapi/uniform-platform-api-openapi.json
operations:
  - GET /api/v1/releases
  - PUT /api/v1/releases
  - PATCH /api/v1/releases
  - DELETE /api/v1/releases
  - GET /api/v1/release-contents
  - DELETE /api/v1/release-contents
  - GET /api/v1/entity-releases
generated: '2026-07-21'
method: generated
---

# Stage and ship changes with Uniform releases

Operations are method + path (no operationIds in the spec). Base URL
`https://uniform.app` (US) or `https://eu.uniform.app` (EU); auth via
`x-api-key` header or bearer with the Developer role; scope with `projectId`.

## Steps

1. **List releases**: `GET /api/v1/releases?projectId=…` (offset/limit
   pagination) to find open releases before creating a new one.
2. **Create or update a release** with `PUT /api/v1/releases`; adjust
   metadata/schedule with `PATCH /api/v1/releases`.
3. **Target content at the release**: entry, asset, and composition reads
   accept a `releaseId` parameter — pass it when reading or writing so
   changes land in the release rather than the live state.
4. **Review what's pending** with `GET /api/v1/release-contents`; remove an
   item with `DELETE /api/v1/release-contents`.
5. **Check which releases touch an entity** with `GET /api/v1/entity-releases`.
6. **Webhooks fire during publishing** (Svix-signed) — consumers can react to
   composition/manifest events; see `asyncapi/uniform-webhooks.yml`.

## Error rules

- `409` on release writes = concurrent modification: re-read, merge, retry.
- `429` back off with jitter; `403` stop, the role lacks access.
- No Idempotency-Key contract; PUT/PATCH by release id are the retry path.
