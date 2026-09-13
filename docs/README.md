# TTBM Documentation

TTBM = **TikTok Broadcast Manager** (working title).

This directory is the planning and implementation-preflight source of truth for the project.

## Reading order

1. [`PRODUCT_REQUIREMENTS.md`](./PRODUCT_REQUIREMENTS.md) — what the product is, who it serves, MVP scope and non-goals.
2. [`USE_CASES.md`](./USE_CASES.md) — user/system behavior and end-to-end flows.
3. [`TIKFINITY_EVENT_CONTRACT.md`](./TIKFINITY_EVENT_CONTRACT.md) — source boundary, normalized event contract, payload-capture checklist.
4. [`DOMAIN_DATA_MODEL.md`](./DOMAIN_DATA_MODEL.md) — entities, relationships, invariants, deletion and retention rules.
5. [`STATE_AND_RULES.md`](./STATE_AND_RULES.md) — session, connection, gift streak and post-LIVE task state machines.
6. [`SCREEN_SPEC.md`](./SCREEN_SPEC.md) — UI information architecture and screen behavior.
7. [`ARCHITECTURE.md`](./ARCHITECTURE.md) — application modules, dependency direction and local-first runtime.
8. [`ERROR_RECOVERY.md`](./ERROR_RECOVERY.md) — disconnects, malformed events, DB failure, backup and restore behavior.
9. [`LOCAL_DATA_PRIVACY.md`](./LOCAL_DATA_PRIVACY.md) — local data policy, sensitive notes, export/delete expectations.
10. [`TEST_ACCEPTANCE.md`](./TEST_ACCEPTANCE.md) — acceptance criteria and executable test scenarios.
11. [`RELEASE_SCOPE.md`](./RELEASE_SCOPE.md) — v0.1 boundary, gates and deferred features.

## Authority order

If documents disagree, use the following precedence until an explicit decision updates the lower document:

1. Source/runtime evidence captured from the current TikFinity Desktop Event API.
2. `TIKFINITY_EVENT_CONTRACT.md`.
3. `STATE_AND_RULES.md` and `DOMAIN_DATA_MODEL.md`.
4. `USE_CASES.md`.
5. `SCREEN_SPEC.md`.
6. `PRODUCT_REQUIREMENTS.md`.

No UI mockup may override a data-source limitation.

## Current technical facts

As verified during planning on 2026-09-14:

- TikFinity documents a local WebSocket Event API at `ws://localhost:21213/` and lists `chat`, `gift`, `share`, `follow`, `like`, `roomUser`, and `subscribe` event envelopes.
- TikFinity states that the Desktop App must run on the same computer as the consumer.
- TikFinity points payload consumers to TikTok-Live-Connector-compatible event structures; TTBM must still capture real payloads before freezing field mappings.
- Tauri 2's official SQL plugin supports SQLite and desktop/mobile platforms. TTBM v0.1 remains Windows-first.

## Change discipline

Any implementation change that alters one of these items must update the corresponding doc in the same PR:

- canonical viewer identity
- gift aggregation semantics
- session boundaries
- task generation/deduplication
- data deletion behavior
- normalized event schema
- persistent schema
- release scope
