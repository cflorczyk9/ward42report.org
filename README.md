# The 42nd Ward Service Report

A single-file, self-updating website that shows how responsive Chicago's 42nd Ward is to resident 311 requests (potholes, graffiti, street lights, traffic signals, and more). Built as an independent civic tool to offer to Alderman Brendan Reilly's office.

Replaces the earlier `ward42-tracker` (development permits) folder, which was retired: the development data showed a decline and worked against him. This one tells a story that flatters him honestly.

## What it does

- Pulls **live** from the City of Chicago 311 open data (dataset `v6vf-nfxy`) on every page load. No backend, no database. Always current.
- Tells a plain-language responsiveness story: how many requests residents file, what share get resolved (~87%), what people ask for most, and how fast each type gets fixed (graffiti in about a day, potholes in under a week).
- Charts (custom, colorblind-checked): what residents ask for, how fast it gets fixed, resolution rate, and a 24-month volume trend.
- Interactive: click a chart bar to filter, click any map pin or table row to open a full detail record, plus type / status / neighborhood / search filters that compose.
- Editorial "civic report" design (serif headlines, mono figures, ink-on-paper), light and dark mode, graceful fallback if the API is slow.

## Important

This is `index.html`, one file. Everything is inline except the 311 API, Leaflet (map), OpenStreetMap tiles, and Google Fonts (all over https).

**It will NOT auto-update as a Claude Artifact** (Artifacts block outbound requests). Host it or open it as a local file.

## Files in this folder

- `index.html` — the full report (deploy this as the main page).
- `embed.html` — a compact widget the Alderman's office can iframe onto ward42chicago.com. Pass the report URL as a query parameter, no file editing needed: `<iframe src="https://your-site/embed.html?report=https://your-site/index.html" width="100%" height="380" style="border:0"></iframe>`. With no `?report=` set, the "full report" link just hides itself.
- `share-card.html` — the source used to generate the social preview image. Not meant to be visited directly. To regenerate the image, open it at exactly 1200x630 and screenshot to `preview.png`.
- `preview.png` — the 1200x630 social share image, referenced by the report's og:image.

## Motion (this round)

Subtle, editorial entrance animations, all in `index.html` (no new files, no libraries):
- Sections fade and rise as you scroll to them.
- The four hero figures and the "87%" count up from zero when they come into view.
- Chart bars grow from the axis; the trend and pothole lines draw across left to right.
- The blue rule under the headline draws across on load, and the "live" dot has a quiet pulse.

All of it is off by default for anyone whose browser is set to "reduce motion," and it degrades to a fully visible, static page if JavaScript is disabled. Entrances play once, so switching theme or resizing won't replay them. The embed widget (`embed.html`) is deliberately left static so it stays lightweight inside someone else's page.

## New features (earlier round)

- **What's on my block:** a resident types an address (or clicks "use my location") and sees recent 311 activity within about two blocks, with a completed/open/canceled breakdown and pins on the map. Address search uses OpenStreetMap Nominatim (no key), with browser geolocation as the no-third-party option.
- **Resolved within a week:** a live stat in the "Getting to done" section. Currently about 81%, and verified honest (85% even after excluding auto-closed request types).
- **Embed widget** and **social share image**, described above.

Note on og:image: it is set to the absolute URL `https://ward42report.org/preview.png`, so Facebook and LinkedIn resolve the card. If the domain ever changes, update `og:image`, `twitter:image`, and `og:url` in `index.html`.

## Deploy (pick one)

**Fastest (~30 seconds), Netlify Drop:**
1. Go to https://app.netlify.com/drop
2. Drag this whole `ward42-service-report` folder onto the page.
3. You get a live URL instantly. That is the link you share.

**Permanent home, GitHub Pages (this repo): `cflorczyk9/ward42report.org`**
1. On GitHub: Settings, Pages, Source = "Deploy from a branch," branch `main` / root.
2. A `CNAME` file is already in the repo, so Pages will serve the site at the custom domain `ward42report.org`.
3. At your domain registrar, point DNS at GitHub Pages:
   - Four `A` records for the apex `ward42report.org` -> `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`
   - One `CNAME` record for `www` -> `cflorczyk9.github.io`
4. Back in Settings, Pages, tick "Enforce HTTPS" once the certificate is issued (can take a few minutes to an hour).

## Verify it worked (10 seconds)

After deploying, open the live URL and check:
1. The four hero figures show real numbers, not dashes. Dashes or blanks mean the page cannot reach the data. If you opened it as a Claude Artifact, that is expected (Artifacts block the live fetch), so host it instead.
2. Type an address into "What's on my block" and hit Search. You should get a count and a map with pins.
3. Paste the URL into a link preview tester (for example, opengraph.xyz) to confirm the social card shows.

