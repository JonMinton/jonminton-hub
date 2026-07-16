# jonminton-hub

Source for https://jonminton.net/ — the top-level routing page for Jon's sites.

Hosted on the Netlify site `jonminton-hub` (NOT repo-linked; deploys are pushed manually). To deploy after editing:

```bash
zip -j site.zip index.html favicon.ico
curl -X POST -H "Authorization: Bearer $NETLIFY_TOKEN" -H "Content-Type: application/zip" \
  --data-binary @site.zip https://api.netlify.com/api/v1/sites/jonminton-hub.netlify.app/deploys
```

(The token from `netlify login` lives in `~/Library/Preferences/netlify/config.json`. Alternatively link this repo to the Netlify site in the Netlify UI for auto-deploys.)

DNS for jonminton.net is managed in Netlify DNS: apex + www + portfolio are Netlify sites; blog, stats, stats-board, food, and games are CNAME records to jonminton.github.io (GitHub Pages).
