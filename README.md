# Client's Package Tracker

A simple, phone-friendly app for yoga teachers to track client packages: how many classes a client has paid for, how many have been taught, what has been paid, and when a package is about to end.

Built as a single HTML file (`index.html`) with no frameworks and no build step. It also ships as an Android app (APK) that wraps the same file.

## Features

**Clients and packages**
- Add clients with name, phone, address, joining weight, package start date, package length (days), and number of classes to teach
- Circular progress ring that fills as classes are taught, animated when you open a client
- "Clients" dropdown on the home screen that opens like a shutter

**Tracking classes**
- Mark a class as **taught**, **skipped by instructor**, or **skipped by client** for any date
- Calendar view of the whole package, colour coded: white (upcoming), green (taught), grey (skipped by instructor), pale red (skipped by client)
- Class log with an Edit mode, so dates cannot be removed by accident
- Client name turns grey once every class in the package is taught

**Money**
- Amount to be paid, amount paid, and balance due per client
- Home screen totals for **Total paid** and **To collect**, each tappable to see the per-client breakdown

**Health and weight**
- Joining weight plus dated weight updates, with the change since joining
- Mark each client as **Has disease** (pale red, with optional details) or **No disease** (green)

**Reminders and contact**
- Red reminder banner for clients whose package is ending within a number of days you choose in Settings, or who have two or fewer classes left
- One-tap **call** and **WhatsApp** buttons for each client

**Other**
- Splash screen with logo fade, white and light green theme, fade effect on every tap
- Backup and restore: share your whole client list as text and paste it back to restore

## Ways to use it

| Version | Where data is saved | Notes |
|---|---|---|
| **Web page** (this repo's `index.html`, e.g. GitHub Pages) | In the browser on that device only | Works on iPhone (Safari, then Share, then Add to Home Screen) and Android (Chrome) |
| **Android APK** | On the phone; optionally online if you sign in with Google | Offline-capable; Google sign-in saves data to your own Supabase project |

## Run it

1. Download `index.html`.
2. Open it in any browser, or host it for free with GitHub Pages (Settings, then Pages, then deploy from the `main` branch).
3. On a phone, open the address and use "Add to Home Screen".

## Optional: online saving with Supabase (Android app)

The Android app can save clients online after signing in with Google. This uses your own free [Supabase](https://supabase.com) project.

1. Create a Supabase project and run this in the SQL Editor:

```sql
create table if not exists public.app_data (
  user_id uuid primary key references auth.users(id) on delete cascade,
  data jsonb not null default '{}'::jsonb,
  updated_at timestamptz not null default now()
);

alter table public.app_data enable row level security;

create policy "own select" on public.app_data
  for select to authenticated using (auth.uid() = user_id);
create policy "own insert" on public.app_data
  for insert to authenticated with check (auth.uid() = user_id);
create policy "own update" on public.app_data
  for update to authenticated
  using (auth.uid() = user_id) with check (auth.uid() = user_id);
create policy "own delete" on public.app_data
  for delete to authenticated using (auth.uid() = user_id);
```

2. Create a Google OAuth **Web application** client with the redirect URI `https://<your-project-ref>.supabase.co/auth/v1/callback`, and enable the Google provider in Supabase (Authentication, then Sign In / Providers).
3. Add `com.rye.packagetracker://auth` to Supabase's redirect URLs (Authentication, then URL Configuration).
4. Put your project URL and **publishable** key in the `SB` and `KEY` constants near the top of the script in `index.html`.

Never put a Supabase secret or service-role key in this file. Only the publishable key belongs here, and row-level security (above) keeps each user's data private.

## Data and privacy

- Clients' names, phone numbers, addresses, weights and health notes are personal data. Keep your backups safe.
- Without signing in, data never leaves the device.
- See [PRIVACY.md](PRIVACY.md) for details.

## Limitations

- The web version has no sync between devices. Use Backup and restore to move data.
- Skipped days do not extend a package's end date.
- This is a record-keeping tool, not medical software. Health notes are for your own reference.

## Tech

Plain HTML, CSS and JavaScript in one file. Android wrapper: a minimal WebView app. Optional cloud storage and login: Supabase (Postgres with row-level security, Google OAuth).
