# Module B assets

| Path | Purpose |
| ---- | ------- |
| [`db/swaploop_admin.sql`](./db/swaploop_admin.sql) | MySQL seed (`swaploop_admin`) — import before running the app |
| [`wireframes/`](./wireframes/) | Screen wireframes (SVG) referenced by the project description |

## Database seed

- Staff password for every seeded account: `password123` (bcrypt hashes in dump)
- Two subscription plans with different discount tiers; partners assigned across both
- Individual `usage_events` for July and August 2026 — aggregate by calendar month (`Asia/Shanghai`) for billing

## Wireframes

Sixteen SVG screens under [`wireframes/`](./wireframes/). See that folder’s README for the file list.
