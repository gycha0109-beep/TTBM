# TikFinity Event Contract

## 1. Purpose

This document defines the source boundary between TikFinity Desktop and TTBM.

TTBM must never let raw TikFinity payload shape leak directly into UI/business logic. The adapter converts source payloads into canonical internal events.

---

## 2. Verified source facts

Verified during planning on 2026-09-14 from TikFinity's documented Event API:

- Endpoint: `ws://localhost:21213/`
- TikFinity Desktop must run on the same computer as the consumer.
- Envelope shape:

```json
{
  "event": "chat",
  "data": {}
}
```

- Documented event names include:
  - `chat`
  - `gift`
  - `share`
  - `follow`
  - `like`
  - `roomUser`
  - `subscribe`
- TikFinity points developers to TikTok-Live-Connector-compatible event payload documentation.

Source references:

- https://tikfinity.zerody.one/tiktok/obsdocks
- https://tikfinity.zerody.one/tiktok/dapi

Important: these facts establish the transport/envelope boundary, **not the final TTBM field mapping**.

---

## 3. Required real-payload capture before implementation freeze

Before the persistent schema and mapper are considered stable, capture real events from at least one test LIVE.

Store sanitized fixtures under:

```text
fixtures/tikfinity/
  chat.json
  gift-single.json
  gift-streak-start.json
  gift-streak-update.json
  gift-streak-end.json
  follow.json
  subscribe.json
  like.json
  share.json
  room-user.json
  unknown.json
```

Capture checklist:

### Viewer identity

- stable numeric/string user ID field
- `uniqueId` / handle field
- nickname/display name field
- avatar URL field
- whether any identity field may be absent per event type

### Gift

- gift ID
- gift name
- repeat count
- repeat end flag
- gift type/streak marker
- unit diamond/coin value, if supplied
- timestamp
- source event identifier, if supplied

### Chat

- stable viewer identity
- chat text
- event timestamp
- event ID, if supplied

### Subscribe

- stable viewer identity
- subscription metadata actually present
- event timestamp

### Follow

- stable viewer identity
- event timestamp

### roomUser

- viewer-count field
- top-gifter/list fields if present
- confirm that it is not treated as a guaranteed individual join event

---

## 4. Source adapter contract

The TikFinity adapter owns:

```text
WebSocket connection
→ raw JSON parsing
→ envelope validation
→ source event classification
→ source field extraction
→ canonical event creation
```

It does **not** own:

- fan persistence rules
- gift totals
- MVP ranking
- After Live task creation
- UI strings
- monthly aggregation

---

## 5. Canonical event envelope

Every accepted source event should normalize into a common envelope.

```ts
interface CanonicalEvent<TPayload> {
  id: string;
  source: 'tikfinity';
  sourceEventId?: string;
  type: CanonicalEventType;
  sessionId: string;
  occurredAtUtc: string;
  receivedAtUtc: string;
  viewer?: CanonicalViewerRef;
  payload: TPayload;
  rawPayload: unknown;
  fingerprint: string;
}
```

Canonical event types:

```text
chat
gift
follow
subscribe
like
share
room_snapshot
unknown
```

Unknown events must not crash the app.

---

## 6. Canonical viewer reference

```ts
interface CanonicalViewerRef {
  platform: 'tiktok';
  platformUserId: string;
  uniqueId?: string;
  nickname?: string;
  displayName?: string;
  avatarUrl?: string;
}
```

Invariant:

```text
Durable viewer identity = platform + platformUserId
```

`uniqueId`, nickname and display name are mutable attributes, not identity keys.

If a source event lacks a stable user ID:

- do not create a durable Viewer using nickname alone
- store the event as unresolved if useful
- exclude unresolved identity from fan-specific aggregates that require durable identity

---

## 7. Chat mapping

Canonical payload target:

```ts
interface ChatPayload {
  text: string;
}
```

Requirements:

