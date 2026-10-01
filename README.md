# jonminton-hub

Source for https://jonminton.net/ — the top-level routing page for Jon's sites.

Hosted on the Netlify site `jonminton-hub` (NOT repo-linked; deploys are pushed manually). To deploy after editing:

```bash
zip -j site.zip index.html favicon.ico
curl -X POST -H "Authorization: Bearer $NETLIFY_TOKEN" -H "Content-Type: application/zip" \
  --data-binary @site.zip https://api.netlify.com/api/v1/sites/jonminton-hub.netlify.app/deploys
```

(The token from `netlify login` lives in `~/Library/Preferences/netlify/config.json`. Alternatively link this repo to the Netlify site in the Netlify UI for auto-deploys.)

DNS for jonminton.net is managed in Netlify DNS: apex + www + portfolio are Netlify sites; blog, stats, stats-board, food, and games are GitHub Pages sites. Since 2026-09-17 they use four A records (185.199.108-111.153) instead of a CNAME to jonminton.github.io, deliberately publishing no AAAA record: IPv6 connections to GitHub Pages were dying on some routes (see `docs/why-my-sites-were-slow.md`). If GitHub ever changes its Pages IPs, update these records. Pre-change snapshot: `docs/dns-snapshot-before-2026-09-17.json`.

The photo beside the company card (laptop width and up; hidden on phones so the tiles stay a no-scroll grid) is Scott Barron's headshot of 23 Sep 2026 (`0043_Retouch_Square`), 320 px greyscale, inlined as a data URI so the two-file deploy above still carries everything. Its button inverts the photo only; the page stays dark.
