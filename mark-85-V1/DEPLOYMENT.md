# Deployment Guide — TechCompare AI

This app is a React (Vite) SPA + an Express API. The backend now uses **SQLite** for
user storage, so it needs a **persistent disk**. That rules out stateless/serverless
hosts (Vercel/Netlify Functions). Recommended free/low-cost options below.

## Security checklist (do this first)

1. **Rotate the leaked SMTP app password.**
   The previous `.env` contained a real Gmail app password that was committed.
   Go to your Google account → Security → App passwords (or the app's less-secure
   app), **revoke `rkcq ahkg pxmc mgcd`**, and generate a fresh one. Put the new
   value only in the platform's secret manager, never in the repo.
2. **Generate a strong `JWT_SECRET`:**
   `node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"`
   and set it in the platform env vars. The server refuses to start without one.
3. `.env` and `database/*.sqlite*` are now gitignored. Never commit real secrets.

## Option A — Render (free, easiest)

1. Push this repo to GitHub.
2. Render → New → Web Service → pick the repo.
3. Runtime: **Docker** (uses the `Dockerfile`).
4. Set env vars: `JWT_SECRET`, `FRONTEND_ORIGIN`, `SMTP_*`, and site URL.
5. A 1 GB persistent disk is attached at `/var/data` (see `render.yaml`) so
   SQLite survives restarts.
6. Free tier spins down after ~15 min idle; wakeups can be slow.

## Option B — Railway (free tier, then cheap)

1. Railway → New Project → Deploy from GitHub.
2. `railway.json` uses the Dockerfile, health check at `/api/health`.
3. Add a **Volume** mounted at `/data` and set `DB_PATH=/data/app.sqlite`.
4. Add env vars: `JWT_SECRET`, `FRONTEND_ORIGIN`, `SMTP_*`.

## Option C — Oracle Cloud Always-Free ARM VM (most control, $0)

Best fit for the "high-traffic + firewall + scale" requirements, but manual:

1. Create an **Always Free** ARM VM (Ampere A1) in Oracle Cloud.
2. Open firewall: `sudo ufw allow OpenSSH`, `sudo ufw allow 80`, `sudo ufw allow 443`.
3. Install Node 22 + Docker, or run the built app directly behind NGINX/Caddy.
4. Use **Cloudflare free** for DNS + CDN + WAF + edge rate limiting + TLS.
5. Serve TLS via Caddy (automatic certs) or NGINX + certbot.
6. Put SQLite on the VM's persistent boot volume (or a block volume).
7. Monitor with UptimeRobot/Prometheus; scrape the API logs.

## Running locally

```
npm install
# set JWT_SECRET in .env
npm run dev        # frontend (Vite, :3000)
npm run server     # API (:5000)
```

## Production notes

- HTTPS is terminated at the reverse proxy / CDN. Set `TRUST_PROXY` to the
  number of proxy hops (default 1) so client IPs and rate limiting are correct.
- Rate limits and hard limits are applied in `backend/server.js`.
- Add `/api/health` to your uptime monitor.
