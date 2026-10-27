# EggHatcher

A small, server-side egg-hatching module for Roblox, written in Luau.

EggHatcher provides weighted pet drops, pity rules, configuration validation, and save/load helpers. All rolls happen on the server, so clients cannot choose or modify the result.

> **Status:** Early project / starter module. The logic has been checked with a 200,000-roll simulation and 39 automated checks.

## Features

- **Weighted rolls:** Assign a relative weight to each pet; EggHatcher calculates the odds.
- **Pity rules:** Guarantee a pet of a minimum rank after a set number of hatches. Multiple rules can apply to one egg.
- **Server-side results:** The server performs the roll and returns the result to the requesting server code.
- **Configuration validation:** Invalid weights, duplicate pet IDs, and impossible pity rules produce clear errors.
- **Odds for UI:** `GetOdds` returns the actual percentage chance for each pet.
- **Pity save/load helpers:** Get and restore per-player pity counters for your own DataStore code.
- **Strict Luau types:** Uses `--!strict` and has no dependencies.

## Installation

### Wally

Add EggHatcher to your `wally.toml` server dependencies. Replace `<username>` with the GitHub or Wally package owner after publishing the package:

```toml
[server-dependencies]
EggHatcher = "<username>/egghatcher@0.1.0"
```

### Rojo or manual installation

Copy `src/init.luau` into a ModuleScript named `EggHatcher`, then place it somewhere server code can require it, such as `ServerScriptService`.

## Quick start

```lua
local EggHatcher = require(path.to.EggHatcher)

local hatcher = EggHatcher.new()

local basicEgg: EggHatcher.EggConfig = {
    id = "BasicEgg",
    pets = {
        { id = "Dog",    weight = 60, rank = 1 }, -- common
        { id = "Cat",    weight = 30, rank = 1 }, -- common
        { id = "Fox",    weight = 9,  rank = 2 }, -- rare
        { id = "Dragon", weight = 1,  rank = 3 }, -- legendary
    },
    pity = {
        { minRank = 2, every = 10 },  -- rare or better at least every 10 hatches
        { minRank = 3, every = 100 }, -- legendary at least every 100 hatches
    },
}

local pet = hatcher:Hatch(player, basicEgg)
print(pet.id) -- "Dog", "Fox", ...
```

For a complete server example with a `RemoteFunction`, input validation, a cooldown, and a currency check, see [`examples/ServerExample.server.luau`](examples/ServerExample.server.luau).

## API

| Function | Description |
| --- | --- |
| `EggHatcher.new(seed?)` | Creates a hatcher. Pass a seed only for reproducible tests. |
| `hatcher:Hatch(player, egg)` | Hatches one egg and returns a `Pet`. |
| `hatcher:HatchMany(player, egg, count)` | Hatches 1 to 100 eggs and returns an array of `Pet`s. |
| `EggHatcher.GetOdds(egg)` | Returns `{ [petId]: percent }` for use in your UI. |
| `EggHatcher.Validate(egg)` | Errors if the configuration is invalid. Called automatically on the first hatch. |
| `hatcher:GetPityState(player)` | Returns a copy of the player's pity counters, ready to save. |
| `hatcher:LoadPityState(player, data)` | Restores saved counters; invalid entries are ignored. |
| `hatcher:ClearPlayer(player)` | Frees the player's data. Call this when the player leaves. |

## Types

```lua
type Pet = { id: string, weight: number, rank: number }
type PityRule = { minRank: number, every: number }
type EggConfig = { id: string, pets: { Pet }, pity: { PityRule }? }
```

`weight` is a relative chance. Weights `60`, `30`, `9`, and `1` produce chances of 60%, 30%, 9%, and 1%, respectively, but any positive weights work.

`rank` describes rarity: a higher rank means a rarer pet.

`every = N` means the Nth hatch is guaranteed at the latest. `every` must be an integer of at least 2.

## How pity works

Each pity rule keeps a counter for each player and egg. A hatch that does not reach the rule's `minRank` increases its counter. When the counter reaches `every - 1`, the next hatch is rolled only among pets with `minRank` or higher, still using their relative weights, and the counter resets.

A pet of a higher rank also resets lower-rank pity rules. If multiple rules trigger on the same hatch, the rule with the highest `minRank` takes priority.

## Security notes

- Call `Hatch` only from server code, and charge the player before hatching.
- Never accept an egg configuration, pet ID, or odds from the client. The client should send only the egg ID and requested count; the server should look up the egg in its own trusted table.
- Rate-limit your remote. The example shows one way to do this.
- EggHatcher does not handle currency or DataStores. It exposes pity counters so your game can save them.

## Saving pity in a DataStore

```lua
-- After the player's data has loaded:
hatcher:LoadPityState(player, data.pity)

-- When saving (for example, during autosave and PlayerRemoving):
data.pity = hatcher:GetPityState(player)
```

## Contributing

Issues and pull requests are welcome. Possible future additions include soft pity and luck multipliers.

## License

MIT. See [`LICENSE`](LICENSE).
