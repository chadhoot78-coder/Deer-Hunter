# Whitetail Tracker

Deer hunt log for LDWF Area 2 with accounts, shared groups, weather/moon, stand analytics and game-cam photos. Static site (one `index.html`) backed by a free Supabase project for accounts and data.

## Files
- `index.html` — the whole app (Log · History · Insights · Cam · Settings). **Edit `CONFIG` at the top of the script once (see step 3).**
- `sw.js` — service worker for offline use. **Bump `CACHE` on every index.html change.**
- `manifest.json`, `icon.png` — Add to Home Screen support

## One-time setup

### 1. Create the Supabase project
supabase.com → New project (free tier). Pick any name and a strong database password (you won't need it in the app).

### 2. Create the tables
SQL Editor → New query → paste and run:

```sql
-- Profiles: one row per account, created automatically on sign-up
create table public.profiles (
  id uuid primary key references auth.users on delete cascade,
  name text,
  group_code text,
  updated_at timestamptz default now()
);

create or replace function public.handle_new_user() returns trigger
language plpgsql security definer set search_path = public as $$
begin
  insert into public.profiles (id, name) values (new.id, new.raw_user_meta_data->>'name');
  return new;
end $$;
create trigger on_auth_user_created after insert on auth.users
  for each row execute procedure public.handle_new_user();

-- Who is in my group (security definer avoids recursive policies)
create or replace function public.group_member_ids() returns setof uuid
language sql security definer stable set search_path = public as $$
  select id from public.profiles
  where group_code is not null
    and group_code = (select group_code from public.profiles where id = auth.uid());
$$;

-- Hunts
create table public.hunts (
  user_id uuid not null default auth.uid() references auth.users on delete cascade,
  id bigint not null,
  hunter text,
  data jsonb not null,
  updated_at timestamptz default now(),
  primary key (user_id, id)
);

-- Photos (stored as compressed JPEG data inside jsonb)
create table public.photos (
  user_id uuid not null default auth.uid() references auth.users on delete cascade,
  id text not null,
  hunt_id bigint,
  data jsonb not null,
  updated_at timestamptz default now(),
  primary key (user_id, id)
);

-- Row-level security: you can do anything to your own rows; you can read your group's rows
alter table public.profiles enable row level security;
alter table public.hunts    enable row level security;
alter table public.photos   enable row level security;

create policy "own profile"    on public.profiles for all    using (id = auth.uid()) with check (id = auth.uid());
create policy "group profiles" on public.profiles for select using (id in (select public.group_member_ids()));

create policy "own hunts"   on public.hunts for all    using (user_id = auth.uid()) with check (user_id = auth.uid());
create policy "group hunts" on public.hunts for select using (user_id in (select public.group_member_ids()));

create policy "own photos"   on public.photos for all    using (user_id = auth.uid()) with check (user_id = auth.uid());
create policy "group photos" on public.photos for select using (user_id in (select public.group_member_ids()));
```

### 3. Connect the app
Supabase → Project Settings → API. Copy **Project URL** and the **anon public** key into the top of the script in `index.html`:

```js
const CONFIG={url:'https://xxxx.supabase.co',key:'eyJ...'};
```

The anon key is designed to be public; the policies above are what keep each person's data private.

### 4. Auth settings (Supabase → Authentication)
- **URL Configuration → Site URL**: your Vercel URL (e.g. `https://deer-hunter.vercel.app`). Password-reset emails link back here.
- **Providers → Email → Confirm email**: leave on if you want people to verify their address before signing in; turn off if you'd rather they get in immediately.

### 5. Deploy (GitHub → Vercel)
Push the four files to the repo root. Vercel: New Project → import → Framework "Other", no build command → Deploy. On iPhone open the URL in Safari → Share → Add to Home Screen.

## How it works
- Anyone with the link can create an account (email + password).
- Hunts and photos save to the phone first, then sync to the account, so the app works in the stand with no signal and catches up later.
- Hunts logged on a phone before accounts existed are moved into the account automatically on first sign-in.
- **Groups**: Settings → Hunting group → enter any code (4+ characters). Everyone who enters the same code sees each other's hunts and cam photos; partners' entries are read-only. Leave any time.
- Photos are shrunk to ~1200px (~100–150 KB each). The free tier's 500 MB database holds a few thousand.

## Updating the app
Edit `index.html`, bump `CACHE` in `sw.js` (v4 → v5 …), push. Phones pick up the new version on the next open with a connection.
