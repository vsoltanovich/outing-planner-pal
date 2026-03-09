[README.md](https://github.com/user-attachments/files/25849416/README.md)
# 🍽️ Outing Planner Pal — Deployment Guide

A real-time group availability planner. One link, everyone picks their times, 
you instantly see when 100% of the group is free.

---

## What you need (both free, no credit card)
- ✅ Supabase account — already done!
- ⬜ Vercel account — takes 2 minutes at vercel.com

---

## Step 1 — Run the database setup in Supabase

1. Go to your Supabase project dashboard
2. Click **SQL Editor** in the left sidebar
3. Paste and run this SQL:

```sql
CREATE TABLE IF NOT EXISTS availability (
  id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  participant_name text NOT NULL,
  cell_key text NOT NULL,
  status text NOT NULL CHECK (status IN ('green', 'red')),
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now(),
  UNIQUE(participant_name, cell_key)
);

CREATE TABLE IF NOT EXISTS participants (
  name text PRIMARY KEY,
  color text NOT NULL,
  joined_at timestamptz DEFAULT now()
);

CREATE TABLE IF NOT EXISTS event_config (
  id integer PRIMARY KEY DEFAULT 1,
  title text DEFAULT 'Let''s Make Plans',
  subtitle text DEFAULT 'Dinner',
  start_date date DEFAULT '2026-04-01',
  end_date date DEFAULT '2026-04-05',
  start_hour numeric DEFAULT 18,
  end_hour numeric DEFAULT 22,
  step numeric DEFAULT 0.5,
  updated_at timestamptz DEFAULT now(),
  CONSTRAINT single_row CHECK (id = 1)
);

ALTER PUBLICATION supabase_realtime ADD TABLE availability;
ALTER PUBLICATION supabase_realtime ADD TABLE participants;
ALTER PUBLICATION supabase_realtime ADD TABLE event_config;
```

4. Click **Run** — you should see "Success"

---

## Step 2 — Deploy to Vercel

### Option A: Drag & Drop (easiest, no GitHub needed)

1. Go to https://vercel.com and sign up / log in
2. From your dashboard click **"Add New Project"**
3. Look for **"Deploy without a Git repository"** or drag-and-drop option
4. Drag the `index.html` file onto the Vercel upload area
5. Click **Deploy**
6. In ~30 seconds you get a URL like: `https://outing-planner-pal.vercel.app`

### Option B: Via GitHub (best for future edits)

1. Create a new GitHub repository (public or private)
2. Upload `index.html` to it
3. Go to https://vercel.com → **Add New Project** → Import from GitHub
4. Select your repo → click **Deploy**
5. Get your URL

---

## Step 3 — Share the link!

Copy your Vercel URL and drop it in your Signal or WhatsApp group:

```
Hey everyone! Pick your available times here:
https://your-app-name.vercel.app
```

That's it. Everyone taps the link, enters their name, and marks their availability.
The grid updates in real-time for everyone simultaneously.

---

## How to reuse for future events

Edit `index.html` and update the CONFIG section near the top of the `<script>` tag:

```javascript
const CONFIG = {
  startDate: '2026-04-01',   // ← change start date
  endDate:   '2026-04-05',   // ← change end date
  startHour: 18,              // ← 18 = 6 PM
  endHour:   22,              // ← 22 = 10 PM
  step:      0.5,             // ← 0.5 = 30-min slots, 1.0 = hourly
};
```

Also update the title/subtitle text in the HTML body near the top.

To reset everyone's responses for a new event, go to Supabase → Table Editor → 
delete all rows from both the `availability` and `participants` tables.

---

## Troubleshooting

**"No response / blank page"**  
→ Make sure the SQL tables were created (Step 1)

**Real-time not updating**  
→ Make sure the `ALTER PUBLICATION` lines ran successfully in Supabase

**Want a custom domain?**  
→ Vercel lets you add a custom domain for free in Project Settings → Domains
