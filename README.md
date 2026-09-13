# TTBM — TikTok Broadcast Manager

**TTBM (TikTok Broadcast Manager)** is a local-first desktop workspace for TikTok LIVE hosts.

The product direction is to capture identifiable LIVE activity locally, build fan-centered history, and turn each finished broadcast into a clear post-LIVE workflow for thank-you messages, fan selfies/photos, thank-you videos, promises, notes, and follow-up tasks.

## Current phase

Planning and contract definition. No production implementation exists yet.

## Documentation

All product and implementation-preflight documents live under [`docs/`](./docs/).

Start with:

1. `docs/PRODUCT_REQUIREMENTS.md`
2. `docs/USE_CASES.md`
3. `docs/TIKFINITY_EVENT_CONTRACT.md`
4. `docs/DOMAIN_DATA_MODEL.md`
5. `docs/STATE_AND_RULES.md`
6. `docs/SCREEN_SPEC.md`
7. `docs/ARCHITECTURE.md`
8. `docs/TEST_ACCEPTANCE.md`

## Working principles

- Local-first; no mandatory cloud database.
- White-theme, people-centered UI rather than spreadsheet/admin UI.
- TikFinity is treated as an adapter/source, not as the domain model.
- Raw source events are preserved where useful; UI and business rules use normalized internal events.
- Data gaps and source uncertainty must be visible rather than silently guessed away.
- MVP avoids AI, TikTok DM automation, bank-account scraping, and full video editing.
