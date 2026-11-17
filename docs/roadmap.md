# Roadmap

Phases are listed in the proposed order. Each phase has a goal and a definition of done. The order and the "done" criteria for phases 1 to 6 are proposals, not decisions.

Nothing in this roadmap has been started. Phase 0 is the first step.

## Phase 0: Prove the unknowns

Throwaway tests. No product code is kept from this phase.

Goal: answer the four questions that could stop the project.

1. Can one custom static SimObject be created with `_EX1` in the current MSFS 2024 build?
2. Can taxiway points, paths and names be read for one stock airport and one third-party airport, and can markers be placed along a named taxiway?
3. Which NOTAM provider can be used, after reading its terms and estimating cost?
4. Is the product name confirmed, and has the NotamsInjector author been contacted about licence and overlap?

Done when each question has a yes or no answer, written down with the evidence.

## Phase 1: First usable version

Goal: a tray app that shows real NOTAMs in the sim for the airports near the aircraft.

Scope:

- Tray app.
- Radius mode.
- Cranes and obstacles.
- Runway closure crosses.
- A viewer that lists all NOTAMs, including those not drawn.
- Local cache.

Done when the tray app runs on a test flight, draws the supported NOTAM types at the correct places, lists the rest in the viewer, and stays within the performance budget in [architecture.md](architecture.md) on the test machine.

## Phase 2: Taxiway closures

Goal: make taxiway closures visible, the gap the reference project left.

Scope, in order:

1. Whole-taxiway closures with barriers and closure markers.
2. Partial closures ("between A and B") on the matched segment.
3. Static vehicles and workers.

Done when each step is tested on at least one stock airport and one third-party airport, and the partial closure step reports its matching result in the viewer for every NOTAM.

## Phase 3: Make it a product

Goal: a version that can be installed and sold.

Scope:

- Installer.
- Settings and viewer app.
- Licence activation.
- Backend proxy, if the chosen provider's terms require it.
- Updates.

Done when a clean Windows machine can install, activate and remove the product, and the provider terms have been read and accepted in writing.

## Phase 4: Flight plan mode

Goal: fetch NOTAMs only for the departure, destination and alternate airports from the user's SimBrief plan.

Done when a test plan is read correctly, the three airports are the only ones fetched, and the Navigraph commercial terms have been confirmed.

## Phase 5: MSFS 2024 EFB app

Goal: show the NOTAM list inside the in-sim EFB.

Done when the EFB app is packaged, appears in the EFB for an aircraft, and reads data from the tray app over localhost.

## Phase 6: Aircraft-specific EFBs

Goal: show NOTAMs in third-party aircraft EFBs (Fenix, PMDG and similar).

Done when a partnership or documented route exists for the aircraft in question. This phase is blocked until then, because no public route has been found (b).

## Later or maybe

- Navaid outages through a WASM module. Requires a WASM module that does not yet exist.
- Animated workers.
- Apron and stand closures.
- MSFS 2020 support.

These items have no committed date and no committed scope.

## Risks

- NOTAM data licence and cost, against a one-time fee.
- Taxiway data quality or coverage in the sim.
- Free-text NOTAMs that cannot be parsed reliably.
- Sim updates breaking object spawning. This has happened before (b).
- No public route into Fenix or PMDG EFBs.
- An existing free project in the same space (NotamsInjector).
- Who creates the 3D models, and at what cost.
- The reported injection crash, if it returns in later builds (b).
- The MIT licence claim of the reference project is unconfirmed, because no LICENSE file is present (b).
