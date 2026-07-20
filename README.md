# Isaac Seed Map Prototype

An early reverse-engineering helper for Binding of Isaac seed/map behavior, focused on Nintendo Switch 2 Repentance observations.

## Current State

This is not a cracked or verified Isaac map generator yet. The HTML page currently uses a deterministic placeholder generator so we can compare seeds, floors, profiles, and observed screenshots in one place while collecting data.

The useful source of truth is the observation data:

- `outputs/isaac-seed-observations.json` - run and floor evidence captured from screenshots and notes.
- `outputs/isaac-player-state.json` - current save/unlock evidence that may affect item pools.
- `outputs/isaac-seed-map-prototype.html` - static prototype with seed import, manual corrections, and observation matching.
- `outputs/isaac-seed-map-notes.md` - working notes and capture workflow.

## Local Use

Open:

```text
outputs/isaac-seed-map-prototype.html
```

The page may auto-load `isaac-seed-observations.json` when browser rules allow it. If not, use the matcher file picker or paste the JSON into the page.

## Data Strategy

Keep layout evidence separate from item/unlock evidence:

- Layout lane: seed, floor, room graph, start-room adjacency, special-room positions.
- Pool lane: character, unlock state, item room/shop/boss/angel/devil results.

Screenshot-only observations should be treated as evidence until their minimaps are traced into coordinates.
