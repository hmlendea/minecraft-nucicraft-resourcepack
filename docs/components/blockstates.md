# Blockstates Component

Deep dive into blockstate files: structure, patterns, and NuciCraft conventions.

## Overview

Blockstate files map block states (properties) to models. Located at `assets/minecraft/blockstates/*.json`.

Two formats:
- **Variants**: Direct state → model mapping
- **Multipart**: Conditional model composition

## File Structure

```
assets/minecraft/blockstates/
├── acacia_fence.json
├── acacia_fence_gate.json
├── cobblestone.json
├── oak_door.json
├── ...
└── (100+ files)
```

## Variants Format

### Simple Block (No Properties)

```json
{
  "variants": {
    "": { "model": "minecraft:block/cobblestone" }
  }
}
```

### Block with Properties

```json
{
  "variants": {
    "facing=north": { "model": "minecraft:block/furnace", "y": 180 },
    "facing=south": { "model": "minecraft:block/furnace" },
    "facing=east":  { "model": "minecraft:block/furnace", "y": 90 },
    "facing=west":  { "model": "minecraft:block/furnace", "y": 270 }
  }
}
```

### Weighted Variants (Texture Variations)

```json
{
  "variants": {
    "": [
      { "model": "minecraft:block/cobblestone", "weight": 3 },
      { "model": "minecraft:block/cobblestone_variated_1", "weight": 1 },
      { "model": "minecraft:block/cobblestone_variated_2", "weight": 1 }
    ]
  }
}
```

**Weight Algorithm**: Random selection proportional to weight. Total weight = 5, so base = 60%, each variation = 20%.

### Complex State Combinations (Door)

```json
{
  "variants": {
    "facing=north,half=lower,hinge=left,open=false,powered=false": { "model": "minecraft:block/oak_door_bottom_left" },
    "facing=north,half=lower,hinge=left,open=true,powered=false": { "model": "minecraft:block/oak_door_bottom_left_open" },
    "facing=north,half=upper,hinge=left,open=false,powered=false": { "model": "minecraft:block/oak_door_top_left" },
    "facing=north,half=upper,hinge=left,open=true,powered=false": { "model": "minecraft:block/oak_door_top_left_open" }
    // ... 32 combinations for door
  }
}
```

## Multipart Format

Used for blocks that compose from parts (fences, walls, pipes).

### Fence Example

```json
{
  "multipart": [
    {
      "apply": { "model": "minecraft:block/oak_fence_post" }
    },
    {
      "when": { "north": "true" },
      "apply": { "model": "minecraft:block/oak_fence_side", "y": 90 }
    },
    {
      "when": { "east": "true" },
      "apply": { "model": "minecraft:block/oak_fence_side", "y": 0 }
    },
    {
      "when": { "south": "true" },
      "apply": { "model": "minecraft:block/oak_fence_side", "y": 270 }
    },
    {
      "when": { "west": "true" },
      "apply": { "model": "minecraft:block/oak_fence_side", "y": 180 }
    }
  ]
}
```

### Wall Example

```json
{
  "multipart": [
    { "apply": { "model": "minecraft:block/cobblestone_wall_post" } },
    { "when": { "up": "true" }, "apply": { "model": "minecraft:block/cobblestone_wall_post" } },
    { "when": { "north": "true" }, "apply": { "model": "minecraft:block/cobblestone_wall_side", "y": 90 } },
    { "when": { "east": "true" }, "apply": { "model": "minecraft:block/cobblestone_wall_side", "y": 0 } },
    { "when": { "south": "true" }, "apply": { "model": "minecraft:block/cobblestone_wall_side", "y": 270 } },
    { "when": { "west": "true" }, "apply": { "model": "minecraft:block/cobblestone_wall_side", "y": 180 } },
    { "when": { "north": "true", "up": "true" }, "apply": { "model": "minecraft:block/cobblestone_wall_side_tall", "y": 90 } }
    // ... tall variants for each direction
  ]
}
```

## NuciCraft Patterns

### Pattern 1: Texture Variations via Weighted Variants

Used for: cobblestone, stone, ores, bricks, logs, terracotta, etc.

```json
{
  "variants": {
    "": [
      { "model": "minecraft:block/<block>", "weight": 3 },
      { "model": "minecraft:block/<block>_variated_1", "weight": 1 },
      { "model": "minecraft:block/<block>_variated_2", "weight": 1 }
    ]
  }
}
```

**Variation Models** in `models/block/variated/` or `models/block/<block>/`:
- Reference variation textures in `textures/block/variated/`
- Same geometry, different texture

### Pattern 2: 3D Models for Complex Blocks

Used for: fences, gates, doors, trapdoors, chains, lanterns, candles, crops, flowers, grass, redstone, buckets, etc.

Blockstate references 3D model directly (no variation).

### Pattern 3: Multipart for Connecting Blocks

Used for: fences, walls, glass panes, iron bars, chains.

### Pattern 4: State-Dependent for Directional/Animated

Used for: doors, trapdoors, fence gates, pistons, observers, dispensers, droppers.

## Property Reference

### Common Properties

| Property | Values | Blocks |
|----------|--------|--------|
| `facing` | north, south, east, west, up, down | Furnace, dispenser, door, pumpkin |
| `half` | upper, lower | Door, bed, tall grass, sunflower |
| `hinge` | left, right | Door |
| `open` | true, false | Door, trapdoor, fence gate |
| `powered` | true, false | Door, trapdoor, piston |
| `north/east/south/west` | true, false | Fence, wall, glass pane, iron bars |
| `up` | true, false | Wall, fence |
| `age` | 0-7 | Crops |
| `waterlogged` | true, false | Many (1.13+) |

### Property Order

In variants format, properties must be **alphabetical**:
- `facing=north,half=lower,hinge=left,open=false` ✓
- `half=lower,facing=north,open=false,hinge=left` ✗

## Model Reference Syntax

```json
{ "model": "minecraft:block/<model_name>" }
```

With transforms:
```json
{ "model": "minecraft:block/<model_name>", "x": 90, "y": 180, "uvlock": true }
```

- `x`, `y`: Rotation in degrees (0, 90, 180, 270)
- `uvlock`: Lock UV to block face (prevents rotation artifacts)

## Validation Rules

1. **All referenced models must exist** in `models/block/`
2. **Variants format**: All property combinations covered or default `""` provided
3. **Multipart format**: Conditions cover all needed connections
4. **Property names**: Match vanilla block state definitions
5. **JSON syntax**: Valid JSON (no trailing commas, quoted keys)

## Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Purple/black block | Model missing | Add model or fix reference |
| Wrong model for state | Property mismatch | Check property names/order |
| Z-fighting | Overlapping multipart | Adjust model geometry |
| Missing variation | Weighted variant not loaded | Check model exists, weight > 0 |

## Adding a New Blockstate

1. Identify block properties (check vanilla wiki)
2. Choose format: variants (simple) or multipart (connecting)
3. Create models first (see [Block Models](./block-models.md))
4. Write blockstate JSON
5. Test in-game (F3+T)
6. Verify all states render correctly

## Related Documents

- [Block Models](./block-models.md)
- [API Usage Examples](../api-usage-examples.md#blockstate-examples)
- [Change Guide](../change-guide.md#adding-a-new-block)
- [Configuration](../configuration.md#blockstates)