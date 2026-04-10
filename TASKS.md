# Heritage Railway Mod — Claude Code Task List
**Transport Fever 2 | Lua Mod**

> Work through phases in order. Each task is independently testable. Do not proceed to the next phase until the current phase is verified in-game or via console output.

---

## Phase 0 — Project Scaffolding

- [ ] **0.1** Create the mod directory structure:
  ```
  heritage_railway/
  ├── mod.lua
  ├── res/
  │   ├── scripts/
  │   │   ├── heritage_check.lua
  │   │   ├── revenue_modifier.lua
  │   │   ├── maintenance_modifier.lua
  │   │   └── tourist_demand.lua
  │   └── ui/
  │       └── heritage_panel.lua
  └── README.md
  ```

- [ ] **0.2** Write `mod.lua` with correct metadata fields: `name`, `description`, `tags`, `minorVersion`, `severityAdd`, `severityRemove`. Set `severityAdd = "NONE"` and `severityRemove = "CRITICAL"` as starting values.

- [ ] **0.3** Register all script files in `mod.lua` using the appropriate TF2 script loading pattern.

- [ ] **0.4** Confirm the mod appears and loads without errors in the TF2 mod manager. Log a startup message to console to verify script execution.

---

## Phase 1 — Heritage Line Flag (UI Toggle)

- [ ] **1.1** Research how TF2 exposes line configuration UI via Lua. Identify the correct callback or hook for adding custom line parameters (check `res/scripts/` API docs and community wiki).

- [ ] **1.2** In `heritage_panel.lua`, add a boolean toggle to the line configuration panel labelled **"Heritage Railway"**. Default value: `false`.

- [ ] **1.3** Persist the toggle state to the line entity using TF2's parameter storage API. Confirm the value survives saving and loading a game.

- [ ] **1.4** Add a read-only status field beneath the toggle that will later display validation feedback (e.g. "Heritage Active", "Steam check failed"). Leave it blank for now.

- [ ] **1.5** Test: open a line's config panel, toggle Heritage on and off, save/load, confirm state is preserved. Log the stored value to console.

---

## Phase 2 — Validation Logic

> All validation functions live in `heritage_check.lua`. Each check returns `true` (pass) or `false, reason_string` (fail).

- [ ] **2.1** Write `checkSteamOnly(lineId)`: query all vehicles on the line. Return `false, "Non-steam traction detected"` if any traction unit is not steam-powered. Identify how TF2 exposes vehicle type/propulsion data from the asset definition.

- [ ] **2.2** Write `checkPassengerOnly(lineId)`: return `false, "Freight stock detected"` if any wagon/carriage has a non-passenger cargo type.

- [ ] **2.3** Write `checkEraMatch(lineId)`: 
  - Read the introduction year of the lead steam engine from its asset data.
  - Check all carriages fall within ±20 years of that introduction year.
  - Return `false, "Non-period stock detected"` on mismatch.

- [ ] **2.4** Write `checkMinimumLength(lineId)`:
  - Calculate total line length in metres using TF2's path/segment API.
  - Return `false, "Line too short (minimum 4 km)"` if under 4000 metres.

- [ ] **2.5** Write `runAllChecks(lineId)`: call checks 2.1–2.4 in order. Return `true, nil` if all pass, or `false, firstFailReason` on first failure.

- [ ] **2.6** Wire `runAllChecks` to the Heritage toggle in `heritage_panel.lua`. Display the failure reason in the status field from task 1.4. If all checks pass, display "Heritage Active ✓".

- [ ] **2.7** Test each check individually by deliberately failing each condition (mixed traction, freight wagon, wrong-era carriage, short line). Confirm correct failure messages appear.

---

## Phase 3 — Maintenance Cost Reduction

- [ ] **3.1** Research TF2's API for modifying vehicle running/maintenance costs. Identify whether this is done via a multiplier callback, entity parameter override, or asset property.

- [ ] **3.2** In `maintenance_modifier.lua`, write `applyMaintenanceReduction(lineId)`: apply a **0.5× multiplier** (50% reduction) to the maintenance cost of all vehicles on a heritage-flagged line.

- [ ] **3.3** Write `removeMaintenanceReduction(lineId)`: restore default maintenance costs when heritage status is lost (toggle turned off, or a check fails).

- [ ] **3.4** Hook both functions to the heritage status state: call `apply` when status becomes active, `remove` when it becomes inactive.

- [ ] **3.5** Test: note baseline maintenance costs on a line, activate heritage status, confirm ~50% reduction in the finance panel. Deactivate and confirm costs restore.

---

## Phase 4 — Fare Multiplier (Length-Scaled)

- [ ] **4.1** In `revenue_modifier.lua`, implement the fare multiplier formula:
  ```lua
  local BASE_MULT = 1.5
  local MAX_MULT  = 2.5
  local SCALE_DIST = 30000  -- metres

  local function fareMultiplier(lineLength)
      local t = math.min(lineLength / SCALE_DIST, 1.0)
      return BASE_MULT + (MAX_MULT - BASE_MULT) * t
  end
  ```

- [ ] **4.2** Research TF2's API for applying a revenue/fare multiplier to a line. Identify the correct hook (per-tick revenue callback, line income modifier, or similar).

- [ ] **4.3** Write `applyFareMultiplier(lineId)`: calculate line length, compute multiplier, apply to line revenue.

- [ ] **4.4** Write `removeFareMultiplier(lineId)`: restore default fare rate.

