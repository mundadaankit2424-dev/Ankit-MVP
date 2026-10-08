# Leave Now — Pune

Leave Now is an installable mobile web app for planning a commute around a required arrival time. It uses TomTom map tiles and live traffic overlays in the Map tab. The personal pattern learner runs in the browser and keeps commute history in this phone's local storage.

## What it does

The interface has a calm light theme by default, a dark theme switch, and five clear tabs: Plan, Map, Journey, Insights, and Settings.

Leave Now is designed as a commute AutoPilot: configure your arrival time, workdays, route, and alert offsets once. The Plan screen can use TomTom's current traffic route duration to update its recommended departure, expected arrival, and traffic outlook. This is a current snapshot, not a forecast for a later time. When the app refreshes its TomTom snapshot, AutoPilot recalculates its reminder times too. If no fresh snapshot is available, it uses your personal trip history or starter estimate.

- Keeps the commute focused with five mobile tabs and remembers your light or dark theme choice.

- Saves a commute, profile, arrival deadline, work days, transport mode, starting estimates, and punctuality buffer on this device.
- Recommends a departure using the current TomTom traffic snapshot when available; otherwise it uses your median and 80th percentile of past trips, grouped by mode and weekday when enough samples exist. The live estimate can change before you travel.
- Compares car and bike using your own logged trips. If there is no history for a mode, it shows an editable starter estimate with low confidence.
- Lets you start/finish a trip manually. The app learns actual travel time and marks the arrival early, on time, or late. Too early / accurate / too late feedback adjusts your personal buffer.
- Shows a weekly punctuality view and weekday trends. Export or clear your local data in Profile & preferences.
- Optional auto-start and arrival detection uses phone location while the app is open. Grant location permission only if you want it.
- Includes a TomTom street map, live traffic overlay, and route line, plus a Google Maps navigation link.
- Refreshes weather at origin and destination through Open-Meteo when the app opens or a route is saved. The typed place text is sent for geocoding. Weather is shown as context; it is not yet used to alter the learned ETA.
- Can send −30 minute, −15 minute, and leave-now reminders. Local reminders are best effort while the app is open. For alerts while it is closed, deploy the optional Cloudflare Worker and enable push.

## TomTom setup

1. In Leave Now, open **Settings → TomTom map setup**.
2. Paste your TomTom API key and tap **Save on this phone**. The key is stored in this browser only; it is not in the source files or ZIP.
3. Save a route and choose an arrival time. Leave Now geocodes the two locations and requests a traffic-aware route from TomTom.
4. The Plan screen uses the current TomTom travel time with your arrival time and buffer to show an estimated departure and arrival. The Plan screen, Map tab, route, and mode refreshes also update AutoPilot reminder times. The snapshot is kept for up to 20 minutes and is not a prediction of future traffic. Traffic may change before leaving.

The current TomTom pricing page lists 20,000 free monthly Routing API requests and 20,000 Geocoding API requests. Review your TomTom account limits and usage. Restrict the key to Search and Routing APIs and your GitHub Pages website address. A browser key is visible in browser network requests, so domain and API restrictions matter. [TomTom pricing](https://docs.tomtom.com/pricing) · [Routing API](https://docs.tomtom.com/routing-api/documentation/tomtom-maps/v1/calculate-route)

## Important live-data limits

This app does **not** read Google's live ETA, traffic, incident, or navigation progress from the Google Maps app. Google Maps does not expose the phone app's private live data to this web app. Without a TomTom key, the ETA and late-risk estimate come from your own recorded trips and editable starting estimates. TomTom's current route duration updates the departure plan and reminder schedule when the app refreshes. It cannot predict traffic at a future departure time, and background reminders cannot refresh traffic data while the app is closed.

“AI” here means an on-device pattern learner using personal trip history; no paid AI service, account, or server upload is used to learn. If the app is closed, phone GPS monitoring stops. The browser and phone OS may delay local notifications; use push setup for reliable scheduled reminders while closed.

## Publish on GitHub Pages

Copy the top-level app files (`index.html`, `app.js`, `ui.js`, `push-ui.js`, `tomtom.js`, manifest, service worker, and icon) to the repository root. Keep `worker/` as a folder with `src/index.js` inside it. Commit and push. GitHub Pages should publish from the `main` branch root. The site URL in this setup is `https://mundadaankit2424-dev.github.io/Ankit-MVP/`.

## Optional free Cloudflare push setup

This Worker sends scheduled push reminders only. It makes **no Google Routes calls** and does not receive your trip history. It receives the notification times, estimated duration summaries, weekday, route label, and push subscription needed to send alerts. Cloudflare has free-tier limits; limits and terms can change.

1. Install Node.js 20 or newer and create/sign in to a Cloudflare account.
2. Open a terminal in `worker/` and run `npm install`, then `npx wrangler login`.
3. Run `npm run db:create`. Copy the D1 database ID returned by Wrangler into `database_id` in `wrangler.jsonc`, replacing `REPLACE_WITH_D1_DATABASE_ID`.
4. Run `npm run db:init` to create the tables.
5. Run `npm run vapid` and `npm run token`. Keep these values private. The VAPID public/private keys are a pair; the app token must never be committed to GitHub.
6. Add these secrets, one at a time:

   ```text
   npx wrangler secret put APP_TOKEN
   npx wrangler secret put VAPID_PUBLIC_KEY
   npx wrangler secret put VAPID_PRIVATE_KEY
   npx wrangler secret put VAPID_SUBJECT
   ```

   Enter the matching generated value when Wrangler prompts. For `VAPID_SUBJECT`, use a contact such as `mailto:you@example.com`.

7. Run `npm run deploy` and copy the `https://…workers.dev` URL shown.
8. In Leave Now, open **Connect free notifications**, enter the Worker URL and the generated app token, and tap **Connect**. Then tap **Enable notifications**, allow the phone permission, and tap **Schedule my alerts**.

The Worker expects the app origin `https://mundadaankit2424-dev.github.io` and app URL `https://mundadaankit2424-dev.github.io/Ankit-MVP/`. If your Pages URL changes, update both values in `worker/wrangler.jsonc` and redeploy. The scheduled trigger checks every minute in India time.

## Data and privacy

- Profile, saved route, preferences, and complete trip history stay in this browser's local storage. Export or clear them from the app.
- If push is connected, Cloudflare stores the web-push subscription and scheduled reminder slots. It does not receive source/destination addresses or detailed travel history.
- If auto-detection is enabled, place text is sent to Open-Meteo to locate the start/end points; GPS positions are compared on-device and are not uploaded by this app. Weather place lookups also send typed text to Open-Meteo. Weather is attributed in the UI.
- OpenStreetMap receives place lookups for the selected route to display locations and directions. Open-Meteo also receives place text for weather and route geocoding.
