# Release Scope — v0.1

## 1. Purpose

This document prevents MVP scope creep. v0.1 is a proof that one TikTok LIVE host can use TTBM locally beside TikFinity and complete the full LIVE → fan history → post-LIVE workflow safely.

---

## 2. Release theme

**One host, one PC, one local database, one reliable workflow.**

The first release is not an agency platform, not an AI assistant, not a TikTok automation bot, and not a video editor.

---

## 3. In scope

### Desktop/runtime

- Windows-first Tauri desktop app
- React/TypeScript UI
- local SQLite
- schema migrations from day one
- local settings/cache

### TikFinity integration

- configurable local WebSocket endpoint
- connect/disconnect/reconnect state
- documented event envelope handling
- real-payload fixtures
- canonical mapper
- unknown/malformed event tolerance
- connection-gap recording

### Sessions

- manual Start LIVE
- manual End LIVE
- explicit active-session state
- closeout/recovery behavior
- session history

### Viewer/fan management

- stable platform identity
- mutable handle/name/avatar updates
- avatar cache
- confirmed session participation
- fan profile
- search/filter
- favorite/tags
- notes
- promises
- back support

### Supported event-derived records

- chat
- gift
- follow
- subscribe
- room-level viewer snapshot where useful
- like/share may be persisted/handled at lower priority if real source volume makes that practical

### Gift correctness

- streak-safe finalization
- duplicate-event protection
- finalized-only authoritative rankings
- today's MVP
- monthly MVP
- top supporters

### After Live

- deterministic rule generation
- task dedupe
- task reasons
- thank-you message workflow
- message templates + clipboard copy
- fan photo/selfie workflow
- thank-you video workflow/status
- promise review
- progress tracking
- sent-item history

### Data safety

- backup package
- restore
- fan data delete behavior
- full reset
- visible source data gaps

### Design

- white/light theme
- people-centered profiles/cards/timelines
- no spreadsheet-primary experience

---

## 4. P1 after v0.1

These are deliberately deferred unless they become necessary to complete the core workflow:

- Calendar view
- custom rule builder for After Live automation
- fan birthday UI
- richer tags/rules
- CSV export
- advanced trend visualizations
- attachment import/copy management beyond simple links
- optional local Wi-Fi/mobile companion exploration

---

## 5. Explicitly out of scope

### AI

- automatic promise extraction
- automatic note summarization
- AI thank-you writing
- fan scoring via LLM

### Messaging automation

- automatic TikTok DM sending
- bulk DM bots
- automated account actions

### Payments

- bank account scraping
- automatic transfer detection
- payment processor integration
- fake conversion of TikTok gifts to exact cash payout

### Cloud/agency

- cloud database
- multi-device sync
- multi-host agency dashboard
- remote manager access
- user accounts/authentication

### Media editing

- full video editor
- CapCut-style timeline
- automatic video generation

### Source expansion

- direct TikTok protocol reverse-engineering as primary source
- Twitch/YouTube/etc adapters
- replacing TikFinity

### Mobile

- full Android/iOS release

---

## 6. Pre-implementation gate

Do not freeze persistent schema until:

- [ ] TikFinity real chat payload captured
- [ ] stable viewer ID confirmed
- [ ] single gift payload captured
- [ ] streak gift lifecycle captured
- [ ] subscribe/follow payloads captured where available
- [ ] roomUser semantics validated
- [ ] source event ID availability determined
- [ ] mapper/idempotency fixtures established

---

## 7. First vertical slice

Build in this order:

```text
1. Tauri shell + white app shell
2. SQLite migration framework
3. TikFinity connection state
4. Start/End LIVE session
5. raw event capture/debug fixture path
6. canonical event mapper
7. stable Viewer identity
8. safe gift persistence/streak logic
9. LIVE MVP/top supporters
10. PEOPLE fan profile/timeline
11. notes/promises/back support
12. End LIVE closeout
13. After Live task generation/dedupe
14. message/photo/video task UI
15. history/monthly MVP
16. backup/restore/delete/reset
```

If step 5–8 reveal incompatible source assumptions, fix contracts before advancing UI breadth.

---

## 8. Definition of Done for v0.1

v0.1 is done only when:

1. A real LIVE can run with TikFinity + TTBM concurrently.
2. Real fixture-tested events ingest without manual database editing.
3. Gift streaks/duplicates do not inflate totals.
4. Same stable Viewer remains one person across name changes.
5. Connection gaps are visible and persist into history.
6. Today's MVP and monthly MVP derive from authoritative finalized gifts.
7. Host can add/use notes, promises and back-support records.
8. Ending a LIVE creates deterministic, deduplicated follow-up tasks.
9. Host can process thank-you message, fan-photo and thank-you-video workflows.
10. Fan profile shows useful cross-session history without requiring a spreadsheet.
11. Backup/restore round trip passes.
12. Fan deletion/full reset match documented behavior.
13. App remains useful for historical browsing without cloud connectivity.
14. Release-blocking cases in `TEST_ACCEPTANCE.md` pass.

---

## 9. Non-blocking polish

The following should not block core v0.1 if data correctness and workflow are stable:

- elaborate transitions/animations
- advanced charts
- theme customization
- multiple language packs
- drag-and-drop dashboards
- deep keyboard shortcut system
- sophisticated gamification/fan levels

These can be layered later without changing the core data contract.
