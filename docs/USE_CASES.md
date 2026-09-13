# Use Cases — TTBM v2

## 1. Scope

TTBM is a local-first TikTok LIVE host operations tool. v0.1 uses TikFinity as the local event source, stores data locally, excludes AI and automated TikTok messaging, and focuses on fan history plus post-LIVE work.

For detailed invariants, also read `TIKFINITY_EVENT_CONTRACT.md`, `DOMAIN_DATA_MODEL.md`, and `STATE_AND_RULES.md`.

## 2. Actors

- **Host** — TikTok LIVE creator using TTBM.
- **TikFinity** — local LIVE event source.
- **Local Storage** — SQLite plus app-managed cache/backups.

## 3. Main flow

```text
Launch → connect TikFinity → Start LIVE → normalize events → resolve fan identity
→ update history/gifts → inspect MVP → add note/promise if needed
→ End LIVE → finalize session → generate After Live tasks
→ process thank-you/photo/video/promise work → review history → backup
```

---

## UC-01 Launch application

Load the local DB, apply safe migrations, restore recent sessions/tasks, and show TikFinity connection state. Missing DB creates a new DB; migration/corruption failure enters recovery instead of unsafe writes.

## UC-02 Connect TikFinity

Connect to the configured local WebSocket, show connection state, receive events, and record/retry disconnects. Reconnect never implies missed events were replayed.

## UC-03 Start LIVE session

Host explicitly starts a session in v0.1. Persist session ID, UTC start time and timezone snapshot before accepting session-owned events. Incoming events while idle may suggest starting a session but must not silently create one.

## UC-04 Auto-register fan

For an identity-bearing event, resolve Viewer by `(platform, platformUserId)`. Create if missing; otherwise refresh mutable handle/name/avatar fields. Never create a durable Viewer from nickname alone.

## UC-05 Cache avatar

Cache observed avatar locally when possible and retain source URL + local path. Fall back to generated/default avatar if download fails.

## UC-06 Record chat

Validate, dedupe, resolve Viewer, persist event, update last activity, update confirmed session participation, and reflect the event in fan activity/timeline.

## UC-07 Record gift safely

Validate/dedupe gift events, resolve Viewer, process streak semantics, finalize authoritative quantity/value, then update session/month ranking. Cumulative streak updates such as `1 → 2 → 3(final)` must equal 3, not 6.

## UC-08 Record subscribe event

Persist the observed subscribe event and timestamp against the Viewer/session. Do not interpret historical subscribe events as guaranteed current subscription state.

## UC-09 Record follow event

Persist the observed follow event and show it in history. Historical observation does not prove current follower state forever.

## UC-10 Confirm participation

The first qualifying identity-bearing event in a session creates one `(sessionId, viewerId)` participation row. Later events update it. Room-level viewer-count snapshots alone do not count as a fan visit.

## UC-11 Calculate today's MVP

Aggregate only authoritative/finalized gifts in the active session by Viewer and display rank #1 as Today's MVP plus top supporters.

## UC-12 View monthly MVP

Use configured local timezone month boundaries converted to UTC, aggregate finalized gifts, show top 1–3 prominently and remaining ranking compactly, and allow opening fan profiles.

## UC-13 View fan profile

Show current profile, tags/favorite, first/last activity, confirmed session count, chat count, gifts, current-month gifts, observed follow/subscribe events, notes, promises, back support, sent items and related tasks.

## UC-14 Search/filter fans

Search handle/name/notes/tags. Filters include VIP, favorite, new, regular, recent subscription event, current-month supporter, recent activity and unresolved work.

## UC-15 Manage note

Host can add, edit and delete local private notes with timestamps.

## UC-16 Record back support

Host manually records currency, amount, date/time and optional memo. Cash support stays separate from TikTok gift units.

## UC-17 Manage promise

Host creates a Viewer-linked promise with optional due date/session. Promise remains open until completed or cancelled and may feed post-LIVE/calendar workflows.

## UC-18 Manage tags

Host applies manual/system tags such as VIP, Favorite, Regular, New and custom tags. Do not silently derive active Subscriber from historical subscribe events.

## UC-19 End LIVE session

Host ends the session. TTBM records cutoff, reconciles provisional gifts, computes final projections, preserves data-gap state, generates/deduplicates post-LIVE tasks, persists end time and closes the session. Unsafe unresolved gift state is surfaced rather than guessed.

## UC-20 Generate After Live tasks

Initial deterministic rules may include: Today's MVP → thank-you message; gift threshold → fan photo; higher threshold → thank-you video; observed subscribe event → thank-you message; open promise → promise review. One task per `(session, viewer, task type)`; multiple triggers become multiple reasons on one task.

## UC-21 Manage message templates

Host creates/edits/deletes named templates with a small validated placeholder set such as `{nickname}`, `{gift_total}`, `{date}`.

## UC-22 Process thank-you message

Select task and template, resolve placeholders, optionally edit, copy to clipboard, send manually outside TTBM, then mark sent and record a SentItem. No automated TikTok DM sending in v0.1.

## UC-23 Process fan-photo task

Open the task, optionally link a local image, send externally, mark sent/completed, and record SentItem.

## UC-24 Process thank-you video task

Track optional stages `TODO → RECORDING → EDITING → READY → SENT`, attach a local file path if useful, and record completion. TTBM is not a full video editor.

## UC-25 View After Live progress

Show total, resolved and remaining work grouped by task type. Completed and skipped are both resolved for progress but remain visually distinct.

## UC-26 View session history

Show start/end/duration, finalized gifts, session MVP/top supporters, chat/subscribe counts, confirmed participants, data-gap warning, post-LIVE completion and memo.

## UC-27 Create local backup

Create a consistent DB snapshot and package settings/templates; optionally include managed cache/media. Do not naively copy a live SQLite/WAL set without a consistency procedure.

## UC-28 Restore local backup

Validate backup/version, create a pre-restore safety snapshot, restore, migrate if necessary and reload. On failure keep/restore the pre-restore state.

## UC-29 Delete fan data

Allow explicit fan-data deletion. Remove host-authored personal data and profile identity data according to the implemented deletion/anonymization policy while preserving only what the documented schema safely permits.

## UC-30 Full reset

After explicit confirmation, reset TTBM-managed DB/cache/settings to new-install state. Linked external user files are not deleted merely because TTBM stored their path.

---

## P1 use cases

- **Calendar** — broadcasts, promises, birthdays, due tasks and custom events.
- **Custom After Live rules** — host-configurable thresholds/automation rules.
- **CSV export** — portability/analysis without turning the main UI into a spreadsheet.

## Acceptance summary

The flow is viable when stable identity survives name changes, gift streaks/duplicates do not inflate totals, disconnect gaps remain visible, sessions have explicit boundaries, fan history is useful without spreadsheets, post-LIVE tasks are deterministic/deduplicated, and local backup/restore/delete paths work.
