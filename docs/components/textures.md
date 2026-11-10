# Textures Component

Deep dive into texture files: formats, animations, variations, and NuciCraft conventions.

## Overview

Textures are PNG images with optional `.mcmeta` animation definitions. Located at `assets/minecraft/textures/block/` and `assets/minecraft/textures/item/`.

## File Structure

```
assets/minecraft/textures/
├── block/
│   ├── cobblestone.png
│   ├── cobblestone.png.mcmeta
│   ├── variated/
│   │   ├── cobblestone_var1.png
│   │   ├── cobblestone_var2.png
│   │   └── ...
│   ├── 3d/
│   │   ├── oak_fence_post.png
│   │   └── ...
│   └── (300+ files)
├── item/
│   ├── diamond_sword.png
│   ├── magic_compass.png
│   ├── musket.png
│   └── (100+ files)
└── ...
```

## Texture Requirements

### Dimensions

- **Must be square**: width = height
- **Must be power of two**: 1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024
- **Standard**: 16x16 (vanilla), 32x32 (Faithful 32x)
- **Max**: 1024x1024 (practical limit)

### Format

- **PNG** (lossless, supports transparency)
- **Color space**: sRGB
- **Bit depth**: 8-bit per channel (32-bit RGBA)

### Naming

- Lowercase with underscores: `cobblestone.png`, `oak_planks.png`
- Match model texture references exactly
- Variation suffixes: `_var1`, `_var2`, `_variated_1`

## Animated Textures (.mcmeta)

### Structure

```json
{
  "animation": {
    "frametime": 4,
    "frames": [0, 1, 2, 3, 4, 5],
    "interpolate": true
  }
}
```

### Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `frametime` | int | 1 | Ticks per frame (1 tick = 1/20 sec) |
| `frames` | array | [0..n-1] | Frame indices or objects |
| `interpolate` | bool | false | Smooth interpolation between frames |

### Frame Object

```json
"frames": [
  { "index": 0, "time": 4 },
  { "index": 1, "time": 2 },
  2,
  3
]
```

### PNG Layout for Animation

Vertical strip: each frame = width × width

```
Frame 0  ← width × width
Frame 1  ← width × width
Frame 2  ← width × width
...
```

Total height = width × frame_count

### Example: Water Still (16x16, 32 frames)

- PNG: 16px wide, 512px tall (16 × 32)
- `.mcmeta`: `"frametime": 2, "interpolate": true`

## Texture Variations

### Purpose

Randomised textures for natural look (cobblestone, ores, logs, etc.)

### Implementation

1. **Variation textures** in `textures/block/variated/`
2. **Variation models** in `models/block/variated/` or `models/block/<block>/`
3. **Blockstate weighted variants** reference variation models

### Example: Cobblestone

**Textures**:
- `textures/block/cobblestone.png` (base)
- `textures/block/variated/cobblestone_var1.png`
- `textures/block/variated/cobblestone_var2.png`

**Models**:
- `models/block/cobblestone.json` → `#all` = `cobblestone`
- `models/block/variated/cobblestone_variated_1.json` → `#all` = `variated/cobblestone_var1`
- `models/block/variated/cobblestone_variated_2.json` → `#all` = `variated/cobblestone_var2`

**Blockstate** (`blockstates/cobblestone.json`):
```json
{
  "variants": {
    "": [
      { "model": "minecraft:block/cobblestone", "weight": 3 },
      { "model": "minecraft:block/variated/cobblestone_variated_1", "weight": 1 },
      { "model": "minecraft:block/variated/cobblestone_variated_2", "weight": 1 }
    ]
  }
}
```

### Variation Guidelines

- **Count**: 2-4 variations typical
- **Weights**: Base higher (3), variations lower (1 each)
- **Visual difference**: Subtle — cracks, shade, pattern shift
- **Tileable**: All variations must tile seamlessly

## 3D Model Textures

