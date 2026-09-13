# Test & Acceptance

## 1. Purpose

This document converts product/data rules into release-blocking acceptance scenarios. Tests should be automated wherever practical and backed by real TikFinity fixtures once captured.

---

# 2. Contract / adapter tests

## AC-EVT-001 — Known event envelope

Given a valid TikFinity envelope:

```json
{"event":"chat","data":{}}
```

When the adapter receives it,
Then the event is classified without crashing and handed to the mapper.

## AC-EVT-002 — Unknown event

Given an unknown event name,
When received,
Then it is classified `unknown`, raw payload is safely preservable/diagnosable, and subsequent valid events still process.

## AC-EVT-003 — Invalid JSON

Invalid JSON must not crash the receiver or mutate domain state.

## AC-EVT-004 — Stable identity mapping

Given two events with the same stable platform user ID but different mutable handle/nickname,
Then one Viewer remains authoritative and mutable profile fields update.

## AC-EVT-005 — Missing stable identity

Given an event with nickname but no confirmed stable platform user ID,
Then TTBM does not create a durable Viewer keyed only by nickname.

---

# 3. Gift tests

## AC-GIFT-001 — Single non-streak gift

Given one valid non-streak gift,
Then one finalized Gift is stored and session total increases exactly once.

## AC-GIFT-002 — Streak does not double-count

Given cumulative updates:

```text
repeatCount 1
repeatCount 2
repeatCount 3 + end
```

Then authoritative quantity is 3, not 6.

## AC-GIFT-003 — Duplicate final event

Given the exact same finalized source gift event twice,
Then total increases once.

## AC-GIFT-004 — Legitimate repeated identical gifts

Given two genuinely distinct identical gifts from the same Viewer close together,
Then idempotency must not collapse them into one merely because gift ID/value/name match.

## AC-GIFT-005 — Provisional gift excluded from closed authority

If session close cannot safely finalize an active streak,
Then it is not silently included as authoritative final total; warning/reconciliation state is visible.

## AC-GIFT-006 — MVP uses finalized gifts only

Session MVP/monthly MVP authority excludes non-finalized or voided gifts.

---

# 4. Viewer/participation tests

## AC-ID-001 — Name change

Given:

```text
platformUserId=1234, uniqueId=mimi
```

and later:

```text
platformUserId=1234, uniqueId=mimi_new
```

Then Viewer count remains one.

## AC-ID-002 — Two different stable IDs, same nickname

Then they remain two different Viewers.

## AC-PART-001 — Participation once per session

Given one Viewer chats five times and gifts twice in one session,
Then one ViewerSessionParticipation row exists for the session and its event count reflects qualifying activity.

## AC-PART-002 — Across sessions

The same Viewer confirmed in three sessions has confirmed-session count three.

## AC-PART-003 — room snapshot is not a visit

A room-level viewer-count event by itself does not create fan participation.

---

# 5. Session tests

## AC-SESSION-001 — Manual start

Starting LIVE durably creates one active session before source events are attributed.

## AC-SESSION-002 — No duplicate active session

Attempting to start another session while one is active is blocked or explicitly resolved.

## AC-SESSION-003 — Manual end

End flow records cutoff, reconciles safe gift state, generates tasks idempotently, stores end time and closes session.

## AC-SESSION-004 — Crash during ending

Given an interrupted ENDING session,
Restart can safely resume/recover closeout without duplicate tasks or gift totals.

---

# 6. Connection/data-gap tests

## AC-CONN-001 — TikFinity closed at startup

TTBM opens and local history remains usable while connection state is disconnected.

## AC-CONN-002 — Disconnect during LIVE

Given active session and source disconnect,
Then a ConnectionGap is created and session `dataGapDetected=true`.

## AC-CONN-003 — Reconnect does not erase gap

After successful reconnect, session remains marked as having a possible missing interval.

## AC-CONN-004 — Gap shown in history

Closed session detail displays data-gap warning.

---

# 7. After Live tests

## AC-TASK-001 — MVP rule

Today's MVP receives one thank-you task when rule is enabled/defaulted.

## AC-TASK-002 — Multi-reason dedupe

