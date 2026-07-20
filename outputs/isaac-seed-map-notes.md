# Isaac Seed Map Prototype Notes

## What exists now

- `isaac-seed-map-prototype.html` is a standalone browser tool.
- It accepts a seed string, floor/stage, platform, version/runtime, and source format.
- It deterministically builds and renders a dungeon-style map from that input.
- The current generator is not claimed to match Nintendo Switch Afterbirth+ exactly.

## Where the real Switch AB+ work plugs in

- `normalizeSeed(value)` handles seed text cleanup.
- `fnv1a(text)` currently converts seed text into a numeric seed.
- `makeRng(seedNumber)` is the placeholder RNG.
- `floorDefinitions` contains named floors, including normal and Repentance alternate paths.
- `generationProfiles` contains version-specific generation constants. The UI derives this internal key from platform + version/runtime.
- `buildFloor(seedText, floorNumber, profileName)` is where the floor layout algorithm lives.

## Updating incorrect maps

Do not overwrite the generator just because one map looks wrong. Save the observation first, then update the algorithm after comparing several records.

Workflow:

1. Set the seed, platform, version/runtime, source format, and floor.
2. Compare the generated map with the real in-game minimap or a trusted screenshot.
3. In the correction panel, choose `Correct`, `Incorrect`, or `Partial match`.
4. Add notes like actual room count, boss direction, treasure direction, shop direction, secret room guess, and source.
5. Save the observation.
6. Export the observation JSON when ready to analyze multiple records.

Correction records are evidence. Code changes should happen later in `floorDefinitions`, `generationProfiles`, `makeRng()`, or `buildFloor()` depending on what the records show.

Examples:

- If only Downpour room counts are wrong, adjust `floorFamilyTuning.waterAlt`.
- If every Switch AB+ floor is wrong for known matching seeds, investigate `makeRng()` and seed decoding.
- If one platform/version differs from another, add or tune a separate profile.

## Data needed to fit the real generator

Collect rows like this:

```text
version, seed, floor, room_count, boss_direction, treasure_direction, shop_direction, visible_grid
Switch AB+, ABCD EFGH, Basement I, 8, east, north, west, ...
```

Good observations include:

- The exact displayed seed.
- Platform, version, release channel, and patch level.
- Game version and whether Repentance is installed.
- Physical cartridge, digital release, virtual cart/digital entitlement, and DLC state when testing Switch, if known.
- Character and mode, if relevant.
- Tainted character variants, especially when shops, pickups, item access, or economy could affect interpretation.
- Current unlock state or item-pool notes, especially if comparing item rooms, shops, boss rewards, angel/devil rooms, or character-dependent routes.
- Full minimap screenshots per floor.
- Any visible special rooms.
- Whether the run was continued from a save.

Start-room adjacency is especially valuable. Record it as directional constraints:

```json
{
  "startRoom": {
    "east": { "exists": true, "type": "curse" },
    "south": { "exists": true, "type": "library" },
    "west": { "exists": false },
    "north": { "exists": true, "type": "unknown" }
  }
}
```

These constraints are useful because a candidate seed algorithm can be rejected quickly if it places the wrong room type or no room in a known direction.

The prototype includes a start-room correction helper with dropdowns for north/east/south/west exits. Use it for fast reroll data:

- `Unknown` means do not include that direction in the JSON.
- `Room` records an exit from start.
- `No room` records a known missing exit.
- Type and shape are optional; leave them unknown unless obvious.

`Save Observation` stores these constraints in browser localStorage and includes them in exported/copied JSON. The website does not write directly back to GitHub or the shared observations file.

Screenshots are useful even before exact tracing. Save them as screenshot evidence with floor label, time, score, curse state, visible special-room icons, and any caption text. Only promote screenshot details into hard coordinate constraints after manually tracing the minimap.

If a floor is affected by Amnesia, Curse of the Lost, or another map-visibility effect, keep the screenshot but mark it as lower-confidence map evidence. The floor label, time, score, and visible special-room icons can still be useful.

