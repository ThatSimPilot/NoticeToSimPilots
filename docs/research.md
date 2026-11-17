# Research findings

This document records what has been found so far. It does not describe built or tested behaviour.

Each finding carries one of three labels:

- **(a) Official docs**: stated in Microsoft or vendor documentation.
- **(b) Forum or reference project**: reported in a forum post, or observed in the NotamsInjector project's code or test reports.
- **(c) Not verified**: not yet checked by us. Treat as a hypothesis.

## 1. What the sim allows

### 1.1 Runtime object creation

- (a) An external application can create an object at a latitude and longitude at runtime with `SimConnect_AICreateSimulatedObject_EX1`, and remove it with `SimConnect_AIRemoveObject`. Microsoft's docs recommend the `_EX1` variant and describe the original function as legacy. The object is named by its "container title".
- (a) Only SimObjects can be created this way. Ordinary scenery-library objects cannot. The docs describe a scenery object as having no simulation elements, and a SimObject as participating in the simulation.
- (a) Non-aircraft SimObject categories listed in the older MSFS 2020 docs include StaticObject, GroundVehicle and Human. This list has not been confirmed against the 2024 documentation (c).
- (b) The reference project's own custom test SimObject spawned successfully in MSFS 2024. This is the main evidence that custom SimObjects work (b). We have not repeated the test (c).
- (b) Raw glTF files cannot be spawned in MSFS 2024. The model must be compiled with the MSFS SDK package tool.

### 1.2 Known stability problem

- (b) A crash-to-desktop after roughly 45 to 60 SimConnect object injections was reported against MSFS 2024 Sim Update 3 beta in May 2025. Staff said the fix was in SU3 flighting build 1.5.8.0. The reporter confirmed in June 2025 that it was fixed.
- The thread describes about 100 create calls with no removals; it does not say whether the limit was on objects alive at once or on total calls.
- Consequence: treat the build containing the fix as the minimum supported version, and re-run a spawn-count test after each sim update (c, until we test it). The object cap in [architecture.md](architecture.md) is a performance choice, not a workaround for this crash.

### 1.3 Object placement

- (b) A developer reported that StaticObject SimObjects do not orient to terrain, and that GroundVehicle objects drive away when spawned. Another answer suggests spawning on the ground at altitude 0 so objects stay on the terrain. This has not been verified by us (c).
- (b) The reference project found that stock object titles are unreliable. The same crane title succeeded in one test run and failed in another. In one run every title was rejected. Shipping our own models avoids depending on what happens to be installed.
- (b) The reference project's scenery-library tests failed with "Object not retained after create" for all 20 airport barrier titles tested.

### 1.4 Reading airport layout

- (a) The SimConnect facilities API (`SimConnect_AddToFacilityDefinition`) exposes airport data including TAXI_POINT, TAXI_PATH, TAXI_NAME and TAXI_PARKING, with counts such as N_TAXI_POINTS and N_TAXI_PATHS. The app could therefore read the taxiway layout as it exists in the user's sim, and match a NOTAM that names a taxiway to real pavement.
- (c) We have not read the full field lists. It is not verified that third-party airport scenery is reflected in this data.
- (b) The reference project takes runway positions from OurAirports data rather than from the sim. Its markers can be misplaced where the sim's airport differs from real-world data. Reading positions from the sim's facilities data would avoid this, but this is not yet tested (c).

### 1.5 Limits on what can be shown

- (b) SimConnect has no API to close a runway. We found no way to make ATC or AI traffic respect a closure. The product is therefore visual and informational only.
- (b) Switching off an ILS, VOR or NDB is not possible through SimConnect alone. The reference project only classifies and logs these outages, and writes a state file for a WASM module that was never built.

## 2. NOTAM data sources

### 2.1 Official US source

