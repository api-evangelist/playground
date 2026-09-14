---
name: playground-parallel-e2e
description: >-
  Run isolated, parallel end-to-end tests (Playwright/Cypress) against Playground
  API without database collisions, using per-runner session identities.
api: Playground REST API
base_url: https://playground.nileslabs.com/api/v1
method: generated
source: https://playground.nileslabs.com/llms-full.txt (Code Integration Recipes)
operations:
  - POST /api/v1/posts
  - GET /api/v1/posts
  - DELETE /api/v1/session/reset
---

# Parallel E2E testing on Playground API

Every session gets a private mutation overlay, so many test runners can create
and mutate data at once without stepping on each other or on the shared baseline.

## The one rule
Give each parallel worker a **unique, stable** `X-Playground-Identity` value and
send it on every request that worker makes. Same identity = same overlay.

## Steps
1. At the start of each worker/spec, mint an id: `const sessionId = 'runner-' + Date.now() + '-' + workerIndex`.
2. Attach it to every request: header `X-Playground-Identity: <sessionId>`.
3. **Arrange** — `POST /api/v1/posts` (or any resource) to seed the state this
   test needs. Assert the `201`/`200` and capture the `local-<uuid>` id.
4. **Act / Assert** — `GET /api/v1/posts` with the same identity header; your
   seeded records are present and isolated from other workers.
5. **Teardown** — `DELETE /api/v1/session/reset` with the worker's identity to
   purge its overlay, or just let it expire (10-day idle TTL).

## Notes
- Snapshot/restore a known fixture state with `GET /api/v1/session/export` and
  `POST /api/v1/session/import`.
- Seed a whole domain in one call: `POST /api/v1/custom/seed` with template
  `ecommerce | saas | blog | crm`.
- No auth is needed; the identity header is isolation, not security.
