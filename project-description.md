# Test Project Outline – Module B – SwapLoop Admin SSR Application

## Competition time

Competitors will have **3 hours** to complete this module.

## Introduction

SwapLoop is a fictional Shanghai community pilot exploring safer alternatives to charging e-bike batteries indoors. Compatible delivery and private e-bikes exchange removable batteries at swap stations; e-bikes with integrated batteries use monitored charging bays; delivery partners can receive controlled priority access; and operators and safety inspectors manage sites, assets, and incidents.

This competition does **not** ask for a finished production platform. This module is a **working prototype** of the **operator / safety / funding console**: a server-rendered administration application that enrols communities, manages station lifecycle, reviews battery health and incidents, configures delivery-partner priority windows, and tracks grant / co-funding / rollout progress.

The application must run **independently**. It reads and writes a compact admin-oriented database seed and queries **live technical state** from the provided **Station Service**. It must not implement or depend on rider-facing REST APIs, QR reservation flows, or SPA rider journeys.

## General Description of Project and Tasks

Implement an independently runnable SSR administration application. Assessors must be able to mark it using only the admin seed and Station Service. Predictable role scoping, clear form validation, and honest operational state (including degraded Station Service mode) are essential.

The following is a high-level overview of the required capabilities; detailed specifications are in the [Requirements](#requirements) section:

- **Staff sign-in:** simple email/password (or seed credentials) with a server-side session; navigation and mutations are role-scoped
- **Community enrolment:** advance communities through enrolment states, including an explicit **not yet covered** label when there is no nearby active station
- **Station lifecycle:** advance stations through lifecycle states with auditable transitions and explicit suspension reasons
- **Live capacity:** refresh Battery Slot and E-bike Charging Bay status from the Station Service (never invent green capacity)
- **Battery fleet:** list batteries by health band; quarantine and retire workflows; thermal-anomaly override always wins
- **Incidents:** assign, inspect, resolve or escalate safety and operational incidents without erasing history
- **Delivery partners:** manage partners and priority windows; priority is capacity preference only, never a safety bypass
- **Funding / rollout:** show grant spend, partner co-funding, revenue inputs, a **derived** grant dependency ratio, and progress toward rollout targets
- **Imports (secondary):** optional CSV import of communities, stations, and historical telemetry when assets include them

### Environment and stack

Build the application using a server-side language and framework available in the competition environment.

- Render pages on the server (HTML forms and redirects). Client-side JavaScript may enhance usability, but the initial content of each page must be server-rendered.
- Use **MySQL** (or the competition-standard relational database) for persistence.
- Import the supplied admin seed from [`assets/data/module-b-seed/`](./assets/data/module-b-seed/). Dataset metadata and restore steps are described in that folder’s `manifest.json` when provided.
- Query live station / cabinet / slot / bay / battery technical status from the provided Station Service under [`assets/station-service/`](./assets/station-service/). Follow the handout at [`assets/handouts/handout-station-service.md`](./assets/handouts/handout-station-service.md) for base URL, endpoints, and degraded-mode expectations.
- Use the fixed staff identities and roles in [`assets/handouts/handout-authentication-and-roles.md`](./assets/handouts/handout-authentication-and-roles.md).
- Use the health-band rules in [`assets/handouts/handout-battery-health.md`](./assets/handouts/handout-battery-health.md) when displaying or applying quarantine / retire decisions.
- Optional import CSV samples (when provided) are under [`assets/data/imports/`](./assets/data/imports/).

The supplied assets are intended to be complete for marking and use only deterministic local data. Until individual asset files are present in this repository, treat the paths above as the required layout assessors will use.

### Technical constraints

- Use `Asia/Shanghai` as the business timezone. Display and store timestamps as ISO 8601 strings with an explicit offset where shown in detail views.
- Persist administrative state in the admin database. Live bay / slot / charging-unit / battery telemetry must be refreshed from the Station Service, not invented in the UI.
- Do **not** call external rider-facing REST APIs. Prefer the identifiers and status fields supplied in the admin seed and Station Service.
- Do **not** model QR reservation flows, idempotency keys, or rider service-session internals as primary admin entities. Prefer direct operator-oriented status fields from the admin seed and Station Service.
- Role-based authorization must protect every mutating form. Suspended staff accounts cannot sign in.
- Server-side validation must produce clear field errors. Prefer accessible forms and tables (labels, focus order, colour-independent status).
- Real payment processing, real government integrations, real IoT hardware control, machine learning, electrochemical simulation, OAuth / SSO, email verification, and complex password-reset flows are out of scope.

### Physical vocabulary

Use this physical vocabulary consistently:

| Term | Meaning |
| ---- | ------- |
| **SwapLoop Station** | Full service location (`SWAP`, `CHARGING`, or `HYBRID`) |
| **Battery Swap Cabinet** | Equipment that stores and charges swappable batteries (may contain multiple slots) |
| **Battery Slot** | One compartment that holds at most one battery (admin/DB unit often labelled swap bay / `SWAP_BAY`) |
| **E-bike Charging Bay** | Bay that charges a whole e-bike with an integrated battery (`BIKE_BAY`) |

Compatibility catalog (do not invent parallel connector codes):

- Swappable packs: `batteryType` `SL-48` \| `SL-60` (derived `voltageClass` `48V` / `60V`)
- Integrated charging: `connectorType` `GB-AC-48` \| `GB-AC-60`

### Framing guardrails

- Enrolment is a **site** process — never score residents, riders, or property managers.
- Uncovered / unenrolled communities are labelled **“not yet covered”**, never “unsafe” or “non-compliant”.
- Tone is operational and safety-forward — no fear-based “fire scoreboard”, no blaming riders or communities.
- Priority windows never bypass compatibility or safety checks — they are capacity preference only.
- Grant dependency ratio is **derived** from ledger inputs — never a free-hand editable “score”.

### Roles

| Role | Scope |
| ---- | ----- |
| `OPERATOR_ADMIN` | District-wide or multi-station operations — community enrolment, station lifecycle, partner setup, funding and rollout dashboards |
| `STATION_OPERATOR` | Assigned station(s) — day-to-day capacity view, maintenance notes, local incident triage |
| `SAFETY_INSPECTOR` | Incident review, quarantine / retirement decisions, inspection notes and resolution |
| `PARTNER_OPERATOR` | Optional; limited to their delivery partner’s priority windows and co-funding context — never bypasses safety rules |

Rider self-registration and rider-facing SPA flows are **out of scope**. Fleet courier accounts may be listed for partner context but are not provisioned in this module.

## Requirements

The SwapLoop admin application shall implement the behaviours below.

### Staff sign-in and role scoping

1. Staff signs in with email/password (or the seed credentials in the authentication handout) against the admin database.
2. The session is server-side (cookie/session). No OAuth and no external bearer-token API are required.
3. Navigation and mutations are role-scoped (own stations / communities / partner only where assigned).
4. Suspended staff accounts cannot sign in.
5. Provide a clear signed-in shell with role-aware navigation, plus explicit access-denied and empty states.

### Community enrolment queue

Communities (residential compounds / mixed-use sites) move through enrolment states, for example:

`APPLIED` → `SITE_SURVEY` → `FUNDING_AGREED` → `INSTALLED` → `COMMISSIONED` → `ACTIVE`

Also include **`NOT_YET_COVERED`** for communities without an active station within their declared service radius.

Requirements:

- Provide an enrolment queue and a community detail view.
- Operators can record survey notes, approximate demand, and funding split when agreeing funding.
- A community without nearby active coverage is shown as **not yet covered**.
- Never score or rank residents or property managers.

### Station lifecycle

Operators advance stations through lifecycle states consistent with the system story, for example:

`PLANNED` / `INSTALLED` / `COMMISSIONED` / `ACTIVE` / `SUSPENDED` / `DECOMMISSIONED`

Station types: `SWAP`, `CHARGING`, `HYBRID`.

Requirements:

- Provide station list and detail views with lifecycle transition forms.
- Suspended stations show an explicit suspension reason (e.g. electrical inspection).
- Lifecycle transitions are auditable (who / when / from → to).
- Live bay / slot / charging-unit status is refreshed from the **Station Service**, not invented in the UI.
- The admin UI must present honest operational state for a suspended station (never as healthy reservable rider capacity).

### Live capacity view

- Show Battery Slots and E-bike Charging Bays for a selected station using Station Service readings.
- Group or label capacity so operators can distinguish swap slots from bike charging bays and see faulted / blocked / unavailable units clearly.
- If the Station Service is unreachable, show a clear degraded-mode message and do **not** invent green capacity.

### Battery fleet and health

Operators and inspectors manage the tracked battery fleet:

- List batteries with lifecycle state and **health band** (`HEALTHY`, `WATCH`, `QUARANTINE`, `RETIRE`, `UNKNOWN`).
- Provide a quarantine queue and retirement workflow.
- Apply versioned health assessments derived from the competition health-band table; a **thermal-anomaly override always wins**.
- Stale / unknown health is a distinct state — never silently treated as healthy.
- Quarantine and retire actions must remove or flag batteries so they are not presented as healthy fleet assets.
- Optional CSV import of historical telemetry or fleet snapshots if provided in assets.

The application may display health results and apply administrative quarantine / retire actions. It uses Station Service readings and/or admin seed fields for health display and fleet actions; full telemetry ingest pipelines for a rider API are out of scope.

### Incidents and inspection

- List open and resolved incidents (thermal anomaly, charging safety cutoff, operational faults).
- Assign an inspector, record inspection notes, resolve or escalate.
- Quarantine or block affected batteries / units when policy requires it.
- Preserve incident history (append notes; do not erase prior resolutions).

### Delivery partners and priority windows

- Manage delivery-partner accounts (name, fleet size, status).
- Configure **priority windows** per partner and station (local time range, reserved capacity share, effective dates).
- Show co-funding contribution against stations / periods where seed data provides it.
- UI copy and validation must make clear that priority never bypasses compatibility or safety checks.

### Funding and rollout dashboard

This module owns funding / rollout analytics in the admin database:

| In | Out |
| -- | --- |
| District safety grant (fixed budget) | Station installation / hardware |
| Delivery-partner co-funding | Operating gap subsidy |

The dashboard must show:

- grant spend vs partner co-funding vs subscription / pay-as-you-go revenue (seeded or imported figures);
- **grant dependency ratio** as a **derived** value (recomputed from ledger inputs — never a free-hand editable score);
- rollout targets: coverage rate, grant dependency trend, incident rate trend;
- progress toward grant phase-out (network becoming self-sustaining).

### Imports and operational overview

- When import CSVs are provided, validate rows and report actionable errors without corrupting existing seed data.
- Provide overview pages: station network table (map widgets are secondary), capacity summary from Station Service, open incidents, and enrolment backlog.

### Required screens / areas

- Staff login and signed-in shell (role-aware navigation)
- Community enrolment queue and community detail
- Station list / detail and lifecycle transition forms
- Live capacity view (Battery Slots + E-bike Charging Bays via Station Service)
- Battery fleet register, quarantine queue, retirement actions
- Incident list / detail / resolve
- Delivery partners and priority-window configuration
- Funding / grant dependency / rollout dashboard
- Optional CSV import results and validation errors
- Access denied / empty / suspended-station states

### Admin database seed expectations

Expected entity groups (names may be adjusted to the provided seed):

- Staff users and roles
- Districts / communities (enrolment state, service radius)
- Stations (type, lifecycle, location, suspension reason)
- Units / slots / charging bays (admin status fields)
- Batteries and health assessments
- Incidents
- Delivery partners and priority windows
- Funding periods / ledger entries / rollout targets

The seed should include communities in multiple enrolment states (including **not yet covered**), at least one **suspended** station, batteries across health bands, and at least one open incident suitable for inspector marking. Provide a deterministic restore path (script or documented import) for assessment.

### Independence

- Markable using only the provided admin seed and Station Service.
- No requirement to call external rider-facing APIs or a separate main backend.

### Delivery priority

**Must have for competition core:** staff auth + roles, community enrolment, station lifecycle, Station Service–backed capacity view, battery quarantine/retire, incident resolve, basic funding/rollout dashboard.

**Secondary:** rich CSV tooling, partner-operator self-service, map widgets, deep telemetry charting.

## Assessment

The solution will be assessed in the latest version of Google Chrome through manual testing against the provided seed and Station Service, database inspection, and expert review. Assessment focuses on observable behaviour rather than the chosen framework.

At minimum, assessors will verify that:

- an operator can sign in and only see / mutate resources allowed by their role (or assignment)
- a community can be moved through enrolment states; a community without nearby active coverage is shown as **not yet covered**
- a station can be suspended with a reason and later returned to active (or decommissioned) with an audit trail
- live capacity for Battery Slots and E-bike Charging Bays is loaded from Station Service (or clearly shows degraded mode if unreachable)
- quarantine and retire workflows remove or flag batteries so they are not presented as healthy fleet assets
- a thermal-anomaly or safety-cutoff related incident can be assigned, inspected, and resolved by a safety inspector
- a partner priority window can be created/edited for a station without implying a safety bypass
- grant dependency ratio is computed from funding inputs, not stored as an arbitrary editable score
- the rollout dashboard shows coverage / dependency / incident trends against targets
- the application runs against its admin seed + Station Service without external backend dependencies
- UI vocabulary matches SwapLoop Station / Battery Swap Cabinet / Battery Slot / E-bike Charging Bay
- CSV import (when provided) validates rows and reports actionable errors without corrupting existing seed data
- forms use server-side validation and basic OWASP-minded protections appropriate to an SSR admin (e.g. CSRF on mutating forms, escaped output, parameterized queries / ORM)

## Mark distribution

The following is a draft distribution. Final criterion-level points must be defined in `marking/marking-scheme.json`.

| WSOS SECTION | Description                            | Points |
| ------------ | -------------------------------------- | -----: |
| 1            | Work organization and self-management  |      5 |
| 2            | Communication and interpersonal skills |      5 |
| 3            | Design Implementation                  |     10 |
| 4            | Front-End Development                  |     25 |
| 5            | Back-End Development                   |     55 |
| **Total**    |                                        | **100** |

## Out of scope

- Rider-facing SPA flows
- Rider reservation / QR / idempotent swap-confirm APIs
- Real payment processing and real government integrations
- Physical lock / charger control and real IoT devices
- Camera-based QR scanning
- Citywide interactive map / motion storytelling
- Campaign marketing site
- OAuth / SSO / email verification / password reset complexity beyond simple staff login
- Scoring or ranking of people or businesses
