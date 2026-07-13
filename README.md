# OneBlock Reborn

OneBlock Reborn is a Forge OneBlock mod built for modpacks. It provides a native OneBlock progression system, built-in phases, custom datapack phases, custom trader pools, saved Origin Blocks, and pack-developer commands for testing phase balance.

## Quick Start

Place an Origin Block, break the generated block, and the OneBlock will keep generating new entries from the current phase.

The current phase is based on the number of blocks mined. Pack authors can add or replace phase behavior with datapack JSON files.

Custom phase files go here:

```text
data/<namespace>/oneblockreborn/phases/<file>.json
```

Custom trader pool files go here:

```text
data/<namespace>/oneblockreborn/trader_pools/<pool>.json
```

## Commands

Anyone can run:

```mcfunction
/oneblockreborn help
```

Pack-developer commands require OP permission level 2:

```mcfunction
/oneblockreborn dev phases
/oneblockreborn dev phase validate
/oneblockreborn dev phase preview <startsAt>
/oneblockreborn dev phase sample <startsAt> <rolls>
/oneblockreborn dev phase exportdocs
/oneblockreborn dev spawnmob <entity>
/oneblockreborn dev held mined <count>
/oneblockreborn dev held phase <startsAt>
/oneblockreborn dev removeall <maxMined> <createChest>
/oneblockreborn dev datapack
```

Useful examples:

```mcfunction
/oneblockreborn dev phases
/oneblockreborn dev phase validate
/oneblockreborn dev phase preview 750
/oneblockreborn dev phase sample 750 1000
/oneblockreborn dev spawnmob minecraft:cow
/oneblockreborn dev held mined 2500
/oneblockreborn dev held phase 1500
```

## Built-In Phase Timeline

| Mined blocks | Phase |
| ---: | --- |
| 0 | Progress |
| 30 | Step 1 |
| 150 | Step 2 |
| 300 | Step 3 |
| 750 | Step 4 |
| 1500 | Step 5 |
| 2000 | Step 6 |
| 2500 | Step 7 |
| 3500 | Step 8 |
| 5000 | Step 9 |

Use `/oneblockreborn dev phases` in-game to see the loaded timeline, including custom phases.

## Custom Phase JSON

A phase JSON file defines when the phase starts, what it is called, and what can generate from the OneBlock.

Basic example:

```json
{
  "starts_at": 6000,
  "name": "Crystal Caverns",
  "entries": [
    { "type": "block", "id": "minecraft:stone", "weight": 20 },
    { "type": "block", "id": "minecraft:amethyst_block", "weight": 4 },
    { "type": "chest", "loot_table": "minecraft:chests/simple_dungeon", "weight": 1 },
    { "type": "mob", "id": "minecraft:bat", "weight": 2 }
  ]
}
```

### Phase Fields

| Field | Required | Description |
| --- | --- | --- |
| `starts_at` | Yes | Mined-block count where this phase begins. |
| `name` | No | Display name. If omitted, the file name is converted into a readable name. |
| `entries` | Yes | Weighted block, chest, and mob entries. |
| `guaranteed` | No | Exact mined-count entries that override the random roll. |
| `stage_actions` | No | Optional actions when this phase is first reached. |

If a custom phase uses the same `starts_at` as a built-in phase, it replaces that phase at that start point.

## Entry Types

### Block Entry

```json
{ "type": "block", "id": "minecraft:grass_block", "weight": 10 }
```

The `id` must be a valid block ID. If the block is missing at runtime, validation will report it.

### Chest Entry

```json
{ "type": "chest", "loot_table": "minecraft:chests/simple_dungeon", "weight": 1 }
```

This places a chest and assigns the loot table.

### Mob Entry

```json
{ "type": "mob", "id": "minecraft:cow", "weight": 2 }
```

Mob entries spawn the mob above the OneBlock, tag it as a OneBlock phase spawn, optionally silence it through config, record it for mob crate release, and grant matching mob advancements when available.

## Mob Count, NBT, And Conditions

