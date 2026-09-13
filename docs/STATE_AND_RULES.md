# State & Rules

## 1. Purpose

TTBM has multiple independent state machines. They must not be collapsed into one `isLive` boolean or ad-hoc UI flags.

This document defines authoritative transitions for:

- TikFinity connection
- LIVE session
- gift streak processing
- After Live tasks
- promises
- thank-you video workflow

---

# 2. Connection state

```text
DISCONNECTED
   ↓ connect
CONNECTING
   ├─ success → CONNECTED
   └─ fail → DISCONNECTED

CONNECTED
   └─ socket lost → RECONNECTING

RECONNECTING
   ├─ success → CONNECTED
   ├─ user stop → DISCONNECTED
   └─ retry policy exhausted → DISCONNECTED
```

## Rules

- Connection state is independent from session state.
- A LIVE session may remain active while connection is `RECONNECTING`.
- Any connection loss during an active session creates a `ConnectionGap`.
- Successful reconnect does not erase `dataGapDetected`.
- Retry behavior should use bounded backoff with visible state, not an infinite silent tight loop.

---

# 3. LIVE session state

```text
IDLE
  ↓ Start LIVE
STARTING
  ├─ success → LIVE
  └─ failure → IDLE

LIVE
  ↓ End LIVE
ENDING
  ├─ finalize success → CLOSED
  └─ recoverable failure → ENDING / retry

CLOSED
```

## Rules

### IDLE

- no session-specific events should be attributed to a non-existent active session
- incoming TikFinity events may trigger a prompt suggesting session start

### STARTING

- create session record transactionally
- set timezone snapshot
- only transition to LIVE when session ID is durable

### LIVE

- accept normalized events
- connection may be connected or degraded
- quick notes/promises allowed

### ENDING

- stop accepting events into the session only after an explicit cutoff is recorded
- reconcile provisional gift streaks
- calculate final projections
- generate After Live tasks idempotently

### CLOSED

- immutable session start/end boundaries except explicit repair/admin flow
- late source events must not silently mutate a closed session without a documented reconciliation path

---

# 4. Session-start policy

v0.1 authority is manual:

```text
Host clicks Start LIVE
```

Optional convenience:

```text
source event arrives while IDLE
→ prompt: "방송 세션을 시작할까요?"
```

Do not auto-create sessions merely because TikFinity emits a background/test event.

---

# 5. Session-end policy

v0.1 authority is manual:

```text
Host clicks End LIVE
```

End flow:

1. capture cutoff timestamp
2. enter ENDING
3. reconcile gift streaks
4. persist final projections
5. generate/dedupe tasks
6. persist end time
7. enter CLOSED
8. navigate to AFTER LIVE summary

If reconciliation cannot safely finalize a gift streak, mark it `PROVISIONAL`/needs review rather than guessing.

---

# 6. Gift state machine

```text
RECEIVED
   ↓ validate
PROVISIONAL (streak-capable)
   ├─ update repeat count → PROVISIONAL
   ├─ repeatEnd → FINALIZED
   └─ invalid/retracted → VOIDED

RECEIVED (non-streak)
   └─ validate → FINALIZED
```

## Gift rules

- only `FINALIZED` values are authoritative for closed-session/monthly totals
- LIVE UI may show provisional amount with a visual temporary state if desired
- repeated cumulative counts must not be summed independently
- duplicate source events must be idempotent
- finalization must be deterministic from fixture-tested source fields

Example:

```text
streak updates: 1, 2, 3(final)
authoritative quantity: 3
NOT 1 + 2 + 3 = 6
```

---

# 7. Viewer participation rule

A fan receives one participation row per session when TTBM observes at least one durable identity-bearing activity.

```text
first qualifying event
→ create ViewerSessionParticipation

later qualifying event
→ update lastConfirmedAtUtc + eventCount
```

Qualifying examples:

- chat
- gift
- follow
- subscribe
- another normalized event with stable viewer identity

Non-qualifying by itself:

- anonymous room viewer count snapshot

---

# 8. Fan lifecycle labels

System labels are projections, not identity.

Suggested deterministic definitions are configurable later; v0.1 may use:

