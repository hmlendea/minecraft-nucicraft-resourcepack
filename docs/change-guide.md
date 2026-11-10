# Change Guide

This document provides safe modification patterns for contributors to the NuciCraft Resource Pack.

## General Principles

1. **Test every change** in Minecraft before committing
2. **Preserve referential integrity** — all references must resolve
3. **Follow existing patterns** — consistency over novelty
4. **Attribute community assets** — update README.md attributions
5. **Update pack.mcmeta description** on releases
6. **Keep changes atomic** — one logical change per commit

## Adding a New Block

### 1. Create Blockstate

**File**: `assets/minecraft/blockstates/<block_name>.json`

**Variants format** (for blocks with discrete states like fence gates):
```json
{
  "variants": {
    "facing=north,open=false": { "model": "minecraft:block/<model_name>" },
    "facing=east,open=false": { "model": "minecraft:block/<model_name>", "y": 90 },
    "facing=south,open=false": { "model": "minecraft:block/<model_name>", "y": 180 },
    "facing=west,open=false": { "model": "minecraft:block/<model_name>", "y": 270 },
    "facing=north,open=true": { "model": "minecraft:block/<model_name>_open" },
    "facing=east,open=true": { "model": "minecraft:block/<model_name>_open", "y": 90 }
  }
}
```

**Multipart format** (for blocks with connecting parts like fences):
```json
{
  "multipart": [
    { "apply": { "model": "minecraft:block/<model_name>_post" } },
    { "when": { "north": "true" }, "apply": { "model": "minecraft:block/<model_name>_side" } },
    { "when": { "east": "true" }, "apply": { "model": "minecraft:block/<model_name>_side", "y": 90 } },
    { "when": { "south": "true" }, "apply": { "model": "minecraft:block/<model_name>_side", "y": 180 } },
    { "when": { "west": "true" }, "apply": { "model": "minecraft:block/<model_name>_side", "y": 270 } }
  ]
}
```

### 2. Create Block Models

**Files**: `assets/minecraft/models/block/<model_name>.json` (and variants)

**Basic model**:
```json
{
  "parent": "minecraft:block/block",
  "textures": {
    "particle": "minecraft:block/<texture_name>",
    "all": "minecraft:block/<texture_name>"
  },
  "elements": [
    {
      "from": [0, 0, 0],
      "to": [16, 16, 16],
      "faces": {
        "up": { "uv": [0, 0, 16, 16], "texture": "#all" },
        "down": { "uv": [0, 0, 16, 16], "texture": "#all" },
        "north": { "uv": [0, 0, 16, 16], "texture": "#all" },
        "south": { "uv": [0, 0, 16, 16], "texture": "#all" },
        "east": { "uv": [0, 0, 16, 16], "texture": "#all" },
        "west": { "uv": [0, 0, 16, 16], "texture": "#all" }
      }
    }
  ]
}
```

### 3. Add Textures

**Files**: `assets/minecraft/textures/block/<texture_name>.png`

- Dimensions: 16x16 (or 32x32 for high-res)
- Power of two dimensions
- Place in appropriate subdirectory if needed

### 4. Add Language Entries

**File**: `assets/minecraft/lang/ro_ro.json`
```json
"block.minecraft.<block_name>": "Nume în Română"
```

**File**: `assets/minecraft/lang/en_us.json` (if different from vanilla)
```json
"block.minecraft.<block_name>": "Custom English Name"
```

### 5. Test

1. Reload resources (F3+T)
2. Place block in creative
3. Verify model, texture, name
4. Test all state variants

## Adding a New Item

### 1. Create Item Model

**File**: `assets/minecraft/models/item/<item_name>.json`

**Simple item** (references block model):
```json
{
  "parent": "minecraft:block/<block_name>",
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
    },
    "ground": {
      "rotation": [0, 0, 0],
      "translation": [0, 3, 0],
      "scale": [0.25, 0.25, 0.25]
    },
    "fixed": {
      "rotation": [0, 180, 0],
      "translation": [0, 0, 0],
      "scale": [0.5, 0.5, 0.5]
    }
  }
}
```

**Custom 3D item**:
```json
{
  "parent": "minecraft:item/generated",
  "textures": {
    "layer0": "minecraft:item/<texture_name>"
  },
  "display": { ... }
}
```

### 2. Add Texture

**File**: `assets/minecraft/textures/item/<texture_name>.png`

### 3. Add Language Entries

**File**: `assets/minecraft/lang/ro_ro.json`
```json
"item.minecraft.<item_name>": "Nume în Română"
```

### 4. Test

1. Reload resources (F3+T)
2. Obtain item in creative
3. Check all perspectives (F5)
4. Check inventory, item frame, ground, head

## Adding Texture Variations

### 1. Create Variation Textures

**Directory**: `assets/minecraft/textures/block/variated/<block_type>/`

**Files**: `<block_type>_<variant_number>.png` (e.g., `cobblestone_1.png`, `cobblestone_2.png`)

### 2. Create Variation Models

