# Custom Item Behaviour

Specification for custom items: magic compass, musket, and extensibility.

## Overview

NuciCraft includes two custom items demonstrating advanced resource pack capabilities:

1. **Magic Compass** — Compass with CustomModelData override
2. **Musket** — Crossbow replacement with custom model and sounds

Both require datapack support for full functionality.

## Magic Compass

### Concept

A compass variant that displays a custom model when `CustomModelData: 1001` is set.

### Asset Chain

```
Datapack (give command)
    → ItemStack with CustomModelData: 1001
    → compass.json override predicate matches
    → magic_compass.json model loaded
    → magic_compass.png texture rendered
```

### Resource Pack Assets

#### Base Model Override (`models/item/compass.json`)

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

#### Custom Model (`models/item/magic_compass.json`)

```json
{
  "parent": "minecraft:item/compass",
  "textures": {
    "layer0": "minecraft:item/magic_compass"
  }
}
```

#### Texture (`textures/item/magic_compass.png`)

Custom 16×16 or 32×32 texture.

### Datapack Integration

#### Give Command

```mcfunction
give @p minecraft:compass{CustomModelData:1001,display:{Name:'{"text":"Magic Compass","color":"aqua"}'}} 1
```

#### Loot Table

```json
{
  "type": "minecraft:block",
  "pools": [{
    "rolls": 1,
    "entries": [{
      "type": "minecraft:item",
      "name": "minecraft:compass",
      "functions": [{
        "function": "minecraft:set_custom_model_data",
        "value": 1001
      }]
    }]
  }]
}
```

#### Advancement Trigger

```json
{
  "criteria": {
    "has_magic_compass": {
      "trigger": "minecraft:inventory_changed",
      "conditions": {
        "items": [{
          "items": ["minecraft:compass"],
          "nbt": "{CustomModelData:1001}"
        }]
      }
    }
  }
}
```

### Behaviour

- **Compass needle**: Still points to spawn (vanilla behaviour preserved)
- **Enchantment glint**: Shows if enchanted (layer1 auto-applied)
- **Durability**: Not applicable (compass has no durability)
- **Stacking**: Stacks with normal compasses (different NBT)

### CustomModelData Values

| Value | Item | Description |
|-------|------|-------------|
| 1001 | Magic Compass | Custom compass model |
| 1002+ | Reserved | Future custom items |

---

## Musket

### Concept

Crossbow replacement with musket model, textures, and sounds.

### Asset Chain

```
Crossbow item (vanilla)
    → crossbow.json model override
    → musket.json model loaded
    → musket.png texture rendered
    → musket sounds played
```

### Resource Pack Assets

#### Base Model Override (`models/item/crossbow.json`)

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

#### Pulling Frames

```json
// models/item/musket_pulling_0.json
{ "parent": "minecraft:item/crossbow", "textures": { "layer0": "minecraft:item/musket_pulling_0" } }

// models/item/musket_pulling_1.json
{ "parent": "minecraft:item/crossbow", "textures": { "layer0": "minecraft:item/musket_pulling_1" } }

// models/item/musket_pulling_2.json
{ "parent": "minecraft:item/crossbow", "textures": { "layer0": "minecraft:item/musket_pulling_2" } }
```

#### Textures

- `textures/item/musket.png` — base
- `textures/item/musket_pulling_0.png` — early pull
- `textures/item/musket_pulling_1.png` — mid pull
- `textures/item/musket_pulling_2.png` — full pull

#### Sounds (`sounds.json`)

```json
{
  "item.musket_shoot": {
    "sounds": [{ "name": "nucicraft:item/musket_shoot", "volume": 1.0, "pitch": 1.0 }]
  }
}
```

Sound files: `sounds/item/musket_shoot.ogg`

### Behaviour

- **Functionally**: Identical to crossbow
- **Charging**: Right-click to charge (pulling animation)
- **Fireworks**: Can shoot firework rockets
- **Enchantments**: Multishot, Quick Charge, Piercing work
- **Durability**: Uses crossbow durability
- **Repair**: Anvil with crossbow
- **Sounds**: Custom shoot sound (retro style)

### Limitations

- **Cannot add new item type** — replaces crossbow
- **Requires crossbow** — no standalone item
- **Datapack not required** — works with vanilla crossbow

---

## Extensibility Pattern

### Adding New Custom Items

#### Option 1: CustomModelData Override (Like Magic Compass)

**Best for**: Variants of existing items (compass, clock, shield, etc.)

**Steps**:
1. Choose base item with override support
2. Allocate CustomModelData value (1002+)
3. Add override to base model
4. Create custom model + texture
5. Datapack to give/set NBT

#### Option 2: Full Replacement (Like Musket)

**Best for**: Complete visual overhaul (crossbow, bow, trident)

**Steps**:
1. Choose base item to replace
2. Override base model entirely
3. Create model + pulling frames if animated
4. Create textures
5. Add custom sounds if desired

#### Option 3: New Item via Datapack + Resource Pack

**Best for**: Truly new items (requires datapack)

**Steps**:
1. Datapack: Custom item with `custom_model_data`
2. Resource pack: Override on base item (e.g., diamond)
3. Model + texture for custom variant

### CustomModelData Allocation

| Range | Purpose |
|-------|---------|
| 1001 | Magic Compass |
| 1002-1999 | Custom item variants |
| 2000-2999 | Seasonal/event items |
| 3000+ | Reserved |

### Base Items Supporting Overrides

| Item | Override Types |
|------|----------------|
| Compass | CustomModelData |
| Clock | CustomModelData |
| Shield | CustomModelData, blocking |
| Crossbow | CustomModelData, pulling |
| Bow | CustomModelData, pulling |
| Trident | CustomModelData |
| Fishing Rod | CustomModelData, cast |
| Carrot on Stick | CustomModelData |
| Warped Fungus on Stick | CustomModelData |
| Brush | CustomModelData |

---

## Datapack-Resource Pack Contract

### Resource Pack Provides

- Model overrides
- Custom models
- Textures
- Sounds
- Language entries

### Datapack Provides

- Item acquisition (loot, crafting, commands)
- CustomModelData assignment
- Custom behaviour (if needed)
- Advancements/statistics

### Version Compatibility

- Resource pack version ≠ datapack version
- Coordinate CustomModelData values
- Document in both READMEs

---

## Testing

### Magic Compass

```mcfunction
# Give
give @p minecraft:compass{CustomModelData:1001} 1

# Verify model
# Hold in hand → should show magic compass texture
# Check GUI → should show magic compass texture
# Drop on ground → should show magic compass texture
```

### Musket

```mcfunction
# Give crossbow (becomes musket)
give @p minecraft:crossbow 1

# Test charging
# Right-click hold → pulling animation frames
# Release → shoot sound + projectile
# Load firework → shoot firework
```

### Automated Test (Conceptual)

```python
# Verify asset chain
assert exists("models/item/compass.json")
assert override_exists("compass.json", "custom_model_data", 1001)
assert exists("models/item/magic_compass.json")
assert exists("textures/item/magic_compass.png")
```

---

## Related Documents

- [Components: Item Models](../components/item-models.md)
- [Components: Textures](../components/textures.md)
- [Components: Language](../components/language.md)
- [Flows: Model Baking](../flows/model-baking.md)
- [API Usage Examples](../api-usage-examples.md#custom-item-examples)
- [Change Guide](../change-guide.md#adding-a-new-item)