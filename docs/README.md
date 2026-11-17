# NoticeToSimPilots

NoticeToSimPilots is a planned Windows add-on for Microsoft Flight Simulator 2024. It fetches real-world NOTAMs (Notices to Air Missions) for the airports near the aircraft and makes the operationally important ones visible inside the simulator. Examples are a taxiway closed for maintenance, shown with closure markers and a service vehicle with workers on the taxiway, or a temporary crane, shown as a crane model at the reported position. The app is a background tray program, with an optional separate settings and viewer program. Later phases add an in-sim EFB (electronic flight bag) app and a flight plan mode that uses SimBrief.

The product is intended for commercial release, most likely for a small one-time fee. A free open-source project with a similar name, NotamsInjector, already exists. See [research.md](research.md) for how it differs.

## Status

Planning only. No code has been written and nothing has been built or tested by us. Every statement in these documents is either a research finding from outside sources, a proposal, or an open question. The confidence level of each finding is marked in [research.md](research.md).

## Feasibility in brief

Confidence labels: (a) official docs, (b) forum post or reference project observation, (c) not yet verified by us.

- Creating custom SimObjects at runtime through SimConnect looks feasible. The API is documented (a). The reference project reports that its own custom test object spawned successfully (b). We have not tested this (c).
- Scenery-library objects cannot be spawned this way. The reference project's tests failed for 20 of 20 airport barrier titles (b).
- Taxiway layout can be read from the sim's own airport data through the SimConnect facilities API (a). Whether third-party airport scenery is reflected is not verified (c).
- SimConnect has no API to close a runway, and no way to make ATC or AI traffic respect a closure (b). The product is therefore visual and informational only.
- The NOTAM data source is the largest unresolved risk. No provider has been chosen, and no commercial redistribution terms have been read (c).
- Aircraft-specific EFB support (Fenix, PMDG) depends on partnerships, not only on engineering (b, c).

## Documents

- [research.md](research.md): what the sim allows, NOTAM data sources, EFB, SimBrief, distribution, the reference project, open questions, and sources.
- [features.md](features.md): what each NOTAM type does in the sim, the two injection modes, the apps, non-goals and known limits.
- [architecture.md](architecture.md): proposed components, data flow, performance budget and technology choices. All of it is proposed.
- [roadmap.md](roadmap.md): phases in order, each with a goal and a definition of done, plus the risks list.
