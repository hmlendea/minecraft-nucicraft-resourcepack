# Configuration

This document describes all configuration files in the NuciCraft Resource Pack, their schemas, and valid values.

## pack.mcmeta

**Location**: Repository root

**Purpose**: Pack metadata and format compatibility declaration.

### Schema

```json
{
  "pack": {
    "pack_format": 107.1,
    "description": "NuciCraft v6.3.5",
    "supported_formats": {
      "min_inclusive": 84,
      "max_inclusive": 999
    }
  }
}
```

### Fields

| Field | Type | Required | Valid Values | Description |
|-------|------|----------|--------------|-------------|
| `pack.pack_format` | Number | Yes | Integer matching target Minecraft version | Current pack format version |
| `pack.description` | String | Yes | `NuciCraft v<semver>` | Display name in resource pack menu |
| `pack.supported_formats.min_inclusive` | Integer | Yes | ≥ 1, ≤ `max_inclusive` | Minimum compatible format |
| `pack.supported_formats.max_inclusive` | Integer | Yes | ≥ `min_inclusive`, typically 999 | Maximum compatible format |

### Pack Format Versions

| Minecraft Version | Pack Format |
|-------------------|-------------|
| 1.21.x | 107.1 |
| 1.20.5 - 1.20.6 | 48 |
| 1.20.3 - 1.20.4 | 41 |
| 1.20 - 1.20.2 | 15 |
| 1.19.3 - 1.19.4 | 12 |
| 1.19 - 1.19.2 | 9 |
| 1.18 - 1.18.2 | 8 |
| 1.17 - 1.17.1 | 7 |
| 1.16.2 - 1.16.5 | 6 |
| 1.16 - 1.16.1 | 5 |

### Validation Rules

- `pack_format` must be within `[min_inclusive, max_inclusive]`
- Client will reject pack if its `pack_format` > client's supported format
- Client will load pack if client's format ≥ `min_inclusive` and ≤ `max_inclusive`
- Description should follow `NuciCraft v<major>.<minor>.<patch>` pattern

## assets/minecraft/atlases/blocks.json

**Location**: `assets/minecraft/atlases/blocks.json`

**Purpose**: Configure texture atlas sources for block textures.

### Schema

```json
{
  "sources": [
    { "type": "directory", "source": "minecraft:block" },
    { "type": "directory", "source": "minecraft:block/3d" },
    { "type": "directory", "source": "minecraft:block/variated" },
    { "type": "single", "source": "minecraft:block/colormap/grass" },
    { "type": "single", "source": "minecraft:block/colormap/foliage" },
    { "type": "paletted_permutations", "source": "minecraft:block/colormap/grass", "palette_key": "minecraft:grass" }
  ]
}
```

### Source Types

| Type | Description | Required Fields |
|------|-------------|-----------------|
| `directory` | All PNG files in directory recursively | `type`, `source` |
| `single` | Single texture file | `type`, `source` |
| `paletted_permutations` | Generate variants from palette | `type`, `source`, `palette_key` |

### Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | String | Yes | Source type: "directory", "single", "paletted_permutations" |
| `source` | String | Yes | Path relative to `textures/` (e.g., `minecraft:block`) |
| `prefix` | String | No | Prefix for generated texture names |
| `palette_key` | String | For paletted_permutations | Palette identifier |
| `filters` | Object | No | Filter conditions for inclusion |

### Validation Rules

- All `source` paths must exist under `assets/minecraft/textures/`
- Directory sources include all `.png` files recursively
- No duplicate texture names across sources (client deduplicates)
- Atlas size limited by GPU/client (typically 8192x8192 or 16384x16384)

## assets/minecraft/lang/*.json

**Location**: `assets/minecraft/lang/`

**Purpose**: Translation overrides for block/item/entity names and UI text.

### Files

| File | Language | Purpose |
|------|----------|---------|
| `ro_ro.json` | Romanian (Romania) | Primary custom language |
| `en_us.json` | English (US) | Minimal overrides |
| `en_gb.json` | English (UK) | Minimal overrides |
| `en_au.json` | English (Australia) | Minimal overrides |
| `en_ca.json` | English (Canada) | Minimal overrides |
| `en_nz.json` | English (New Zealand) | Minimal overrides |
| `en_ud.json` | English (Pirate) | Minimal overrides |
| `languages.json` | Metadata | Language list for client |

### Schema (Translation Files)

```json
{
  "block.minecraft.oak_planks": "Placi de stejar",
  "item.minecraft.diamond_sword": "Sabie de diamant",
  "block.nucicraft.magic_compass": "Busolă Magică"
}
```

### Schema (languages.json)

```json
[
  { "code": "ro_ro", "name": "Română (România)", "region": "RO", "bidirectional": false },
  { "code": "en_us", "name": "English (US)", "region": "US", "bidirectional": false }
]
```

### Fields (languages.json)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `code` | String | Yes | Language code (matches filename) |
| `name` | String | Yes | Display name |
| `region` | String | Yes | Region code |
| `bidirectional` | Boolean | Yes | RTL support |

### Validation Rules

- Translation keys must exactly match vanilla keys
- Keys follow pattern: `<type>.<namespace>.<path>` (e.g., `block.minecraft.oak_planks`)
- Custom keys for new blocks/items: `block.nucicraft.<name>`, `item.nucicraft.<name>`
- No duplicate keys within a file
- `languages.json` must list all provided language files

## assets/minecraft/blockstates/*.json

**Location**: `assets/minecraft/blockstates/`

**Purpose**: Map block state properties to model variants.

### Variants Format

