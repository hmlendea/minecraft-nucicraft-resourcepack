# Item Models Component

Deep dive into item model files: structure, display transforms, overrides, and NuciCraft conventions.

## Overview

Item models define 3D geometry and display transforms for items. Located at `assets/minecraft/models/item/*.json`.

## File Structure

```
assets/minecraft/models/item/
├── diamond_sword.json
├── crossbow.json
├── compass.json
├── magic_compass.json
├── musket.json
├── musket_pulling_0.json
├── musket_pulling_1.json
├── musket_pulling_2.json
└── (100+ files)
```

## Model Structure

### Basic Fields

```json
{
  "parent": "minecraft:item/handheld",
  "textures": {
    "layer0": "minecraft:item/diamond_sword"
  },
  "display": { ... },
  "overrides": [ ... ]
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `parent` | Yes* | Parent model (usually generated/handheld) |
| `textures` | Yes | Layer textures |
| `display` | No | Transform per view |
| `overrides` | No | Conditional model switching |

*Parent provides base geometry.

### Common Parents

| Parent | Use Case |
|--------|----------|
| `minecraft:item/generated` | Flat 2D items (sword, pickaxe, food) |
| `minecraft:item/handheld` | 3D handheld (tools, weapons) |
| `minecraft:item/compass` | Compass (special animation) |
| `minecraft:item/clock` | Clock (special animation) |
| `minecraft:item/crossbow` | Crossbow (pulling animation) |
| `minecraft:item/bow` | Bow (pulling animation) |
| `minecraft:item/fishing_rod` | Fishing rod (cast animation) |
| `minecraft:item/shield` | Shield (blocking animation) |

### Textures (Layers)

```json
"textures": {
  "layer0": "minecraft:item/diamond_sword",
  "layer1": "minecraft:item/enchantment_glint",
  "layer2": "minecraft:item/custom_overlay"
}
```

- `layer0` = base texture
- `layer1` = enchantment glint (auto-applied)
- `layer2+` = custom overlays
- All layers rendered additively

### Display Transforms

```json
"display": {
  "thirdperson_righthand": {
    "rotation": [-90, 55, -45],
    "translation": [0, 2.5, 0],
    "scale": [0.68, 0.68, 0.68]
  },
  "firstperson_righthand": {
    "rotation": [0, -90, 25],
    "translation": [1.13, 3.2, -1.13],
    "scale": [0.68, 0.68, 0.68]
  },
  "firstperson_lefthand": { ... },
  "thirdperson_lefthand": { ... },
  "gui": {
    "rotation": [30, 225, 0],
    "translation": [0, 0, 0],
    "scale": [0.62, 0.62, 0.62]
  },
  "ground": {
    "rotation": [0, 0, 0],
    "translation": [0, 3, 0],
    "scale": [0.25, 0.25, 0.25]
  },
  "fixed": {
    "rotation": [0, 0, 0],
    "translation": [0, 0, 0],
    "scale": [0.5, 0.5, 0.5]
  },
  "head": { ... }
}
```

#### Transform Fields

| Field | Type | Description |
|-------|------|-------------|
| `rotation` | [x,y,z] | Degrees, applied in order X→Y→Z |
| `translation` | [x,y,z] | Units (1/16 block) |
| `scale` | [x,y,z] | Multiplier |

#### View Contexts

| Context | When Used |
|---------|-----------|
| `thirdperson_righthand` | Other players, right hand |
| `thirdperson_lefthand` | Other players, left hand |
| `firstperson_righthand` | Your view, right hand |
| `firstperson_lefthand` | Your view, left hand |
| `gui` | Inventory, creative tab, JEI |
| `ground` | Dropped on ground |
| `fixed` | Item frame, armor stand |
| `head` | Worn on head |

### Overrides (Conditional Models)

```json
"overrides": [
  {
    "predicate": { "custom_model_data": 1001 },
    "model": "minecraft:item/magic_compass"
  },
  {
    "predicate": { "pulling": 1, "pull": 0.65 },
    "model": "minecraft:item/bow_pulling_1"
  }
]
```

#### Predicate Fields

| Field | Type | Description |
|-------|------|-------------|
| `custom_model_data` | int | CustomModelData NBT value |
| `pulling` | 0/1 | Is pulling (bow/crossbow) |
| `pull` | float | Pull progress (0-1) |
| `blocking` | 0/1 | Is blocking (shield) |
| `damaged` | 0/1 | Is damaged |
| `damage` | float | Damage ratio (0-1) |
| `lefthanded` | 0/1 | In left hand |
| `cooldown` | float | Cooldown progress |

**Evaluation**: First matching predicate wins. Order matters.

## NuciCraft Patterns

### Pattern 1: Standard Generated Item

```json
{
  "parent": "minecraft:item/generated",
  "textures": {
    "layer0": "minecraft:item/diamond"
  }
}
```

### Pattern 2: Handheld Tool/Weapon

```json
{
  "parent": "minecraft:item/handheld",
  "textures": {
    "layer0": "minecraft:item/diamond_sword"
  },
  "display": {
    "thirdperson_righthand": { "rotation": [-90, 55, -45], "translation": [0, 2.5, 0], "scale": [0.68, 0.68, 0.68] },
    "firstperson_righthand": { "rotation": [0, -90, 25], "translation": [1.13, 3.2, -1.13], "scale": [0.68, 0.68, 0.68] },
    "gui": { "rotation": [30, 225, 0], "scale": [0.62, 0.62, 0.62] },
    "ground": { "translation": [0, 3, 0], "scale": [0.25, 0.25, 0.25] }
  }
}
```

### Pattern 3: Custom Model Data Override (Magic Compass)

**Base Model** (`compass.json`):
```json
{
  "parent": "minecraft:item/compass",
  "overrides": [
    {
      "predicate": { "custom_model_data": 1001 },
      "model": "minecraft:item/magic_compass"
    }
  ]
}
```

**Override Model** (`magic_compass.json`):
```json
{
  "parent": "minecraft:item/compass",
  "textures": {
    "layer0": "minecraft:item/magic_compass"
  }
}
```

### Pattern 4: Pulling Animation (Musket/Crossbow)

**Base Model** (`crossbow.json`):
```json
{
  "parent": "minecraft:item/crossbow",
  "textures": {
    "layer0": "minecraft:item/musket"
  },
  "overrides": [
    { "predicate": { "pulling": 1 }, "model": "minecraft:item/musket_pulling_0" },
    { "predicate": { "pulling": 1, "pull": 0.65 }, "model": "minecraft:item/musket_pulling_1" },
    { "predicate": { "pulling": 1, "pull": 0.9 }, "model": "minecraft:item/musket_pulling_2" }
  ]
}
```

**Pulling Frames** (`musket_pulling_0.json`):
```json
{
  "parent": "minecraft:item/crossbow",
  "textures": {
    "layer0": "minecraft:item/musket_pulling_0"
  }
}
```

### Pattern 5: Multi-Layer (Bow-style)

```json
{
  "parent": "minecraft:item/generated",
  "textures": {
    "layer0": "minecraft:item/bow",
    "layer1": "minecraft:item/bow_pulling_0",
    "layer2": "minecraft:item/bow_pulling_1",
    "layer3": "minecraft:item/bow_pulling_2"
  },
  "overrides": [
    { "predicate": { "pulling": 1 }, "model": "minecraft:item/bow_pulling_0" },
    { "predicate": { "pulling": 1, "pull": 0.65 }, "model": "minecraft:item/bow_pulling_1" },
    { "predicate": { "pulling": 1, "pull": 0.9 }, "model": "minecraft:item/bow_pulling_2" }
  ]
}
```

## Coordinate System (Display Transforms)

- **Rotation**: Euler angles in degrees, order X→Y→Z
- **Translation**: 1 unit = 1/16 block = 1 pixel on 16x texture
- **Scale**: 1.0 = original size

### Common Transform Values

| View | Rotation | Translation | Scale |
|------|----------|-------------|-------|
| GUI | [30, 225, 0] | [0, 0, 0] | [0.62, 0.62, 0.62] |
| Ground | [0, 0, 0] | [0, 3, 0] | [0.25, 0.25, 0.25] |
| Fixed | [0, 0, 0] | [0, 0, 0] | [0.5, 0.5, 0.5] |
| Thirdperson RH | [-90, 55, -45] | [0, 2.5, 0] | [0.68, 0.68, 0.68] |
| Firstperson RH | [0, -90, 25] | [1.13, 3.2, -1.13] | [0.68, 0.68, 0.68] |

## Validation Rules

1. **Parent exists** — valid model path
2. **Texture references exist** — all layer textures in `textures/item/`
3. **Override models exist** — all referenced models present
4. **Predicate keys valid** — only known predicate fields
5. **No circular overrides** — A→B→A not allowed
6. **JSON syntax valid** — no trailing commas

## Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Item invisible | Parent missing | Fix parent path |
| Magenta texture | Texture missing | Add texture or fix reference |
| Wrong hand position | Bad display transforms | Copy from similar vanilla item |
| Override not working | Predicate mismatch | Check predicate fields, order |
| Animation not playing | Wrong parent | Use compass/clock/crossbow/bow parent |

## Adding a New Item Model

1. Choose parent (generated, handheld, crossbow, etc.)
2. Create texture(s) in `textures/item/`
3. Write model JSON
4. If override: add to base model overrides
5. Test in-game (F3+T)
6. Check all views (GUI, hand, ground, frame)

## Performance Considerations

- **Layer count**: More layers = more draw calls
- **Override count**: Evaluated each frame; keep minimal
- **Texture size**: 16x standard; 32x+ uses more atlas space
- **Custom model data**: Requires datapack to set NBT

## Related Documents

- [Block Models](./block-models.md)
- [Textures](./textures.md)
- [API Usage Examples](../api-usage-examples.md#model-examples)
- [Behaviour: Custom Item Behaviour](../behaviour/custom-item-behaviour.md)
- [Change Guide](../change-guide.md#adding-a-new-item)
- [Configuration](../configuration.md#modelsitem)