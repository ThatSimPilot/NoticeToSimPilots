# Features

This document describes what NoticeToSimPilots is planned to do. None of it is built or tested. Where a behaviour depends on research that is not yet verified, the confidence is stated.

Confidence levels used below:

- **High**: the method is documented by the sim's official API and the reference project already implements it (see [research.md](research.md)). We have not tested it ourselves.
- **Medium**: the method is plausible, but depends on data or behaviour not yet verified.
- **Best effort**: the result depends on text matching that will sometimes fail.
- **List only**: the NOTAM is shown in the viewer but is not drawn in the sim.

## 1. NOTAM handling by type

Every NOTAM appears in the viewer, whether or not it can be drawn. Each entry carries a note saying whether it was placed in the sim.

| NOTAM type | In the sim | Confidence |
|---|---|---|
| Obstacle or crane with coordinates | Crane or mast model at the reported position. Height matched where possible. Lit or unlit per the NOTAM. | High |
| Runway closed | Closure crosses at both runway ends. | High |
| Taxiway closed (whole taxiway) | Barriers and closure markers at each end. Optional parked service vehicle and workers. | Medium. Depends on taxiway data read from the sim (see [research.md](research.md), section 1.4). |
| Taxiway closed (part, "between A and B") | Same as whole taxiway, on the matched segment only. | Best effort |
| Apron or stand closed | List only at first. Cones or barriers in a later phase. | List only, later phase |
| Navaid, lighting, procedure, airspace, vague work notices | Shown in the viewer only. | List only |

Notes on the table:

- Runway and taxiway placement uses airport data read from the sim where possible. The reference project used external OurAirports data for runways, which can be offset from the sim's airport. Reading from the sim is the planned approach, and it has not been tested (c).
- Navaid outages (ILS, VOR, NDB) are not in scope for the first versions. The sim cannot switch them off through SimConnect alone (b).
- Vague notices are not placed, because there is no reliable position to place them at.

## 2. Injection modes

An injection mode decides which airports the app fetches NOTAMs for, and therefore which objects can appear in the sim.

### 2.1 Radius mode (default at first)

- The app fetches NOTAMs for airports within a set distance of the aircraft.
- The default radius is not yet decided. The reference project uses 50 nm for fetching and places objects within 10 to 15 nm (b). We propose to follow a similar approach, to be confirmed.
- Objects are created only for airports near the aircraft, and removed when the aircraft moves out of range or the NOTAM expires.

### 2.2 Flight plan mode (later phase)

- The app reads the user's latest SimBrief flight plan and fetches NOTAMs only for the departure, destination and alternate airports.
- This is planned for phase 4 (see [roadmap.md](roadmap.md)).
- It reduces the number of data requests to about three airports per flight, which helps data cost.
- Field names in the SimBrief response have not been checked (c).

## 3. Apps

| App | Status | What it does |
|---|---|---|
| Tray app | Required. First phase. | Runs in the background. Tracks aircraft position, fetches and parses NOTAMs, creates and removes sim objects. Does everything needed for the feature to work. |
| Settings and NOTAM viewer | Optional. Early phase for the viewer, later for the full product. | Lets the user configure settings and view all NOTAMs, including those not drawn in the sim. Does not need to run for the sim feature to work. |
| MSFS 2024 EFB app | Later phase (phase 5). | Shows the NOTAM list inside the in-sim EFB. Depends on the 2024 EFB API (see [research.md](research.md), section 3). |
| Aircraft-specific EFB support (Fenix, PMDG and similar) | Later phase (phase 6). | Depends on partnerships with the aircraft developers. There is no public route found so far (b). |

## 4. Non-goals

These are out of scope, now and in the planned versions:

- Changing ATC or AI traffic behaviour. The sim offers no route for this (b).
- Enforcing closures. Closures are visual and informational only.
- Xbox and PlayStation. The product depends on a PC tray executable.
- Replacing a real briefing tool. The product must carry a notice that it is not for real-world navigation.

## 5. Known limits

- **Data coverage.** Not every NOTAM can be placed. Free text often lacks coordinates or taxiway names.
- **Data licence.** The NOTAM source is not yet chosen. Commercial use may be restricted or cost money per request. Until this is settled, the product cannot be sold (c).
- **Sim updates.** Object spawning has broken in past sim updates (b). A sim update may break the feature without warning.
- **Object count.** The app will cap the number of spawned objects to stay light (see [architecture.md](architecture.md)). A crash after roughly 45 to 60 injections was reported in May 2025 and marked fixed in a later beta build (b); the safe count in the current build is not yet measured (c).
- **Model quality.** Models will be original and low polygon (proposal). Their look is not yet designed.
- **Positioning.** Objects are placed at the reported coordinates. Ground orientation is not verified (c).
- **Aircraft EFBs.** Support in third-party aircraft depends on those developers, not on this project's code.
- **Platform.** PC only.

## 6. Required notice

The app and its viewer must show a notice that the information is not for real-world navigation, and that the user must consult official NOTAM sources and their own briefing before flight.
