# WDC Collections (Desktop)

The single desktop web app for the **Wyoming Dinosaur Center** digital collections system — sites, specimen catalog, intake, reference data, and user administration. Views are shown or hidden by the signed-in user's role. (Repo is named `admin` for historical reasons; it is the desktop Collections app.)

---

## Overview

One desktop app serves every desk role. Any user with an assigned role can sign in; the header shows their role and the app reveals only the features they need. The **Users** panel is admin-only. The mobile, offline-first **field app** remains a separate app.

---

## Features

- **User management** — invite new users via magic link, assign roles, reset passwords, delete accounts
- **Role columns** — users organized by role in a column-per-role layout; click to select, then act
- **Sites** — interactive map (Leaflet) with a minimap, satellite/topographic/street base layers and a Macrostrat geology overlay; per-species **periodic-table tiles** (diet-colored border, species code, specimen-count badge) at each site; site list + filters (species, epoch, has-specimens, include-archived); right-hand details panel with create/edit and click-the-map GPS
- **Specimens** — the **Catalog** sub-tab browses/searches all occurrences with filters (site, formation, taxon, stage, and a **Needs cataloging** quick filter) and pagination; full Darwin Core record editor (cataloging, taxonomy, event, location, geology, element/condition, measurements, removal, notes, arrays); **photo gallery** (signed URLs + lightbox); **chain-of-custody** timeline with add-event; **storage location** capture (cabinet/drawer/shelf → `in_storage` custody event); **CSV export** with Darwin Core headers. The **Intake** sub-tab is a scaffold for future VLM digitization
- **Reference Data** (under Info) — manage `formations`, `genera` (periodic-table symbol + diet + PBDB/GBIF IDs), `taxa` (species code + diet), `elements`, and `vocabularies`; edits flow to all WDC apps
- **Tab navigation** — Users, Sites, Specimens, Info. Specimens has Catalog + Intake sub-tabs; Reference Data is a sub-tab within Info
- **Role-based access** — any user with a role can sign in; the header shows the role, and the **Users** tab is admin-only. Interns and staff can create and edit sites; reference-data and user management stay restricted. The UI mirrors the rules; the real enforcement is Supabase RLS + the `admin-users` Edge Function
- **No service worker** — intentionally excluded to avoid caching stale UI

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

Any role can sign in to the desktop app; features are gated per role (see the matrix the team maintains). Effective access today:

| Role | Access |
|------|--------|
| `intern` | Field data entry (seasonal); view all; create/edit specimens; **create + edit sites**; intake |
| `staff` | Field collection + cataloging; edit specimens, sites, custody; intake |
| `management` | Edit any record; archive sites; edit reference data; Info tab |
| `researcher` | Read-only |
| `admin` | Full access; **Users** panel (invite / roles / reset / delete) |

*(The former `registrar` role was merged into `staff`.)*

Enforcement lives in Supabase RLS + the `admin-users` Edge Function; the UI mirrors it. Users is the only panel gated in the UI today (admin-only); finer per-action limits are enforced by RLS and tightened over time.

---

## Invite & Password Reset Flow

Both send email automatically via the project's configured SMTP (Resend), with a copyable backup link shown to the admin in case the email doesn't arrive.

1. Admin enters email + role in the Users tab → **+ New User** (or picks a user → **Password**)
2. The `admin-users` Edge Function creates/sets the account and **emails** the recipient a link to `onboard.html`
3. The recipient clicks the link, lands on `onboard.html`, and sets their name, phone, mailing address, and password
4. On submit they are routed to the appropriate app

---

## Edge Function

The `admin-users` function handles all privileged user operations using the Supabase service-role key. It is called from the app with a Bearer token and **verifies the caller is an admin server-side** (403 otherwise) — so relaxing the UI login to all roles does not expose these operations.

Supported actions:

| Action | Description |
|--------|-------------|
| `invite` | Create the user, set role, and **email an invite** (`inviteUserByEmail`) → `onboard.html`; returns a backup link |
| `update_role` | Change a user's role in the `profiles` table |
| `reset_password` | **Email a recovery link** (`resetPasswordForEmail`) → `onboard.html`; returns a backup link |
| `deactivate` | Delete a user account (cannot delete self) |

Email requires a configured SMTP provider (Resend) — see the `supabase` repo README. Redeploy after changes: `supabase functions deploy admin-users`.

---

## Setup

### 1. Supabase configuration

Deploy the schema from the `supabase` repo (`schema.sql`, then `reference-schema.sql`, then `reference-genera.sql` — see that repo's README).

In **Authentication → URL Configuration**:
- **Site URL**: `https://{your-github-org}.github.io`
- **Redirect URLs**: include `https://{your-github-org}.github.io/**` (must cover `.../admin/onboard.html`, or invite/reset redirects are blocked)

### 2. Email (SMTP)

Invite and password-reset emails send through the project's SMTP provider. In **Authentication → Emails / SMTP**, enable custom SMTP (Resend) with a **verified sender address/domain**. Custom SMTP is available on the Supabase free tier; the built-in sender is capped at ~2 emails/hour and won't work for real onboarding.

### 3. Edge Function secrets

Set these in **Supabase → Edge Functions → Secrets**:

| Secret | Value |
|--------|-------|
| `SITE_URL` | `https://{your-github-org}.github.io/admin/` |
| `FIELD_URL` | `https://{your-github-org}.github.io/field/` |

Deploy with `supabase functions deploy admin-users`.

### 4. App configuration

Update credentials in `index.html` and `onboard.html`:

```javascript
const SUPABASE_URL  = 'https://your-project.supabase.co';
const SUPABASE_ANON = 'your-anon-public-key';
```

### 5. Deploy

Push to GitHub. Enable **GitHub Pages** on the repo. The app is live at `https://{org}.github.io/admin/`.

### 6. Make yourself admin

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