- preserve raw payload
- resolve viewer before fan timeline update
- repeated identical text is not automatically duplicate; idempotency must use source identity/fingerprint, not message text alone

---

## 8. Gift mapping

Canonical target:

```ts
interface GiftPayload {
  giftId: string;
  giftName?: string;
  repeatCount: number;
  repeatEnd: boolean;
  giftType?: number | string;
  unitValue?: number;
  totalValue?: number;
}
```

### Gift streak invariant

Do not sum every repeated streak update as a new full gift.

Preferred domain behavior:

```text
non-streak gift
→ finalize immediately

streak-capable gift
→ create/update provisional streak state
→ finalize once end signal is confirmed
```

Alternative delta processing is acceptable if mathematically equivalent and idempotent.

The fixture tests must prove that a streak progressing 1 → 2 → 3 ends as total quantity 3, not 6.

---

## 9. Follow mapping

Canonical target:

```ts
interface FollowPayload {}
```

A follow event records that TTBM observed a follow event. It does not prove the viewer remains a follower forever.

---

## 10. Subscribe mapping

Canonical target:

```ts
interface SubscribePayload {
  tier?: string;
  months?: number;
  sourceMetadata?: Record<string, unknown>;
}
```

A subscribe event means:

> TTBM observed a subscribe-related source event at a specific time.

It does **not** automatically mean:

- currently active subscription
- automatic renewal succeeded
- not cancelled
- not expired

The UI must preserve this distinction.

---

## 11. Like / share mapping

These are optional engagement records in v0.1.

Do not let high-volume like events block gift/chat processing. The ingestion design should permit batching or lower-priority persistence if required after profiling.

---

## 12. roomUser mapping

Normalize to `room_snapshot`, not `viewer_join`, unless real source evidence proves a specific individual-join contract.

Example target:

```ts
interface RoomSnapshotPayload {
  viewerCount?: number;
  sourceData?: Record<string, unknown>;
}
```

Do not increment a fan's participation count from room-level viewer-count snapshots.

---

## 13. Idempotency

Preferred key order:

1. reliable source event ID, if proven available and stable
2. deterministic fingerprint based on event-specific immutable fields

Example fingerprint inputs:

```text
source
sessionId
event type
platformUserId (if any)
source timestamp or received-time bucket
event-specific ID/value fields
```

Fingerprint design must be validated against real captured fixtures. It must not collapse legitimate repeated gifts/chats merely because content matches.

---

## 14. Time semantics

- `receivedAtUtc` = when TTBM receives the source message
- `occurredAtUtc` = source event time if reliable, otherwise documented fallback to receive time

The mapping must record whether source time was available if later auditability requires it.

All stored timestamps use UTC. UI aggregation applies configured local timezone.

---

## 15. Disconnect behavior

A reconnect does not imply replay.

On disconnect during an active session:

- record disconnect time
- set session `dataGapDetected=true`
- reconnect with bounded backoff
- record reconnect time
- show a visible warning
- do not fabricate events for the missing interval

---

## 16. Unknown/malformed payload behavior

Unknown event:

```text
preserve raw payload
→ classify `unknown`
→ diagnostic log
→ do not crash session
```

Malformed known event:

```text
preserve raw payload when parseable
→ validation failure record
→ no unsafe domain mutation
→ surface diagnostic count in developer/debug view
```

---

## 17. Contract freeze gate

The TikFinity adapter is not considered implementation-ready until all are true:

- [ ] real chat fixture captured
- [ ] real single gift fixture captured
- [ ] real gift streak sequence captured
- [ ] stable viewer ID confirmed
- [ ] avatar/name fields confirmed
- [ ] follow fixture captured
- [ ] subscribe fixture captured or explicitly marked unavailable in test account
- [ ] roomUser semantics verified from runtime payload
- [ ] event ID availability verified
- [ ] idempotency strategy tested against fixtures
- [ ] normalized mapper tests written
