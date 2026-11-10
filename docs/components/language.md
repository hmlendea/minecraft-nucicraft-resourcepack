# Language Component

Deep dive into language files: structure, conventions, and NuciCraft translation approach.

## Overview

Language files provide translations for block/item/entity names, UI text, and messages. Located at `assets/minecraft/lang/*.json`.

## File Structure

```
assets/minecraft/lang/
├── ro_ro.json          # Romanian (primary custom language)
├── en_us.json          # US English (minimal overrides)
├── en_gb.json          # UK English (minimal overrides)
├── en_au.json          # Australian English (minimal overrides)
├── en_ca.json          # Canadian English (minimal overrides)
├── en_nz.json          # New Zealand English (minimal overrides)
├── en_pt.json          # Pirate English (fun variant)
└── languages.json      # Language metadata
```

## languages.json

Defines available languages for the pack.

```json
[
  {
    "code": "ro_ro",
    "name": "Română (România)",
    "region": "RO",
    "bidirectional": false
  },
  {
    "code": "en_us",
    "name": "English (US)",
    "region": "US",
    "bidirectional": false
  }
  // ... other English variants
]
```

### Fields

| Field | Description |
|-------|-------------|
| `code` | Language code (matches filename) |
| `name` | Display name in language menu |
| `region` | Region code |
| `bidirectional` | RTL support (false for Latin scripts) |

## Translation File Structure

### Key Format

```
<category>.<namespace>.<key>
```

### Categories

| Category | Examples |
|----------|----------|
| `block` | `block.minecraft.cobblestone` |
| `item` | `item.minecraft.diamond_sword` |
| `entity` | `entity.minecraft.zombie` |
| `effect` | `effect.minecraft.speed` |
| `enchantment` | `enchantment.minecraft.sharpness` |
| `potion` | `potion.minecraft.healing` |
| `death` | `death.attack.magic` |
| `gui` | `gui.inventory`, `gui.advancements` |
| `commands` | `commands.give.success` |
| `chat` | `chat.type.text` |
| `stat` | `stat.mine_block.minecraft.cobblestone` |

### Romanian Example (ro_ro.json)

```json
{
  "block.minecraft.cobblestone": "Pietriș",
  "block.minecraft.stone": "Piatră",
  "block.minecraft.diamond_ore": "Minereu de Diamant",
  "block.minecraft.gold_ore": "Minereu de Aur",
  "block.minecraft.iron_ore": "Minereu de Fier",
  "block.minecraft.coal_ore": "Minereu de Cărbune",
  "block.minecraft.redstone_ore": "Minereu de Redstone",
  "block.minecraft.emerald_ore": "Minereu de Smarald",
  "block.minecraft.lapis_ore": "Minereu de Lapis Lazuli",
  "block.minecraft.nether_gold_ore": "Minereu de Aur al Nether-ului",
  "block.minecraft.deepslate_diamond_ore": "Minereu de Diamant din Deepslate",
  "item.minecraft.diamond_sword": "Sabie de Diamant",
  "item.minecraft.diamond_pickaxe": "Ciocan de Diamant",
  "item.minecraft.diamond_axe": "Topor de Diamant",
  "item.minecraft.diamond_shovel": "Lopată de Diamant",
  "item.minecraft.diamond_hoe": "Motoasă de Diamant",
  "item.minecraft.magic_compass": "Busolă Magică",
  "item.minecraft.musket": "Moschet",
  "entity.minecraft.creeper": "Creeper",
  "entity.minecraft.zombie": "Zombie",
  "entity.minecraft.skeleton": "Schelet",
  "entity.minecraft.spider": "Păianjen",
  "entity.minecraft.enderman": "Enderman",
  "death.attack.magic": "a fost distrus de magie",
  "death.attack.musket": "a fost împușcat",
  "gui.inventory": "Inventar",
  "gui.crafting": "Fabricare",
  "gui.advancements": "Realizări"
}
```

### English Override Example (en_us.json)

```json
{
  "item.minecraft.magic_compass": "Magic Compass",
  "item.minecraft.musket": "Musket",
  "death.attack.magic": "was destroyed by magic",
  "death.attack.musket": "was shot"
}
```

## NuciCraft Translation Strategy

### Primary Language: Romanian (ro_ro)

- **Complete coverage**: All vanilla keys overridden
- **Custom keys**: Magic compass, musket, custom death messages
- **Style**: Natural Romanian, not literal translation
- **Maintenance**: Updated when vanilla adds keys

### English Variants: Minimal Overrides

- Only custom items (magic compass, musket)
- Custom death messages
- Vanilla names preserved
- Reason: English is fallback; Romanian is feature

### Adding New Translations

1. **Identify key**: Use F3+H in-game for advanced tooltips, or check vanilla assets
2. **Add to ro_ro.json**: Full Romanian translation
3. **Add to en_us.json**: Only if custom item/feature
4. **Test**: Switch language in-game, verify display

## Key Discovery

### In-Game (F3+H)

Enable advanced tooltips → hover item/block → shows translation key.

### Vanilla Assets

Extract from `client.jar/assets/minecraft/lang/en_us.json`.

### Common Patterns

```
block.minecraft.<block_id>
item.minecraft.<item_id>
entity.minecraft.<entity_id>
effect.minecraft.<effect_id>
enchantment.minecraft.<enchantment_id>
potion.minecraft.<potion_id>
death.attack.<cause>
gui.<screen>.<element>
commands.<command>.<feedback>
stat.<type>.minecraft.<id>
```

## Special Keys

### Custom Items

```
item.minecraft.magic_compass
item.minecraft.musket
item.minecraft.musket_pulling_0
item.minecraft.musket_pulling_1
item.minecraft.musket_pulling_2
```

### Custom Death Messages

```
death.attack.magic
death.attack.musket
```

### Custom Blocks (if any)

```
block.minecraft.custom_block
```

## Validation Rules

1. **Valid JSON** — no trailing commas, quoted keys
2. **UTF-8 encoding** — supports Romanian diacritics (ă, â, î, ș, ț)
3. **Key format** — category.namespace.key
4. **No duplicate keys** — last wins
5. **languages.json matches** — each .json file listed

## Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Name not translated | Key missing / wrong | Add correct key to ro_ro.json |
| Shows key instead of name | languages.json missing entry | Add language to languages.json |
| Romanian not in menu | languages.json missing ro_ro | Add ro_ro entry |
| Diacritics broken | File not UTF-8 | Save as UTF-8 |
| English shows custom name | en_us.json has override | Remove if not desired |

## Maintenance

### When Vanilla Updates

1. Compare vanilla en_us.json with previous version
2. Identify new keys
3. Translate new keys for ro_ro.json
4. Add custom keys if new features

### Automation Opportunity

Script to:
- Fetch vanilla translation file
- Diff against ro_ro.json
- Report missing keys
- Auto-translate (with review)

## Related Documents

- [Configuration](../configuration.md#lang)
- [API Usage Examples](../api-usage-examples.md#language-examples)
- [Change Guide](../change-guide.md#modifying-existing-assets)
- [Dependencies](../dependencies.md#community-pack-dependencies)