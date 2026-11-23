# Winter Watch Hard practice

Public GitHub Pages frontend: `index.html`. Private Groq relay: `relay/server.mjs`, started with `npm start` on Render. The frontend can also call Groq directly using a visitor key saved in that visitor's browser. The shared relay mode is intended for a public link that does not ask visitors for a key.

An optional [Cloudflare Worker relay](cloudflare/README.md) is prepared in `cloudflare/`. It is not connected to the public page yet; Render remains the live relay until the Cloudflare Worker has its own secret and passes a public-link test.

## Data workflow

1. Fetch the lecturer's `routes-named.csv`, `ops.csv`, `tonight.csv` and `plan.md` from the BDAIB hub.
2. Use ten documented OpenStreetMap points, with disputed route labels flagged in the map table. The R212 label says Belturbet, whereas [the official road schedule](https://www.irishstatutebook.ie/eli/2012/si/54/made/en/print) traces R212 through Ballyhaise toward Clones. This is a source-data conflict, not an invitation to silently relabel the route.
3. Fetch a fresh 0 cm soil-surface forecast for all ten points from Open-Meteo on every refresh. Apply the plan's temperature, wet-road, depot stock, truck/driver and departure rules in JavaScript.
4. On first visit, fetch each point for each of five winters from Open-Meteo's ERA5 historical archive: 50 route-winter requests. Reduce hourly air, 0–7 cm soil, dew point and precipitation readings into one route-night summary for 18:00–09:00. Cache the nightly summaries in browser storage, so the archive request resumes after interruption and does not repeat on each page load.
5. Join by `route_id` and night-start date, then show per-route counts and the gap check. Send only ten current-route lines and ten history lines to the AI.

## Verified checks on 23 November 2026

- Operations log: 7,560 rows, 439 runs, 1,152.45 tonnes. The brief rounds this to 1,152.5 tonnes.
- Historical join in a fresh Edge browser: 7,560/7,560 matched, zero gaps and zero short or invalid nights.
- ERA5 0–7 cm soil frost count: 406 route-nights. This is 47.3% below the brief's rough 770 guide. A deeper, modelled soil layer is not interchangeable with a surface or road-deck measurement. Point placement also differs. The page states this difference explicitly.
- Current forecast loaded ten locations and displayed a recommendation for each route. The live weather happened to produce ten “skip” calls during this check, so a cold-night manual test is still needed to exercise truck scheduling.
- A browser call from the page to Groq with an intentionally invalid test key returned HTTP 401, establishing that direct browser requests reach Groq rather than failing at CORS. No real key was used for this test.
- The new Render service has `GROQ_API_KEY` configured. A real question from the public page reached Groq and returned an answer with token and cost receipt. The five benchmark questions passed after a prompt correction; see [HANDIN.md](HANDIN.md) for observed answers and costs.

## Sources and licence

- Teaching files: [BDAIB Winter Watch Hard brief](https://bdaib-hub.pages.dev/pack/winter-watch-k7p2qm#hard).
- Map: [OpenStreetMap contributors, ODbL](https://www.openstreetmap.org/copyright). Point links and selection notes are on the page.
- Weather: [Open-Meteo forecast](https://open-meteo.com/en/docs) and [historical archive](https://open-meteo.com/en/docs/historical-weather-api), [CC BY 4.0](https://open-meteo.com/en/license). The free API is for noncommercial use and has [published limits](https://open-meteo.com/en/terms). The historical values are reanalysis estimates, not field measurements.
- AI: Groq `openai/gpt-oss-20b`; [published model pricing](https://console.groq.com/docs/models). The page shows usage and an estimated cost in cent after an actual response.

## Hosting and key

Publish the `main` branch root through GitHub Pages. Create a separate Render Free Node web service from this repository with build command `npm install --omit=dev`, start command `npm start`, and health path `/health`. Add `GROQ_API_KEY` in Render **Environment**; never commit it or paste it into the HTML. The new relay URL is `https://winter-hard-api.onrender.com/ask`. The free service may sleep when idle; check cold-start behavior before handing in.

The Hard brief describes direct browser BYOK. The shared relay is an additional route to meet the user's requirement that anyone with the link can use the chatbot without entering a key. The direct Groq mode and Save/Forget controls remain in Settings for testing the brief's original flow.

## Governance

The Duty Engineer makes the final treatment decision and records it with the forecast and reason in the night log before 16:30. The tool cannot dispatch a truck or edit the log. The Senior Engineer reviews repeat icing on treated routes; unresolved map mismatches and modelled weather are decision limits.
