# WDC Admin Console

A desktop web app for administering the **Wyoming Dinosaur Center** digital collections system. Manages users, roles, and provides access to collections data across all WDC apps.

---

## Overview

The admin console is the control panel for the WDC digital collections platform. Admins invite new users, assign roles, manage passwords, and will access site and specimen data — including intake and cataloging — as those modules are built out.

---

## Features

- **User management** — invite new users via magic link, assign roles, reset passwords, delete accounts
- **Role columns** — users organized by role in a column-per-role layout; click to select, then act
- **Tab navigation** — Users, Sites, Specimens, Info. Intake and Catalog are sub-tabs within the Specimens page
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
wdc-admin/
├── index.html          # Complete single-file admin app
├── onboard.html        # New user account setup (name, phone, address, password)
├── sw.js               # Self-destructing service worker (clears any old caches)
└── README.md           # This file
```

The Edge Function lives in the `wdc-supabase` repo at `supabase/functions/admin-users/index.ts`.

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

Tabs currently showing "coming soon" will be built out in future phases:

- [x] Users — invite, role management, password reset
- [ ] Sites — view, edit, and archive dig sites / localities
- [ ] Specimens — browse and search all specimen records
  - [ ] Intake — digitize paper field sheets (VLM-assisted), review before commit
  - [ ] Catalog — formal cataloging, taxonomy, storage, and Darwin Core export

Audit log / full change history is deferred to a later admin & reporting phase.

---

*Wyoming Dinosaur Center · Thermopolis, Wyoming*
