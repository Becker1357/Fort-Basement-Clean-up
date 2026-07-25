# Basement Manifest — deploy guide

You have two files:

- **index.html** — the whole app (all 13 photos are baked in).
- **README.md** — this guide.

Deploying is two parts: **(A)** a tiny free database so all 7 people's answers pool into one board, and **(B)** putting the page on a public URL. Budget ~10 minutes.

---

## A. Turn on sharing (Supabase) — ~5 min

Without this step the page still runs, but each person only sees their *own* answers on their *own* device. The database is what pools everyone.

1. Go to **supabase.com** → sign in → **New project** (Free plan). Give it any name + database password. Wait ~2 min for it to finish setting up.

2. Left sidebar → **SQL Editor** → **New query**. Paste this and click **Run**:

   ```sql
   create table if not exists kv (
     key text primary key,
     value jsonb,
     updated_at timestamptz default now()
   );
   alter table kv enable row level security;
   create policy "read"   on kv for select using (true);
   create policy "insert" on kv for insert with check (true);
   create policy "update" on kv for update using (true) with check (true);
   create policy "delete" on kv for delete using (true);
   ```

   (This makes one simple key/value table the app reads and writes.)

3. Left sidebar → **Project Settings** → **API**. Copy two values:
   - **Project URL** — looks like `https://abcdxyz.supabase.co`
   - **anon public** key — the long string under *Project API keys* (the one labeled `anon` / `public`)

4. Open **index.html** in any text editor. The very top of the `<script>` has a CONFIG block:

   ```js
   const SUPABASE_URL = "";   // paste Project URL here
   const SUPABASE_KEY = "";   // paste anon public key here
   ```

   Paste your two values between the quotes and save. That's it — the app now shares.

> The anon key is designed to be public and is safe to ship inside the page. The real "door key" is the URL itself, so only share it with the house. You can tighten access later in Supabase (Auth + Row Level Security) if you ever want to.

---

## B. Put it online — pick ONE

### Option 1 — Netlify drag-and-drop (no terminal)
1. Go to **app.netlify.com** → sign in.
2. **Add new site** → **Deploy manually**.
3. Drag the folder that contains `index.html` onto the page.
4. You get a live URL instantly (rename it under Site settings if you like).

### Option 2 — Surge (fastest via terminal)
```bash
npm install -g surge
cd path/to/basement-manifest-site
surge .
```
Follow the prompts (email + pick a domain). Done.

### Option 3 — GitHub Pages (nice if you're using Claude Code)
```bash
cd path/to/basement-manifest-site
git init && git add . && git commit -m "Basement manifest"
# create an empty repo on github.com first, then:
git remote add origin https://github.com/<you>/<repo>.git
git push -u origin main
```
Then on GitHub: **repo → Settings → Pages → Source: `main` / root → Save.** Your URL appears in ~1 minute.

---

## Share it
Send the URL to the house. Each person opens it, types their name once, and starts tagging. Everything — including any Round-2 photos added from the **Manage** tab — pools in your Supabase project.

## Housekeeping
- **See raw data / wipe it:** Supabase → **Table editor** → `kv`. Or use the app's **Manage ▸ Reset everything**.
- **Back up before a reset:** **Manage ▸ Export everything** copies all answers as text you can paste back in later.
- **Add more photos anytime:** **Manage ▸ Add a photo (round 2)** — everyone sees it.
