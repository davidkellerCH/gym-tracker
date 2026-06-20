# Ironlog

A minimal, offline-friendly gym tracker that lives on your phone's home screen. Log your lifts, see your last few sessions at a glance, and track progress with built-in charts. No app store, no subscription — just a single HTML file backed by a free database, behind a login.

**Live app:** https://davidkellerch.github.io/gym-tracker/

---

## What it does

- **Workout days (A / B / C / D)** — group exercises however you split your training. Rename, add, or remove days anytime.
- **Add exercises by name** — type a name, pick a day, start logging. No fixed list.
- **Fast set logging** — weight + reps per set, with your previous numbers shown as a faded "target to beat." Reps that beat last time tick green.
- **Last 3 sessions inline** — recent history above today's entry, with full history expandable.
- **Per-exercise notes** — keep cues like "push for 15 reps" or "try 47.5 next time."
- **Progress charts** (tap ⋯ → *View progress*):
  - Top-set weight over time
  - Estimated 1RM trend (Epley formula — an estimate, not a literal max)
  - Total volume per session (weight × reps)
- **Works offline** — logging and charts work without signal; data syncs to the cloud when you're back online.
- **Login-protected** — your data is private to your account.
- **Backup export** — download your whole log as JSON anytime (⚙ menu).

---

## How it works

The entire app is **one `index.html` file**. There's no build step and no framework. It uses:

- **GitHub Pages** to host the file for free at a public URL.
- **Supabase** (free tier) for the database and authentication. Your whole log is stored as a single JSON record, owned by your account.
- **Browser storage** as an offline cache, so the app opens and works even with no connection. Changes sync to Supabase once you're online.
- **PWA install** — "Add to Home Screen" in Safari makes it open fullscreen like a native app.

### A note on the public key

This is a static site, so the Supabase URL and **anon key** sit in `index.html` for anyone to see. That's expected: the anon key is designed to be public. Security comes from **login plus row-level rules**, not from hiding the key. Without a valid account, that key can't read or write anything. The repository contains **no personal training data** — all of that lives only in the private Supabase database.

---

## Run your own copy

You don't need access to anyone else's database — set up your own in about 15 minutes. Everything below is free and needs no credit card.

### 1. Create a Supabase project

1. Go to [supabase.com](https://supabase.com) and sign in (GitHub login is easiest).
2. Create a **new project**. Pick a region near you. Wait ~2 min for it to provision.
3. Open the **SQL Editor**, paste this, and click **Run**:

```sql
create table workouts (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references auth.users(id) default auth.uid(),
  data jsonb not null,
  updated_at timestamptz default now()
);

alter table workouts enable row level security;

create policy "own rows" on workouts
  for all using (auth.uid() = user_id) with check (auth.uid() = user_id);
```

This creates a table where each row is owned by a user, and a rule so a signed-in user can only ever read or write their own rows.

### 2. Create your account

In Supabase → **Authentication → Users → Add user**. Enter your email and a password, and tick **Auto Confirm User** so you can sign in right away (no confirmation email). There is deliberately **no sign-up inside the app** — accounts are created here only, so no stranger can register.

### 3. Add your credentials to the app

Open `index.html` and find the config block near the top of the `<script>`:

```javascript
const SUPABASE_URL = "YOUR_SUPABASE_URL";   // e.g. https://xxxxx.supabase.co
const SUPABASE_KEY = "YOUR_SUPABASE_ANON_KEY";
```

Replace the two placeholders with your own values (Project Settings → API). Use the base Project URL — **without** any `/rest/v1/` on the end; the app adds that itself.

### 4. Host it on GitHub Pages

1. Create a new **public** repository on GitHub.
2. Upload your edited `index.html` (**Add file → Upload files → Commit**).
3. Repo **Settings → Pages → Deploy from a branch** → `main` / root → **Save**.
4. After ~1 minute you get a URL like `https://yourname.github.io/your-repo/`.

### 5. Install on your iPhone

1. Open the Pages URL in **Safari**.
2. Tap **Share → Add to Home Screen**.
3. Open it, sign in with the account from step 2, and start logging. A fresh account starts empty — add your days and exercises.

---

## Want it even more private?

- **Private repo:** hides the URL and key entirely, but GitHub Pages on private repos needs a paid plan.
- **Self-hosted proxy:** a small serverless function (e.g. Cloudflare Worker) can hold the keys server-side so they never appear in the browser. More setup; usually unnecessary once login is in place.

---

## Tech notes

- **No dependencies.** Plain HTML, CSS, and vanilla JavaScript. Charts are hand-drawn SVG, so they work fully offline.
- **Auth.** Email + password via Supabase Auth (GoTrue). The browser holds a short-lived access token that auto-refreshes; sign-out clears it.
- **Data model.** One Supabase row per user holds a JSON object: `{ groups: [...], exercises: [{ id, group, name, comment, sessions: [{ date, sets: [{ weight, reps }] }] }] }`.
- **Estimated 1RM** uses the Epley formula: `weight × (1 + reps / 30)`, taken from your best set each session.
- **Offline behaviour.** The app caches your latest data locally. Edits made offline are kept and pushed to Supabase next time it connects.

---

*Built as a personal project. Free to copy and adapt.*
