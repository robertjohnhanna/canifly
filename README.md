# seehear

**One map. Every keyless public signal. Two audio domains you can click and hear.**

A single self-contained HTML file OSINT dashboard: live earthquakes, military
aircraft, emergency squawks, the ISS, satellites, natural disasters, global
disaster alerts, rocket launches, geopolitics — plus a scanner-style radio
panel spanning **police/fire (OpenMHz)**, **aviation ATC (LiveATC)** with live
METARs, and **broadcast/internet radio (radio-browser)**.

No backend. No build step. No API keys. No `npm install`. **Open `index.html`
directly in a browser.** Fully responsive — works on iPhone, iPad, and desktop.

## Run it

```
open index.html      # or just double-click it
```

MapLibre GL and satellite.js load from CDN; every data feed is fetched
client-side from keyless public APIs. Nothing is installed or served. Every
feed either renders on the map or plays audio in-app — nothing is a bare link
out to another site.

## SITREP — situational awareness at a glance

On load, seehear locates you (instant IP fix via ipwho.is, refined by GPS if
you allow the browser prompt — denial is silently tolerated), flies the map to
your position, drops a pulsing marker, and builds a live **SITREP briefing**
that refreshes every 60 s:

The SITREP reflects the **current map area** (viewport), not a fixed point —
pan or zoom and it re-reads what's in view (nearest ATC tower, aircraft, ships,
quakes, disasters, broadcast stations…). It has three heights: minimise (▾),
normal, and **▲ expand** to fill its whole bay over the map.

- **Anomalies** — a correlation pass across every in-view feed surfaces the
  outliers at the top (aircraft squawking 7500/7600/7700, red/orange GDACS
  alerts, M5+ quakes). When anything is flagged, the **SITREP title turns
  red** (warnings) or **amber** (alerts).
- ✈ every aircraft in view (auto-enabled layer, adsb.lol point query)
- 🎖 military aircraft in view, with closest callsign
- 🚨 aircraft squawking 7700/7600/7500 worldwide, with distance to nearest
- 🌍 nearest earthquake and 🔥 nearest natural event, with distances
- 🌀 nearest global disaster (GDACS)
- 📟 **auto-tuned police/fire audio** — the OpenMHz system matching your
  city/region starts monitoring automatically
- 🛬 nearest ATC tower with a one-tap LISTEN chip
- 📡 broadcast stations in view with a one-tap OPEN chip

Every row has a VIEW/LISTEN/OPEN action. The SITREP is a bottom bar spanning
the gap between the side panels; its header carries a UTC clock, your resolved
location, and a **⌖ ME** button that recenters on you. The map keeps its
content centred in the open area above. There is no top chrome — the two side
panels (OBJECTS, left; radio, right) and the SITREP bar are all anchored to the
screen edges. Position is resolved client-side and never leaves the browser.

Units are imperial (mi/ft, °F, mph) and the interface is set in a minimalist
all-caps style, with measurement units left in natural case.

Every marker is a purpose-drawn icon: aircraft and ships are silhouettes
rotated to their heading/course, and hazards carry type-specific glyphs
(fire, volcano, cyclone, flood, launch, antenna…). The map itself is
chrome-free — no on-map control widgets — with zoom by scroll/pinch and
recenter via ⌖ ME.

## Live map layers

| Group | Layer | Source | Refresh |
|---|---|---|---|
| Air | Aircraft (in view, civil) | adsb.lol point query | 30 s |
| | Military aircraft | adsb.lol ADS-B | 60 s |
| | Emergency squawks (7700/7600/7500) | adsb.lol | 60 s |
| Surface | Amtrak trains | amtraker v3 | 60 s |
| Space | ISS live position | wheretheiss.at | 15 s |
| | Bright satellites | CelesTrak TLE → SGP4 propagated in-browser | 60 s |
| | GPS constellation | CelesTrak TLE → SGP4 | 2 min |
| | Starlink constellation (sampled) | CelesTrak TLE → SGP4 | 2 min |
| | Rocket launches (upcoming, at the pad) | Launch Library 2 | 60 min |
| Hazard | Earthquakes (24 h, M2.5+) | USGS | 60 s |
| | Natural events — wildfires, storms, volcanoes, ice | NASA EONET v3 | 10 min |
| | Global disasters — quakes/cyclones/floods (alert-graded) | GDACS (UN/EC) | 30 min |
| | Live seismic (real-time, global) | EMSC seismicportal | 2 min |
| | River / flood gauges (in view) | USGS water services | 10 min |
| | Holocene volcanoes | Smithsonian GVP (curated static) | — |
| Conflict | Ukraine frontline | DeepStateMap | 30 min |
| Signals | Broadcast stations (in view) | radio-browser | on pan |
| | ATC towers | LiveATC (static set) | — |

**Starts checked:** all aircraft (civil + military) — every other intel feed
loads in the background so the SITREP and anomaly detection stay complete
without cluttering the map. All signal-map layers start unchecked.

**Dossier:** right-click (desktop) or long-press (touch) anywhere →
reverse-geocoded place (Nominatim) + country profile (RestCountries) + head of
state (Wikidata SPARQL) + Wikipedia summary.

**Search:** place name or raw `lat,lon` via OSM Nominatim.

## The radio panel — the point of this tool

1. **Police / Fire — OpenMHz.** Pick any of ~440 trunked-radio systems → live
   calls auto-play newest-last through one `<audio>` element, with a scan mode
   that cycles systems. It starts **paused**; the ▶/⏸ button toggles the
   auto-monitor, and only present-and-newer calls play (no backlog replay).
   Unencrypted talkgroups only; expect the trunk-recorder delay.
2. **Aviation ATC — LiveATC + METARs.** Major world airports with live METAR
   weather (aviationweather.gov) inline in each row. ▶ tries the direct mp3
   stream (`<audio>` playback isn't CORS-gated); ↗ opens LiveATC's official
   player as the guaranteed fallback.
3. **Broadcast — radio-browser.** Thousands of internet-radio stations,
   defaulting to your country → tap to play in-app through the shared `<audio>`
   element. Search by name for any station worldwide.

## Mobile

Designed for iPhone/iPad as first-class targets: bottom MAP / LAYERS / RADIO
navigation with slide-up sheets, 44 px touch targets, safe-area (notch/home
indicator) insets, long-press dossier, `viewport-fit=cover`, momentum
scrolling. Desktop gets side-by-side panels.

## Design notes & honest caveats

- **`<audio>` playback is not gated by CORS** — only JSON `fetch()` is. So
  OpenMHz `.m4a` calls and LiveATC mp3 streams play cross-origin fine; only
  the OpenMHz *JSON API* carries a CORS/Cloudflare risk.
- **Every feed either renders on the map or plays audio in-app.** Link-only
  features (ones that just open another website with no in-app payoff) were
  deliberately removed.
- Every layer loader is defensive with silent auto-retry (exponential backoff):
  one dead feed never breaks the board, and failures show a neutral count
  rather than an error.
- **Dropped because they require a key** (violating the keyless rule):
  OpenAQ v3, NASA FIRMS (MAP_KEY), APRS.fi, OpenSky, aisstream, Shodan,
  Global Fishing Watch, Sentinel Hub.

## Provenance

The list of public endpoints (and the keyless/keyed split) draws on
ShadowBroker's data-source table
([github.com/BigBodyCobain/Shadowbroker](https://github.com/BigBodyCobain/Shadowbroker),
AGPL-3.0). Only the *list of public endpoints* — public knowledge — is reused,
not any of their code. ShadowBroker is the full-fidelity Docker build; seehear
is deliberately the opposite: minimal, keyless, one file.
