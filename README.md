# WDC Admin Console

A desktop web app for administering the **Wyoming Dinosaur Center** digital collections system. Manages users, roles, and provides access to collections data across all WDC apps.

---

## Overview

The admin console is the control panel for the WDC digital collections platform. Admins invite new users, assign roles, manage passwords, and will access site and specimen data — including intake and cataloging — as those modules are built out.

---

## Features

- **User management** — invite new users via magic link, assign roles, reset passwords, delete accounts
- **Role columns** — users organized by role in a column-per-role layout; click to select, then act
- **Sites** — interactive map (Leaflet) with a minimap, satellite/topographic/street base layers and a Macrostrat geology overlay; per-species **periodic-table tiles** (diet-colored border, species code, specimen-count badge) at each site; site list + filters (species, epoch, has-specimens, include-archived); right-hand details panel with create/edit and click-the-map GPS
- **Specimens** — the **Catalog** sub-tab browses/searches all occurrences with filters (site, formation, taxon, stage, and a **Needs cataloging** quick filter) and pagination; full Darwin Core record editor (cataloging, taxonomy, event, location, geology, element/condition, measurements, removal, notes, arrays); **photo gallery** (signed URLs + lightbox); **chain-of-custody** timeline with add-event; **storage location** capture (cabinet/drawer/shelf → `in_storage` custody event); **CSV export** with Darwin Core headers. The **Intake** sub-tab is a scaffold for future VLM digitization
- **Reference Data** (under Info) — manage `formations`, `genera` (periodic-table symbol + diet + PBDB/GBIF IDs), `taxa` (species code + diet), `elements`, and `vocabularies`; edits flow to all WDC apps
- **Tab navigation** — Users, Sites, Specimens, Info. Specimens has Catalog + Intake sub-tabs; Reference Data is a sub-tab within Info
- **Admin-only access** — non-admin accounts are rejected at sign-in with a clear error message
- **No service worker** — intentionally excluded to avoid caching stale admin UI

---

## Tech Stack

| Layer | Service |
|-------|---------|
| Database & Auth | [Supabase](https://supabase.com) (PostgreSQL + Auth) |
| User Management API | Supabase Edge Function (`admin-users`) |
| Hosting | [GitHub Pages](https://pages.github.com) |
| Runtime | Vanilla JS — no framework, no build step |

---

## Project Structure

```
admin/
├── index.html          # Complete single-file admin app (Users, Sites, Specimens, Info)
├── onboard.html        # New user account setup (name, phone, address, password)
├── sw.js               # Self-destructing service worker (clears any old caches)
├── assets/
│   └── dino/           # (legacy) dinosaur flag icons — superseded by CSS letter tiles
└── README.md           # This file
```

Third-party libraries (Supabase JS, Leaflet + MiniMap) load from CDN. The Edge Function lives in the `supabase` repo at `supabase/functions/admin-users/index.ts`.

Maps use free, no-key tile sources: Esri World Imagery (satellite), OpenTopoMap (topographic), OpenStreetMap (street), and Macrostrat (geology overlay).

---

## User Roles

| Role | Description |
|------|-------------|
| `intern` | Field data entry |
| `staff` | Field data entry |
| `registrar` | Collections management; audit log access |
| `management` | Edit any record; archive sites; audit log access |
| `researcher` | Read access |
| `admin` | Full access; can use this admin console |

---

## Invite Flow

1. Admin enters email and role in the Users tab → clicks **+ New User**
2. The `admin-users` Edge Function creates the auth account and generates a magic link pointing to `onboard.html`
3. Admin copies the link and sends it to the new user
4. New user clicks the link, lands on `onboard.html`, sets their name, phone, mailing address, and password
5. On submit, they are routed to the field app (non-admin) or admin app (admin)

---

## Edge Function

The `admin-users` function handles all privileged user operations using the Supabase service role key. It is called from the admin app with a Bearer token (the signed-in admin's access token).

Supported actions:

| Action | Description |
|--------|-------------|
| `invite` | Create user account + generate onboarding magic link |
| `update_role` | Change a user's role in the `profiles` table |
| `reset_password` | Send a password reset email |
| `deactivate` | Delete a user account |

---

## Setup

### 1. Supabase configuration

Ensure the database schema is deployed (see `wdc-supabase` or `wdc-field` repos for `schema.sql`).

In **Authentication → URL Configuration**:
- **Site URL**: `https://{your-github-org}.github.io`
- **Redirect URLs**: `https://{your-github-org}.github.io/**`

### 2. Edge Function secrets

Set these in **Supabase → Edge Functions → Secrets**:

| Secret | Value |
|--------|-------|
| `SITE_URL` | `https://{your-github-org}.github.io/admin/` |
| `FIELD_URL` | `https://{your-github-org}.github.io/field/` |

### 3. App configuration

Update credentials in `index.html` and `onboard.html`:

```javascript
const SUPABASE_URL  = 'https://your-project.supabase.co';
const SUPABASE_ANON = 'your-anon-public-key';
```

### 4. Deploy

Push to GitHub. Enable **GitHub Pages** on the repo. The app is live at `https://{org}.github.io/admin/`.

### 5. Make yourself admin

```sql
update profiles set role = 'admin'::user_role
where email = 'your@email.com';
```

---

## Roadmap

- [x] Users — invite, role management, password reset
- [x] Reference Data — formations, genera, taxa, elements, vocabularies
- [x] Sites — map with species tiles, filters, create/edit
- [x] Specimens — browse/search, full Darwin Core editor, photos, chain of custody, storage location, CSV export
  - [x] Catalog — browse/search + full record editor (with a "Needs cataloging" quick filter)
  - [ ] Intake — VLM-assisted field-sheet digitization (scaffold in place; not yet wired to a vision model)
- [ ] Darwin Core Archive (DwC-A) ZIP export (occurrence.csv + eml.xml + meta.xml) for iDigBio/GBIF
- [ ] Audit log / full change-history views
- [ ] Loan management

Reference-data tables are seeded by `reference-schema.sql` and `reference-genera.sql` in the `supabase` repo — run those (after `schema.sql`) before the Sites/Specimens/Reference Data screens will show data.

---

*Wyoming Dinosaur Center · Thermopolis, Wyoming*
