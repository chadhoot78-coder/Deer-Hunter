# Whitetail Tracker

Single-page deer hunt log (PWA) for LDWF Area 2. Hunts live in the browser's localStorage, cam photos in IndexedDB, weather from Open-Meteo (free, no key).

## Files
- `index.html` — the whole app (Log · History · Insights · Cam · Settings)
- `sw.js` — service worker for offline use. **Bump `CACHE` on every index.html change.**
- `manifest.json`, `icon.png` — Add to Home Screen support

## Deploy (GitHub → Vercel)
1. Put these four files at the root of the repo and push.
2. Vercel: New Project → import the repo → Framework "Other", no build command → Deploy.
3. Open the URL in Safari on iPhone → Share → Add to Home Screen.

Any static HTTPS host works (Netlify, GitHub Pages, Cloudflare Pages, nginx). HTTPS is required for geolocation and the service worker.

## Updating
Edit `index.html`, change `CACHE` in `sw.js` (v3 → v4 …), push. Phones pick up the new version on the next open with a connection.

## Optional: share hunts with partners (Supabase)
One person sets this up once; everyone enters the same values under Settings → Share.

1. Create a free project at supabase.com. Copy the **Project URL** and the **anon public** key (Settings → API).
2. SQL Editor → run:

```sql
create table public.hunts (
  group_code text not null,
  id bigint not null,
  hunter text,
  data jsonb not null,
  updated_at timestamptz default now(),
  primary key (group_code, id)
);
alter table public.hunts enable row level security;
create policy "group read"   on public.hunts for select using (true);
create policy "group insert" on public.hunts for insert with check (true);
create policy "group update" on public.hunts for update using (true);
create policy "group delete" on public.hunts for delete using (true);
```

3. In the app: Settings → your name, a group code your buddies will also use, the URL and anon key → Save → Sync now.
4. Turn on "Include partners' hunts in stats" to see everyone's sits in the stand scorecard and wind charts.

Notes: the anon key is meant to be public; the group code is what keeps groups apart, so pick something not guessable. Hunts sync both ways on every save; photos stay on each phone. Delete a hunt in the app and it's removed from the shared table too.
