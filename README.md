# Heritage Railway
### A Transport Fever 2 Mod

Transform your older railways into living heritage lines. Flag any steam-hauled, period-matched passenger route as a Heritage Railway to unlock reduced maintenance costs, premium ticket fares scaled by line length, and tourist demand that grows with your service frequency.

---

## Features

**Heritage Line Designation**
Manually flag any qualifying line as a Heritage Railway via the line configuration panel. When all conditions are met, three economic bonuses activate automatically. If the line falls out of qualification — say, a steam engine is swapped for a diesel — bonuses are suspended until conditions are restored.

**Reduced Maintenance Costs**
Heritage lines benefit from a 50% reduction in vehicle running costs, representing volunteer labour, charitable funding, and preservation society support. This applies to all rolling stock on the line for as long as heritage status is active.

**Premium Ticket Fares**
Passengers pay a heritage premium. The fare multiplier scales linearly with line length — from 1.5× at the minimum qualifying distance up to a cap of 2.5× at 30 km and beyond — reflecting the greater draw and experience of a longer scenic run.

**Tourist Passenger Generation**
Each station on a heritage line generates additional tourist visitors. The volume scales with service frequency, capping at 4 trains per hour, encouraging players to run a meaningful timetable rather than a token service.

---

## How to Activate a Heritage Line

1. Build a line using **steam traction only**
2. Ensure all rolling stock is **passenger carriages** — no freight
3. Ensure all carriages are **within 20 years** of the lead engine's introduction date
4. Ensure the line is at least **4 km long**
5. Open the line configuration panel and toggle **Heritage Railway** to on
6. The status field will confirm *"Heritage Active ✓"* when all conditions are met

If any condition is not met, the status field will display the reason (e.g. *"Non-period stock detected — bonuses paused"*).

---

## Qualification Conditions

| Condition | Requirement |
|---|---|
| Traction type | Steam engines only — no diesel or electric |
| Stock type | Passenger carriages only — no freight wagons |
| Era matching | All carriages within ±20 years of the lead engine's introduction year |
| Minimum length | Line must be at least 4 km end-to-end |

All four conditions must be satisfied simultaneously. Bonuses are removed immediately if any condition is broken mid-game.

---

## Economic Bonuses

| Bonus | Value | Notes |
|---|---|---|
| Maintenance reduction | 50% of base running cost | Applies to all vehicles on the line |
| Fare multiplier | 1.5× – 2.5× base fare | Scales linearly with line length; caps at 30 km |
| Tourist generation | Up to 50 passengers/station/month | Scales with service frequency; caps at 4 trains/hour |

These values are tuned for balance against the vanilla TF2 economy. Heritage lines should be profitable and distinctive — they are not intended as a primary money-printing strategy.

---

## Compatibility

- Tested against vanilla TF2 rolling stock
- Compatible with most rolling stock packs provided vehicles carry correct era and propulsion metadata
- May conflict with mods that substantially alter the passenger demand or revenue systems — check mod descriptions for economy overhaul warnings
- If you encounter a conflict, please open an issue with the name of the conflicting mod

---

## Installation

**Via Steam Workshop**
Subscribe to the mod. It will download and activate automatically.

**Manual Installation**
1. Download or clone this repository
2. Copy the `heritage_railway/` folder to your TF2 mods directory:
   - **Windows:** `%APPDATA%\Transport Fever 2\mods\`
   - **Linux:** `~/.local/share/Transport Fever 2/mods/`
3. Enable the mod in the TF2 mod manager before starting a new game

---

## Known Limitations

- Heritage status is per-line and must be set manually for each qualifying line
- The era-matching check reads vehicle introduction years from asset metadata; rolling stock mods that omit or incorrectly set this field may not validate correctly
- Tourist demand is recalculated monthly in-game time — there is a short lag before frequency changes are reflected in passenger numbers

---

## Tunable Constants

For players comfortable editing Lua, all key values are defined as named constants at the top of `mod.lua` and can be adjusted to taste:

```lua
MIN_LENGTH_M       = 4000   -- Minimum line length in metres
ERA_WINDOW_YRS     = 20     -- Permitted year gap between engine and carriage eras
MAINTENANCE_MULT   = 0.5    -- Running cost multiplier (0.5 = 50% reduction)
BASE_FARE_MULT     = 1.5    -- Fare multiplier at minimum qualifying length
MAX_FARE_MULT      = 2.5    -- Fare multiplier cap
FARE_SCALE_DIST_M  = 30000  -- Distance at which max fare multiplier is reached
MAX_TOURISTS       = 50     -- Peak tourist passengers per station per in-game month
FREQ_CAP           = 4      -- Trains/hour at which tourist bonus is maximised
```

---

## Contributing

Issues and pull requests are welcome. If you're reporting a bug, please include:

- A description of the issue and steps to reproduce it
- Your TF2 version number
- A list of other active mods
- Any relevant console output

---

## Licence

MIT Licence. See `LICENSE` for details.
