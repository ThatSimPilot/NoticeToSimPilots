# Architecture

Everything in this document is a **proposal**. Nothing has been built, prototyped or measured. Where a proposal depends on an unverified research finding, the finding is referenced in [research.md](research.md).

## 1. Components

### 1.1 Tray app (proposed)

- **Proposed technology:** C# with .NET. The reason given is that Microsoft ships a managed SimConnect library. The reference project shows that Python also works (b).
- **Responsibilities:**
  - Track aircraft position from SimConnect.
  - Decide which airports are in scope (radius mode, or flight plan mode in a later phase).
  - Fetch NOTAMs, through the cache or the backend proxy.
  - Parse and classify each NOTAM by type and placeability.
  - Read the airport layout from the sim (taxiway points, paths and names).
  - Create and remove sim objects.
  - Serve a local HTTP API on localhost for the viewer and, later, the EFB app.

### 1.2 Settings and viewer (proposed)

- A web page served by the tray app and opened in a small window.
- Proposed so that the same UI can be reused inside the EFB later. Whether the in-sim EFB can reach localhost is not yet verified (c).

### 1.3 Community package (proposed)

- Original SimObject models, compiled with the MSFS SDK package tool:
  - closure cross (runway)
  - barriers
  - cones
  - crane
  - service vehicle
  - worker
- Models must be original, not reused from base-game assets. This is planned because of the reading of the Game Content Usage Rules (b, one developer's reading).
- Later: the MSFS 2024 EFB app package.

### 1.4 Backend proxy and cache (proposed, conditional)

- Holds the NOTAM provider key, so the key is not shipped inside the desktop app.
- Caches NOTAM results per airport, so cost grows with distinct airports, not users.
- Checks licence entitlement.
- Needed only if the chosen provider's terms require it. This is not yet known (c).

### 1.5 Installer (proposed)

- Installs the tray app.
- Copies the community package into the MSFS Community folder.
- Optionally starts the tray app with Windows.

## 2. Data flow

```
Sim position (SimConnect)
  -> airports in scope (radius mode; flight plan mode later)
  -> NOTAM fetch (via local cache; backend proxy if required)
  -> parse and classify
  -> match to sim airport layout (taxiway, runway, stand data from SimConnect)
  -> create or remove sim objects near the aircraft
  -> expose full NOTAM list over localhost to viewer and, later, EFB
```

Notes on the flow:

- The NOTAM list is always stored, including items that cannot be drawn. The viewer reads from this list.
- Object creation and removal happen only for NOTAMs in range of the aircraft.
- The matching step is the most likely point of failure. Taxiway matching is best effort for partial closures (see [features.md](features.md)).

## 3. Performance budget (proposed)

The add-on must be light on performance. The figures below are targets to test against, not measured results.

| Item | Proposed target |
|---|---|
| Aircraft position poll | Every 1 to 5 seconds. |
| Per-frame work | None. The app does not run in the sim's frame loop. |
| Objects spawned | Only for airports near the aircraft (about 10 nm). Removed when out of range. |
| Maximum spawned objects | About 50 at any time. A starting figure chosen to keep the load small, not a measured limit. The crash once reported at 45 to 60 injections is marked fixed (b); the real safe count must be measured in the current build (c). |
| Model complexity | Static models with low polygon counts. No AI behaviour. |
| NOTAM fetch | Each airport fetched at most every 15 minutes. Results cached locally. |

## 4. Technology choices

| Area | Proposed choice | Basis | Status |
|---|---|---|---|
| Tray app language | C# / .NET | Managed SimConnect library from Microsoft. | Proposed. Python (reference project) is the alternative. |
| Sim interface | SimConnect `_EX1` object creation | Official docs recommend `_EX1` (a). | Proposed. Not tested by us. |
| Object models | Original models, compiled with MSFS SDK | Raw glTF cannot be spawned (b). Stock titles are unreliable (b). | Proposed. |
| Airport layout | SimConnect facilities API | Documented TAXI_POINT, TAXI_PATH and related fields (a). | Proposed. Third-party scenery coverage not verified (c). |
| Local cache | Local database (SQLite is used by the reference project) | Reference project (b). | Proposed. |
| Settings and viewer UI | Web page served on localhost | Reuse in EFB later. | Proposed. |
| EFB app | MSFS 2024 EFB API, TypeScript or JSX, Node and npm | Official docs (a). | Proposed, later phase. |
| Backend | Small service that holds the NOTAM key and caches results | Keys cannot be shipped in a desktop app. | Proposed, conditional on provider terms. |
| Installer | To be chosen | Not yet researched. | Open. |

## 5. Open architecture questions

- Which NOTAM provider is used. This decides whether the backend is needed.
- Whether the tray app is written in C# or Python. This is a choice between two options with a working reference (b) and a managed SimConnect library (a).
- How the license key and activation work for a one-time fee.
- How updates are delivered to users.
- Whether the viewer is a separate program or only a page served by the tray app.
