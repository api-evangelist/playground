---
name: playground-chaos-simulation
description: >-
  Exercise a frontend's loading, error, and rate-limit states on demand by
  triggering Playground API's built-in latency / status / rate-limit simulation.
api: Playground REST API
base_url: https://playground.nileslabs.com/api/v1
method: generated
source: https://playground.nileslabs.com/llms-full.txt (Network Simulation Capabilities)
operations:
  - GET /api/v1/posts
  - GET /api/v1/users
---

# Network / chaos simulation on Playground API

Any endpoint accepts simulation controls, as a query param OR a header, so you
can drive UI edge cases without touching your application code.

## Controls
- **Latency** — `?_delay=1500` or `X-Simulate-Delay: 1500` (1-5000 ms). Test
  spinners, skeletons, and timeout handling.
- **HTTP status** — `?_status=500` or `X-Simulate-Status: 500` (any 400-599).
  Test error boundaries and retry logic.
- **Rate limit** — `?_ratelimit=true` or `X-Simulate-RateLimit: true`. Returns
  `429 Too Many Requests` with a `Retry-After` header. Test backoff.

## Steps
1. **Slow response** — `GET /api/v1/posts?_delay=3000`; confirm your loading UI
   holds for 3s then renders.
2. **Server error** — `GET /api/v1/users?_status=503`; confirm your error state
   and any retry kick in.
3. **Throttled** — `GET /api/v1/posts?_ratelimit=true`; read `Retry-After` and
   confirm your client waits before retrying.
4. Combine with real queries: `GET /api/v1/posts?user_id=1&_delay=1000` still
   returns real filtered data, just late.

## Notes
- Simulation is opt-in per request; remove the param/header for a normal response.
- Errors use the `{ "error": ..., "message": ... }` envelope.
