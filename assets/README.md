# Module B assets

Competitor-facing assets for the SwapLoop Admin SSR application. Paths below match `project-description.md`. Files marked **TODO** are not yet in this repository and must be supplied before competition use.

## Planned layout

```text
assets/
  data/
    module-b-seed/          # compact swaploop_admin MySQL seed + manifest
    imports/                # optional CSV samples (communities, stations, telemetry)
  handouts/
    handout-authentication-and-roles.md
    handout-station-service.md
    handout-battery-health.md
  station-service/          # runnable mock Station Service (live technical status)
```

## Included now

- Placeholder directories only (this README and `.gitkeep` files).

## Still required (TODO)

| Asset | Purpose |
| ----- | ------- |
| `data/module-b-seed/` | Compact admin schema seed with staff, communities (incl. not yet covered), stations (incl. suspended), batteries across health bands, open incident, partners, funding ledger |
| `data/imports/` | Optional CSV import samples and invalid-row cases |
| `handouts/handout-authentication-and-roles.md` | Seed staff credentials and role/assignment rules |
| `handouts/handout-station-service.md` | Base URL, endpoints, degraded-mode behaviour |
| `handouts/handout-battery-health.md` | Health-band table + thermal-anomaly override |
| `station-service/` | Independently runnable mock exposing live station / slot / bay / battery technical status |

## Alignment notes

- The application must remain markable with **admin seed + Station Service only**.
- Physical vocabulary: SwapLoop Station, Battery Swap Cabinet, Battery Slot, E-bike Charging Bay.
- Compatibility labels: `SL-48` / `SL-60`, `GB-AC-48` / `GB-AC-60` (do not invent parallel codes).
