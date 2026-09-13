# Screen Specification

## 1. Visual direction

Default UI is **white-theme, creator-friendly, people-centered**.

Avoid:

- dense spreadsheet tables as the primary interaction
- dark default theme
- enterprise-admin visual language
- excessive charts that do not help the host act

Prefer:

- white background
- light gray surfaces
- rounded cards
- soft borders/shadows
- lavender primary accent
- mint completion/success
- pink relationship accents
- gold/yellow reserved for MVP/VIP/highlight
- avatars, badges, timeline, compact ranking cards

Primary desktop layout:

```text
┌─────────────┬──────────────────────────────┬────────────────────┐
│ Sidebar     │ Main content                 │ Optional detail    │
│             │                              │ drawer             │
│ LIVE        │                              │                    │
│ PEOPLE      │                              │                    │
│ AFTER LIVE  │                              │                    │
│ HISTORY     │                              │                    │
│ SETTINGS    │                              │                    │
└─────────────┴──────────────────────────────┴────────────────────┘
```

---

# 2. Global shell

## Sidebar

Items:

- LIVE
- PEOPLE
- AFTER LIVE
- HISTORY / MONTHLY
- SETTINGS

P1:

- CALENDAR

Bottom utility area:

- current local data status
- optional backup reminder
- app version

## Global source status

Always reachable/visible enough to answer:

- TikFinity connected?
- active LIVE session?
- data gap detected?

Connection indicators:

```text
Connected        mint/green
Connecting       neutral animated
Reconnecting     amber
Disconnected     gray/red-accent warning
Data gap         amber warning badge
```

---

# 3. LIVE screen

## Goal

Give the host the useful current state without demanding interaction.

## Header

- screen title: `LIVE`
- session status
- elapsed time
- TikFinity connection status
- Start LIVE / End LIVE action

Do not hide End LIVE in a menu.

## Summary cards

Candidate cards:

1. Today's MVP
2. Session Gift Total
3. Confirmed Participants
4. Subscribe Events

Do not label confirmed participants as total viewers.

## Top supporters

Presentation:

- #1 larger MVP card
- #2/#3 secondary cards
- #4–#10 compact rows/cards

Card content:

- avatar
- handle/nickname
- rank
- finalized gift total
- useful tag/badge

Click opens Viewer detail drawer or PEOPLE profile.

## Live activity rail

Compact event feed for important events:

- finalized gift
- subscribe event
- follow event
- notable manual action

High-volume chat should not dominate the screen by default.

## Quick actions

When a fan is selected:

- Add Note
- Add Promise
- Record Back Support
- Favorite

## Data-gap banner

If a source disconnect occurs during LIVE:

```text
⚠ TikFinity connection was interrupted from 22:14:08 to 22:15:31.
Some LIVE events may be missing.
```

Banner remains visible for the session; reconnection does not clear the warning.

---

# 4. PEOPLE screen

## Goal

Let the host recognize and understand a fan quickly.

## Left rail / list

- search field
- saved filters/categories
- fan list with avatar, name, badges and small key metric

Suggested filters:

- All
- Favorites
- VIP
- New
- Regular
- This month supporters
- Recent subscribe event
- Needs follow-up

## Fan profile header

- avatar
- current `@handle`
- nickname/display name
- favorite toggle
- tags
- first confirmed / last confirmed activity

## Key stat cards

- This month gifts
- Total gifts
- Confirmed sessions
- Chat count

Optional observed subscription indicator should use wording that does not claim active subscription unless verified.

## Tabs

### Timeline

Chronological relationship/activity history.

Examples:

```text
Today 22:14   Gift · Galaxy ×1
Today 21:03   Chat
Sep 10        Subscribe event observed
Sep 08        Gift · Rose ×120
```

### Notes

- add
- edit
- delete
- timestamps

### Promises

- open first
- due date
- complete/cancel/reopen

### Sent

- thank-you message
- fan photo
- thank-you video
- other

## Manual back-support card

Show separately from TikTok Gift metrics.

Example:

```text
Back support
KRW 50,000 · Sep 13
```

---

# 5. AFTER LIVE screen

## Goal

Turn the end of a stream into an actionable checklist.

## Session header

- broadcast date/time
- duration
- today's MVP
- source data-gap status
- overall progress

## Progress

```text
9 / 12 resolved
75%
```

Completed and skipped use distinct visual treatments.

## Task groups

1. Thank-you messages
2. Fan photo/selfie
3. Thank-you videos
4. Promise review
5. Other/content notes

Each task card:

- Viewer avatar/name
- task type
- reason badges
- current status
- primary action

Example reasons:

```text
MVP
50K+ gift
Subscribe event
Open promise
```

## Thank-you message detail

- template selector
- generated preview
- editable text area
- Copy button
- Mark Sent button
- Skip button

No direct TikTok DM automation in v0.1.

## Fan-photo detail

- optional local file link
- Open File/Folder
- Mark Sent
- Skip

## Thank-you video detail

- stage selector: TODO / RECORDING / EDITING / READY / SENT
- optional file link
- memo

---

# 6. HISTORY / MONTHLY screen

## Monthly landing

Primary view should feel like ranking/content, not spreadsheet.

### Podium

- #1 large
- #2/#3 secondary
- #4–#10 compact list

### Useful secondary cards

- new supporters
- rising supporters
- most confirmed sessions
- sessions this month

Avoid implying unsupported audience-retention metrics.

## Session history

Cards/list:

- date
- duration
- MVP
- finalized gift total
- confirmed participants
- After Live completion
- data-gap badge

Click opens session detail.

---

# 7. SETTINGS screen

Sections:

## TikFinity

- WebSocket endpoint
- connection test
- current status
- reconnect behavior display

Default endpoint candidate:

```text
ws://localhost:21213/
```

## Local data

- app data path
- database status
- cache size
- backup
- restore
- full reset

## Timezone

- configured timezone
- default locale-derived/Asia-Seoul for initial Korean setup

## Message templates

- list
- create/edit/delete
- supported placeholder reference

## Tags

- custom tags
- reserved-tag explanation

## After Live rules

P0 may show fixed/default thresholds with limited settings; P1 can provide full rule builder.

---

# 8. Dialogs / drawers

## Start LIVE dialog

- confirms new session
- shows source connection state
- if disconnected, warn but allow explicit start if product chooses

## End LIVE dialog

- confirms session close
- if source disconnected or provisional gifts exist, show warning

## Quick Note

- Viewer fixed
- single focused input
- Save / Cancel

## Quick Promise

- Viewer fixed
- content
- optional due date

## Record Back Support

- Viewer fixed
- currency
- amount
- date/time
- memo

## Fan delete

Must explicitly describe what is deleted/anonymized according to implemented policy.

## Full reset

High-friction confirmation; recommend backup first.

---

# 9. Empty states

Examples:

### No session

```text
No LIVE session is running.
Connect TikFinity and start a session when you go live.
```

### No fan history

Use friendly explanatory empty state, not blank table.

### No post-LIVE work

```text
All clear. No follow-up tasks remain for this session.
```

---

# 10. Accessibility / usability baseline

- visible keyboard focus
- minimum practical contrast
- do not rely on color alone for task/connection states
- target hit areas suitable for fast use during LIVE
- destructive actions require explicit confirmation
- rank/status icons accompanied by text where ambiguity matters
- scrolling must remain smooth with large fan/event history; virtualize if needed

---

# 11. Responsive scope

v0.1 targets desktop first.

Recommended minimum supported working width should be established during implementation after prototype testing. Narrow desktop windows may collapse the detail drawer before collapsing main information hierarchy.

Mobile UI is not v0.1 scope.