**Directory**: `assets/minecraft/models/block/variated/<block_type>/`

**Files**: `<block_type>_<variant_number>.json`

Each model references its specific texture:
```json
{
  "parent": "minecraft:block/<base_model>",
  "textures": {
    "all": "minecraft:block/variated/<block_type>/<block_type>_<variant_number>"
  }
}
```

### 3. Create Blockstate with Random Variants

**File**: `assets/minecraft/blockstates/<block_type>.json`

```json
{
  "variants": {
    "": [
      { "model": "minecraft:block/variated/<block_type>/<block_type>_1", "weight": 1 },
      { "model": "minecraft:block/variated/<block_type>/<block_type>_2", "weight": 1 },
      { "model": "minecraft:block/variated/<block_type>/<block_type>_3", "weight": 1 }
    ]
  }
}
```

### 4. Test

1. Reload resources
2. Place many blocks
3. Verify random distribution
4. Check no identical adjacent (probabilistic)

## Modifying Existing Assets

### Changing a Texture

1. Edit PNG file in place
2. Maintain dimensions (power of two)
3. If animated, update .mcmeta if frame count changes
4. Test in-game

### Changing a Model

1. Edit JSON file
2. Verify all texture references still resolve
3. Check parent chain for cycles
4. Test all block states that use this model

### Changing a Blockstate

1. Edit JSON file
2. Verify all referenced models exist
3. Test all state combinations in-game

### Renaming a Block/Item

1. Update language files (all languages)
2. If custom key (nucicraft namespace), update model references if needed
3. Test in all supported languages

## Updating Community Pack Assets

### Procedure

1. Download latest community pack release
2. Extract to temporary location
3. Compare with current vendored assets:
   ```bash
   diff -r <temp>/assets/minecraft/ assets/minecraft/ | grep -E "^(Only in|Files.*differ)"
   ```
4. Copy new/changed assets
5. Resolve conflicts (NuciCraft customisations take priority)
6. Update attribution in README.md if needed
7. Test thoroughly
8. Commit with message: `chore: update <pack-name> to <version>`

### Conflict Resolution

| Conflict Type | Resolution |
|---------------|------------|
| Same file, different content | Keep NuciCraft version if customised; otherwise take upstream |
| Upstream adds new files | Add them |
| Upstream removes files | Remove if not customised |
| Upstream renames files | Follow rename, update references |

## Updating Pack Format (Minecraft Version Upgrade)

### Procedure

1. Wait for stable Minecraft release
2. Check new pack_format on Minecraft Wiki
3. Update `pack.mcmeta`:
   ```json
   {
     "pack": {
       "pack_format": <new_format>,
       "description": "NuciCraft v<new_version>",
       "supported_formats": {
         "min_inclusive": 84,
         "max_inclusive": 999
       }
     }
   }
   ```
4. Check for asset schema changes:
   - Blockstate format changes
   - Model format changes
   - Atlas format changes
   - New block/item states
5. Update affected assets
6. Test on new version
7. Release as minor version bump

## Common Pitfalls

| Pitfall | Symptom | Prevention |
|---------|---------|------------|
| Typo in model path | Purple/black checkerboard | Copy-paste paths, test after change |
| Typo in texture path | Magenta texture | Verify texture exists, test |
| Missing variant in blockstate | Missing model for state | Test all state combinations |
| Circular parent chain | Model load error | Keep parent chains shallow |
| Non-power-of-two texture | Texture not loading | Validate dimensions |
| Invalid JSON syntax | Pack fails to load | Use JSON validator |
| Missing language key | Vanilla name shown | Add to all language files |
| Atlas overflow | Missing textures in-game | Limit unique textures |
| High-poly model | Low FPS | Vet model complexity |

## Validation Checklist Before Commit

- [ ] All JSON files valid syntax (`jq empty file.json`)
- [ ] All referenced models exist
- [ ] All referenced textures exist
- [ ] Texture dimensions are powers of two
- [ ] Language keys added for all supported languages
- [ ] Tested in Minecraft (F3+T reload)
- [ ] No purple/black checkerboard
- [ ] No magenta textures
- [ ] Custom names appear correctly
- [ ] Performance acceptable
- [ ] Attribution updated if community assets changed

## Commit Message Format

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Types**:
- `feat` — New block/item/feature
- `fix` — Bug fix (missing model, wrong texture, etc.)
- `chore` — Maintenance (community pack update, version bump)
- `refactor` — Model/texture reorganisation
- `docs` — Documentation updates

**Examples**:
```
feat(block): add 3D model for cherry fence gate
fix(texture): correct magenta texture on variated obsidian
chore(deps): update VanillaTweaks to 2024-01-15
refactor(models): consolidate fence post templates
docs: update change guide with variation procedure
```

## Review Checklist

For pull requests:

- [ ] Changes follow patterns in this guide
- [ ] All validation checklist items pass
- [ ] Attribution updated for community assets
- [ ] pack.mcmeta version bumped if release
- [ ] No unrelated changes
- [ ] Commit messages follow format