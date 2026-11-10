# Quick Start Guide

Get up and running with NuciCraft Resource Pack in 5 minutes.

## For Players

### 1. Download

Go to [GitHub Releases](https://github.com/your-repo/nucicraft-resourcepack/releases) and download the latest `nucicraft_vX.Y.Z.zip`.

### 2. Install

1. Open Minecraft
2. Options → Resource Packs → **Open Pack Folder**
3. Copy the downloaded ZIP into that folder
4. Close folder, return to Resource Packs screen
5. Click the pack to move it to **Selected** (right side)
6. Move to **top of list** for priority
7. Click **Done**

### 3. Enable Romanian Names (Optional)

1. Options → Language
2. Select **Română (România)**
3. Restart Minecraft (required)

### 4. Play

Join any server or singleplayer world. The pack works client-side automatically.

---

## For Server Admins

### Server-Prompted Installation

Add to `server.properties`:

```properties
resource-pack=https://github.com/your-repo/nucicraft-resourcepack/releases/download/v6.3.5/nucicraft_v6.3.5.zip
resource-pack-sha1=abcdef1234567890abcdef1234567890abcdef12
resource-pack-prompt="NuciCraft Resource Pack - 3D models, custom names, and more"
resource-pack-enforce=false
```

**Get SHA1**:
```bash
sha1sum nucicraft_v6.3.5.zip
```

Restart server. Players prompted on join.

---

## For Developers

### 1. Clone Repository

```bash
git clone https://github.com/your-repo/nucicraft-resourcepack.git
cd nucicraft-resourcepack
```

### 2. Explore Structure

```
assets/minecraft/
├── blockstates/     # Block state → model mappings
├── models/
│   ├── block/       # 3D block models
│   └── item/        # Item models + display transforms
├── textures/
│   ├── block/       # Block textures (PNG + .mcmeta)
│   └── item/        # Item textures
├── lang/            # Translations (ro_ro.json primary)
├── atlases/         # Texture atlas sources
├── shaders/         # Shader files (experimental)
└── sounds/          # Custom sounds (OGG)
```

### 3. Make a Change

Example: Add a texture variation for cobblestone

1. Create variation texture: `assets/minecraft/textures/block/variated/cobblestone_var1.png`
2. Create model: `assets/minecraft/models/block/cobblestone_variated_1.json`
3. Update blockstate: `assets/minecraft/blockstates/cobblestone.json` — add weighted variant
4. Test: In Minecraft, press **F3+T** to reload resource packs

### 4. Validate

```bash
# Check JSON syntax
find assets -name "*.json" -exec jq empty {} \;

# Check texture dimensions (requires Python + Pillow)
python3 -c "
from PIL import Image
import os
for root, dirs, files in os.walk('assets/minecraft/textures'):
    for f in files:
        if f.endswith('.png'):
            p = os.path.join(root, f)
            with Image.open(p) as img:
                w, h = img.size
                if w != h or (w & (w - 1)) != 0:
                    print(f'Non-power-of-two: {p} = {w}x{h}')
"
```

### 5. Submit

1. Commit changes with descriptive message
2. Open Pull Request
3. CI runs basic validation
4. Maintainer reviews and merges

---

## Key Files to Know

| File | Purpose |
|------|---------|
| `pack.mcmeta` | Pack metadata, format version, description |
| `assets/minecraft/blockstates/*.json` | Block state → model mapping |
| `assets/minecraft/models/block/*.json` | 3D block geometry |
| `assets/minecraft/models/item/*.json` | Item models + display |
| `assets/minecraft/textures/block/*.png` | Block textures |
| `assets/minecraft/lang/ro_ro.json` | Romanian translations |
| `assets/minecraft/atlases/blocks.json` | Texture atlas sources |
| `.github/workflows/release-asset.yml` | Release automation |

---

## Common Tasks

| Task | Files to Edit |
|------|---------------|
| Add 3D model for block | `models/block/`, `blockstates/` |
| Add texture variation | `textures/block/variated/`, `models/block/`, `blockstates/` |
| Rename block/item (Romanian) | `lang/ro_ro.json` |
| Rename block/item (English) | `lang/en_us.json` |
| Add custom item | `models/item/`, `models/item/` (override), datapack |
| Add sound | `sounds/`, `sounds.json` |
| Update pack version | `pack.mcmeta`, tag release |

---

## Testing Checklist

Before submitting PR:

- [ ] JSON syntax valid (`jq empty`)
- [ ] Textures are square power-of-two (16, 32, 64, 128...)
- [ ] All model references exist
- [ ] All texture references exist
- [ ] Blockstate variants reference valid models
- [ ] Language keys follow convention (`block.minecraft.*`, `item.minecraft.*`)
- [ ] Tested in-game (F3+T reload)
- [ ] No purple/black (missing model) or magenta (missing texture)

---

## Useful Commands

```bash
# Reload resource packs in-game
# Press F3+T

# View client logs
# ~/.minecraft/logs/latest.log

# Search for blockstate
grep -r "cobblestone" assets/minecraft/blockstates/

# Find model parents
grep -r '"parent"' assets/minecraft/models/block/ | head -20

# Count assets
find assets -name "*.json" | wc -l
find assets -name "*.png" | wc -l
```

---

## Next Steps

- Read [Change Guide](./change-guide.md) for detailed procedures
- Read [Components](./components/) for asset type deep-dives
- Read [Architecture](./architecture.md) for system overview
- Browse [API Usage Examples](./api-usage-examples.md) for patterns

---

## Support

- **Issues**: GitHub Issues
- **Questions**: GitHub Discussions
- **Documentation**: This `docs/` directory