## Unlocks and item pools

Unlock state should be tracked, but it should not be mixed up with the room-layout problem too early.

Separate the evidence into two lanes:

- Layout lane: seed, floor, start room, room graph, special-room positions.
- Pool lane: character, unlock state, item-room item, shop stock, boss reward, angel/devil items, trinkets, pickups.

This matters because a seed mapper might predict the floor graph correctly while item results still differ because an item is locked, a character changes access, or a pool has been altered by progression. Use `isaac-player-state.json` as the current unlock/environment scratchpad, and use observation-level `unlockProfile` / `unlockNotes` when a run has special context.

With enough matching seed/map pairs, the placeholder RNG and layout rules can be replaced piece by piece and tested against real layouts.

## Compatibility caution

Do not assume a seed is universal. A seed may resolve differently across:

- PC, Nintendo Switch, PlayStation, Xbox, and iOS ports.
- Rebirth, Afterbirth, Afterbirth+, Repentance, and Repentance+ where that release actually exists for the platform being tested.
- Patch releases within the same expansion.
- Console physical, digital, and virtual-cart style entitlement builds, if their packaged versions differ.
- Daily/challenge/special-seed modes versus ordinary seeded runs.
- Normal path versus alternate path floors, especially Downpour/Dross, Mines/Ashpit, Mausoleum/Gehenna, Corpse, and Home.
- Purchased entitlement versus the effective runtime that is actually active.

The safest internal model is:

```text
seed + platform + version + release/patch + mode -> observed result
```

The prototype keeps this metadata on imported seeds and warns when a seed reported for one environment is tested against another derived generation bucket.

The prototype also warns when a selected floor requires a release that the current platform/runtime does not support, for example using Downpour with an Afterbirth+ runtime.

## Entitlement vs runtime

For map generation, the observed runtime matters more than the store entitlement. If a Switch install has Repentance floors, items, bosses, or menus active even though only Afterbirth+ was purchased, record it as:

```text
platform: Nintendo Switch
entitlement: Afterbirth+
effective runtime: Repentance
```

Use platform `Nintendo Switch`, version/runtime `Afterbirth+ purchase, Repentance active`, and the appropriate source format for that case. The prototype derives the internal `AB+ Switch purchase, Repentance active` bucket from those fields. This lets observations preserve the weird purchase state while still allowing Repentance-only floors like Downpour, Dross, Mines, Ashpit, Mausoleum, Gehenna, Corpse, and Home.

## Switch version naming

For original Nintendo Switch observations, use `Repentance` as the effective runtime when Repentance content is visible. Do not label original Switch observations as `Repentance+` unless the game itself clearly reports Repentance+ or Repentance+ content is confirmed on that exact platform/build.

For Nintendo Switch 2 observations, use platform `Nintendo Switch 2` and version/runtime `Repentance` when the game appears to be running regular Repentance. Do not assume Switch 2 means Repentance+.

Keep `Repentance+` as a separate source label for PC or Switch 2 seed reports. Those seeds should be treated as cross-version data until proven compatible.

## Online seed sources

Useful public sources are mostly forum-style text, not clean APIs:

- Steam Community discussions for The Binding of Isaac: Rebirth.
- Reddit posts and comments from Isaac communities.
- Wiki pages that list special seeds or challenge-style seeds.
- Personal seed lists, screenshots, and old forum archives.

Steam and Reddit usually should not be fetched directly from a standalone browser file because cross-origin rules and bot protection can block it. The current prototype instead has a forum-paste importer: copy text from a source, paste it into the tool, and extract seed-shaped values.

A later local fetcher can collect source pages into a JSON file with fields like:

```json
[
  {
    "seed": "ABCD EFGH",
    "source": "Steam Community",
    "url": "https://steamcommunity.com/app/250900/discussions/",
    "notes": "User-reported strong item start",
    "platform": "PC",
    "version": "Afterbirth+",
    "release": "unknown patch"
  }
]
```