Mob entries can include a count range, NBT, and conditions.

```json
{
  "type": "mob",
  "id": "minecraft:zombie",
  "weight": 3,
  "count": [1, 3],
  "nbt": "{CustomName:'\"Sky Zombie\"',CanPickUpLoot:1b}",
  "conditions": {
    "dimension": "minecraft:overworld",
    "biome": "minecraft:plains",
    "min_mined": 6000,
    "max_mined": 7000,
    "time": "night",
    "weather": "clear"
  }
}
```

`count` can be either a single number or a two-number range:

```json
"count": 2
```

```json
"count": [1, 4]
```

Supported conditions:

| Condition | Values |
| --- | --- |
| `dimension` | Resource location, such as `minecraft:overworld` or `minecraft:the_nether`. |
| `biome` | Resource location, such as `minecraft:plains`. |
| `min_mined` | Minimum mined-block count. |
| `max_mined` | Maximum mined-block count. |
| `time` | `day` or `night`. |
| `weather` | `clear`, `raining`, or `thundering`. |

If an entry's conditions do not match, it is skipped for that roll. If no entries match, the OneBlock falls back to stone.

## Guaranteed Entries

Guaranteed entries trigger at exact mined-block counts and override the normal random roll.

```json
{
  "starts_at": 6000,
  "name": "Crystal Caverns",
  "entries": [
    { "type": "block", "id": "minecraft:stone", "weight": 20 }
  ],
  "guaranteed": [
    { "at": 6000, "type": "block", "id": "minecraft:amethyst_block", "weight": 1 },
    { "at": 6050, "type": "mob", "id": "minecraft:allay", "weight": 1 }
  ]
}
```

## Stage Actions

Stage actions run when a phase is first reached.

```json
{
  "starts_at": 6000,
  "name": "Crystal Caverns",
  "entries": [
    { "type": "block", "id": "minecraft:calcite", "weight": 10 }
  ],
  "stage_actions": {
    "world_border_size": 750,
    "spawn_stage_trader": true,
    "build_end_portal": false,
    "message": "Reached %Level% at %X% %Y% %Z%"
  }
}
```

Supported stage actions:

| Field | Description |
| --- | --- |
| `world_border_size` | Sets world border size if native world border config is enabled. |
| `spawn_stage_trader` | Spawns a stage trader if native stage traders are enabled. |
| `build_end_portal` | Builds the configured compact end portal if native end portal handling is enabled. |
| `message` | Sends a message to nearby players. Supports `%X%`, `%Y%`, `%Z%`, and `%Level%`. |

## How Weights Work

Each entry has a `weight`. Higher weight means more common.

Example:

```json
[
  { "type": "block", "id": "minecraft:dirt", "weight": 9 },
  { "type": "block", "id": "minecraft:diamond_ore", "weight": 1 }
]
```

Total weight is 10. Dirt has a 90% chance and diamond ore has a 10% chance.

Use this command to inspect the real loaded percentages:

```mcfunction
/oneblockreborn dev phase preview <startsAt>
```

Use this command to simulate the phase:

```mcfunction
/oneblockreborn dev phase sample <startsAt> <rolls>
```

## Mob Spawning Behavior

When a mob entry is selected:

1. The entity is created from its resource ID.
2. Optional NBT is applied.
3. The mob is moved to the block above the OneBlock.
4. The mob gets the tag `oneblockreborn.phase_spawn`.
5. The mob is silenced if `muteSpawnedMobs` is enabled.
6. The mob is recorded for visiting trader mob crate release.
7. Mob achievements are granted if enabled and an advancement exists.
8. If the OneBlock position is empty after the mob spawn, it is filled with grass.

The native mob replacement system only affects mobs tagged with `oneblockreborn.phase_spawn`.

## Built-In Mobs

