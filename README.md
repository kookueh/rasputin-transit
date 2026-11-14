# RASPUTIN Transit

Phone web app (PWA) for Singapore: weather alerts, bus timings with live bus positions, MRT disruption alerts, and one-tap transit directions to work and home. Same Warmind look as the Rasputin console.

## Layout
A bottom icon dock switches between five panels, and tapping any card title collapses or expands it (remembered on the phone).
- **Home:** alerts (weather, rain, lightning, UV, MRT) and directions to work and home.
- **Bus:** nearby stops (pick services to track), arrivals and a live bus radar.
- **Sky:** rain radar (NEA frames over a OneMap map, last 40 minutes), conditions (temperature, humidity, wind, UV, lightning) and the forecast.
- **Road:** free taxis (counts within 500 m / 1 km / 2 km, trend, and a learned typical-for-this-hour baseline for Work and Home), HDB carpark availability and LTA traffic cameras near Me, Work or Home.
- **Setup:** addresses, weather areas, tracked stops, proxy.

All the Sky and Road data comes straight from data.gov.sg and OneMap (no key). data.gov.sg rate-limits bursts, so the app queues its requests about a second apart; the first Sky load takes roughly 15 seconds.

## What works with no setup
- **Weather alerts:** NEA 2-hour forecast, 24-hour outlook and rain gauges from data.gov.sg (open, no key). Pick your home and work areas in Config.
- **Directions:** one tap opens Google Maps with live transit directions from where you are. Add your addresses in Config.

## What needs the transit proxy
Bus arrivals, live bus positions, bus stop search and MRT alerts come from LTA DataMall, which needs a key and does not allow calls straight from a web page. A tiny Cloudflare Worker (`worker/worker.js`) holds the key.

### Setup
1. **LTA key.** Request API access at https://datamall.lta.gov.sg (free; approval can take a day or two). You get an `AccountKey`.
2. **Cloudflare Worker.** Create a free Cloudflare account, then Workers & Pages > Create > Worker. Paste in `worker/worker.js` and deploy. In the Worker's Settings > Variables and Secrets add:
   - `LTA_KEY` (secret): your AccountKey
   - `APP_TOKEN` (secret): any long random string
   - `ALLOWED_ORIGIN` (text): your GitHub Pages origin, for example `https://yourname.github.io`
   Copy the Worker's URL (`https://<name>.<you>.workers.dev`).
3. **GitHub Pages.** Upload everything in this folder except `worker/` to a repository (the web upload is fine) and turn on Pages for it.
4. **On your phone.** Open the Pages link in Safari, Share > Add to Home Screen. Open the app, scroll to Config, paste the Worker URL and the same `APP_TOKEN`, then add your home and work addresses and areas and search for your bus stops.

Everything you enter in Config is stored on the phone only. Nothing personal is in the repository, so the public Pages site is safe.

## To verify on first live use
- The bus arrival response includes each bus's latitude and longitude (shown as a map link and distance). If a bus shows no position, LTA is not reporting one for it.
- The MRT alert parsing assumes LTA's `TrainServiceAlerts` shape (`value.Status`, `value.Message[]`). Check it against a real disruption.

## Data
Weather: data.gov.sg / NEA, Singapore Open Data Licence. Transport: LTA DataMall, subject to its terms.

