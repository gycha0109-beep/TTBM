# Local Data & Privacy

## 1. Purpose

TTBM stores fan-facing operational data on the host's device. This document defines the default local-data posture and deletion/export expectations.

---

## 2. Default posture

- no mandatory cloud account
- no mandatory cloud database
- no automatic fan-data upload
- no silent analytics containing fan notes or message content
- host controls backup location
- linked external media remains outside TTBM unless explicitly copied into managed storage

---

## 3. Data categories

### Source-derived public/profile data

Examples:

- platform user ID
- handle
- nickname/display name
- avatar URL
- public LIVE activity events

### Host-authored private data

Examples:

- notes
- promises
- back-support records
- custom tags
- private task memos

Host-authored private data should receive the strictest local-only treatment.

### Operational metadata

Examples:

- sessions
- connection gaps
- task status
- backup metadata

### Raw source payloads

May contain more data than the UI uses. Retain only when needed for debugging/audit/contract stability and review retention policy before production release.

---

## 4. Storage locations

Implementation must use platform application-data directories rather than arbitrary project folders.

Logical layout candidate:

```text
TTBM app data/
  ttbm.db
  avatars/
  managed-media/
  logs/
  backups/ (only if user chooses app location)
```

Exact OS paths are implementation-time facts and must be documented once Tauri app identifiers are frozen.

---

## 5. Notes and sensitive context

TTBM cannot control what a host types into freeform notes, so notes may contain sensitive information.

Requirements:

- notes stay local by default
- notes are not written to normal diagnostics/logs
- notes are included in backup only because the user explicitly backs up their TTBM data
- fan-data deletion removes associated notes under the implemented policy

---

## 6. Chat text

If full chat text is retained, it increases data volume and privacy sensitivity.

Before production, explicitly choose one policy:

### Option A — retain full chat text

Pros: richer timeline/search.
Cons: larger/sensitive local dataset.

### Option B — retain only selected/important chat text

Pros: lower retention burden.
Cons: less complete relationship history.

### Option C — retain chat events/counts but expire raw text after a retention period

Potential balanced approach.

v0.1 implementation must not accidentally make an undecided retention policy permanent.

---

## 7. Raw payload retention

Raw payloads are useful during early development, especially while TikFinity field contracts are being discovered.

Recommended phases:

- development: retain fixtures/raw diagnostic samples locally
- production: minimize raw payload retention or define bounded retention if the normalized record is sufficient

Do not upload raw payloads automatically.

---

## 8. Avatar cache

Avatar cache is a convenience copy.

- safe to clear without deleting core Viewer identity
- refreshed when source URL changes/when cache is stale according to future policy
- removed when full reset occurs
- fan deletion should remove fan-specific cached avatar if no longer referenced

---

## 9. External file links

When TTBM stores a path to a user-selected image/video:

- path is metadata only unless user explicitly imports/copies file into managed storage
- deleting TTBM task/history must not delete the original external file by default
- full reset must not recursively delete user folders outside app-managed storage

---

## 10. Backup

Backup may contain private fan notes and support history.

Requirements:

- clearly tell the user backup contains TTBM data
- user chooses destination
- no automatic cloud upload in v0.1
- versioned format and manifest
- consider optional encryption only after threat model/product need is defined; do not claim encrypted backups unless implemented and tested

---

## 11. Fan data deletion

The fan-profile UI must offer an explicit delete action.

The final behavior must match `DOMAIN_DATA_MODEL.md` and be stated in confirmation text.

At minimum:

- delete host-authored notes/promises/back-support/sent metadata for that Viewer as defined
- remove mutable personal profile display fields
- remove fan-specific avatar cache
- handle historical source/aggregate references according to documented anonymization/deletion implementation

Do not present deletion as complete if identifiable copies remain in managed storage outside the documented policy.

---

## 12. Full reset

Full reset removes all TTBM-managed local state:

- DB
- settings
- managed cache
- managed local media, if confirmation explicitly includes it

It does not delete arbitrary externally linked files.

Before reset, UI should offer backup.

---

## 13. Logs

Diagnostics must avoid:

- private note content
- full message templates with private edits unless required for explicit debug export
- unrestricted raw chat dumps
- secrets/tokens if any integration credentials appear later

Logs should be bounded/rotated once production logging exists.

---

## 14. Telemetry

v0.1 should operate without mandatory telemetry.

If optional telemetry is introduced later:

- opt-in/out behavior must be explicit
- never send fan notes, chat text, raw payloads, viewer IDs, handles, gift histories or linked filenames as analytics payloads without a separately reviewed policy

---

## 15. Privacy acceptance gates

Before release verify:

- app works with network unavailable for historical browsing
- no cloud account required
- fan delete removes documented local private records
- full reset removes managed state
- avatar cache clears safely
- external linked media survives reset unless explicitly imported/managed and deletion was disclosed
- logs do not contain note content in ordinary operation
- backup content and destination are user-visible/controlled
