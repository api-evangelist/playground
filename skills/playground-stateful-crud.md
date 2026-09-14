---
name: playground-stateful-crud
description: >-
  Create a record and prove it persists across subsequent reads in the same
  Playground API session — the core differentiator over static mock APIs.
api: Playground REST API
base_url: https://playground.nileslabs.com/api/v1
method: generated
source: openapi/playground-openapi.json + https://playground.nileslabs.com/llms-full.txt
operations:
  - POST /api/v1/posts
  - GET /api/v1/posts
  - GET /api/v1/posts/{id}
  - PATCH /api/v1/posts/{id}
  - DELETE /api/v1/posts/{id}
  - DELETE /api/v1/session/reset
---

# Stateful CRUD on Playground API

Playground keeps your mutations in a private per-session overlay laid over an
immutable global baseline. Unlike JSONPlaceholder, a `POST` you make shows up in
your later `GET`s — but only for your session.

## Session identity
- Browser: send `credentials: 'include'` (fetch) / `withCredentials: true` (Axios).
- Node / Playwright / cURL: send header `X-Playground-Identity: <your-session-id>`.

## Steps
1. **Create** — `POST /api/v1/posts` with `{ "user_id": 1, "title": "...", "body": "..." }`.
   The response has a `local-<uuid>` `id` and `"_sandbox": "created"`.
2. **List** — `GET /api/v1/posts`. Your new post appears at the top of `data[]`
   (newest overlay records first); the response also carries a `pagination` object.
3. **Read one** — `GET /api/v1/posts/{id}` using the returned `local-<uuid>`.
4. **Patch** — `PATCH /api/v1/posts/{id}` with only the fields to change; response
   shows `"_sandbox": "updated"`.
5. **Delete** — `DELETE /api/v1/posts/{id}` returns `204 No Content`. It is removed
   from your view only; the global baseline is untouched.
6. **Undo everything** — `DELETE /api/v1/session/reset` purges the whole overlay
   and returns you to clean baseline data.

## Conventions to respect
- Pagination: `?page=1&limit=10` (max `limit` 200). Sorting: `?_sort=title&_order=desc`.
- Search: `?q=term`. Filter posts by author: `?user_id=1`.
- Overlay cap: 30 created records per resource; overlays expire after 10 days idle.
- Errors return `{ "error": ..., "message": ... }` (not RFC 9457).
