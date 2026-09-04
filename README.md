# MediHist — Patient Medical History Portal

A single-file (no build step, no framework) patient medical-history portal backed by
**Supabase** (Postgres + PostgREST). Select a patient chart, log medications, doctor
visits, family history and old reports, and print a full clinical summary.

## Features

- Patient chart selector + global search across patients, meds, diagnoses, visits, reports
- Per-patient tabs: General Vitals (auto BMI), Medications, Doctor Visits, Family History,
  Reports & Files, Printable Clinical Summary
- Full CRUD against Supabase (`patients`, `medications`, `doctor_visits`,
  `family_history`, `medical_reports`)
- Live connection indicator in the header + on-screen translation of Postgres errors
  (RLS `42501`, not-null `23502`, FK `23503`, …)
- Dark / light theme, print-friendly summary, responsive layout

## Repository layout

```
├── index.html          the entire application (HTML + CSS + JS + supabase-js CDN)
├── netlify.toml        Netlify config (publish dir = root, security headers + CSP)
├── supabase/
│   ├── 01_fix_rls.sql                 REQUIRED — RLS policies for the anon key
│   ├── 02_seed_demo_data.sql          optional demo patients
│   └── 03_lock_down_to_logged_in_users.sql   optional: per-user access control
```

## Setup (one time, in the Supabase dashboard)

1. Create the five tables (or keep the ones you already have — the column names used by
   the app are listed in `supabase/01_fix_rls.sql` comments / `02_seed_demo_data.sql`).
2. Open **SQL Editor** and run `supabase/01_fix_rls.sql`. Without it the anon key used by
   a web page cannot read or write the child tables (`42501` errors).
3. Optionally run `supabase/02_seed_demo_data.sql` for demo charts.
4. Put your project URL + publishable key in `index.html`
   (`SUPABASE_URL`, `SUPABASE_ANON_KEY` — already set for this project).

Run locally: just open `index.html`, or `python3 -m http.server` in this folder.

## Deploy

### Option A — GitHub Pages (free, straight from this repo)

1. Push this repo to GitHub (see commands below).
2. Repo → **Settings → Pages**.
3. *Build and deployment → Source:* **Deploy from a branch** → branch `main` → folder `/ (root)`.
4. Wait ~1 min → your site is `https://<username>.github.io/<repo-name>/`.

> GitHub Pages on the free plan only works for **public** repositories
> (private repos need a paid GitHub plan). The app is a single static file with no asset
> paths, so it works from any base URL without extra configuration.

### Option B — Netlify (auto-deploys on every push)

1. <https://app.netlify.com> → **Add new site → Import an existing project → GitHub**.
2. Build command: *(leave empty)* — `netlify.toml` already sets the publish directory
   and the security headers.
3. Deploy. Every subsequent `git push` redeploys automatically.

## Pushing this folder to a new GitHub repo

```bash
git init
git add .
git commit -m "MediHist: Supabase-connected patient history portal"
git branch -M main
git remote add origin https://github.com/<your-username>/<repo-name>.git
git push -u origin main
```

## ⚠️ Security

The publishable/anon key inside `index.html` is **meant to be public** — Supabase's
security boundary is Row Level Security, not key secrecy. What that key can do is exactly
what your policies allow the `anon` role:

- `supabase/01_fix_rls.sql` = fully open (fine for demos with fake data),
- `supabase/03_lock_down_to_logged_in_users.sql` = per-user access after adding
  Supabase Auth (use this before storing real patient records).

Never commit or publish the **secret / service_role** key.
