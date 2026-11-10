# Data Model

This document defines the domain entities, their attributes, relationships, and invariants for the NuciCraft Resource Pack.

## Domain Entities

### PackMetadata

**Location**: `pack.mcmeta`

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `pack.pack_format` | Integer | Yes | Minecraft resource pack format version (107.1) |
| `pack.description` | String | Yes | Human-readable description ("NuciCraft v6.3.5") |
| `pack.supported_formats.min_inclusive` | Integer | Yes | Minimum compatible pack_format (84) |
| `pack.supported_formats.max_inclusive` | Integer | Yes | Maximum compatible pack_format (999) |

**Invariants**:
- `pack_format` must be within `[min_inclusive, max_inclusive]`
- `description` must follow pattern `NuciCraft v<semver>`
- `min_inclusive` ≤ `max_inclusive`

### BlockState

**Location**: `assets/minecraft/blockstates/<block>.json`

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `variants` | Object | Conditional* | Map of state property combinations → model references |
| `multipart` | Array | Conditional* | List of conditional model applications |

*Exactly one of `variants` or `multipart` must be present.

**Variant Entry**:
| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `model` | String | Yes | Model path (e.g., `minecraft:block/oak_fence_gate`) |
| `x` | Integer | No | X rotation (0, 90, 180, 270) |
| `y` | Integer | No | Y rotation (0, 90, 180, 270) |
| `uvlock` | Boolean | No | Lock UV to model orientation |

**Multipart Entry**:
| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `when` | Object | No | State property conditions (e.g., `{ "north": "true" }`) |
| `apply` | Object/Array | Yes | Model reference(s) to apply when conditions met |

**Invariants**:
- All referenced models must exist under `models/block/`
- Property names must match Minecraft's block state definitions for that block
- Variant keys must be valid combinations of boolean/enum properties
- No duplicate variant keys
- Multipart `when` conditions must be mutually exclusive or order-defined

### BlockModel

**Location**: `assets/minecraft/models/block/**/*.json`

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `parent` | String | No | Parent model path for inheritance |
| `ambientocclusion` | Boolean | No | Enable ambient occlusion (default: true) |
| `display` | Object | No | Display transforms (see ItemModel) |
| `textures` | Object | No | Texture variable assignments (e.g., `{ "particle": "minecraft:block/oak_planks" }`) |
| `elements` | Array | No | Geometry elements (cubes) |
| `gui_light` | String | No | Light type for GUI: "front" or "side" |

**Element**:
| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `from` | [Number, Number, Number] | Yes | Min corner (x, y, z) in 0-16 space |
| `to` | [Number, Number, Number] | Yes | Max corner (x, y, z) in 0-16 space |
| `rotation` | Object | No | Element rotation |
| `shade` | Boolean | No | Enable face shading (default: true) |
| `faces` | Object | Yes | Face definitions |

**Face**:
| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `uv` | [Number, Number, Number, Number] | No | UV coordinates [u1, v1, u2, v2] |
| `texture` | String | Yes | Texture variable reference (e.g., `#particle`, `#side`) |
| `cullface` | String | No | Face to cull against: "up", "down", "north", "south", "east", "west" |
| `rotation` | Integer | No | Texture rotation (0, 90, 180, 270) |
| `tintindex` | Integer | No | Tint index for biome coloring |

**Invariants**:
- `from` coordinates ≤ `to` coordinates
- All coordinates in range [0, 16]
- UV coordinates in range [0, 16] (texture atlas space)
- Texture variables must be defined in `textures` or inherited
- Parent chain must not contain cycles
- If `elements` present, `parent` should not define conflicting geometry

### ItemModel

**Location**: `assets/minecraft/models/item/*.json`

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `parent` | String | No | Parent model path |
| `textures` | Object | No | Texture variable assignments |
| `display` | Object | No | Display transforms |
| `overrides` | Array | No | Conditional model overrides |

**Display Transform** (per perspective):
| Perspective | Type | Description |
|-------------|------|-------------|
| `firstperson_righthand` | Object | Right hand first-person |
| `firstperson_lefthand` | Object | Left hand first-person |
| `thirdperson_righthand` | Object | Right hand third-person |
| `thirdperson_lefthand` | Object | Left hand third-person |
| `gui` | Object | Inventory/GUI |
| `head` | Object | Worn on head |
| `ground` | Object | Dropped on ground |
| `fixed` | Object | Item frame |

**Transform Object**:
| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `rotation` | [Number, Number, Number] | No | X, Y, Z rotation |
| `translation` | [Number, Number, Number] | No | X, Y, Z translation |
| `scale` | [Number, Number, Number] | No | X, Y, Z scale |