```json
{
  "variants": {
    "facing=north,open=false": { "model": "minecraft:block/oak_fence_gate" },
    "facing=south,open=true": { "model": "minecraft:block/oak_fence_gate", "y": 180 }
  }
}
```

### Multipart Format

```json
{
  "multipart": [
    { "apply": { "model": "minecraft:block/oak_fence_post" } },
    { "when": { "north": "true" }, "apply": { "model": "minecraft:block/oak_fence_side" } }
  ]
}
```

### Validation Rules

- Exactly one of `variants` or `multipart` per file
- All referenced models must exist in `models/block/`
- Property names must match block's state definition
- Variant keys: comma-separated `property=value` pairs
- Multipart `when`: object with property conditions
- Multipart `apply`: model reference or array of references

## assets/minecraft/models/block/*.json

**Location**: `assets/minecraft/models/block/` and subdirectories

**Purpose**: Define 3D geometry for blocks.

### Schema

```json
{
  "parent": "minecraft:block/block",
  "ambientocclusion": true,
  "textures": {
    "particle": "minecraft:block/oak_planks",
    "side": "minecraft:block/oak_planks"
  },
  "elements": [
    {
      "from": [0, 0, 0],
      "to": [16, 16, 16],
      "faces": {
        "up": { "uv": [0, 0, 16, 16], "texture": "#particle" },
        "down": { "uv": [0, 0, 16, 16], "texture": "#particle" }
      }
    }
  ]
}
```

### Validation Rules

- `from`/`to` coordinates in range [0, 16]
- `from` ≤ `to` for each axis
- UV coordinates in range [0, 16] (atlas space)
- Texture variables (`#name`) must be defined in `textures` or inherited
- Parent chain must not cycle
- Element rotation origin within [0, 16] if specified

## assets/minecraft/models/item/*.json

**Location**: `assets/minecraft/models/item/`

**Purpose**: Define 3D geometry and display transforms for items.

### Schema

```json
{
  "parent": "minecraft:block/oak_planks",
  "display": {
    "firstperson_righthand": {
      "rotation": [0, 45, 0],
      "translation": [0, 0, 0],
      "scale": [0.5, 0.5, 0.5]
    },
    "gui": {
      "rotation": [30, 225, 0],
      "translation": [0, 0, 0],
      "scale": [0.625, 0.625, 0.625]
    }
  }
}
```

### Display Perspectives

| Perspective | Typical Use |
|-------------|-------------|
| `firstperson_righthand` | Held in right hand (first person) |
| `firstperson_lefthand` | Held in left hand (first person) |
| `thirdperson_righthand` | Held in right hand (third person) |
| `thirdperson_lefthand` | Held in left hand (third person) |
| `gui` | Inventory, creative menu |
| `head` | Worn on head |
| `ground` | Dropped item |
| `fixed` | Item frame |

### Validation Rules

- Transform values follow Minecraft conventions
- Parent references valid (block or item model)
- Texture variables resolve to `textures/item/`
- Override predicates use valid custom model data

## Texture Files (.png + .mcmeta)

**Location**: `assets/minecraft/textures/**/`

### PNG Requirements

- Dimensions: powers of two (16, 32, 64, 128, 256, 512, 1024...)
- Format: PNG (RGBA)
- No interlacing required but recommended off

### Animation Metadata (.mcmeta)

```json
{
  "animation": {
    "frametime": 2,
    "frames": [0, 1, 2, 3, 4, 5],
    "interpolate": true
  }
}
```

### Animation Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `frametime` | Integer | 1 | Ticks per frame (1 tick = 1/20 sec) |
| `frames` | Array | All frames | Frame index sequence |
| `interpolate` | Boolean | false | Smooth interpolation between frames |

### Validation Rules

- Animated texture: height = width × frame count (vertical strip)
- Frame indices within frame count
- `frametime` > 0

## .github/workflows/release-asset.yml

**Location**: `.github/workflows/release-asset.yml`

**Purpose**: Automated release packaging.

### Configuration

```yaml
name: Release Asset
on:
  release:
    types: [published]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Create release package
        run: |
          zip -r nucicraft_${{ github.ref_name }}.zip assets/ pack.mcmeta pack.png
      - name: Compute SHA1
        run: echo "SHA1: $(sha1sum nucicraft_${{ github.ref_name }}.zip | cut -d' ' -f1)"
      - name: Upload release asset
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ release.upload_url }}
          asset_path: ./nucicraft_${{ github.ref_name }}.zip
          asset_name: nucicraft_${{ github.ref_name }}.zip
          asset_content_type: application/zip
```

### Triggers

- `release.published` — When a GitHub Release is published (not draft)

### Outputs

- ZIP file: `nucicraft_<tag>.zip` (e.g., `nucicraft_v6.3.5.zip`)
- Contains: `assets/`, `pack.mcmeta`, `pack.png`
- SHA1 checksum in release notes

## Environment Variables

None required — all configuration is file-based.

## Configuration Validation Checklist

Before release, verify:

- [ ] `pack.mcmeta` format matches target Minecraft version
- [ ] `pack.mcmeta` description follows `NuciCraft v<semver>`
- [ ] `supported_formats` range includes target format
- [ ] `atlases/blocks.json` sources all exist
- [ ] All `blockstates/*.json` model references resolve
- [ ] All `models/**/*.json` texture references resolve
- [ ] All texture PNGs have power-of-two dimensions
- [ ] All `.mcmeta` animation files valid JSON
- [ ] `lang/ro_ro.json` keys match vanilla (spot check)
- [ ] `lang/languages.json` lists all provided languages
- [ ] Release workflow triggers on tag push