- (a) The FAA moved to its new NOTAM Management Service (NMS) and shut down the legacy US NOTAM System in April 2026 (news and FAA sources).
- (a) The FAA's NMS FAQ says developers must request access to the new NMS API (REST) by emailing notams@faa.gov, to get a clientId and clientSecret. It is not self-serve. The FAQ does not say whether access is free or whether commercial apps may redistribute the data.
- (b) A third-party catalogue says the API documentation is not published to unauthenticated visitors, and a third-party client library describes OAuth 2.0 client-credentials tokens. Neither is confirmed from an FAA source (c).
- (b) The older SWIM Cloud Distribution Service (SCDS) feed of NOTAMs (Solace JMS) needs a SWIM account and a signed FAA Access Agreement. A 2021 FAA meeting summary says it is being retired along with the Federal NOTAM System, and that new Access Agreements were due in 2024. Do not build on it.
- (c) The status of the older api.faa.gov NOTAM API is unverified.
- (c) Whether the FAA will grant NMS access to a commercial third-party app, and on what terms, is unknown. Asking costs one email.

### 2.2 Commercial providers

The table below lists what was found in web searches in November 2026. Vendor pages may be out of date. None of these figures has been checked against a current price list by us.

| Provider | What is recorded | Free or personal key | Confidence |
|---|---|---|---|
| SkyLink | Entry plan about $16 to $19 per month (vendor pages disagree: $15.99, $16, $18.59 and $19 all appear). Pro $49 per month. The page describes global airport coverage but does not name its data source. Its GitHub repo describes an FAA SWIM feed, so non-US coverage is unconfirmed. | 1,000 requests per month free, described as a trial for personal projects. | Vendor pages (b), not verified (c) |
| Notamify | API needs a Pro subscription at $24.9 per month, which includes 50 credits and a 7-day trial. Credit packs: $15 for 50, $79 for 300, $199 for 1,000, about $0.20 to $0.30 a credit. Newer docs list per-endpoint credit costs (for example 5 credits for a briefing request); older docs say 1 credit per request. The active NOTAMs endpoint takes up to 5 airports per request. Global coverage, data source not named. The V2 endpoint offers interpreted NOTAMs with affected infrastructure and map-ready context "where available". | No free API tier. | Vendor page (b), not verified (c) |
| FAA NMS API | See 2.1. US only. | Request access by email. | Official FAQ (a), cost and terms (c) |
| PreflightAPI | US airports. NOTAMs from FAA NMS. Paid tiers not published. | 5,000 calls per month free. | Third-party write-up only (b), not verified (c) |
| ICAO API Data Service | API key required. Paid booster packs, prices not found. Registering means accepting ICAO terms and some purchases may need a licence agreement. | 100 free calls, too few to use. | Vendor page (b), not verified (c) |
| Aviation Edge | Trial rates "from $15". Full price list, plans and data source not found on the NOTAM page. | None found. | Vendor page (b), not verified (c) |
| Boeing / Jeppesen | Basic NOTAMs and Smart NOTAMs APIs. Pricing is shown only after login. Business-to-business contracts only. | Free test of any number of APIs. | Vendor page (b), not verified (c) |
| DTN | NOTAM API exists (first release April 2024, one station per request at first). Pricing not found. | None found. | Vendor page (b), not verified (c) |
| Cirium | Commercial. Pricing not public or not checked. | None found. | Not verified (c) |
| Apify NOTAM and TFR feed | Community-run scraper actor, from $3 per 1,000 results. | Free trial, size not found. | Vendor listing (b), not verified (c). Scraped data, so licence is doubtful. |
| AVWX | Aviation weather API with hobby, pro and enterprise tiers. Basic METAR and TAF parsing is free. An open-source plan exists. Whether it serves NOTAMs was not confirmed. | Free tier for weather. | Not verified (c) |
| CheckWX | Used by the reference project with a user-supplied free-tier key. Terms not checked. | User-supplied free key. | Observed in reference project (b), terms not verified (c) |

