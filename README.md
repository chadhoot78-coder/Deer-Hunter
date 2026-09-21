# Whitetail Tracker

Single-page deer hunt log (PWA). Data lives in the browser's localStorage; weather from Open-Meteo (free, no key).

## Files
- `index.html` — the whole app
- `sw.js` — service worker for offline use (bump `CACHE` on every index.html change)
- `manifest.json`, `icon.png` — Add to Home Screen support

## Deploy (GitHub → Vercel)
1. Create a repo, add these four files at the root, push.
2. In Vercel: New Project → import the repo → Framework "Other", no build command → Deploy.
3. Open the URL on your iPhone in Safari → Share → Add to Home Screen.

Any static host works (Netlify, GitHub Pages, Cloudflare Pages, nginx). It must be served over HTTPS for geolocation and the service worker.

## Updating
Edit `index.html`, change `CACHE` in `sw.js` (v1 → v2…), push. Phones pick up the new version on the next open with a connection.
