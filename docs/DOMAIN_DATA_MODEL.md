# Domain & Data Model

## 1. Purpose

This document defines TTBM's durable domain entities, relationships, invariants, derived data, and deletion behavior.

The model is local-first and SQLite-oriented, but domain rules must not depend on a particular SQL library.

---

## 2. Design rules

1. Stable platform identity is separate from mutable display/profile data.
2. Raw source events and derived domain records are conceptually separate.
3. Event ingestion must be idempotent.
4. Gift streaks must not inflate totals.
5. Monthly/session totals are derived from finalized records, not handwritten cache values as authority.
6. Manual cash/back support remains separate from TikTok gift units.
7. Fan-specific private notes are deletable.
8. Data gaps are explicit session metadata.

---

## 3. Entity map

```text
Viewer
 ├──< ViewerSessionParticipation >── LiveSession
 ├──< LiveEvent >────────────────── LiveSession
 ├──< Gift >─────────────────────── LiveSession
 ├──< SubscriptionEvent
 ├──< FollowEvent
 ├──< Note
 ├──< BackSupport
 ├──< Promise
 ├──< ViewerTag >── Tag
 ├──< AfterLiveTask >────────────── LiveSession
 └──< SentItem >─────────────────── LiveSession?

LiveSession
 ├──< ConnectionGap
 ├──< LiveEvent
 └──< AfterLiveTask
```

---

## 4. Viewer

```text
Viewer
- id                     UUID/ULID/internal ID
- platform               TEXT, initially `tiktok`
- platformUserId         TEXT
- uniqueId               TEXT nullable
- nickname               TEXT nullable
- displayName            TEXT nullable
- avatarUrl              TEXT nullable
- avatarLocalPath        TEXT nullable
- firstSeenAtUtc         TEXT/INTEGER timestamp
- lastSeenAtUtc          TEXT/INTEGER timestamp
- favorite               BOOLEAN
- createdAtUtc
- updatedAtUtc
```

Unique invariant:

```text
UNIQUE(platform, platformUserId)
```

### Mutable-name policy

`uniqueId`, nickname, display name and avatar may be overwritten with latest observed values.

Future optional table if name history becomes useful:

```text
ViewerProfileHistory
```

Not required for v0.1.

---

## 5. LiveSession

```text
LiveSession
- id
- startedAtUtc
- endedAtUtc nullable
- timezone
- status
- dataGapDetected BOOLEAN
- memo nullable
- createdAtUtc
- updatedAtUtc
```

Invariants:

- only one active session per local profile in v0.1
- ended session must have `endedAtUtc`
- active session must not be used as a monthly authority after forced-corrupt close until reconciliation finishes

---

## 6. ConnectionGap

```text
ConnectionGap
- id
- sessionId
- source
- disconnectedAtUtc
- reconnectedAtUtc nullable
- reason nullable
```

If any ConnectionGap exists for an active/completed session, `dataGapDetected=true` remains sticky unless future source replay proves the gap fully recovered.

v0.1 assumes no replay guarantee.

---

## 7. LiveEvent

Immutable-ish source record:

```text
LiveEvent
- id
- sessionId
- viewerId nullable
- source
- sourceEventId nullable
- eventType
- occurredAtUtc
- receivedAtUtc
- fingerprint
- rawPayload JSON/TEXT
- validationStatus
- createdAtUtc
```

Indexes/constraints:

```text
UNIQUE(source, sourceEventId) WHERE sourceEventId IS NOT NULL
UNIQUE(fingerprint)
INDEX(sessionId, occurredAtUtc)
INDEX(viewerId, occurredAtUtc)
INDEX(eventType, occurredAtUtc)
```

Raw events are not edited for UI corrections. Corrections apply to derived/domain layers.

---

## 8. ViewerSessionParticipation

```text
ViewerSessionParticipation
- sessionId
- viewerId
- firstConfirmedAtUtc
- lastConfirmedAtUtc
- eventCount
```

Constraint:

```text
PRIMARY KEY(sessionId, viewerId)
```

A row means:

> TTBM observed at least one identity-bearing activity for this Viewer in this session.

It does not mean the app observed exact enter/leave boundaries.

---

## 9. Gift

```text
Gift
- id
- sessionId
- viewerId
- sourceEventId nullable
- sourceStreakKey nullable
- giftId
- giftName nullable
- repeatCount
- unitValue nullable
- totalValue nullable
- state
- startedAtUtc nullable
- finalizedAtUtc nullable
- occurredAtUtc
- updatedAtUtc
```

State:

```text
PROVISIONAL
FINALIZED
VOIDED
```

Only `FINALIZED` gift records participate in authoritative MVP/monthly totals.

### Streak invariant

For one logical streak, updates replace/advance the same logical streak total or apply exact deltas. They never independently add all cumulative repeat counts.

---

## 10. SubscriptionEvent

```text
SubscriptionEvent
- id
- eventId
- sessionId
- viewerId
- tier nullable
- months nullable
- occurredAtUtc
- sourceMetadata nullable
```

This table stores observed subscription events, not guaranteed current subscription state.

A future manually maintained state can be separate if required.

---

## 11. FollowEvent

```text
FollowEvent
- id
- eventId
- sessionId
- viewerId
- occurredAtUtc
```