- [ ] **4.5** Hook both to heritage status changes (same pattern as Phase 3).

- [ ] **4.6** Test: run a heritage line, open the finance panel, confirm per-passenger revenue is elevated. Test a short line (~5 km) and a long line (~25 km) to confirm scaling is working correctly.

---

## Phase 5 — Tourist Passenger Generation

- [ ] **5.1** Research TF2's passenger demand/generation API. Identify how to inject additional passenger demand at a specific station or city node.

- [ ] **5.2** In `tourist_demand.lua`, implement the frequency-scaling formula:
  ```lua
  local MAX_TOURISTS = 50   -- per station per in-game month (tune via playtesting)
  local FREQ_CAP     = 4    -- trains/hour cap

  local function touristBonus(trainsPerHour)
      local t = math.min(trainsPerHour / FREQ_CAP, 1.0)
      return math.floor(MAX_TOURISTS * t)
  end
  ```

- [ ] **5.3** Write `getLineFrequency(lineId)`: calculate trains per hour using headway data from TF2's line statistics API.

- [ ] **5.4** Write `applyTouristDemand(lineId)`: for each station on the line, inject `touristBonus(frequency)` additional passenger demand. Tag injected demand as heritage-origin to allow clean removal.

- [ ] **5.5** Write `removeTouristDemand(lineId)`: remove all heritage-tagged demand injections from all stations on the line.

- [ ] **5.6** Hook `applyTouristDemand` to a simulation tick callback (not every tick — run on a timer, e.g. every in-game month). Recalculate frequency each time so the bonus responds to service changes.

- [ ] **5.7** Test: run a heritage line with low frequency (1 train/hour) and observe modest tourist numbers. Increase to 4+ trains/hour and confirm passenger generation increases. Deactivate heritage and confirm tourist demand drops.

---

## Phase 6 — State Management & Edge Cases

- [ ] **6.1** Handle the case where a heritage line **loses qualification mid-game** (player swaps in a diesel, adds freight, etc.). Validation should run on each simulation tick (or on a short timer). If checks fail, deactivate heritage status and remove all bonuses immediately.

- [ ] **6.2** Handle **line deletion**: ensure all bonuses and tourist demand injections are cleaned up when a heritage-flagged line is removed.

- [ ] **6.3** Handle **vehicle replacement**: if a player replaces a vehicle on a heritage line, re-run all checks. Update UI status field accordingly.

- [ ] **6.4** Handle **new game / save-load**: ensure heritage state is correctly restored on load. Test by saving mid-game with an active heritage line and reloading.

- [ ] **6.5** Add a **console command or debug toggle** to print current heritage state for all lines (active/inactive, which checks pass/fail, current multipliers). This is a development aid — can be removed before release.

---

## Phase 7 — Balancing & Playtesting

- [ ] **7.1** Play a full game session from early-mid game. Attempt to build a viable heritage line as soon as steam engines are available. Note whether the bonuses make it economically compelling without being dominant.

- [ ] **7.2** Tune `MAX_TOURISTS` up or down based on observed passenger generation. Target: heritage lines should be profitable and distinctive, not a primary money-printer.

- [ ] **7.3** Tune `BASE_MULT` and `MAX_MULT` if the fare premium feels too weak or too strong relative to vanilla line revenue.

- [ ] **7.4** Test with the **minimum length line** (just over 4 km). Confirm it is marginally viable but not highly profitable — the multiplier at short distance should be modest.

- [ ] **7.5** Test with a **maximum-length line** (~30+ km). Confirm the 2.5× cap prevents runaway profits on long routes.

- [ ] **7.6** Check for **compatibility issues** with any commonly used mods (rolling stock packs, economy overhauls). Note any conflicts in README.

---

## Phase 8 — Polish & Release Prep

- [ ] **8.1** Write player-facing UI strings for all status messages. Ensure they are clear and non-technical (e.g. "All heritage conditions met ✓", "Non-period stock detected — bonuses paused").

- [ ] **8.2** Add a **tooltip** to the Heritage toggle explaining the conditions and bonuses in plain language.

- [ ] **8.3** Remove or gate all debug logging behind a `DEBUG = false` flag in `mod.lua`.

- [ ] **8.4** Write `README.md` covering: what the mod does, how to activate a heritage line, the four qualification conditions, and the three bonuses with approximate values.

- [ ] **8.5** Test a **clean install** — remove and reinstall the mod, start a new game, confirm no errors on load.

- [ ] **8.6** Package for Steam Workshop upload: verify mod thumbnail, description, and tags are set correctly in `mod.lua`.

---

## Constants Reference (tune during Phase 7)

| Constant | Starting Value | Purpose |
|---|---|---|
| `MIN_LENGTH_M` | 4000 | Minimum line length for heritage status |
| `ERA_WINDOW_YRS` | 20 | Max year gap between engine and carriage eras |
| `MAINTENANCE_MULT` | 0.5 | Running cost multiplier (lower = cheaper) |
| `BASE_FARE_MULT` | 1.5 | Fare multiplier at minimum qualifying length |
| `MAX_FARE_MULT` | 2.5 | Fare multiplier cap at 30 km+ |
| `FARE_SCALE_DIST_M` | 30000 | Distance at which max fare multiplier is reached |
| `MAX_TOURISTS` | 50 | Peak tourist passengers added per station per month |
| `FREQ_CAP` | 4 | Trains/hour at which tourist bonus is maximised |
