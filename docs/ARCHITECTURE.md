# Architecture

## 1. Goal

TTBM v0.1 is a Windows-first local desktop application. The architecture must keep TikFinity-specific transport details isolated from the host-facing domain and UI.

Proposed stack:

- Tauri 2
- React
- TypeScript
- SQLite via Tauri SQL plugin
- local filesystem/cache via Tauri filesystem/path APIs

No mandatory cloud backend.

---

## 2. High-level runtime

```text
TikTok LIVE
   ↓
TikFinity Desktop
   ↓ ws://localhost:21213/
TTBM TikFinity Adapter
   ↓
Canonical Event Pipeline
   ↓
Domain Services / Rules
   ↓
Repositories
   ↓
SQLite + local managed files
   ↑
Application Services
   ↑
React UI
```

---

## 3. Dependency direction

Allowed direction:

```text
UI
↓
Application
↓
Domain
↓
Ports / repository interfaces
↑
Infrastructure implementations
```

TikFinity adapter is infrastructure. Domain must not import TikFinity raw payload types.

UI must not parse TikFinity raw JSON.

---

## 4. Suggested source layout

```text
src/
  app/
    router/
    providers/
    shell/

  features/
    live/
    people/
    after-live/
    history/
    settings/

  application/
    session/
    viewers/
    tasks/
    backup/

  domain/
    events/
    viewer/
    session/
    gift/
    participation/
    promise/
    task/
    sent-item/

  integrations/
    tikfinity/
      client.ts
      envelope.ts
      mapper.ts
      reconnect.ts

  db/
    migrations/
    repositories/
    queries/

  shared/
    time/
    ids/
    validation/
    errors/
    ui/

src-tauri/
  src/
  capabilities/
```

Exact paths may change, but dependency boundaries should remain.

---

## 5. TikFinity integration

Responsibilities:

- connect/disconnect/reconnect
- parse JSON envelope
- validate known event envelope
- preserve unknown events
- normalize source fields
- emit canonical events
- expose connection status

Non-responsibilities:

- calculate MVP
- update monthly totals directly
- generate tasks
- decide VIP
- mutate React state as domain authority

---

## 6. Canonical event pipeline

Recommended sequence:

```text
Socket message
→ JSON parse
→ source envelope validation
→ canonical mapper
→ identity resolution
→ idempotency check
→ raw/canonical event persistence
→ event-specific domain processor
→ transaction commit
→ UI query invalidation/update
```

Important ordering:

- domain success should not be shown until durable write succeeds
- gift event + derived gift state should commit atomically where possible
- task generation at session close should be rerunnable

---

## 7. Persistence architecture

SQLite is the system of record for v0.1 local structured data.

Use migrations from the beginning.

Prefer repository/query modules rather than SQL scattered throughout React components.

Example ports:

```ts
interface ViewerRepository {}
interface SessionRepository {}
interface EventRepository {}
interface GiftRepository {}
interface TaskRepository {}
interface BackupService {}
```

Avoid premature generic repository abstractions if they hide useful SQL semantics.

---

## 8. Transaction boundaries

Candidate boundaries:

### Ingest identity-bearing event

```text
resolve/upsert Viewer
+ insert LiveEvent
+ update participation
+ event-specific derived write
= one transaction when practical
```

### Finalize gift

```text
idempotency check
+ streak update/finalization
+ event record
= atomic
```

### End session

Do not make the entire potentially long UI closeout a single giant DB transaction. Use staged idempotent operations:

```text
capture cutoff
→ reconcile gifts
→ generate tasks transactionally/idempotently
→ persist end state
```

If interrupted, reopening app should detect incomplete ENDING state and resume/recover safely.

---

## 9. Query strategy

Do not persist every UI aggregate as authoritative state unless profiling proves necessary.

Start with indexed queries for:

- current-session supporter totals
- monthly supporter totals
- fan timeline
- confirmed participation count
- unresolved task count

If performance later requires projections/materialized counters, they must be rebuildable from authoritative records.

---

## 10. UI state vs persisted state

### Persisted authority

- sessions
- events
- gifts
- viewers
- notes
- promises
- tasks
- sent items
- settings that affect semantics

### Ephemeral UI state

- selected fan
- open drawer
- current filters
- modal open state
- transient toast
- temporary text before save

Do not make React local state the only copy of important post-LIVE progress.

---

## 11. File handling

Two modes may coexist:

### Managed cache

Avatar cache and internally generated/managed files live inside app-owned directories.

### External linked file

For fan photo/video tasks, v0.1 may store a user-selected file path without copying the file.

The UI must distinguish a link/reference from an app-managed copy.

Full reset must not delete arbitrary external linked files.

---

## 12. Backup architecture

Backup service should produce a versioned package.

Candidate content:

```text
manifest.json
 database snapshot
 settings.json (if outside DB)
 optional managed avatar/media cache
```

Manifest fields:

- backup format version
- app version
- schema version
- createdAtUtc
- included components

Restore validates manifest before replacing active data.

---

## 13. Logging

Local diagnostics should include:

- connection transitions
- mapper validation failure
- unknown source event type
- DB errors
- migration errors
- backup/restore failure

Do not log private note content by default.

Do not dump raw sensitive payloads into unbounded logs.

Raw source payloads stored in DB are subject to local retention/deletion policy.

---

## 14. Performance priorities

Order of importance during active LIVE:

1. gift correctness
2. durable event ingestion
3. connection health
4. chat/identity activity
5. UI animation
6. lower-value high-volume engagement events

If like events become extremely high volume, batch/drop non-authoritative UI detail before compromising gift correctness.

---

## 15. Security boundary

The app listens/initiates only local integrations required by product scope.

- do not expose a LAN HTTP server by default
- do not upload fan records by default
- do not execute arbitrary template code
- sanitize/render chat text as text, not trusted HTML
- treat source JSON and linked filenames as untrusted input

---

## 16. Testing architecture

Use layers:

- pure domain unit tests
- canonical mapper fixture tests
- repository integration tests on temporary SQLite DB
- migration tests
- session closeout/task-generation tests
- backup/restore round-trip tests
- UI component tests for critical states
- optional end-to-end desktop smoke after shell exists

See `TEST_ACCEPTANCE.md`.

---

## 17. First implementation slice

Recommended vertical slice:

```text
Tauri shell
→ SQLite migration 001
→ TikFinity connection status
→ Start/End session
→ capture raw chat/gift fixture
→ canonical mapper
→ Viewer identity
→ gift-safe persistence
→ LIVE MVP card
→ PEOPLE profile
```

Do not implement all screens before validating real TikFinity payload contracts.
