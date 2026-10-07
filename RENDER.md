# Host the lustpress API on Render (free)

Do this once. ~10 minutes, no code changes needed — Render builds the
`Dockerfile` already in this folder.

## 1. Put this folder on GitHub

Render deploys from git, so the `api/` folder needs to be a GitHub repo:

1. Go to https://github.com/new → repo name `lustpress-api` → Public or
   Private (either works) → Create. Do NOT tick "Add a README".
2. Upload the files: on the new repo page click **"uploading an existing
   file"** → drag in **everything inside the `api/` folder**
   (`src/`, `Dockerfile`, `render.yaml`, `package.json`, `bun.lock`,
   everything — but NOT `node_modules/`) → Commit.
   - Faster with git (PowerShell, inside the `api/` folder):
     `git init; git add .; git commit -m "lustpress api";`
     `git branch -M main; git remote add origin https://github.com/YOU/lustpress-api.git;`
     `git push -u origin main`
   - If you keep site + api in ONE repo instead, set Render's
     **Root Directory** to `api` in step 2 below.

## 2. Create the Render service

1. https://dashboard.render.com → **New → Blueprint** → connect GitHub →
   select `lustpress-api` → Apply. (`render.yaml` fills everything in:
   Docker runtime, health check `/`, free plan.)
   - No Blueprint option? Use **New → Web Service** → same repo →
     Runtime **Docker** → Health Check Path `/` → Create.
2. Wait for **Live** (first build ~3–5 min). Open the service URL —
   you should see `{"success":true,...,"version":"8.5.1-alpha"}`.
3. Sanity-check a scraper in the browser:
   `https://YOUR-NAME.onrender.com/xnxx/search?key=test&page=1`
   → `{"success":true,"data":[...]}`.

## 3. Point the website at it

1. Vercel dashboard → project `i-like-it` → Settings → Environment
   Variables → edit `LUSTPRESS_URL` to
   `https://YOUR-NAME.onrender.com/` → **Save** → Deployments →
   **Redeploy** (env changes need a rebuild).
2. Done. The site picks up the new API on that build.

## Notes

- **Free plan sleeps** after ~15 min idle; first visit then takes ~50s
  and that request may render without scraper rows (the site degrades,
  never errors). Upgrade the service to **Starter** (~$7/mo) for always-on.
- **Pornhub on Render:** datacenter IPs get Pornhub's bot wall, so
  pornhub/youporn stay disabled on the site (`enabled: false` in the
  site's `src/lib/sources.ts`) with the embed-direct fallback player.
  If `/pornhub/get?id=<real-id>` ever returns a real title from Render,
  flip them to `enabled: true` and redeploy the site.
- **Env vars** you may add later on the Render dashboard (service →
  Environment): `REDIS_URL` (New → Redis on Render, cache survives
  restarts), `USER_AGENT`, `EXPIRE_CACHE`.
- Never commit real `.env` secrets — the repo's `.env` is local-only
  (PORT/LUSTPRESS_GRAPHQL/EXPIRE_CACHE are safe defaults).
