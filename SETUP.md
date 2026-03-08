# Bus Tracker — Setup Guide

## Files
- `index.html` — passenger page (share this link)
- `admin.html` — driver/operator page (keep private)

---

## Step 1: Create a Free Supabase Project

1. Go to https://supabase.com and create a free account
2. Click **New Project** — give it any name (e.g. "bus-tracker")
3. Choose a region close to you, set a password, click Create
4. Wait ~2 minutes for it to spin up

---

## Step 2: Create the Database Table

In your Supabase project, click **SQL Editor** in the left sidebar and run this:

```sql
create table trip_state (
  id integer primary key default 1,
  status text default 'inactive',
  trip_name text,
  origin text,
  destination text,
  youtube_id text,
  maps_url text,
  lat double precision,
  lng double precision,
  eta_destination text,
  eta_next_stop text,
  distance_remaining text,
  progress_pct integer default 0,
  stops jsonb,
  updated_at timestamptz default now()
);

-- Allow public reads (passengers can see trip state)
-- Admin writes are protected by the anon key being device-only
alter table trip_state enable row level security;

create policy "Public read" on trip_state for select using (true);
create policy "Anon write" on trip_state for insert with check (true);
create policy "Anon update" on trip_state for update using (true);
```

Click **Run**.

---

## Step 3: Get Your Keys

In Supabase, go to **Project Settings → API**:

- Copy **Project URL** → paste into admin.html "Supabase URL" field
- Copy **anon / public** key → paste into admin.html "Supabase Anon Key" field
- Also paste both into `index.html` where it says `YOUR_SUPABASE_URL` and `YOUR_SUPABASE_ANON_KEY`

---

## Step 4: Get a Google Directions API Key

1. Go to https://console.cloud.google.com
2. Create a new project (or use existing)
3. Enable the **Directions API**
4. Go to **Credentials → Create Credentials → API Key**
5. Copy the key → paste into admin.html "Google Directions API Key" field
6. Restrict the key to **Directions API** only for safety

**Cost:** You get $200/month free credit. At ~15 requests per trip you'd need
to run ~2,600 trips/month before paying anything.

---

## Step 5: Host on GitHub Pages (Free)

1. Create a free GitHub account at https://github.com
2. Create a new **public** repository
3. Upload `index.html` and `admin.html`
4. Go to **Settings → Pages → Source → main branch** → Save
5. Your links will be:
   - Passenger: `https://yourname.github.io/reponame/`
   - Admin/Driver: `https://yourname.github.io/reponame/admin.html`

---

## How It Works on Trip Day

1. Open `admin.html` on the bus phone
2. Enter trip details (name, origin, destination, stops)
3. Hit **Start Trip**
4. The phone grabs GPS, calls Google for ETA, pushes everything to Supabase
5. Passengers visit `index.html` — it polls Supabase every 30 seconds
6. As you depart each stop, tap **Mark Departed** — timeline updates live
7. Hit **End Trip** when you arrive — passenger page clears

That's it. Total ongoing cost: $0.
