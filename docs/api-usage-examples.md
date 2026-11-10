# API Usage Examples

This document provides practical examples for working with NuciCraft Resource Pack assets programmatically or via datapacks.

## Table of Contents

- [Blockstate Examples](#blockstate-examples)
- [Model Examples](#model-examples)
- [Texture Examples](#texture-examples)
- [Language Examples](#language-examples)
- [Custom Item Examples](#custom-item-examples)
- [Datapack Integration](#datapack-integration)
- [Shader Examples](#shader-examples)
- [Sound Examples](#sound-examples)

---

## Blockstate Examples

### Simple Variant Blockstate

```json
{
  "variants": {
    "": { "model": "minecraft:block/cobblestone" }
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

### Multipart Blockstate (Fence)

```json
{
  "multipart": [
    { "apply": { "model": "minecraft:block/oak_fence_post" } },
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

### State-Dependent Variants (Door)

```json
{
  "variants": {
    "facing=north,half=lower,hinge=left,open=false": { "model": "minecraft:block/oak_door_bottom_left" },
    "facing=north,half=lower,hinge=left,open=true": { "model": "minecraft:block/oak_door_bottom_left_open" },
    "facing=north,half=upper,hinge=left,open=false": { "model": "minecraft:block/oak_door_top_left" },
    "facing=north,half=upper,hinge=left,open=true": { "model": "minecraft:block/oak_door_top_left_open" }
  }
}
```

---

## Model Examples

### Basic Block Model (Cube)

```json
{
  "parent": "minecraft:block/block",
  "textures": {
    "particle": "minecraft:block/cobblestone",
    "all": "minecraft:block/cobblestone"
  },
  "elements": [
    {
      "from": [0, 0, 0],
      "to": [16, 16, 16],
      "faces": {
        "down":  { "uv": [0, 0, 16, 16], "texture": "#all", "cullface": "down" },
        "up":    { "uv": [0, 0, 16, 16], "texture": "#all", "cullface": "up" },
        "north": { "uv": [0, 0, 16, 16], "texture": "#all", "cullface": "north" },
        "south": { "uv": [0, 0, 16, 16], "texture": "#all", "cullface": "south" },
        "west":  { "uv": [0, 0, 16, 16], "texture": "#all", "cullface": "west" },
        "east":  { "uv": [0, 0, 16, 16], "texture": "#all", "cullface": "east" }
      }
    }
  ]
}
```

### Model with Texture Variables

```json
{
  "parent": "minecraft:block/block",
  "textures": {
    "particle": "minecraft:block/oak_log_top",
    "side": "minecraft:block/oak_log",
    "end": "minecraft:block/oak_log_top"
  },
  "elements": [
    {
      "from": [0, 0, 0],
      "to": [16, 16, 16],
      "faces": {
        "down":  { "uv": [0, 0, 16, 16], "texture": "#end", "cullface": "down" },
        "up":    { "uv": [0, 0, 16, 16], "texture": "#end", "cullface": "up" },
        "north": { "uv": [0, 0, 16, 16], "texture": "#side", "cullface": "north" },
        "south": { "uv": [0, 0, 16, 16], "texture": "#side", "cullface": "south" },
        "west":  { "uv": [0, 0, 16, 16], "texture": "#side", "cullface": "west" },
        "east":  { "uv": [0, 0, 16, 16], "texture": "#side", "cullface": "east" }
      }
    }
  ]
}
```

### Item Model with Display Transforms

```json
{
  "parent": "minecraft:item/handheld",
  "textures": {
    "layer0": "minecraft:item/diamond_sword"
  },
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
    }
  }
}
```

### Model with Override (Custom Model Data)

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

### Multi-Layer Item Model

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

---

## Texture Examples

### Animated Texture (.mcmeta)

```json
{
  "animation": {
    "frametime": 4,
    "frames": [0, 1, 2, 3, 4, 5],
    "interpolate": true
  }
}
```

**Requirements**:
- PNG is vertical strip (width x height x frames)
- Each frame = width x width
- Place `.mcmeta` alongside `.png`

---

## Language Examples

### Romanian Translation Entry

```json
{
  "block.minecraft.cobblestone": "Pietriș",
  "block.minecraft.diamond_ore": "Minereu de Diamant",
  "item.minecraft.diamond_sword": "Sabie de Diamant",
  "item.minecraft.magic_compass": "Busolă Magică",
  "item.minecraft.musket": "Moschet",
  "death.attack.magic": "a fost distrus de magie"
}
```

### English Override Entry

```json
{
  "block.minecraft.cobblestone": "Cobblestone",
  "item.minecraft.magic_compass": "Magic Compass",
  "item.minecraft.musket": "Musket"
}
```

### Language File Structure

```json
{
  "block.minecraft.<block_id>": "Display Name",
  "item.minecraft.<item_id>": "Display Name",
  "entity.minecraft.<entity_id>": "Display Name",
  "effect.minecraft.<effect_id>": "Display Name",
  "enchantment.minecraft.<enchantment_id>": "Display Name",
  "potion.minecraft.<potion_id>": "Display Name",
  "death.attack.<cause>": "Death message",
  "gui.<screen>.<element>": "UI text",
  "commands.<command>.<feedback>": "Command output"
}
```

---

## Custom Item Examples

### Magic Compass (Datapack Required)

**Datapack Function** (`data/namespace/functions/give_magic_compass.mcfunction`):
```mcfunction
give @p minecraft:compass{CustomModelData:1001,display:{Name:'{"text":"Magic Compass","color":"aqua"}'}} 1
```

**Model Override** (`assets/minecraft/models/item/compass.json`):
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

**Custom Model** (`assets/minecraft/models/item/magic_compass.json`):
```json
{
  "parent": "minecraft:item/compass",
  "textures": {
    "layer0": "minecraft:item/magic_compass"
  }
}
```

### Musket (Crossbow Replacement)

**Model** (`assets/minecraft/models/item/crossbow.json`):
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

---

## Datapack Integration

### Custom Model Data Items

```mcfunction
# Give item with custom model data
give @p minecraft:diamond{CustomModelData:1002} 1

# Detect in predicate
execute if entity @p[nbt={SelectedItem:{id:"minecraft:diamond",CustomModelData:1002}}] run say "Holding custom diamond"
```

### Loot Table with Custom Model Data

```json
{
  "type": "minecraft:block",
  "pools": [
    {
      "rolls": 1,
      "entries": [
        {
          "type": "minecraft:item",
          "name": "minecraft:diamond",
          "functions": [
            {
              "function": "minecraft:set_custom_model_data",
              "value": 1002
            }
          ]
        }
      ]
    }
  ]
}
```

### Advancement Trigger

```json
{
  "criteria": {
    "has_magic_compass": {
      "trigger": "minecraft:inventory_changed",
      "conditions": {
        "items": [
          {
            "items": ["minecraft:compass"],
            "nbt": "{CustomModelData:1001}"
          }
        ]
      }
    }
  }
}
```

---

## Shader Examples

### Basic Shader Include

```glsl
// shaders/core/basic.vsh
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

```glsl
// shaders/core/basic.fsh
#version 120
uniform sampler2D Texture;
varying vec2 texCoord;
void main() {
  gl_FragColor = texture2D(Texture, texCoord);
}
```

**Note**: Shaders in resource packs are experimental. Most users use Iris/OptiFine shader packs instead.

---

## Sound Examples

### Sound Event Definition

```json
// assets/minecraft/sounds.json
{
  "item.musket_shoot": {
    "sounds": [
      { "name": "nucicraft:item/musket_shoot", "volume": 1.0, "pitch": 1.0 }
    ]
  },
  "entity.player.retro_hurt": {
    "sounds": [
      { "name": "nucicraft:damage/retro_hurt", "volume": 0.8, "pitch": [0.9, 1.1] }
    ]
  }
}
```

### Sound File Requirements

- Format: OGG Vorbis
- Sample rate: 44100 Hz or 48000 Hz
- Mono or stereo
- Place in `assets/minecraft/sounds/` matching JSON path

### Playing via Command

```mcfunction
playsound nucicraft:item.musket_shoot master @a ~ ~ ~ 1 1
```

---

## Validation Scripts

### JSON Syntax Check

```bash
#!/bin/bash
# validate-json.sh
find assets -name "*.json" -exec jq empty {} \; -print
echo "All JSON files valid"
```

### Texture Dimension Check

```python
#!/usr/bin/env python3
# validate-textures.py
from PIL import Image
import os

def check_texture(path):
    try:
        with Image.open(path) as img:
            w, h = img.size
            if w != h or (w & (w - 1)) != 0:
                print(f"WARNING: {path} is {w}x{h} (not square power of two)")
                return False
    except Exception as e:
        print(f"ERROR: {path}: {e}")
        return False
    return True

for root, dirs, files in os.walk("assets/minecraft/textures"):
    for f in files:
        if f.endswith(".png"):
            check_texture(os.path.join(root, f))
```

### Reference Integrity Check (Conceptual)

```python
# Check all model references exist
# Check all blockstate model references exist
# Check all texture references exist
# Check all language keys valid
```

---

## Related Documents

- [Components: Blockstates](./components/blockstates.md)
- [Components: Block Models](./components/block-models.md)
- [Components: Item Models](./components/item-models.md)
- [Components: Textures](./components/textures.md)
- [Components: Language](./components/language.md)
- [Flows: Model Baking](./flows/model-baking.md)
- [Behaviour: Custom Item Behaviour](./behaviour/custom-item-behaviour.md)
- [Configuration](./configuration.md)