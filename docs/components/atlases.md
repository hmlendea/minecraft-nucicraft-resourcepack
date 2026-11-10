# Atlases Component

Deep dive into texture atlas configuration: sources, stitching, and NuciCraft conventions.

## Overview

Texture atlases combine individual textures into large sheets for GPU efficiency. Configured via `assets/minecraft/atlases/*.json`.

## File Structure

```
assets/minecraft/atlases/
├── blocks.json
└── (other atlases: items, particles, etc. — usually vanilla)
```

## blocks.json Structure

```json
{
  "sources": [
    {
      "type": "directory",
      "source": "minecraft:textures/block",
      "prefix": "block/"
    },
    {
      "type": "directory",
      "source": "minecraft:textures/block/variated",
      "prefix": "block/variated/"
    },
    {
      "type": "directory",
      "source": "minecraft:textures/block/3d",
      "prefix": "block/3d/"
    }
  ]
}
```

### Source Object Fields

| Field | Required | Description |
|-------|----------|-------------|
| `type` | Yes | `"directory"` or `"single"` |
| `source` | Yes | Resource location (namespace:path) |
| `prefix` | No | Prefix for atlas keys |
| `filter` | No | Regex filter for files |

### Source Types

#### Directory Source

Scans directory recursively for PNG files.

```json
{
  "type": "directory",
  "source": "minecraft:textures/block",
  "prefix": "block/"
}
```

- Each PNG → atlas entry with key `prefix + relative_path`
- e.g., `textures/block/cobblestone.png` → `block/cobblestone`

#### Single Source

Single texture file.

```json
{
  "type": "single",
  "source": "minecraft:textures/block/cobblestone",
  "prefix": "block/"
}
```

## Atlas Stitching Process

1. **Load sources** in order
2. **Collect textures** from each source
3. **Sort** by size (largest first) for packing efficiency
4. **Pack** into atlas pages (power-of-two dimensions)
5. **Generate UV coordinates** for each texture
6. **Upload** to GPU

## NuciCraft Configuration

### Current blocks.json

```json
{
  "sources": [
    {
      "type": "directory",
      "source": "minecraft:textures/block",
      "prefix": "block/"
    }
  ]
}
```

**Note**: Single source with recursive scan. Subdirectories (variated/, 3d/) included automatically.

### Implications

- All block textures under `textures/block/` stitched
- Keys: `block/cobblestone`, `block/variated/cobblestone_var1`, `block/3d/oak_fence_post`
- Model references use these keys: `"texture": "block/cobblestone"`

## Atlas Limits

### Practical Limits

| Limit | Value | Notes |
|-------|-------|-------|
| Max atlas size | 8192×8192 or 16384×16384 | GPU/driver dependent |
| Max textures | ~2000-4000 | Depends on texture sizes |
| Max unique textures | Atlas area / (16×16) | ~262K for 8K atlas |

### Monitoring

Check `logs/latest.log`:
```
[TextureAtlas] Stitched 1234 textures into 8192x8192 atlas
```

Or in-game F3 → "Texture Atlas" section.

### Overflow Symptoms

- Magenta textures in-game
- "Atlas overflow" in logs
- Missing textures for recently added assets

## Managing Atlas Usage

### Reduce Unique Textures

1. **Consolidate variations** — fewer variation textures
2. **Reuse textures** — same texture for similar blocks
3. **Remove unused** — delete orphaned textures
4. **Use smaller textures** — 16x instead of 32x where possible

### Estimate Usage

```bash
# Count unique block textures
find assets/minecraft/textures/block -name "*.png" | wc -l

# Estimate atlas slots (16x16 each)
# 1000 textures ≈ 1000 slots ≈ 16K×16K if all 16x
```

## Custom Atlases (Advanced)

### Item Atlas

Usually vanilla. Can override:

```json
// assets/minecraft/atlases/items.json
{
  "sources": [
    { "type": "directory", "source": "minecraft:textures/item", "prefix": "item/" }
  ]
}
```

### Particle Atlas

```json
// assets/minecraft/atlases/particles.json
{
  "sources": [
    { "type": "directory", "source": "minecraft:textures/particle", "prefix": "particle/" }
  ]
}
```

## Validation

### Check JSON Syntax

```bash
jq empty assets/minecraft/atlases/blocks.json
```

### Verify Sources Exist

```bash
# Check each source directory exists
ls assets/minecraft/textures/block/
ls assets/minecraft/textures/block/variated/
ls assets/minecraft/textures/block/3d/
```

### In-Game Verification

1. Enable pack
2. F3 → Texture Atlas
3. Check usage % and texture count
4. Look for missing textures (magenta)

## Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Textures missing | Source directory wrong | Fix `source` path |
| Atlas overflow | Too many textures | Reduce variations, consolidate |
| Wrong texture on model | Prefix mismatch | Align prefix with model references |
| Duplicate entries | Multiple sources overlap | Deduplicate sources |

## Related Documents

- [Textures](./textures.md)
- [Block Models](./block-models.md)
- [Configuration](../configuration.md#atlasesblocksjson)
- [API Usage Examples](../api-usage-examples.md)
- [Troubleshooting](../troubleshooting.md#atlas-overflow)