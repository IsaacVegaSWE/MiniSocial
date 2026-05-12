# MiniSocial

A barebones social media web app. Users sign up, follow each other, write posts, and see a feed.

**Live stack:**
- Frontend → GitHub Pages (static HTML/CSS/JS)
- Backend  → Railway or Render (Node.js + Express)
- Database → Supabase (PostgreSQL)

---

## Project structure

```
minisocial/
├── backend/
│   ├── middleware/
│   │   └── auth.js          JWT verification middleware
│   ├── routes/
│   │   ├── auth.js          POST /api/auth/signup, /api/auth/login
│   │   ├── posts.js         POST /api/posts, GET /api/posts/feed
│   │   └── users.js         GET /api/users, POST /api/users/:id/follow
│   ├── db.js                PostgreSQL pool + table init
│   ├── server.js            Express entry point
│   ├── package.json
│   ├── .env.example         Copy to .env — never commit .env
│   └── .gitignore
│
├── frontend/
│   ├── css/
│   │   └── style.css        All styles
│   ├── js/
│   │   └── api.js           fetch() wrapper, token helpers
│   ├── index.html           Auth redirect router
│   ├── login.html           Sign-in page
│   ├── signup.html          Registration page
│   └── feed.html            Main app (compose + feed + sidebar)
│
├── .gitignore
└── README.md
```

---

## Deployment — step by step

### 1. Database (Supabase)

1. Go to https://supabase.com and create a free project.
2. Open **SQL Editor** and run:

```sql
CREATE TABLE IF NOT EXISTS users (
  id            SERIAL PRIMARY KEY,
  username      TEXT UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  created_at    TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS posts (
  id         SERIAL PRIMARY KEY,
  user_id    INTEGER REFERENCES users(id) ON DELETE CASCADE,
  content    TEXT NOT NULL CHECK (char_length(content) <= 280),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS follows (
  follower_id  INTEGER REFERENCES users(id) ON DELETE CASCADE,
  following_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
  PRIMARY KEY (follower_id, following_id)
);
```

3. Go to **Settings → Database** and copy the connection string (URI format).
   It looks like: `postgresql://postgres:[password]@db.xxxx.supabase.co:5432/postgres`

---

### 2. Backend (Railway)

1. Push this repo to GitHub.
2. Go to https://railway.app → New Project → Deploy from GitHub repo.
3. Select the repo. Under **Settings → Source**, set the **Root Directory** to `backend`.
4. Under **Variables**, add:

| Key            | Value                                |
|----------------|--------------------------------------|
| `DATABASE_URL` | Your Supabase connection string      |
| `JWT_SECRET`   | Any long random string (32+ chars)   |
| `FRONTEND_URL` | `https://yourusername.github.io`     |
| `PORT`         | `3000`                               |

5. Railway auto-deploys on every push. Copy the generated domain — it looks like `https://minisocial-production.up.railway.app`.

> **Render alternative:** Same flow. New Web Service → connect repo → set root to `backend` → add the same 4 env vars → deploy.

---

### 3. Frontend (GitHub Pages)

1. Open `frontend/js/api.js` and replace the placeholder:
   ```js
   const API_BASE = 'https://your-backend.railway.app';
   //                ↑ paste your Railway domain here
   ```
2. Commit and push.
3. In your GitHub repo → **Settings → Pages** → Source: **Deploy from a branch** → branch `main`, folder `/frontend`.
4. Your site will be live at `https://yourusername.github.io/minisocial/` within a minute.

---

## How passwords are secured

On signup:
```
bcrypt.hash(plaintext_password, saltRounds=12)
  → "$2b$12$...72-char-hash-with-salt-embedded..."
```
Only the hash is stored. On login, `bcrypt.compare()` re-hashes the attempt and compares — the plaintext password is never saved anywhere.

---

## API reference

| Method | Route                     | Auth | Description                     |
|--------|---------------------------|------|---------------------------------|
| POST   | `/api/auth/signup`        | No   | Register; returns JWT           |
| POST   | `/api/auth/login`         | No   | Login; returns JWT              |
| GET    | `/api/posts/feed`         | Yes  | Posts from followed + own posts |
| POST   | `/api/posts`              | Yes  | Create a post (≤280 chars)      |
| GET    | `/api/users`              | Yes  | All users with follow status    |
| POST   | `/api/users/:id/follow`   | Yes  | Toggle follow/unfollow          |
| GET    | `/health`                 | No   | Health check                    |

JWT goes in the `Authorization: Bearer <token>` header.

---

## Local development

```bash
# Backend
cd backend
cp .env.example .env   # fill in your Supabase URL + JWT secret
npm install
npm run dev            # starts on http://localhost:3000

# Frontend
# Open frontend/index.html directly in a browser, OR:
cd frontend
npx serve .            # serves on http://localhost:3000 (use a different port for backend)
```

Set `API_BASE` in `js/api.js` to `http://localhost:3000` while developing locally, and remember to set `FRONTEND_URL=http://localhost:5000` (or `*`) in your `.env` for CORS.