**Override Entry**:
| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `predicate` | Object | Yes | Custom model data conditions |
| `model` | String | Yes | Model to use when predicate matches |

**Invariants**:
- Display transform values follow Minecraft conventions
- Parent references valid
- Texture variables resolve to `textures/item/`
- Override predicates use valid custom model data keys

### Texture

**Location**: `assets/minecraft/textures/**/*.png` (+ optional `.mcmeta`)

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `width` | Integer | Implicit | Image width (power of 2) |
| `height` | Integer | Implicit | Image height (power of 2) |
| `animation` | Object | No | Animation metadata (from .mcmeta) |

**Animation Metadata** (`.mcmeta`):
| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `animation.frametime` | Integer | No | Ticks per frame (default: 1) |
| `animation.frames` | Array | No | Frame index sequence |
| `animation.interpolate` | Boolean | No | Interpolate between frames |

**Invariants**:
- Dimensions are powers of two (16, 32, 64, 128, 256...)
- PNG format
- If animated, frame count = height / width (vertical strip)
- Animation metadata valid JSON

### AtlasSource

**Location**: `assets/minecraft/atlases/blocks.json`

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `type` | String | Yes | Source type: "directory", "single", "paletted_permutations" |
| `source` | String | Yes | Path or namespace |
| `prefix` | String | No | Prefix for generated names |
| `filters` | Object | No | Filter conditions |

**Invariants**:
- All sources must exist under `textures/`
- Directory sources include all PNG files recursively
- No duplicate texture names across sources

### LanguageEntry

**Location**: `assets/minecraft/lang/<lang>.json`

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `key` | String | Yes | Translation key (e.g., `block.minecraft.oak_planks`) |
| `value` | String | Yes | Translated text |

**Invariants**:
- Keys must match vanilla translation keys exactly
- Values are free-form text (no format validation)
- No duplicate keys within a file

### Shader

**Location**: `assets/minecraft/shaders/core/*.glsl`

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `program` | String | Implicit | Shader program name (filename) |
| `vertex` | String | Yes | Vertex shader source |
| `fragment` | String | Yes | Fragment shader source |

**Invariants**:
- GLSL version compatible with client
- Referenced by client shader system

### SoundEvent

**Location**: `assets/minecraft/sounds/**/*.ogg`

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | String | Implicit | Sound event name (path relative to sounds/) |
| `file` | String | Implicit | OGG file path |

**Invariants**:
- OGG format
- Sample rate compatible with client
- Referenced by client sound system

## Relationships

```
PackMetadata (1) ──► (N) BlockState
PackMetadata (1) ──► (N) BlockModel
PackMetadata (1) ──► (N) ItemModel
PackMetadata (1) ──► (N) Texture
PackMetadata (1) ──► (1) AtlasConfig
PackMetadata (1) ──► (N) LanguageEntry
PackMetadata (1) ──► (N) Shader
PackMetadata (1) ──► (N) SoundEvent

BlockState ──► (references) BlockModel
BlockModel ──► (references) Texture (via texture variables)
ItemModel ──► (references) BlockModel OR Texture
AtlasConfig ──► (sources) Texture
```

## State Transitions

None — all data is static. The only "state" is the client's loaded asset registry, which is rebuilt on:
- Client startup
- Resource pack reload (F3+T)
- Pack enable/disable

## Invariants Summary

| Invariant | Scope | Enforcement |
|-----------|-------|-------------|
| `pack_format` in supported range | PackMetadata | Client loader |
| All blockstate model refs exist | BlockState → BlockModel | Client model baking |
| All model texture refs resolve | BlockModel/ItemModel → Texture | Client texture atlas |
| No parent cycles | BlockModel/ItemModel | Client model baking |
| Texture dimensions power of 2 | Texture | Client texture loading |
| Atlas capacity not exceeded | AtlasConfig → Texture | Client atlas building |
| Language keys match vanilla | LanguageEntry | Manual verification |
| JSON schema compliance | All JSON | Client parser |

## Versioning and Migration

- **Pack version**: In `pack.mcmeta` description (`NuciCraft v<version>`)
- **Format version**: `pack_format` (107.1 for Minecraft 1.21.x)
- **Migration**: None — replace entire pack on update
- **Backward compatibility**: `supported_formats` range allows older clients to load

## Extension Points

1. **New blocks/items**: Add blockstate + model + texture
2. **Texture variations**: Add to `textures/block/variated/` + model in `models/block/variated/`
3. **Custom items**: Add item model + texture + language entry
4. **New languages**: Add `lang/<code>.json`
5. **Shaders**: Add to `shaders/core/`
6. **Sounds**: Add to `sounds/`