# Optional Cloudflare Worker relay

This is a Cloudflare alternative to the live Render relay. The GitHub Pages app
still uses Render until this Worker has been deployed, given a private key, and
tested from the public page. The Worker returns the same `/health` and `/ask`
JSON shapes as `relay/server.mjs`.

## Deploy from this GitHub repository

1. In Cloudflare, open **Workers & Pages → Create application → GitHub** and
   connect `x25237489-cpu/winter-hard-watch-cloudfare-version` on the `main` branch.
2. The Wrangler config `wrangler.jsonc` and entrypoint `worker.mjs` are at repository root, so leave the **Root directory** at its default, with Worker name `winter-hard-api-cloudflare`.
   Use no build command and the default deploy command `npx wrangler deploy`.
3. In the new Worker's **Settings → Variables and Secrets**, add a **Secret**
   named `GROQ_API_KEY`. Put the key in Cloudflare's secret field, never in
   GitHub, HTML, chat, or a plain-text variable. Deploy the secret change.
4. Open the Worker's `https://<name>.<account>.workers.dev/health`. It should
   report `{"status":"ok","configured":true}`. Record the exact URL.
5. Update the frontend `RELAY_BASE` constant to that Worker hostname (without `/ask`), then test
   a real question from the public GitHub Pages link. Keep the Render service
   available until that test succeeds.

The Workers Free plan currently allows 100,000 requests per day. This Worker
also applies an approximate 6 requests/minute per IP and 30/minute per
Cloudflare location. Those limits are not a strict global budget, and an HTTP
Origin check is not authentication against scripts. Set a Groq project usage
limit if available. Do not assume that moving to Cloudflare eliminates Groq
quota or cost exposure.

The Cloudflare dashboard's GitHub integration is separate from the Codex
plugin catalog. No Cloudflare management plugin was available in the connected
catalog when this alternative was prepared.
