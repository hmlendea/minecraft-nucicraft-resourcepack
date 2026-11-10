# Variation Rendering Behaviour

Specification for texture variation rendering: selection, distribution, and visual consistency.

## Overview

Texture variations provide randomised appearance for blocks (cobblestone, ores, logs, etc.) using weighted random selection at placement time.

## Architecture

```
Block Placement → BlockState Lookup → Weighted Variant Selection → Model → BakedModel → Render
```

## Variant Selection Algorithm

### At Block Placement

1. **Get** blockstate for block
2. **Check** variants array for default state `""`
3. **If array**: Weighted random selection
4. **If single**: Use that model
5. **Cache** selection per block position (persistent)

### Weighted Random Selection

```json
"variants": {
  "": [
    { "model": "minecraft:block/cobblestone", "weight": 3 },
    { "model": "minecraft:block/cobblestone_variated_1", "weight": 1 },
    { "model": "minecraft:block/cobblestone_variated_2", "weight": 1 }
  ]
}
```

**Algorithm**:
1. Sum all weights = 5
2. Generate random float [0, 5)
3. Select variant where cumulative weight > random value

**Distribution**:
- Base (weight 3): 60%
- Variation 1 (weight 1): 20%
- Variation 2 (weight 1): 20%

### Persistence

- Selection stored in chunk data (block state)
- Same block position → same variation (deterministic)
- World save/load preserves variation
- **Not** re-randomised on resource pack reload

## Variation Model Structure

### Base Model

```json
// models/block/cobblestone.json
{
  "parent": "minecraft:block/block",
  "textures": {
    "particle": "minecraft:block/cobblestone",
    "all": "minecraft:block/cobblestone"
  }
}
```

### Variation Model

```json
// models/block/variated/cobblestone_variated_1.json
{
  "parent": "minecraft:block/block",
  "textures": {
    "particle": "minecraft:block/cobblestone",
    "all": "minecraft:block/variated/cobblestone_var1"
  }
}
```

### Key Points

- **Same geometry** (parent), different texture
- **Particle texture** = base texture (consistent breaking particles)
- **Texture path** points to variation in `textures/block/variated/`

## Texture Requirements

### Variation Textures

Location: `assets/minecraft/textures/block/variated/`

Naming: `<block>_var1.png`, `<block>_var2.png`, etc.

### Visual Consistency Rules

1. **Tileable**: All variations must tile seamlessly with themselves
2. **Compatible**: Variations should tile with each other (no visible seams)
3. **Subtle**: Differences should be natural (cracks, shade, pattern shift)
4. **Same palette**: Similar color range to base

### Testing Tileability

Place 10×10 grid of block → check for visible grid lines.

## Blockstate Configuration

### Standard Pattern

```json
{
  "variants": {
    "": [
      { "model": "minecraft:block/<block>", "weight": 3 },
      { "model": "minecraft:block/variated/<block>_variated_1", "weight": 1 },
      { "model": "minecraft:block/variated/<block>_variated_2", "weight": 1 }
    ]
  }
}
```

### Blocks Using Variations

| Block | Variations | Weights |
|-------|------------|---------|
| Cobblestone | 2 | 3:1:1 |
| Stone | 2 | 3:1:1 |
| Stone Bricks | 2 | 3:1:1 |
| Deepslate | 2 | 3:1:1 |
| Andesite | 2 | 3:1:1 |
| Diorite | 2 | 3:1:1 |
| Granite | 2 | 3:1:1 |
| Coal Ore | 2 | 3:1:1 |
| Iron Ore | 2 | 3:1:1 |
| Gold Ore | 2 | 3:1:1 |
| Diamond Ore | 2 | 3:1:1 |
| Emerald Ore | 2 | 3:1:1 |
| Redstone Ore | 2 | 3:1:1 |
| Lapis Ore | 2 | 3:1:1 |
| Copper Ore | 2 | 3:1:1 |
| Logs (all) | 2-3 | 3:1:1 |
| Terracotta (all) | 2 | 3:1:1 |
| Bricks | 2 | 3:1:1 |
| Nether Bricks | 2 | 3:1:1 |
| End Stone | 2 | 3:1:1 |
| Obsidian | 2 | 3:1:1 |

## Distribution Quality

### Visual Clustering

Random placement can create clusters of same variation.

**Mitigation**:
- Higher base weight reduces variation frequency
- More variations (3-4) reduces clustering perception
- Acceptable for natural materials

### Per-Chunk Distribution

Each chunk independently randomised → chunk boundaries may show pattern shifts.

**Not fixable** without global noise function (not available in vanilla).

## Performance

### Runtime Cost

- **Zero** at render time (model already selected)
- **One-time** at placement (weighted random)
- **Memory**: Block state stores variant index (minimal)

### Atlas Impact

Each variation = unique texture = atlas slot.

| Variations per Block | Atlas Slots |
|---------------------|-------------|
| 1 (base only) | 1 |
| 2 variations | 3 |
| 3 variations | 4 |
| 4 variations | 5 |

**Budget**: ~2000 textures max → ~500 blocks with 3 variations each.

## Adding New Variations

### Procedure

1. Create variation textures in `textures/block/variated/`
2. Create variation models in `models/block/variated/`
3. Update blockstate with weighted variants
4. Test: place 50+ blocks, verify distribution
5. Verify tileability at chunk boundaries

### Weight Tuning

| Goal | Weights |
|------|---------|
| Subtle (mostly base) | 5:1:1 |
| Balanced | 3:1:1 |
| Varied (more variation) | 2:1:1 |
| Equal | 1:1:1 |

## Debugging

### Verify Distribution

```mcfunction
# Fill area with block
/fill ~ ~ ~ ~10 ~ ~10 minecraft:cobblestone
# Count variations visually or via structure block
```

### Check Model References

```bash
# Verify all variation models exist
ls assets/minecraft/models/block/variated/

# Verify blockstate references valid models
jq '.variants.""[] | .model' assets/minecraft/blockstates/cobblestone.json
```

### Logs

No specific logs — variation selection is silent.

## Related Documents

- [Components: Blockstates](../components/blockstates.md)
- [Components: Block Models](../components/block-models.md)
- [Components: Textures](../components/textures.md)
- [Flows: Pack Load](../flows/pack-load.md)
- [Flows: Model Baking](../flows/model-baking.md)
- [Flows: Render Frame](../flows/render-frame.md)
- [Change Guide](../change-guide.md#adding-texture-variations)