Given Viewer is MVP, exceeds gift threshold, and has a subscribe event,
If all trigger the same THANK_YOU_MESSAGE task type,
Then one task exists with multiple reasons.

## AC-TASK-003 — Rerun safe

Running task generation twice produces identical task cardinality and no duplicates.

## AC-TASK-004 — Complete message

Marking a thank-you message as sent records completion and a SentItem.

## AC-TASK-005 — Skip is not sent

Skipping a task resolves it for progress if configured but does not create SentItem.

## AC-TASK-006 — Progress

Progress counts resolved tasks according to documented COMPLETED/SKIPPED policy and visually distinguishes the two.

---

# 8. Notes/promises/back support

## AC-NOTE-001

Create, edit, search and delete note without exposing content in normal diagnostic logs.

## AC-PROMISE-001

Promise transitions OPEN → COMPLETED/CANCELLED and can be reopened.

## AC-BACK-001

Back support requires positive amount and currency and remains separate from TikTok gift totals.

## AC-BACK-002

KRW and another currency are not naively summed into one total.

---

# 9. Time/month tests

## AC-TIME-001 — UTC persistence

Stored event/session timestamps use UTC authority.

## AC-TIME-002 — Local month boundary

An event near midnight/month boundary is assigned according to configured timezone month, not naive UTC calendar month.

## AC-TIME-003 — Timezone setting change

Changing display timezone does not rewrite stored event timestamps.

---

# 10. Backup/restore tests

## AC-BACKUP-001 — Round trip

Create representative local state, create backup, restore into clean state, and verify equivalent:

- viewers
- sessions
- gifts
- notes
- promises
- tasks
- sent items
- templates/settings required by format

## AC-BACKUP-002 — Corrupt/invalid package

Invalid backup is rejected before replacing active DB.

## AC-BACKUP-003 — Restore failure safety

A failed restore does not destroy the pre-restore local dataset.

## AC-BACKUP-004 — Live DB consistency

Backup procedure produces a consistent SQLite snapshot; test under WAL/use mode selected by implementation.

---

# 11. Delete/privacy tests

## AC-DELETE-001 — Fan deletion

Fan deletion removes exactly the categories promised by UI/docs and leaves no undeclared host-authored private note data for that Viewer in managed stores.

## AC-DELETE-002 — Full reset

Full reset removes TTBM-managed DB/settings/cache.

## AC-DELETE-003 — External linked file preserved

A linked external image/video survives fan deletion/full reset unless it had explicitly been imported into managed storage and deletion behavior was disclosed.

## AC-LOG-001

Normal diagnostic logs do not contain note content.

---

# 12. UI acceptance

## AC-UI-001 — White-theme default

Primary app launches in approved light/white visual system.

## AC-UI-002 — No spreadsheet primary UX

LIVE, PEOPLE and AFTER LIVE primary interactions use cards/profiles/timeline/checklist rather than requiring spreadsheet-style table management.

## AC-UI-003 — Connection states accessible

Host can identify connected/reconnecting/disconnected and session data-gap status without opening developer tools.

## AC-UI-004 — Fast fan detail

From supporter ranking or fan list, host can reach fan detail and quick note/promise action with a small number of interactions.

## AC-UI-005 — Destructive confirmation

Fan delete, full reset and restore-over-current-data require explicit confirmation.

---

# 13. Performance baseline

Exact numeric SLA is deferred until fixture/runtime profiling, but acceptance requires:

- gift processing remains correct under concurrent chat/like traffic
- high-volume low-value events do not freeze critical UI indefinitely
- fan list/timeline remains usable with realistic accumulated local data
- no unbounded in-memory event queue during prolonged DB/source failure

A synthetic load fixture suite should be added after real event shapes are known.

---

# 14. v0.1 release gates

Release is blocked unless:

- [ ] real TikFinity fixture set exists
- [ ] stable viewer identity field is runtime-confirmed
- [ ] gift streak mapper tests pass
- [ ] event idempotency tests pass
- [ ] session recovery tests pass
- [ ] task dedupe/rerun tests pass
- [ ] monthly timezone test passes
- [ ] backup/restore round trip passes
- [ ] fan delete/full reset behavior matches docs
- [ ] data-gap state is visible
- [ ] no mandatory cloud dependency exists