### Purpose

Textures mapped onto 3D geometry (fences, doors, chains, etc.)

### Structure

`textures/block/3d/` — mirrors model structure

### Example: Oak Fence

- `textures/block/3d/oak_fence_post.png` — post texture
- `textures/block/3d/oak_fence_side.png` — side texture

**Models** reference these directly:
```json
"textures": {
  "particle": "minecraft:block/oak_planks",
  "texture": "minecraft:block/3d/oak_fence_post"
}
```

## Connected Textures (CTM) — Not Used

NuciCraft does **not** use CTM (Connected Textures Mod) — requires mod. Uses vanilla multipart blockstates instead.

## Texture Atlas

All textures stitched into atlas at load time.

### Source Definition

`assets/minecraft/atlases/blocks.json`:
```json
{
  "sources": [
    { "type": "directory", "source": "minecraft:textures/block", "prefix": "block/" }
  ]
}
```

### Implications

- Each unique texture = atlas slot
- Atlas size limited (typically 8192x8192 or 16384x16384)
- Too many variations → atlas overflow → magenta textures
- Monitor: `logs/latest.log` for "Atlas overflow"

## Item Textures

### Location

`assets/minecraft/textures/item/`

### Requirements

Same as block textures (square, power of two, PNG)

### Layers

Item models use layers:
- `layer0` = base
- `layer1` = enchantment glint (auto)
- `layer2+` = overlays

Each layer = separate texture file.

### Example: Diamond Sword

- `textures/item/diamond_sword.png` (layer0)

### Example: Bow (Multi-Layer)

- `textures/item/bow.png` (layer0)
- `textures/item/bow_pulling_0.png` (layer1)
- `textures/item/bow_pulling_1.png` (layer2)
- `textures/item/bow_pulling_2.png` (layer3)

## Custom Item Textures

### Magic Compass

- `textures/item/magic_compass.png`

### Musket

- `textures/item/musket.png`
- `textures/item/musket_pulling_0.png`
- `textures/item/musket_pulling_1.png`
- `textures/item/musket_pulling_2.png`

## Validation Checklist

- [ ] Square dimensions
- [ ] Power of two dimensions
- [ ] PNG format
- [ ] Transparency preserved (if needed)
- [ ] `.mcmeta` valid JSON (if animated)
- [ ] Animation PNG height = width × frames
- [ ] All model references resolve
- [ ] No duplicate textures (waste atlas space)
- [ ] Variation textures tile seamlessly

## Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Magenta texture | Missing file / wrong path | Add texture or fix reference |
| Animation not playing | Bad .mcmeta / wrong PNG layout | Fix .mcmeta, verify PNG height |
| Texture stretched | Non-square / non-power-of-two | Resize to 16, 32, 64, 128... |
| Atlas overflow | Too many unique textures | Reduce variations, consolidate |
| Seams on variations | Non-tileable variation | Fix variation texture edges |

## Tools

### Validate Dimensions (Python)

```python
from PIL import Image
import os

for root, dirs, files in os.walk("assets/minecraft/textures"):
    for f in files:
        if f.endswith(".png"):
            p = os.path.join(root, f)
            with Image.open(p) as img:
                w, h = img.size
                if w != h or (w & (w - 1)) != 0:
                    print(f"ISSUE: {p} = {w}x{h}")
```

### Validate .mcmeta

```bash
find assets -name "*.mcmeta" -exec jq empty {} \;
```

### Check Atlas Usage (In-Game)

F3 → "Texture Atlas" section shows usage %.

## Related Documents

- [Block Models](../components/block-models.md)
- [Item Models](../components/item-models.md)
- [Atlases](../components/atlases.md)
- [Blockstates](../components/blockstates.md)
- [API Usage Examples](../api-usage-examples.md#texture-examples)
- [Change Guide](../change-guide.md#adding-texture-variations)
- [Configuration](../configuration.md#texture-files)