### New

Viewer first confirmed within recent threshold (exact threshold to be decided).

### Regular

Repeated confirmed participation above a threshold.

### VIP

Manual tag in v0.1 unless a deterministic threshold is explicitly configured.

### Favorite

Manual host choice.

### Subscriber

Do not derive current-active subscriber status solely from historical subscribe events. Prefer wording such as `구독 이벤트 확인` unless manually maintained.

---

# 9. After Live task state

```text
PENDING
  ├─ start → IN_PROGRESS
  ├─ complete directly → COMPLETED
  └─ skip → SKIPPED

IN_PROGRESS
  ├─ complete → COMPLETED
  ├─ skip → SKIPPED
  └─ reset → PENDING

COMPLETED
  └─ reopen → PENDING (explicit user action)

SKIPPED
  └─ reopen → PENDING
```

## Completion rule

Progress denominator counts all generated tasks except any future task types explicitly marked informational.

Default:

```text
progress = COMPLETED / (PENDING + IN_PROGRESS + COMPLETED)
```

Whether `SKIPPED` counts as resolved is a product decision. Recommended v0.1:

```text
resolved = COMPLETED + SKIPPED
progress = resolved / total
```

UI should distinguish skipped from completed.

---

# 10. After Live generation rules

Initial defaults are product defaults, not hard-coded forever.

Candidate v0.1 rules:

```text
TODAY_MVP
→ THANK_YOU_MESSAGE

Gift total >= configurable photo threshold
→ FAN_PHOTO

Gift total >= configurable video threshold
→ THANK_YOU_VIDEO

Subscribe event observed
→ THANK_YOU_MESSAGE

Open promise related to this session/viewer
→ PROMISE_REVIEW
```

## Dedupe invariant

At most one task per:

```text
(sessionId, viewerId, taskType)
```

If multiple rules match:

```text
1 task
+ multiple AfterLiveTaskReason rows
```

Task generation is rerunnable and idempotent.

---

# 11. Promise state

```text
OPEN
  ├─ fulfill → COMPLETED
  └─ cancel → CANCELLED

COMPLETED
  └─ reopen → OPEN

CANCELLED
  └─ reopen → OPEN
```

Promises with due dates appear in future calendar/P1 views.

---

# 12. Thank-you video sub-workflow

A THANK_YOU_VIDEO task may have an optional workflow stage separate from the generic task status.

```text
TODO
→ RECORDING
→ EDITING
→ READY
→ SENT
```

Mapping recommendation:

```text
TODO       => task PENDING
RECORDING  => task IN_PROGRESS
EDITING    => task IN_PROGRESS
READY      => task IN_PROGRESS
SENT       => task COMPLETED
```

Do not introduce a full video-editor domain into v0.1.

---

# 13. Sent-item rule

A task completion that represents something actually sent can create `SentItem`.

Examples:

```text
THANK_YOU_MESSAGE completed as sent
→ SentItem(MESSAGE)

FAN_PHOTO completed as sent
→ SentItem(FAN_PHOTO)

THANK_YOU_VIDEO completed as sent
→ SentItem(THANK_YOU_VIDEO)
```

Checking a task as skipped must not create a sent item.

---

# 14. Back-support rule

Manual back support:

- requires viewer
- requires amount > 0
- requires currency
- records user-selected occurrence date/time
- remains separate from TikTok gift totals
- may optionally create an After Live task only through an explicit future rule, not implicitly in v0.1 unless configured

---

# 15. Time and month boundary

Persist UTC.

For display/month queries:

```text
configured timezone
→ local month start
→ convert to UTC lower bound
→ next local month start
→ convert to UTC exclusive upper bound
```

Do not group UTC timestamps by naive UTC month when user's configured timezone differs.

---

# 16. Failure rules

- unknown source event → preserve/log, no crash
- malformed event → no unsafe domain mutation
- DB write failure → do not acknowledge domain success to UI
- reconnect → never imply missed events recovered
- backup failure → keep original DB untouched
- restore failure → roll back to pre-restore safety snapshot

See `ERROR_RECOVERY.md` for operational detail.
