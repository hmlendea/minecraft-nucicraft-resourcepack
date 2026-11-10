# API Reference Index

Complete reference for all NuciCraft Resource Pack asset schemas and interfaces.

## Asset Types

| Asset Type | Directory | Schema Reference |
|------------|-----------|------------------|
| Pack Metadata | Root | [pack.mcmeta](#packmetameta) |
| Blockstates | `assets/minecraft/blockstates/` | [Blockstate Schema](#blockstate-schema) |
| Block Models | `assets/minecraft/models/block/` | [Block Model Schema](#block-model-schema) |
| Item Models | `assets/minecraft/models/item/` | [Item Model Schema](#item-model-schema) |
| Textures | `assets/minecraft/textures/` | [Texture Specification](#texture-specification) |
| Atlases | `assets/minecraft/atlases/` | [Atlas Schema](#atlas-schema) |
| Languages | `assets/minecraft/lang/` | [Language Schema](#language-schema) |
| Shaders | `assets/minecraft/shaders/` | [Shader Specification](#shader-specification) |
| Sounds | `assets/minecraft/sounds/` | [Sound Specification](#sound-specification) |

---

## pack.mcmeta

```json
{
  "pack": {
    "pack_format": 107.1,
    "supported_formats": {
      "min_inclusive": 84,
      "max_inclusive": 999
    },
    "description": "NuciCraft v6.3.5"
  }
}
```

### Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `pack_format` | number | Yes | Target pack format (107.1 = 1.21.x) |
| `supported_formats.min_inclusive` | int | Yes | Minimum compatible format |
| `supported_formats.max_inclusive` | int | Yes | Maximum compatible format |
| `description` | string | Yes | Display name in resource pack menu |

### Pack Format Versions

| Minecraft Version | Pack Format |
|-------------------|-------------|
| 1.21.x | 107.1 |
| 1.20.5-1.20.6 | 48 |
| 1.20.3-1.20.4 | 41 |
| 1.20-1.20.2 | 15 |
| 1.19.3-1.19.4 | 12 |
| 1.19-1.19.2 | 9 |
| 1.18-1.18.2 | 8 |

---

## Blockstate Schema

### Variants Format

```json
{
  "variants": {
    "<state_key>": { "model": "<model_path>", "x": 0, "y": 0, "uvlock": false },
    "": [ { "model": "<model_path>", "weight": 1 }, ... ]
  }
}
```

### Multipart Format

```json
{
  "multipart": [
    { "apply": { "model": "<model_path>", "x": 0, "y": 0 } },
    { "when": { "<property>": "<value>" }, "apply": { "model": "<model_path>", "x": 0, "y": 0 } }
  ]
}
```

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `variants` | object | State key → model mapping |
| `multipart` | array | Conditional model composition |
| `model` | string | Model resource location |
| `x`, `y` | int | Rotation (0, 90, 180, 270) |
| `uvlock` | bool | Lock UV to face |
| `weight` | int | Selection weight for random variant |
| `when` | object | Property conditions |

---

## Block Model Schema

```json
{
  "parent": "<parent_model>",
  "ambientocclusion": true,
  "textures": {
    "particle": "<texture>",
    "<variable>": "<texture>"
  },
  "elements": [
    {
      "from": [x, y, z],
      "to": [x, y, z],
      "rotation": { "origin": [x, y, z], "axis": "x|y|z", "angle": 0, "rescale": false },
      "shade": true,
      "faces": {
        "down|up|north|south|west|east": {
          "uv": [u1, v1, u2, v2],
          "texture": "#<variable>",
          "cullface": "down|up|north|south|west|east",
          "tintindex": -1,
          "rotation": 0
        }
      }
    }
  ],
  "display": { "<view>": { "rotation": [], "translation": [], "scale": [] } },
  "gui_light": "front|side"
}
```

### Coordinate System

- Range: 0-16 (1 block = 16 units)
- Origin: (0,0,0) = min corner
- Axes: X=east, Y=up, Z=south

### Common Parents

| Parent | Description |
|--------|-------------|
| `minecraft:block/block` | Full cube |
| `minecraft:block/cube_column` | Column (logs) |
| `minecraft:block/cube_bottom_top` | Distinct top/bottom |
| `minecraft:block/cross` | Cross (flowers) |
| `minecraft:block/template_fence_post` | Fence post |
| `minecraft:block/template_fence_side` | Fence side |

---

## Item Model Schema

```json
{
  "parent": "<parent_model>",
  "textures": {
    "layer0": "<texture>",
    "layer1": "<texture>",
    "layer2": "<texture>"
  },
  "display": {
    "thirdperson_righthand": { "rotation": [], "translation": [], "scale": [] },
    "firstperson_righthand": { "rotation": [], "translation": [], "scale": [] },
    "gui": { "rotation": [], "translation": [], "scale": [] },
    "ground": { "rotation": [], "translation": [], "scale": [] },
    "fixed": { "rotation": [], "translation": [], "scale": [] }
  },
  "overrides": [
    { "predicate": { "custom_model_data": 1001 }, "model": "<model_path>" }
  ]
}
```

### Common Parents

| Parent | Use Case |
|--------|----------|
| `minecraft:item/generated` | 2D flat items |
| `minecraft:item/handheld` | 3D tools/weapons |
| `minecraft:item/compass` | Compass animation |
| `minecraft:item/crossbow` | Crossbow pulling |
| `minecraft:item/bow` | Bow pulling |

### Predicate Fields

| Field | Type | Description |
|-------|------|-------------|
| `custom_model_data` | int | CustomModelData NBT |
| `pulling` | 0/1 | Is pulling |
| `pull` | float | Pull progress (0-1) |
| `blocking` | 0/1 | Is blocking |
| `damaged` | 0/1 | Is damaged |
| `cooldown` | float | Cooldown progress |

---

## Texture Specification

### Requirements

| Property | Requirement |
|----------|-------------|
| Format | PNG |
| Dimensions | Square, power of two (16, 32, 64, 128, 256, 512, 1024) |
| Color Space | sRGB |
| Bit Depth | 8-bit per channel (RGBA) |

### Animation (.mcmeta)

```json
{
  "animation": {
    "frametime": 4,
    "frames": [0, 1, 2, 3],
    "interpolate": true
  }
}
```

### PNG Layout for Animation

Vertical strip: height = width × frame_count

---

## Atlas Schema

```json
{
  "sources": [
    {
      "type": "directory|single",
      "source": "namespace:path",
      "prefix": "optional/prefix/",
      "filter": "regex"
    }
  ]
}
```

### Source Types

| Type | Description |
|------|-------------|
| `directory` | Recursive scan for PNGs |
| `single` | Single texture file |

---

## Language Schema

### languages.json

```json
[
  { "code": "ro_ro", "name": "Română (România)", "region": "RO", "bidirectional": false }
]
```

### Translation Files

```json
{
  "block.minecraft.<id>": "Display Name",
  "item.minecraft.<id>": "Display Name",
  "entity.minecraft.<id>": "Display Name",
  "death.attack.<cause>": "Death message",
  "gui.<screen>.<element>": "UI text"
}
```

### Key Categories

| Prefix | Category |
|--------|----------|
| `block.` | Blocks |
| `item.` | Items |
| `entity.` | Entities |
| `effect.` | Status effects |
| `enchantment.` | Enchantments |
| `potion.` | Potions |
| `death.attack.` | Death messages |
| `gui.` | UI text |
| `commands.` | Command feedback |
| `stat.` | Statistics |

---

## Shader Specification

### Vertex Shader (.vsh)

```glsl
#version 120
uniform mat4 ModelView;
uniform mat4 Projection;
attribute vec4 Position;
attribute vec2 TexCoord;
varying vec2 texCoord;
void main() {
  texCoord = TexCoord;
  gl_Position = Projection * ModelView * Position;
}
```

### Fragment Shader (.fsh)

```glsl
#version 120
uniform sampler2D Texture;
varying vec2 texCoord;
void main() {
  gl_FragColor = texture2D(Texture, texCoord);
}
```

### Location

`assets/minecraft/shaders/core/` — experimental, not actively used.

---

## Sound Specification

### sounds.json

```json
{
  "event.name": {
    "sounds": [
      { "name": "namespace:path", "volume": 1.0, "pitch": 1.0 }
    ]
  }
}
```

### Sound File Requirements

| Property | Requirement |
|----------|-------------|
| Format | OGG Vorbis |
| Sample Rate | 44100 Hz or 48000 Hz |
| Channels | Mono or stereo |
| Location | `assets/minecraft/sounds/` matching JSON path |

### Custom Events (NuciCraft)

| Event | Description |
|-------|-------------|
| `item.musket_shoot` | Musket fire sound |
| `entity.player.retro_hurt` | Retro damage sound |

---

## Cross-References

| Document | Description |
|----------|-------------|
| [Components: Blockstates](../components/blockstates.md) | Blockstate deep dive |
| [Components: Block Models](../components/block-models.md) | Block model deep dive |
| [Components: Item Models](../components/item-models.md) | Item model deep dive |
| [Components: Textures](../components/textures.md) | Texture deep dive |
| [Components: Atlases](../components/atlases.md) | Atlas deep dive |
| [Components: Language](../components/language.md) | Language deep dive |
| [Flows: Pack Load](../flows/pack-load.md) | Pack loading flow |
| [Flows: Model Baking](../flows/model-baking.md) | Model baking flow |
| [Flows: Render Frame](../flows/render-frame.md) | Render frame flow |
| [Behaviour: Variation Rendering](../behaviour/variation-rendering.md) | Variation behaviour |
| [Behaviour: Custom Item Behaviour](../behaviour/custom-item-behaviour.md) | Custom item behaviour |
| [Configuration](../configuration.md) | Configuration reference |
| [API Usage Examples](../api-usage-examples.md) | Practical examples |