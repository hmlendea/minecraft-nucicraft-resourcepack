# Block Models Component

Deep dive into block model files: structure, geometry, and NuciCraft conventions.

## Overview

Block models define 3D geometry for blocks. Located at `assets/minecraft/models/block/*.json` and subdirectories.

## File Structure

```
assets/minecraft/models/block/
├── cobblestone.json
├── oak_fence_post.json
├── oak_fence_side.json
├── oak_door_bottom_left.json
├── variated/
│   ├── cobblestone_variated_1.json
│   └── ...
├── andesite/
│   ├── andesite_variated_1.json
│   └── ...
├── birch_log/
│   ├── birch_log.json
│   └── ...
└── (500+ files)
```

## Model Structure

### Basic Fields

```json
{
  "parent": "minecraft:block/block",
  "ambientocclusion": true,
  "textures": {
    "particle": "minecraft:block/cobblestone",
    "all": "minecraft:block/cobblestone"
  },
  "elements": [
    { ... }
  ],
  "display": { ... }
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `parent` | No | Parent model to inherit from |
| `ambientocclusion` | No | Enable AO (default true) |
| `textures` | Yes* | Texture variable mappings |
| `elements` | Yes* | 3D geometry elements |
| `display` | No | Transform overrides per view |
| `gui_light` | No | "front" or "side" for GUI rendering |

*Either parent must provide, or define locally.

### Texture Variables

```json
"textures": {
  "particle": "minecraft:block/cobblestone",
  "top": "minecraft:block/grass_block_top",
  "bottom": "minecraft:block/dirt",
  "side": "minecraft:block/grass_block_side",
  "#custom": "minecraft:block/custom_texture"
}
```

- Keys without `#` = variable names referenced in elements as `#key`
- `particle` = breaking particles, map icon
- Values = texture paths (namespace:path or minecraft:block/name)

### Elements (Geometry)

Each element is a cuboid with 6 faces.

```json
"elements": [
  {
    "from": [0, 0, 0],
    "to": [16, 16, 16],
    "rotation": { "origin": [8, 8, 8], "axis": "y", "angle": 45, "rescale": false },
    "shade": true,
    "faces": {
      "down":  { "uv": [0, 0, 16, 16], "texture": "#all", "cullface": "down", "tintindex": 0 },
      "up":    { "uv": [0, 0, 16, 16], "texture": "#all", "cullface": "up" },
      "north": { "uv": [0, 0, 16, 16], "texture": "#all", "cullface": "north" },
      "south": { "uv": [0, 0, 16, 16], "texture": "#all", "cullface": "south" },
      "west":  { "uv": [0, 0, 16, 16], "texture": "#all", "cullface": "west" },
      "east":  { "uv": [0, 0, 16, 16], "texture": "#all", "cullface": "east" }
    }
  }
]
```

#### Element Fields

| Field | Type | Description |
|-------|------|-------------|
| `from` | [x,y,z] | Min corner (0-16) |
| `to` | [x,y,z] | Max corner (0-16) |
| `rotation` | object | Optional rotation |
| `shade` | bool | Apply lighting (default true) |
| `faces` | object | Face definitions |

#### Face Fields

| Field | Type | Description |
|-------|------|-------------|
| `uv` | [u1,v1,u2,v2] | Texture coordinates (0-16) |
| `texture` | string | Texture variable `#name` |
| `cullface` | string | Face to cull against (down/up/north/south/east/west) |
| `tintindex` | int | Color tint index (-1 = none, 0 = foliage, 1 = grass) |
| `rotation` | int | Texture rotation (0, 90, 180, 270) |

### Display Transforms

```json
"display": {
  "thirdperson_righthand": { "rotation": [0,0,0], "translation": [0,0,0], "scale": [1,1,1] },
  "firstperson_righthand": { "rotation": [0,0,0], "translation": [0,0,0], "scale": [1,1,1] },
  "gui": { "rotation": [30,225,0], "translation": [0,0,0], "scale": [0.62,0.62,0.62] },
  "ground": { "rotation": [0,0,0], "translation": [0,3,0], "scale": [0.25,0.25,0.25] },
  "fixed": { "rotation": [0,0,0], "translation": [0,0,0], "scale": [0.5,0.5,0.5] }
}
```

## Common Parent Models

| Parent | Use Case |
|--------|----------|
| `minecraft:block/block` | Full cube (cobblestone, stone) |
| `minecraft:block/cube` | Base cube, no faces |
| `minecraft:block/cube_bottom_top` | Different top/bottom (logs, furnaces) |
| `minecraft:block/cube_column` | Column (logs, quartz) |
| `minecraft:block/cross` | Cross model (flowers, grass) |
| `minecraft:block/template_fence_post` | Fence post |
| `minecraft:block/template_fence_side` | Fence side |
| `minecraft:block/template_door_bottom` | Door bottom |
| `minecraft:block/template_door_top` | Door top |
| `minecraft:block/template_trapdoor` | Trapdoor |

