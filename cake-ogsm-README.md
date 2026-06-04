# Cake OGSM Platform

Internal OGSM data entry and presentation tool for the Cake Indonesia Business Team.

---

## What you need

| Tool | Purpose | Cost |
|---|---|---|
| VSCode | Edit the code | Free |
| GitHub | Host the repo + serve the app | Free |
| Supabase | Database + user auth | Free tier |

No server. No build step. One HTML file.

---

## Step 1 — Set up Supabase

1. Go to [supabase.com](https://supabase.com) → **New project** → name it `cake-ogsm`
2. Wait for it to provision (~1 min), then go to the **SQL Editor**
3. Run this SQL to create the required tables:

```sql
-- Slides (admin-defined structure)
create table slides (
  id          text primary key,
  title       text not null,
  fields      jsonb not null default '[]',
  position    integer default 0,
  created_at  timestamptz default now()
);

-- Entries (team data input)
create table entries (
  id          uuid primary key default gen_random_uuid(),
  slide_id    text references slides(id) on delete cascade,
  slide_title text,
  data        jsonb not null default '{}',
  period      text,
  note        text,
  by          text,
  created_at  timestamptz default now()
);

-- Users with roles
create table users (
  id          uuid primary key default gen_random_uuid(),
  email       text unique not null,
  role        text check (role in ('admin', 'team')) default 'team',
  created_at  timestamptz default now()
);
```

4. Go to **Settings → API** and copy:
   - **Project URL** (looks like `https://xxxx.supabase.co`)
   - **anon public** key (long string starting with `eyJ...`)

---

## Step 2 — Paste credentials into the app

Open `index.html` in VSCode. Find these two lines near the top (around line 10):

```js
const SUPABASE_URL = 'YOUR_SUPABASE_URL';
const SUPABASE_KEY = 'YOUR_SUPABASE_ANON_KEY';
```

Replace with your actual values:

```js
const SUPABASE_URL = 'https://xxxx.supabase.co';
const SUPABASE_KEY = 'eyJhbGciOiJIUzI1NiIs...';
```

Save the file.

---

## Step 3 — Push to GitHub

### First time setup

```bash
# In VSCode terminal (Ctrl + `)
git init
git add .
git commit -m "Initial commit"

# Create a new repo on github.com, then:
git remote add origin https://github.com/YOUR_USERNAME/cake-ogsm.git
git branch -M main
git push -u origin main
```

### Enable GitHub Pages

1. Go to your repo on GitHub
2. **Settings → Pages**
3. Under **Source**, select `Deploy from a branch`
4. Branch: `main`, folder: `/ (root)`
5. Click **Save**

Your app will be live at:
```
https://YOUR_USERNAME.github.io/cake-ogsm/
```
(takes about 1–2 minutes to deploy the first time)

### Every future update

```bash
git add .
git commit -m "describe what changed"
git push
```
GitHub Pages auto-deploys on every push. Changes go live in ~30 seconds.

---

## Step 4 — Add team members

### In Supabase Auth

1. Go to **Authentication → Users → Invite user**
2. Enter each team member's email
3. They receive an email to set their password

### Set their role

In the **SQL Editor**, run:

```sql
-- Add an admin
insert into users (email, role) values ('you@cake.com', 'admin');

-- Add a team member
insert into users (email, role) values ('colleague@cake.com', 'team');
```

Repeat for each person. Their role determines what they see when they sign in.

---

## How to use the app

### Admin
- **Slide builder** — Create slides, add/remove fields (number, text, percent, currency, date, URL)
- **Data entry** — Fill in data for any slide
- **History** — See all entries, delete if needed
- **Present** button — Full-screen presentation overlay
- **Export** button — Downloads a standalone HTML file ready for sharing

### Team
- **Data entry** — Select a slide, fill in the latest values, hit Save
- **History** — View all past entries (read-only)

---

## Folder structure

```
cake-ogsm/
├── index.html    ← entire app (edit credentials here)
└── README.md     ← this file
```

---

## Demo mode

If Supabase is not yet configured, the app runs in **demo mode** using browser localStorage. This works for a single user but data is not shared across devices or team members.

Demo credentials:
- Admin: `admin@cake.com` / `admin123`
- Team: `team@cake.com` / `team123`

---

## Making changes in VSCode

The entire app is in `index.html`. Open it and use **Go to Line** (`Ctrl+G`) to jump to sections:

| What to change | Search for |
|---|---|
| Supabase credentials | `YOUR_SUPABASE_URL` |
| App name / branding | `Cake Indonesia` |
| Demo passwords | `admin@cake.com` |
| Default slide templates | `function seedSlides()` |
| Colors | `:root {` (CSS variables at top) |

---

## Troubleshooting

**"Invalid credentials" on login**
→ Make sure the email exists in both Supabase Auth (invited) AND the `users` table (with correct role)

**Changes not showing after push**
→ Wait 30–60 seconds, then hard refresh the browser (`Ctrl+Shift+R`)

**App shows demo mode even after adding credentials**
→ Check for typos in the URL and key. The URL should start with `https://` and the key should start with `eyJ`

**Data not saving**
→ Open browser DevTools → Console and look for error messages. Usually a Supabase RLS (Row Level Security) policy issue — disable RLS on all three tables in Supabase → Table Editor → [table] → RLS toggle
