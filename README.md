# GTA IV Mods

A collection of GTA IV gameplay and vehicle utilities powered by ScriptHookDotNet.

This README documents `GTAIV_CombinedScripts_ConvoyCrashFix.cs`, which combines Any Taxi, Bodyguards, Convoy, Vehicle Browser, Vehicle Controls, and Vehicle Editor in one script.

<a href="https://www.youtube.com/watch?v=6ZeMM30cjbk&t=450s">
  <img src="https://img.youtube.com/vi/6ZeMM30cjbk/maxresdefault.jpg" alt="GTA IV Mods gameplay video" width="800">
</a>

## Installation

1. [Download GTA IV Mode Files](https://github.com/IAVARABBASOV/gta4-modes/releases/download/gta4mode/GTA.IV.Mode.Files.7z).
2. Extract the `.7z` archive using 7-Zip or another compatible archive manager.
3. Copy the extracted files into your GTA IV game directory, alongside `GTAIV.exe`. Back up existing files before replacing them.
4. For the version described here, place `GTAIV_CombinedScripts_ConvoyCrashFix.cs` in the game's `scripts` folder. Ensure that the required GTA IV ScriptHookDotNet runtime and its dependencies are installed.
5. Remove or move older copies of these same scripts out of the `scripts` folder to avoid duplicate features and key handlers.
6. Launch GTA IV.

### Requirements

- GTA IV The Complete Edition (the target game edition for this mod pack).
- A working GTA IV ScriptHookDotNet installation compatible with your game build.
- 7-Zip or another `.7z` archive extractor.

Console commands below are entered in the ScriptHookDotNet in-game console. Its opening key depends on your runtime configuration.

## Keyboard Controls

| Key | Action |
| --- | --- |
| **N** | Start Any Taxi; cancel while waiting to enter; update the destination during or after a ride |
| **M** | Toggle 4x time acceleration while the taxi is driving |
| **Insert** | Spawn a bodyguard using the last selected model (maximum 20 tracked bodyguards) |
| **Delete** | Remove one bodyguard, starting with the oldest tracked one |
| **F1** | Toggle convoy passenger shooting mode |
| **F6** | Open or close Vehicle Browser |
| **F7** | Open or close Vehicle Controls |
| **F8** | Add one convoy vehicle (maximum 3) |
| **F9** | Remove the last convoy vehicle and its crew |
| **F12** | Unlock all islands |
| **Z** | Toggle indicator lights on your current vehicle |
| **X** | Toggle hazard lights on your current vehicle |
| **R** | Repair and wash your current vehicle and all active convoy vehicles |

**R** is handled by both the convoy and vehicle utility scripts. The vehicle utility handler also sets maximum health and armor to 100 and adds $1,000 when your balance is below $1,000,000, even when you are on foot. It does not explicitly refill current health or armor.

## Any Taxi

Use a nearby vehicle as a passenger ride to your map waypoint.

1. Set a waypoint on the map.
2. Stand within 5 meters of a vehicle and press **N**. The script uses its NPC driver or creates one if the driver's seat is empty. It skips a vehicle you are driving yourself.
3. Your character enters a passenger seat; nearby group members may enter too.
4. Press **M** during the ride to switch between normal speed and 4x time acceleration.
5. To change the route, move your waypoint and press **N** again. After arrival, **N** starts another trip to the selected waypoint.

Pressing **N** while waiting to enter cancels the ride. Leaving the vehicle during or after the trip ends taxi mode. Walking more than 8 meters away while waiting also cancels it.

The ride includes temporary collision protection and driver protection. Time acceleration resets on arrival or cleanup. If no waypoint is selected when you board, select one and press **N** to begin driving.

## Bodyguards

Spawn armed companions that join your player group. Each receives your currently equipped weapon and an MP5, with 30,000 rounds for each assigned weapon.

- **Insert** repeats the last selected model; the initial model is `F_Y_STRIPPERC01`.
- **Delete** removes one bodyguard per press, starting with the oldest tracked one.
- The script allows up to **20 tracked bodyguards**. Actual group behavior and vehicle seating remain subject to the game's limits.

### Console Commands

| Command | Action |
| --- | --- |
| `bodyguards [page] [search]` | List available ped models, 10 per page, optionally filtered by name |
| `bodyguard` | Spawn the currently selected model |
| `bodyguard <number or PedModel>` | Select a model and spawn a bodyguard |

Examples:

```text
bodyguards
bodyguards 2
bodyguards 1 stripper
bodyguard F_Y_STRIPPERC01
bodyguard 1
```

Model numbers refer to the full sorted model list, including when the displayed results are filtered. The list is read from the game's `common/data/peds.ide` and the TLAD/TBoGT equivalents when present; only models validated by the runtime are included.

## Convoy

Build an escort of up to **3 vehicles**, each with **4 crew members**: one driver and three passengers. Use vehicles that support all four required seats; crew creation fails if the selected vehicle cannot accommodate them.

- **F8** adds a vehicle using the remembered vehicle and crew models. Initial defaults are `SULTAN` and `F_Y_STRIPPERC02`.
- **F9** removes only the most recently added vehicle and its crew. Repeat to remove the entire convoy.
- **F1** toggles passenger shooting mode while the driver continues following. New vehicles inherit the current shooting mode.
- **R** repairs and washes every active convoy vehicle.
- **F12** unlocks all islands, even without an active convoy.

### Console Commands

| Command | Action |
| --- | --- |
| `convoy [VehicleModel] [PedModel or number]` | Add one vehicle, optionally selecting its vehicle and crew models |
| `convoypeds [page] [search]` | List available crew models, 10 per page |
| `convoyped` | Display the currently selected crew model |
| `convoyped <PedModel or number>` | Select the crew model for future convoy vehicles |
| `convoyend` | Remove the last convoy vehicle and its crew |

Examples:

```text
convoy
convoy SULTAN
convoypeds 1 stripper
convoyped F_Y_STRIPPERC02
convoy SULTAN F_Y_STRIPPERC02
convoyend
```

The vehicle model supplied to `convoy` is remembered for subsequent **F8** spawns. Crew selection affects future vehicles; existing crews keep their models. Crew model numbers use the same full sorted ped list as bodyguards.

### Following and Stability

This version follows the player's live vehicle or character, with speed-dependent formation spacing, gradual acceleration, approach braking, and spacing checks against the player and other convoy vehicles. It also recovers stalled or stale driving tasks and accounts for different road heights when spawning and following.

Catch-up commands have a ceiling of **300 km/h**. This is a controller limit; actual speed depends on the vehicle, traffic, road layout, and game AI.

Crew members spawn directly into their seats and remain outside the player group. Invalid vehicles and drivers are cleaned up, and several entity-access paths handle objects disappearing during gameplay.

### Diagnostics

The script automatically writes `ConvoyDiagnostics.csv` beside the script, with at most one row per convoy vehicle per second. It records spacing, actual and commanded speeds, braking limits, task changes, and vehicle state. When the file exceeds 2 MiB, it is copied to `ConvoyDiagnostics.csv.previous` and a new log begins. Logging stops if the script cannot write the file.

## Vehicle Browser

Press **F6** to browse vehicle categories and spawn a selected model.

The menu supports:

- Spawning a vehicle and placing you in its driver's seat.
- Replacing your current vehicle; this deletes the old vehicle.
- Random primary and specular colors.
- Damage-proof spawning through the **Spawn Invincible** option.

### Console Commands

| Command | Action |
| --- | --- |
| `vb` | Open Vehicle Browser |
| `sv <VehicleModel>` | Spawn a vehicle by model name |
| `lv [Category]` | List vehicles, optionally filtering category names |

Examples:

```text
vb
sv SULTAN
lv
```

The `sv` command uses default spawn options without random colors or invincibility. Use the menu to enable those options.

## Vehicle Controls

Press **F7** to open or close the utility window while using a vehicle.

- Toggle the hood and trunk.
- Open or close each of the four passenger doors.
- Toggle indicator and hazard lights.
- Use **Z**, **X**, and **R** for the keyboard shortcuts listed above.

Independent left/right turn signals are not implemented in this version: the **LEFT** button only displays a message, while **RIGHT** toggles the vehicle's shared indicator-light property. **HAZARD** toggles the hazard-light property. The window and keyboard shortcuts maintain separate toggle states, so mixing them may require an extra press to reach the intended light state.

## Vehicle Editor

Sit in a vehicle and enter:

```text
EditCar
```

The editor lets you change primary, specular, and two feature colors; lock or unlock doors; and open or close individual passenger doors, the hood, and the trunk.

## Video

Click the preview above to watch the gameplay demonstration. This README describes the supplied combined script; the video may show an earlier version.
