---
name: Manage Uniform compositions
description: >-
  Read, upsert, and audit compositions (Canvas) and their component
  definitions in a Uniform project via the Platform API.
api: openapi/uniform-platform-api-openapi.json
operations:
  - GET /api/v1/canvas
  - PUT /api/v1/canvas
  - DELETE /api/v1/canvas
  - GET /api/v1/canvas-definitions
  - PUT /api/v1/canvas-definitions
  - GET /api/v1/canvas-history
generated: '2026-07-21'
method: generated
---

# Manage Uniform compositions

The Uniform Platform API has **no operationIds** — operations are identified
by method + path. Base URL is `https://uniform.app` (US) or
`https://eu.uniform.app` (EU); all paths already include `/api/v1`.

## Auth

Send a service-account key or personal access token in the `x-api-key`
header (or as a bearer token). The credential needs the Developer role on the
target project. See `authentication/uniform-authentication.yml`.

## Steps

1. **Scope every call to a project.** Nearly all operations require the
   `projectId` query parameter.
2. **Read component definitions first** — `GET /api/v1/canvas-definitions`
   returns the component types a composition may use. Never invent component
   types.
3. **Fetch compositions** with `GET /api/v1/canvas`. List calls paginate with
   `offset`/`limit`; pass `withTotalCount=true` for a total.
4. **Upsert a composition** with `PUT /api/v1/canvas` (create and update are
   the same upsert call, idempotent by composition id). A `409` means a
   conflicting concurrent change — re-read, merge, retry.
5. **Audit changes** with `GET /api/v1/canvas-history`.
6. **Delete** with `DELETE /api/v1/canvas` only on explicit instruction.

## Error rules

- `401` re-check the `x-api-key` credential and region host.
- `403` the key's role lacks project access — do not retry.
- `429` rate limited: back off and retry with jitter.
- There is **no Idempotency-Key contract**; only PUT upserts are safely
  retryable. Do not blind-retry POSTs.