| Mined blocks | Built-in mobs added |
| ---: | --- |
| 0 | None |
| 30 | None |
| 150 | None |
| 300 | Chicken, pig |
| 750 | Chicken, pig, cow, sheep |
| 1500 | Chicken, pig, cow, sheep, wolf, cat |
| 2000 | None |
| 2500 | Chicken, pig, cow, sheep, wolf, cat, llama, goat, horse, donkey, fox |
| 3500 | Chicken, pig, cow, sheep, wolf, cat, llama, goat, horse, donkey, fox |
| 5000 | Chicken, pig, cow, sheep, wolf, cat, llama, goat, horse, donkey, fox |

Custom phases can add any valid entity ID from Minecraft or installed mods.

## Trader Pools

Trader pools let packs override stage and visiting trader offers.

Path:

```text
data/<namespace>/oneblockreborn/trader_pools/<pool>.json
```

Common pool names:

```text
visiting
stage_150
stage_300
stage_750
stage_1500
stage_2000
stage_2500
stage_3500
stage_5000
```

Use matching names for custom stages, such as `stage_6000`.

## Pack-Dev Testing Workflow

1. Add or edit custom phase JSON in your datapack.
2. Reload datapacks.
3. Run:

```mcfunction
/oneblockreborn dev phase validate
```

4. Preview the target phase:

```mcfunction
/oneblockreborn dev phase preview 6000
```

5. Simulate balance:

```mcfunction
/oneblockreborn dev phase sample 6000 1000
```

6. Test specific mobs:

```mcfunction
/oneblockreborn dev spawnmob minecraft:zombie
```

7. Set a held Origin Block to the target progress:

```mcfunction
/oneblockreborn dev held mined 6000
```

8. Place the Origin Block and play-test the phase.

## Full Custom Phase Example

```json
{
  "starts_at": 6000,
  "name": "Crystal Caverns",
  "entries": [
    { "type": "block", "id": "minecraft:stone", "weight": 20 },
    { "type": "block", "id": "minecraft:calcite", "weight": 8 },
    { "type": "block", "id": "minecraft:amethyst_block", "weight": 4 },
    { "type": "chest", "loot_table": "minecraft:chests/simple_dungeon", "weight": 1 },
    {
      "type": "mob",
      "id": "minecraft:bat",
      "weight": 2,
      "count": [1, 2],
      "conditions": {
        "time": "night"
      }
    },
    {
      "type": "mob",
      "id": "minecraft:zombie",
      "weight": 1,
      "nbt": "{CustomName:'\"Crystal Guard\"',CanPickUpLoot:1b}",
      "conditions": {
        "min_mined": 6100,
        "weather": "clear"
      }
    }
  ],
  "guaranteed": [
    { "at": 6000, "type": "block", "id": "minecraft:amethyst_block", "weight": 1 },
    { "at": 6250, "type": "mob", "id": "minecraft:allay", "weight": 1 }
  ],
  "stage_actions": {
    "world_border_size": 750,
    "spawn_stage_trader": true,
    "message": "Reached %Level%!"
  }
}
```

## Troubleshooting

Run validation first:

```mcfunction
/oneblockreborn dev phase validate
```

Common issues:

| Problem | What to check |
| --- | --- |
| Phase does not appear | File path must be `data/<namespace>/oneblockreborn/phases/<file>.json`. Reload datapacks after editing. |
| Entry never appears | Check weight, conditions, and whether a guaranteed entry is overriding the roll. |
| Mob does not spawn | Check entity ID with `/oneblockreborn dev spawnmob <entity>`. |
| Custom NBT fails | Make sure NBT is valid Mojangson inside a JSON string. Escape quotes as needed. |
| Chances feel wrong | Run `/oneblockreborn dev phase preview <startsAt>` and `/oneblockreborn dev phase sample <startsAt> <rolls>`. |
| Trader has default trades | Make sure the trader pool file name matches the stage, such as `stage_6000.json`. |

## Config Notes

Several systems can be enabled or disabled in the common config, including:

- native stage actions
- native stage traders
- native world border
- native end portal handling
- visiting traders
- expanded mob crates
- native mob achievements
- bossbar progress display
- mob replacement
- trial system

Pack authors should test custom phases with the same config values their modpack will ship.