Common fixes:
- **Blank stat tiles:** the host or network is blocking the City of Chicago API, or you opened the file as an Artifact. Host it on Netlify or GitHub Pages.
- **No social preview image on Facebook or LinkedIn:** those two require an absolute image URL. It is already set to `https://ward42report.org/preview.png`. If you host on a different domain, update `og:image` and `twitter:image` in `index.html` to match.
- **Address search does nothing:** OpenStreetMap's geocoder occasionally rate-limits. Wait a few seconds and try again, or use "Use my location."

## Staying current (it's automatic)

The page has no database and no build step. Every visit runs live queries against the City of Chicago 311 feed in the browser and computes the trailing 12 months from the current date, so the data is never stale. There is nothing to schedule or rebuild.

The one thing outside your control is the City itself. If they ever retire, rename, or restructure the 311 dataset, the page would quietly show its "couldn't load" banner. To catch that:

- **`.github/workflows/health-check.yml`** runs daily, checks the 311 feed still returns Ward 42 data with the exact fields the site uses, and **fails if anything changed**. GitHub emails you (the repo owner) on a failed run, so you find out the next morning instead of months later. You can also run it on demand from the repo's **Actions** tab.
- Heads-up: GitHub pauses scheduled workflows after ~60 days with no commits to the repo. If that happens it emails you, and any commit (or clicking "Enable" in the Actions tab) restarts it.
- Keep **auto-renew on** for the domain at Namecheap so the registration never lapses.
- The **social card** (`preview.png`) is number-free on purpose, so it can never show outdated figures.

## Traffic tracking (Google Analytics)

GA4 is **already active** with Measurement ID `G-QNM7KJZD5M` (set once via `window.GA_ID` near the top of `index.html`). To point it at a different property, change that one value.

GA4 auto-tracks page views, outbound clicks (including the "Report it to 311" button), and scroll depth. The page also fires a custom `view_request` event whenever someone opens a request's detail card, so you can see what people dig into. Data only starts appearing once the site is deployed and getting visits (check Analytics > Realtime).

Privacy note: GA4 sets cookies. If you'd rather not, a cookieless alternative like Plausible or GoatCounter drops in the same way and needs no consent banner. Say the word and I'll swap it.

## Other features added

- **Social share cards** (Open Graph / Twitter meta): a posted link shows a title, description, and the `preview.png` image card. The image and URL tags already point at the absolute `https://ward42report.org/` domain.
- **"Report it to 311" call-to-action** near the top: turns the report from something you read into something you act on, and links straight to 311.chicago.gov.
- **"View the raw Ward 42 data" link** in the methodology: anyone can inspect or download the underlying records, which reinforces the transparency angle.

## How the "compare to the city" section is framed (important)

The comparison is built to be favorable AND honest, because a cherry-picked chart is easy to discredit:

- **It compares on speed, not completion rate.** Ward 42 resolves ~87% of requests, but the citywide rate is ~94%, so Ward 42 is actually *below* average on completion rate. That comparison is deliberately NOT shown, because it hurts him.
- **Speed is where the ward wins.** It ranks 6th of 50 wards on average time to close a request, and per request type it beats the city on potholes (~4 days vs ~16) and graffiti, ties on traffic signals, and is slower on street lights (~12 vs ~6).
- **The street-light row is shown even though he loses it.** Including the one category where the ward lags is what makes the wins credible. If you'd rather drop it, say so, but I recommend keeping it.
- **The rank caveat is stated on the page:** each ward files a different mix of requests, so the fairest read is type by type. That line is there on purpose so the rank can't be dismissed as spin.
- **The pothole-over-time chart** drops the current partial month and notes that low-volume months can swing.

## Honest methodology notes (read before sending)

- **Resolution time is an estimate.** The 311 dataset has no "closed date" field, so time-to-resolve is approximated from each request's last-update date (`last_modified_date`). The site says this plainly in the caption and methodology.
- Request types that auto-close instantly (information-only calls, some parking/Divvy logs) are excluded from the resolution-time chart so they don't fake a "0 days" result.
- "Cab Feedback" runs ~48 days and is flagged as an outlier rather than blended in.
- The current calendar month is drawn dashed on the trend chart and labeled "in progress" so it doesn't read as a drop.
- 311 resolution is performed by City departments; the ward office routes and expedites. The framing is "how the ward's requests are handled," not a claim that the alderman personally fixes potholes.

## Tweaking

- **Time window:** the headline/charts use a rolling 12 months, the trend uses 24. Search the script for the rolling-window date math to change it.
- **Higher traffic:** if it ever hits Chicago's anonymous rate limit (~1,000 req/hour), request a free Socrata app token and append `&$$app_token=YOUR_TOKEN` to the endpoints. Not needed for normal use.
