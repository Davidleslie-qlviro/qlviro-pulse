# Qlviro Pulse

7-filter creator screener. Hand in a handle, get a verdict.

## What it does

`POST /api/screen` with `{ "handle": "@example" }` → JSON scorecard across 7 filters (1–5 each), total /35, and a verdict.

| Band | Range |
|------|-------|
| 🟢 PASS  | 28+   |
| 🟡 MAYBE | 22–27 |
| 🔴 KILL  | <22   |

The 7 filters: engagement, pain signal, pay capacity, niche clarity, active monetiser, posting cadence, Qlviro fit.

## Stack

- **Runtime:** Vercel serverless function (Node 20+)
- **Model:** `claude-haiku-4-5-20251001`
- **Tool:** Anthropic `web_search_20250305` (up to 8 searches per screen)
- **Frontend:** Single static `index.html`, Tailwind via CDN, dark mode

## Local sanity check (optional)

```bash
npm install
# no local dev server needed — just deploy
```

## Deploy

1. **Push to GitHub**
   ```bash
   cd qlviro-pulse
   git init
   git add .
   git commit -m "Pulse v1"
   git branch -M main
   git remote add origin git@github.com:Davidleslie-qlviro/qlviro-pulse.git
   git push -u origin main
   ```

2. **Import to Vercel**
   - vercel.com → **Add New → Project** → pick `qlviro-pulse`
   - Framework Preset: **Other**
   - Build/Output: leave blank
   - Environment Variables → add `ANTHROPIC_API_KEY` (from Console: Daveiph5@icloud.com)
   - Deploy

3. **Point pulse.qlviro.com**
   - Vercel project → **Settings → Domains** → add `pulse.qlviro.com`
   - At your DNS provider, add the CNAME record Vercel shows you
   - Wait ~1 min for the cert

## Test

```bash
curl -X POST https://pulse.qlviro.com/api/screen \
  -H "Content-Type: application/json" \
  -d '{"handle":"@realestaterood"}'
```

Or just open `https://pulse.qlviro.com/` and type `@realestaterood` in the box.