- (c) "NOTAMET" was searched for and not found. The closest match is ICAO's NOTAMeter, which is a web tool that reports the age of NOTAMs by state. It is not an API.
- (b) A forum post says autorouter licenses its NOTAM data from EUROCONTROL's EAD, which is commercial. A UK NATS bulletin feed is free but covers the UK only. No free global NOTAM source was found.
- (b) Notamify's terms (a possibly dated copy) say redistribution, resale or commercial use of its data needs written approval, but that this does not apply to data taken through its official API. They also say API keys must not be shared with third parties, and that higher rate limits and bulk exports need a separate written agreement. Nothing was found about end users of an app built on the API.
- (c) No other provider's redistribution or commercial-use terms have been read. This must be settled before anything is sold.
- (b) The reference project's main source is notams.online. It reads that site's responses by decoding them with a key taken from the site's JavaScript. This is not a licensed API and is not suitable for a paid product.

### 2.3 What NOTAM text looks like

- (b) NOTAM content is mostly free text.
- Obstacle and crane NOTAMs usually carry coordinates and height, so they are the easiest to place.
- Runway closures name the runway.
- Taxiway closures name the taxiway. Partial closures ("between A and B") need matching against the taxiway graph, and are best effort.
- Vague notices, for example work in progress near an apron, cannot be placed. They will be list-only.
- (a) Digital NOTAM (AIXM) carries real geometry. EUROCONTROL notes that current practice often leaves precise coordinates out of the NOTAM text (a).

### 2.4 Consequence for a paid product

- An API key cannot be shipped inside a desktop app without exposing it.
- A public release therefore needs either a small backend that holds the key and caches results, or a bring-your-own-key option. This is a design choice, not a settled decision.
- With a shared server cache, data cost grows with the number of distinct airports requested, not with the number of users.
- A one-time fee against recurring data cost is a business risk that is not yet resolved.
- Bring-your-own-key candidates with a free personal tier: SkyLink (1,000 requests per month) and PreflightAPI (5,000 per month, US only). With flight plan mode at about three airports per flight, 1,000 requests covers hundreds of flights a month. This is arithmetic on vendor figures, not a tested number.
- A user-supplied key moves the data cost and the provider's terms onto the user's own account, which suits a one-time fee. Notamify's ban on sharing keys with third parties needs a direct answer on whether a user entering their own key into a third-party app is allowed (c).
- Per-credit pricing (Notamify) makes every shared-cache refresh cost money. Flat-rate plans, or the FAA NMS if access is granted, suit a hosted cache better. This is an inference, not a quote.

## 3. EFB (electronic flight bag)

- (a) MSFS 2024 has an EFB API. Apps are written in TypeScript or JSX, built with Node and npm from a template sample, and shipped as an add-on package. Once packaged, they appear in the EFB for all aircraft.
- (a) Developers can write EFB apps but cannot replace the EFB itself.
- (b) In-sim JavaScript can call a local web server with fetch or WebSockets. HTTPS is not required, but correct CORS headers are. This means an EFB app could read from the tray app over localhost. This was not verified specifically for the 2024 EFB (c).
- (b) No public third-party app API was found for the Fenix A320 EFB.
- (b) PMDG's tablet documentation says apps may be contributed by partnered developers. Aircraft-specific EFB support therefore depends on partnerships, not only on engineering.

## 4. SimBrief

- (b) A user's latest flight plan (OFP) can be fetched from the SimBrief XML fetcher endpoint, by Pilot ID (`userid`) or by username. JSON output is available by adding `json=1` or the newer `json=v2`. Only the latest plan can be fetched.
- (b) API keys are needed for generating flight plans. Whether a key is needed just to fetch was not confirmed (c).
- (c) The exact field names for origin, destination and alternate were not checked.
- (c) Commercial-use terms were not found. They need to be asked of Navigraph.
- Flight plan mode also cuts NOTAM requests to about three airports per flight, which helps data cost.

## 5. Distribution

- (a) The MSFS Marketplace is for in-sim packages that install without external installers.
- Because this product needs a tray executable, the realistic route is selling outside the Marketplace, for example from an own site or a third-party store (b, community guidance). This is a plan, not a ruling.
- For the same reason the product is PC only.
- (b) One developer's reading of the Game Content Usage Rules is that they appear to bar redistributing modified base-game content for payment. This is one developer's reading, not a ruling (b). The plan is to ship original 3D models rather than reuse Asobo assets.