Observed-event semantics only.

---

## 12. Note

```text
Note
- id
- viewerId
- content
- createdAtUtc
- updatedAtUtc
```

Notes are host-authored private data.

Requirements:

- editable
- deletable
- searchable locally
- excluded from telemetry if telemetry is ever added

---

## 13. BackSupport

```text
BackSupport
- id
- viewerId
- currency
- amountMinorUnit
- occurredAtUtc
- memo nullable
- createdAtUtc
- updatedAtUtc
```

Use integer minor units for fiat where practical.

Examples:

```text
KRW 50,000 → amountMinorUnit = 50000
USD 12.34 → amountMinorUnit = 1234
```

Do not merge this into TikTok Gift totals.

---

## 14. Promise

```text
Promise
- id
- viewerId
- sessionId nullable
- content
- dueAtUtc nullable
- status
- createdAtUtc
- completedAtUtc nullable
- updatedAtUtc
```

Status:

```text
OPEN
COMPLETED
CANCELLED
```

---

## 15. Tag / ViewerTag

```text
Tag
- id
- name
- type
- createdAtUtc

ViewerTag
- viewerId
- tagId
- createdAtUtc
```

Constraint:

```text
UNIQUE(viewerId, tagId)
```

Reserved/system concepts may include:

- VIP
- Favorite (can remain Viewer boolean instead)
- Regular
- New

`Subscriber` must not silently mean currently active subscription unless manually controlled or source-verifiable.

---

## 16. AfterLiveTask

```text
AfterLiveTask
- id
- sessionId
- viewerId nullable
- taskType
- status
- dueAtUtc nullable
- memo nullable
- createdAtUtc
- completedAtUtc nullable
- updatedAtUtc
```

Task types initially:

```text
THANK_YOU_MESSAGE
FAN_PHOTO
THANK_YOU_VIDEO
PROMISE_REVIEW
CONTENT_NOTE
```

Status:

```text
PENDING
IN_PROGRESS
COMPLETED
SKIPPED
```

Deduplication constraint:

```text
UNIQUE(sessionId, viewerId, taskType)
```

If a global session task has no viewer, SQLite null-uniqueness behavior must be handled with a dedicated generated key or partial indexes. Do not assume nullable unique columns dedupe global tasks correctly.

---

## 17. AfterLiveTaskReason

```text
AfterLiveTaskReason
- id
- taskId
- reasonType
- reasonValue nullable
```

Example:

```text
Task = THANK_YOU_MESSAGE for @mimi
Reasons:
- TODAY_MVP
- GIFT_THRESHOLD: 50000
- SUBSCRIBE_EVENT
```

One task, multiple reasons.

---

## 18. MessageTemplate

```text
MessageTemplate
- id
- name
- content
- createdAtUtc
- updatedAtUtc
```

Supported variable set must be explicit and validated.

v0.1 candidate variables:

```text
{nickname}
{gift_total}
{date}
```

No arbitrary template code execution.

---

## 19. SentItem

```text
SentItem
- id
- viewerId
- sessionId nullable
- type
- status
- localFilePath nullable
- memo nullable
- sentAtUtc nullable
- createdAtUtc
- updatedAtUtc
```

Types:

```text
MESSAGE
FAN_PHOTO
THANK_YOU_VIDEO
OTHER
```

---

## 20. Derived queries

### Today's/session MVP

```text
SUM(Gift.totalValue)
WHERE sessionId = ?
AND state = FINALIZED
GROUP BY viewerId
ORDER BY SUM(...) DESC
```

### Monthly MVP

Use configured timezone boundaries converted to UTC query bounds.

```text
[start-of-month local → UTC, start-of-next-month local → UTC)
```

Then aggregate finalized gifts only.

### Confirmed participation count

```text
COUNT(ViewerSessionParticipation)
WHERE viewerId = ?
```

Never derive from `roomUser` snapshots alone.

---

## 21. Delete semantics

### Delete a note/promise/back support/sent item

Hard delete is acceptable for v0.1 after explicit user action.

### Delete a fan profile

Preferred behavior:

1. delete host-authored personal data tied to Viewer
2. remove/censor mutable profile identity fields
3. keep non-identifying historical aggregates only if needed for session-stat consistency
4. document exact implementation in migration/schema before release

A practical v0.1 option is to anonymize historical event references:

```text
platformUserId → irreversible local anonymized token or detached reference
names/avatar/notes → deleted
```

Do not promise full anonymized aggregate preservation until implemented and tested.

### Full reset

Delete:

- SQLite DB
- avatar cache
- managed attachment metadata
- local application settings

User-linked external files must not be deleted unless TTBM copied them into its managed storage and the reset screen explicitly says so.

---

## 22. Schema migration rules

- every migration has monotonically increasing version
- migration must run transactionally where supported
- schema change and corresponding domain doc change ship together
- migration tests cover upgrade from previous released version
- never silently drop user data

---

## 23. Open decisions before first schema freeze

- internal IDs: UUID vs ULID vs integer primary key
- raw JSON storage encoding
- whether gifts use one logical row per streak or event rows + projection table
- how anonymous/unresolved source events are retained
- whether profile-name history is required
- exact fan-delete anonymization strategy
- attachment storage: copy-managed vs reference-only default
