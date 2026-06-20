# Ironlog

A minimal, offline-friendly gym tracker that lives on your phone's home screen. Log your lifts, see your last few sessions at a glance, and track progress with built-in charts. No app store, no subscription — just a single HTML file backed by a free database.

**Live app:** https://davidkellerch.github.io/gym-tracker/

---

## What it does

- **Workout days (A / B / C / D)** — group exercises however you split your training. Rename, add, or remove days anytime.
- **Add exercises by name** — type a name, pick a day, start logging. No fixed list.
- **Fast set logging** — weight + reps per set, with your previous numbers shown as a faded "target to beat." Reps that beat last time tick green.
- **Last 3 sessions inline** — see recent history above today's entry, with full history expandable.
- **Per-exercise notes** — keep cues like "push for 15 reps" or "try 47.5 next time."
- **Progress charts** (tap ⋯ → *View progress*):
  - Top-set weight over time
  - Estimated 1RM trend (Epley formula — an estimate, not a literal max)
  - Total volume per session (weight × reps)
- **Works offline** — logging and charts work without signal; data syncs to the cloud when you're back online.
- **Backup export** — download your whole log as JSON anytime (⚙ menu).

---

## How it works

The entire app is **one `index.html` file**. There's no build step and no framework. It uses:

- **GitHub Pages** to host the file for free at a public URL.
- **Supabase** (free tier) as the database. Your whole log is stored as a single JSON record.
- **Browser storage** as an offline cache, so the app opens and works even with no connection. Changes sync to Supabase automatically once you're online.
- **PWA install** — "Add to Home Screen" in Safari makes it open fullscreen like a native app.

Your data is just a JSON object: a list of workout days and a list of exercises, each with dated sessions of sets. Simple and portable.

---

## Run your own copy

You don't need access to anyone else's database — set up your own in about 10 minutes. Everything below is free and needs no credit card.

### 1. Create a Supabase project

1. Go to [supabase.com](https://supabase.com) and sign in (GitHub login is easiest).
2. Create a **new project**. Pick a region near you. Wait ~2 min for it to provision.
3. Open the **SQL Editor**, paste this, and click **Run**:

```sql
create table workouts (
  id uuid default gen_random_uuid() primary key,
  data jsonb not null,
  updated_at timestamptz default now()
);

insert into workouts (data) values ('{}'::jsonb);

alter table workouts enable row level security;

create policy "public access" on workouts
  for all using (true) with check (true);
```

4. Go to **Project Settings → API** and copy two values:
   - **Project URL** (looks like `https://xxxxx.supabase.co`)
   - **anon public** key (a long string under "Project API keys")

### 2. Add your credentials to the app

Open `index.html` and find the config block near the top of the `<script>`:

```javascript
const SUPABASE_URL = "YOUR_SUPABASE_URL";   // e.g. https://xxxxx.supabase.co
const SUPABASE_KEY = "YOUR_SUPABASE_ANON_KEY";
```

Replace the two placeholders with your own values. Use the base Project URL — **without** any `/rest/v1/` on the end; the app adds that itself.

### 3. Host it on GitHub Pages

1. Create a new **public** repository on GitHub.
2. Upload your edited `index.html` to it (**Add file → Upload files → Commit**).
3. In the repo: **Settings → Pages → Deploy from a branch**, choose `main` / root, **Save**.
4. After ~1 minute you'll get a URL like `https://yourname.github.io/your-repo/`.

### 4. Install on your iPhone

1. Open the Pages URL in **Safari**.
2. Tap **Share → Add to Home Screen**.
3. It now opens fullscreen like a native app.

On first launch with an empty database, the app starts blank — just add your days and exercises. (This repo's copy is pre-seeded with the owner's starting log; a fresh Supabase project of your own will start empty.)

---

## A note on privacy

Because the repo is **public**, the Supabase URL and anon key in `index.html` are visible to anyone who looks. The anon key is designed to be public for browser apps, but combined with the open table policy above, **anyone who has the URL and key could read or write that database.**

For a personal gym log this is usually an acceptable trade-off. If you want it locked down, options include:

- Making the GitHub repo private (note: GitHub Pages on private repos requires a paid plan).
- Replacing the open policy with Supabase **Auth** so only a signed-in user can read/write.

If you'd like the locked-down version, that's a reasonable next step — it's a moderate change to both the SQL policy and the app's data calls.

---

## Tech notes

- **No dependencies.** Plain HTML, CSS, and vanilla JavaScript. Charts are hand-drawn SVG, so they work fully offline.
- **Data model.** One Supabase row holds a JSON object: `{ groups: [...], exercises: [{ id, group, name, comment, sessions: [{ date, sets: [{ weight, reps }] }] }] }`.
- **Estimated 1RM** uses the Epley formula: `weight × (1 + reps / 30)`, taken from your best set each session.
- **Offline behaviour.** The app caches your latest data locally. Edits made offline are kept and pushed to Supabase next time it connects.

---

*Built as a personal project. Free to copy and adapt.*