## NuciCraft Patterns

### Pattern 1: Full Cube with Variations

**Base Model** (`cobblestone.json`):
```json
{
  "parent": "minecraft:block/block",
  "textures": {
    "particle": "minecraft:block/cobblestone",
    "all": "minecraft:block/cobblestone"
  }
}
```

**Variation Model** (`variated/cobblestone_variated_1.json`):
```json
{
  "parent": "minecraft:block/block",
  "textures": {
    "particle": "minecraft:block/cobblestone",
    "all": "minecraft:block/variated/cobblestone_var1"
  }
}
```

### Pattern 2: Directional Logs

**Model** (`birch_log/birch_log.json`):
```json
{
  "parent": "minecraft:block/cube_column",
  "textures": {
    "particle": "minecraft:block/birch_log_top",
    "end": "minecraft:block/birch_log_top",
    "side": "minecraft:block/birch_log"
  }
}
```

### Pattern 3: Fence Post/Side

**Post** (`oak_fence_post.json`):
```json
{
  "parent": "minecraft:block/template_fence_post",
  "textures": {
    "particle": "minecraft:block/oak_planks",
    "texture": "minecraft:block/oak_planks"
  }
}
```

**Side** (`oak_fence_side.json`):
```json
{
  "parent": "minecraft:block/template_fence_side",
  "textures": {
    "particle": "minecraft:block/oak_planks",
    "texture": "minecraft:block/oak_planks"
  }
}
```

### Pattern 4: Cross Model (Flowers, Grass)

```json
{
  "parent": "minecraft:block/cross",
  "textures": {
    "particle": "minecraft:block/poppy",
    "cross": "minecraft:block/poppy"
  }
}
```

### Pattern 5: Complex 3D (Door, Chain, Lantern)

Multiple elements with precise coordinates. See `oak_door_bottom_left.json`, `chain.json`, `lantern.json`.

## Coordinate System

- Origin: Block corner (0,0,0) = min x, min y, min z
- Range: 0 to 16 (1 block = 16 units)
- Axes: X = east, Y = up, Z = south
- Faces: down (y=0), up (y=16), north (z=0), south (z=16), west (x=0), east (x=16)

## UV Mapping

- UV coordinates in texture pixels (0-16 for 16x16 texture)
- [u1, v1, u2, v2] = left, top, right, bottom
- Origin: top-left of texture
- For 16x16 texture: full face = [0, 0, 16, 16]

## Tinting

| Tint Index | Source | Blocks |
|------------|--------|--------|
| 0 | Foliage color (biome) | Leaves, vines |
| 1 | Grass color (biome) | Grass block top, tall grass |
| 2 | Water color | Water, lily pad |
| -1 | None | Most blocks |

```json
"faces": {
  "up": { "uv": [0,0,16,16], "texture": "#top", "tintindex": 1 }
}
```

## Validation Rules

1. **Coordinates in [0,16]** — `from` < `to` for all axes
2. **UV coordinates in [0,16]** — matches texture dimensions
3. **Texture variables defined** — all `#var` in `textures`
4. **Parent exists** — if specified
5. **No circular parent chain** — max depth ~10
6. **JSON syntax valid** — no trailing commas

## Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Magenta texture | Texture missing | Add texture or fix reference |
| Stretched texture | Wrong UV | Fix UV coordinates |
| Dark faces | Missing cullface | Add cullface |
| Z-fighting | Coplanar faces | Adjust coordinates |
| Rotation wrong | Wrong origin/axis | Fix rotation object |

## Adding a New Block Model

1. Determine geometry type (cube, cross, column, custom)
2. Choose parent or create custom elements
3. Create texture(s) in `textures/block/`
4. Write model JSON
5. Reference in blockstate
6. Test in-game (F3+T)

## Performance Considerations

- **Element count**: Fewer = faster. Target < 20 per model.
- **Face count**: Cullface reduces rendered faces.
- **Texture atlas**: Each unique texture = atlas slot.
- **Transparency**: Transparent faces cost more (sorting).

## Related Documents

- [Blockstates](./blockstates.md)
- [Textures](./textures.md)
- [API Usage Examples](../api-usage-examples.md#model-examples)
- [Change Guide](../change-guide.md#adding-a-new-block)
- [Configuration](../configuration.md#modelsblock)