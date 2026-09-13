# Product Requirements — TTBM

## 1. Product statement

**TTBM (TikTok Broadcast Manager)** is a local-first desktop workspace for TikTok LIVE hosts. It captures identifiable LIVE activity, builds fan-centered history, and turns each finished broadcast into a clear follow-up workflow.

The product should answer three questions quickly:

1. **During LIVE:** Who is here, who is supporting, and who needs attention?
2. **After LIVE:** Who do I need to thank, send a fan selfie/photo to, make a thank-you video for, or follow up with?
3. **Next LIVE:** What history, notes, promises, and previous support exist for this person?

TTBM is not primarily a statistics product. Statistics exist to support host operations and fan relationship continuity.

---

## 2. Primary user

### TikTok LIVE host

A creator who streams frequently and currently relies on memory, scattered notes, chat history, spreadsheets, messaging apps, or ad-hoc lists to remember supporters and post-LIVE obligations.

### Secondary future users

Not in v0.1:

- host managers
- agencies managing multiple hosts
- mobile-only hosts
- other LIVE platforms

---

## 3. Core problems

Hosts increasingly need to manage more than the LIVE itself.

Common operational problems:

- remembering who supported the stream today
- identifying today's and this month's top supporters
- remembering repeat supporters when their handle/display name changes
- retaining context about fans without opening spreadsheets
- recording off-platform/back-channel support manually
- remembering promises made during LIVE
- tracking thank-you messages sent after LIVE
- tracking fan selfies/photos that still need to be sent
- tracking thank-you videos that need recording/editing/sending
- knowing what remains unfinished after the stream ends
- reviewing past interactions before the next LIVE

---

## 4. Product principles

### 4.1 Local-first

- No mandatory cloud account.
- No mandatory cloud database.
- Primary data stays on the host's computer.
- Past records remain readable without network access.
- User controls backup, restore, export, and deletion.

### 4.2 People-first UI

The app must not feel like Excel, Airtable, or an enterprise admin console.

Primary UI patterns:

- profile cards
- avatars
- badges
- timelines
- supporter ranking cards
- checklists
- progress indicators
- compact detail drawers
- calendar/event cards

### 4.3 White theme

Default visual direction:

- bright white background
- light gray surfaces
- restrained lavender as primary accent
- mint for success/completion
- pink for relationship/fan-related accents
- gold/yellow reserved for MVP/VIP/highlight states

Dark mode is not an MVP requirement.

### 4.4 Minimal interaction during LIVE

The host should rarely type during LIVE.

Allowed quick actions:

- add note
- add promise
- favorite/unfavorite
- record back support

Everything else should prefer automatic ingestion or post-LIVE processing.

### 4.5 Honest data semantics

The product must not imply more source certainty than actually exists.

Examples:

- do not label confirmed activity as total unique viewers
- do not call a past `subscribe` event an always-current subscription state unless verified
- do not hide source disconnect gaps
- do not convert TikTok gifts and cash support into one fake currency total

---

## 5. MVP goals

v0.1 must prove that one host can run TTBM beside TikFinity and complete this loop:

```text
Start TTBM
→ connect to TikFinity
→ start LIVE session
→ ingest fan events
→ resolve stable fan identity
→ aggregate gifts safely
→ show current top supporters / MVP
→ add notes/promises when needed
→ end LIVE
→ generate post-LIVE tasks
→ process thank-you / fan-photo / video / promise tasks
→ review fan history later
→ back up local data
```

---

## 6. MVP functional requirements

### LIVE

- connect to local TikFinity Event API
- show connection state
- manually start/end a TTBM LIVE session
- ingest supported normalized events
- show data-gap warning when source connectivity is lost
- show today's MVP and top supporters
- show session gift total
- show confirmed participating fans, not unverifiable total audience identity
- quick note / promise / back-support actions

### PEOPLE

- fan profile keyed by stable platform identity
- current handle/display name/avatar
- confirmed participation sessions
- chat count
- TikTok gift totals
- current-month gift total
- subscribe-event history
- follow-event history
- notes
- promises
- tags/favorite
- manual back-support history
- sent-items history

### AFTER LIVE

- automatically generate tasks from deterministic rules
- deduplicate same fan + same task type + same session
- retain multiple reasons for one task
- thank-you message template and clipboard copy
- fan-photo/selfie task tracking
- thank-you video task tracking
- promise review tasks
- completion progress

### HISTORY

- session history
- monthly MVP / top supporters
- session data-gap indicator

### SETTINGS / SAFETY

- timezone
- message templates
- backup
- restore
- fan data deletion
- full reset

---

## 7. Explicit non-goals for v0.1

- AI-generated messages or AI note extraction
- automatic TikTok DM sending
- automatic bank-account scraping or payment detection
- cloud sync
- account/login system
- manager/agency dashboard
- full mobile app
- full video editor
- guaranteed current subscription status
- guaranteed identity of every viewer who silently enters/leaves
- TikFinity replacement
- direct TikTok protocol reverse-engineering as the primary path

---

## 8. Success criteria

The MVP is successful when all are true:

1. One host can run a real LIVE with TikFinity and TTBM simultaneously.
2. TTBM survives normal reconnects without silently inventing missing events.
3. Repeat-gift/streak behavior does not inflate gift totals.
4. Duplicate source events do not inflate totals.
5. The same stable TikTok user remains one fan when mutable names change.
6. The host can find a fan's useful history in seconds.
7. Ending a LIVE creates a useful post-LIVE work queue.
8. Same-session duplicate follow-up tasks are not generated.
9. The host can finish the post-LIVE queue without external spreadsheets.
10. Local backup and restore reproduce the same usable records.

---

## 9. Product risks

### Source-contract risk

TikFinity payload shape can change or differ from assumptions. Mitigation: adapter boundary, raw payload preservation, fixture-based contract tests, real payload capture before schema freeze.

### Identity risk

Mutable names are unsafe identity keys. Mitigation: require stable platform user ID before creating durable fan identity.

### Gift-accounting risk

Gift streak semantics can create severe double-counting. Mitigation: provisional/finalized gift handling and idempotency.

### Data-loss risk

Local-only data can be lost with a failed PC. Mitigation: clear backup UX and tested restore.

### Privacy risk

Host-written fan notes may contain sensitive context. Mitigation: local-first storage, explicit deletion controls, no silent cloud sync.

---

## 10. Product naming

`TTBM` and `TikTok Broadcast Manager` are working names. Naming/branding can change without affecting domain terminology or persisted schema.
