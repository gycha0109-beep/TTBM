# Error & Recovery

## 1. Principle

TTBM must fail visibly and conservatively. It must not hide data loss, invent missing LIVE events, or report an operation as complete before durable local persistence succeeds.

---

## 2. TikFinity unavailable at app start

Expected behavior:

- app still opens
- historical local data remains usable
- source status shows `DISCONNECTED`
- host can inspect PEOPLE/HISTORY/AFTER LIVE
- reconnect remains available

Do not block the entire application because TikFinity is closed.

---

## 3. Disconnect during LIVE

On socket loss:

1. transition to `RECONNECTING`
2. record `ConnectionGap.disconnectedAtUtc`
3. set active session `dataGapDetected=true`
4. keep local session open
5. retry using bounded backoff
6. on reconnect, record `reconnectedAtUtc`
7. retain visible gap warning until session close/history

Never claim the gap was recovered unless a future source replay mechanism is explicitly proven.

---

## 4. Invalid JSON / malformed event

### Invalid JSON

- reject event
- increment diagnostic counter/log
- do not crash receiver

### Known envelope, malformed payload

- preserve source data when safely possible
- mark validation failure
- perform no unsafe domain mutation
- continue receiving subsequent events

### Unknown event type

- classify as unknown
- preserve/debug-log within retention limits
- continue session

---

## 5. Duplicate event

If idempotency key/fingerprint already exists:

- treat as already processed
- do not increment gift/chat/follow/subscription projections twice
- diagnostic tracing may record duplicate detection

A duplicate is not an application error unless rate/shape suggests mapper failure.

---

## 6. Gift streak unresolved at End LIVE

Do not guess a final count.

Possible v0.1 behavior:

- leave streak `PROVISIONAL`
- exclude it from authoritative closed-session totals
- show `gift reconciliation needed` warning
- allow later fixture-informed repair/admin path

If source runtime evidence establishes a safe timeout/finalization rule, document and test it before enabling.

---

## 7. SQLite write failure

When a durable write fails:

- rollback transaction
- do not update UI as if write succeeded
- show a concise local-data error
- keep receiver alive if safe
- if DB becomes unavailable/corrupt, stop further mutating operations and enter degraded/recovery mode

Gift correctness is more important than keeping decorative UI live.

---

## 8. App crash/restart during LIVE

On startup:

- detect session in `LIVE`, `STARTING`, or `ENDING`
- show interrupted-session recovery UI
- do not silently start a second session

Candidate recovery choices:

- resume existing session if appropriate and source state supports it
- close at last known safe timestamp with interruption marker

Exact automated recovery policy must be tested before release.

---

## 9. App crash during End LIVE

Session closeout steps must be idempotent.

On restart, an `ENDING` session can resume:

1. verify cutoff
2. reconcile safe gift state
3. rerun task generation (deduped)
4. finalize session if all required steps succeed

---

## 10. Avatar/cache failure

Avatar download/cache failure never blocks event ingestion.

Fallback:

- use source URL if currently reachable, or
- generated initials/default avatar

Cache corruption can be cleared/rebuilt independently from core DB.

---

## 11. Linked media missing

If a user-linked photo/video path no longer exists:

- retain task/history record
- show `file missing`
- allow relink
- do not mark sent/completed automatically

---

## 12. Backup failure

- active database remains untouched
- partial backup package is removed/marked invalid
- user receives failure reason where available
- no success message until package validation completes

---

## 13. Restore failure

Before restore:

- validate manifest/version
- create pre-restore safety snapshot when possible

On failure:

- do not leave half-restored active state
- restore/repoint to safety snapshot
- surface recovery instructions

---

## 14. Migration failure

Migrations must be transactional where supported.

Failure behavior:

- abort migration set
- do not continue normal mutating runtime on uncertain schema
- preserve old data
- offer backup/recovery diagnostics

---

## 15. Disk full / permission loss

Treat inability to write as a critical local-data condition.

- notify immediately
- stop claiming events/tasks are saved
- do not silently buffer indefinitely in memory
- allow export/backup only if filesystem permits

---

## 16. Clock/timezone change

Persist UTC source/receive timestamps. A settings timezone change affects display and future aggregation boundaries, not the stored event timestamps.

Session stores timezone snapshot for audit/display consistency.

---

## 17. Recovery acceptance gates

Before v0.1 release, manually or automatically verify:

- TikFinity closed at startup
- disconnect/reconnect during active session
- duplicate gift
- malformed event followed by valid event
- app restart with active/incomplete session
- DB write failure simulation where practical
- backup failure
- backup/restore round trip
- missing linked media
- cache clear and rebuild