## 6. Reference project: NotamsInjector

Source: [NotamsInjector](https://github.com/Hegewiese/NotamsInjector). We cloned and read the code.

### 6.1 What it is

- A Python 3.11 Windows tray app, using PySide6 for the UI, python-SimConnect, a SQLite cache, and PyInstaller for a single .exe. About 6,800 lines.
- Targets MSFS 2020 and 2024.
- The README says the licence is MIT. The repository contains no LICENSE file, so reuse rights must be confirmed with the author (b).
- Latest commit seen: 14 April 2026.

### 6.2 What it does

- Polls aircraft position. Fetches NOTAMs for airports within a radius (default 50 nm). Re-fetches after moving 5 nm, or every 15 minutes. Caches results in SQLite.
- Obstacles and cranes: spawns stock sim objects by title at the NOTAM coordinates when the aircraft is within 10 to 15 nm. Removes them when out of range or expired. Titles tried include `Construction_BigCrane01`, `Pipelay_Crane_SK3000`, wind turbine, antenna and chimney models. Skips natural obstacles such as trees. Reads lit or unlit wording and sets the object's `LIGHT BEACON` variable. Can add a vertical column of beacon objects to highlight an obstacle.
- Runway closed: ships a generated SimObject package with a flat yellow closed-runway cross in three sizes (arms 10 m, 18 m and 27 m, chosen by runway width, proportions from ICAO Annex 14). Placed at both runway ends, using threshold coordinates from OurAirports `runways.csv`.
- Taxiway closed: detected and logged only. The code has a TODO noting that barrier placement needs taxiway coordinates. No workers or vehicles.
- ILS, VOR and NDB unserviceable: classified in detail, but only logged. Waiting on a WASM module.
- ATIS unserviceable: alerts when the radio is tuned to that frequency.
- In-sim text alerts through SimConnect text messages. A debug window with a NOTAM table, an actions list and a log. A settings window.
- Airport data from OurAirports CSV, with optional OpenAIP enrichment that needs an API key.
- Not present: an EFB app, flight plan mode, an installer, a backend.

### 6.3 Lessons carried over

- Raw glTF cannot be spawned. Models must be compiled with the MSFS SDK package tool (b).
- Scenery-library models do not work (b).
- Stock titles are unreliable (b).
- It uses the legacy create call, not `_EX1` (b).
- It takes runway positions from OurAirports, so markers can be misplaced (b).
- Its duplicate check compares only the first 12 characters of the NOTAM text, which is crude (b). Our design must not copy this.

### 6.4 How NoticeToSimPilots differs

- Taxiway closures made visible. The reference project left this gap.
- Workers and vehicles.
- Geometry read from the sim, not from external data.
- Licensed data (planned, not yet secured).
- An installer, with a separate settings and viewer app.
- Flight plan mode.
- EFB apps.

## 7. Open questions

1. Which NOTAM provider can be used commercially, and at what cost per user and per airport?
2. Does the chosen provider's licence require a backend proxy?
2a. Will the FAA grant NMS API access to a commercial app, and may the data be cached and redistributed?
2b. Does SkyLink return NOTAMs for non-US airports, and do SkyLink's and Notamify's terms allow a user's own key inside a third-party app?
3. Does the SimConnect facilities data include taxiway geometry for third-party airport scenery?
4. Does `_EX1` creation of a custom static SimObject work in the current MSFS 2024 build?
5. Does the crash after 45 to 60 injections still occur in the current build? What is the safe object count?
6. Does a StaticObject stay on terrain when spawned at altitude 0?
7. Which SimConnect SimObject categories are valid for MSFS 2024?
8. Can an EFB app reach a localhost server in the 2024 EFB?
9. What are the SimBrief commercial-use terms, and is an API key needed to fetch a plan?
10. Does the reference project's MIT licence claim apply, and will its author allow reuse?
11. Is the Game Content Usage Rules reading correct for base-game assets?
12. Who creates the 3D models, and at what cost?

## 8. Sources

- [NotamsInjector (reference project)](https://github.com/Hegewiese/NotamsInjector)
- [SimConnect_AICreateSimulatedObject_EX1](https://docs.flightsimulator.com/msfs2024/html/6_Programming_APIs/SimConnect/API_Reference/AI_Object/SimConnect_AICreateSimulatedObject_EX1.htm)
- [SimConnect_AddToFacilityDefinition](https://docs.flightsimulator.com/msfs2024/html/6_Programming_APIs/SimConnect/API_Reference/Facilities/SimConnect_AddToFacilityDefinition.htm)
- [SimObjects (MSFS 2024 docs)](https://docs.flightsimulator.com/msfs2024/html/5_Content_Configuration/Modular_SimObjects/SimObjects/SimObjects.htm) (returned 404 when fetched directly; findings come from the search summary of this page, so re-check it)
- [Electronic Flight Bag API](https://docs.flightsimulator.com/msfs2024/html/6_Programming_APIs/EFB/Electronic_Flight_Bag_API.htm)
- [SU3 beta SimConnect injection crash thread](https://devsupport.flightsimulator.com/t/su3-beta-1-5-3-0-simconnect-injections-exceeding-a-max-number-45-60-causes-2024-ctd/14411)
- [AICreateSimulatedObject StaticObject orientation thread](https://devsupport.flightsimulator.com/t/simconnect-aicreatesimulatedobject-staticobject-doesnt-orient-to-terrain-groundvehicle-drives-away/3557)
- [FSNetworkHttpRequestGet thread](https://devsupport.flightsimulator.com/t/whats-the-proper-way-to-use-fsnetworkhttprequestget/16832)
- [Reusing base content in a payware package thread](https://devsupport.flightsimulator.com/t/reusing-base-content-in-a-payware-package-allowed-or-not/17510)
- [PMDG Tablet Introduction (PDF)](https://downloads.pmdg.com/supportfiles/Docs/PMDG_Tablet_Introduction.pdf)
- [FAA NOTAM FAQs](https://www.faa.gov/about/initiatives/notam/faqs)
- [EAA news on FAA NOTAM system, April 2026](https://www.eaa.org/eaa/news-and-publications/eaa-news-and-aviation-news/2026-04-16-faas-new-notam-system-online-april-18)
- [FAA NMS API (third-party catalogue)](https://apis.io/apis/faa/faa-nms-api/)
- [ICAO API data samples](https://www.icao.int/api-data-samples)
- [Notamify NOTAM API](https://notamify.com/notam-api)
- [Notamify API usage](https://notamify.com/notamify-api-usage)
- [Notamify terms](https://notamify.com/terms)
- [SkyLink NOTAM API blog](https://skylinkapi.com/blog/notam-api-live-structured-notams-any-airport/)
- [SkyLink-API/notam-api on GitHub](https://github.com/SkyLink-API/notam-api)
- [PreflightAPI listing](https://hnshowcase.com/post/47241523/preflightapi-us-airports-weather-notams-and-more-via-one-api)
- [ICAO API Data Service](https://www.icao.int/api-data-service-1)
- [ICAO NOTAMeter](https://www.icao.int/notameter)
- [Boeing developer APIs](https://developer.boeing.com/apis)
- [Aviation Edge NOTAM API](https://aviation-edge.com/notam-api/)
- [EUROCONTROL DNOTAM geometrical and geographical data](https://ext.eurocontrol.int/aixm_confluence/display/DNOTAM/Geometrical+and+geographical+data)
- [Navigraph: fetching OFP data](https://developers.navigraph.com/docs/simbrief/fetching-ofp-data)
- [Navigraph: using the SimBrief API](https://developers.navigraph.com/docs/simbrief/using-the-api)
- [MSFS Marketplace introduction](https://www.flightsimulator.com/guide/introduction-to-the-msfs-marketplace/)
