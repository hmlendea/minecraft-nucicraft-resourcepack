# Dependencies

This document catalogues all external and internal dependencies of the NuciCraft Resource Pack.

## External Dependencies (Community Packs)

All community pack assets are **vendored** (copied into the repository) under standard Minecraft asset paths. There are no runtime dependencies.

### Direct Vendored Dependencies

| Community Pack | Author | Version/Date | Assets Integrated | Path |
|----------------|--------|--------------|-------------------|------|
| VanillaTweaks | VanillaTweaks Team | Various | Texture tweaks, model variations | `textures/block/`, `models/block/` |
| Faithful 32x | Faithful Team | 32x | 32x item textures | `textures/item/` |
| MCjossic | MCjossic | Various | 3D amethyst, iron bars, dripstone | `models/block/`, `textures/block/` |
| Saplistic | Saplistic | Various | 3D chains | `models/block/`, `textures/block/` |
| Ecorn | Ecorn | v2.3.1 | 3D doors, trapdoors | `models/block/`, `textures/block/` |
| Ivango | Ivango | Various | 3D buckets | `models/block/`, `textures/block/` |
| JMHMirror | JMHMirror | Various | 3D goat horn | `models/item/`, `textures/item/` |
| D_Dimensional | D_Dimensional | v1.3.4 | 3D torches, lanterns | `models/block/`, `textures/block/` |
| ZzartySt | ZzartySt | Various | 3D armor trims | `textures/trims/`, `models/item/` |
| PiXeLGaMeS_or_not | PiXeLGaMeS_or_not | Various | Musket (crossbow replacement) | `models/item/`, `textures/item/` |
| Ensis | Ensis | Various | 3D beetroots, carrots | `models/block/`, `textures/block/` |
| Vexcenot | Vexcenot | Various | 3D crops (melon, potato, pumpkin, sweet berry) | `models/block/`, `textures/block/` |
| Hecco | Hecco | Various | 3D flowers | `models/block/`, `textures/block/` |
| Relidium | Relidium | v2 | Variated obsidian | `textures/block/variated/`, `models/block/variated/` |
| Infina | Infina | Various | Variated bricks | `textures/block/variated/`, `models/block/variated/` |
| Oslypsis | Oslypsis | Various | Colorful cobblestone | `textures/block/variated/`, `models/block/variated/` |
| Amethyst Geode | Amethyst Geode | Various | Glowstone variations | `textures/block/variated/`, `models/block/variated/` |
| PatchWolf | PatchWolf | v1.1 | Improved prismarine bricks | `textures/block/`, `models/block/` |
| Mexorax | Mexorax | Various | Animated candles | `textures/block/`, `models/block/` |
| PixalThePixeled | PixalThePixeled | Various | Animated crying obsidian | `textures/block/`, `models/block/` |
| DudeAbide | DudeAbide | Various | 3D redstone dust | `models/block/`, `textures/block/` |
| Luracasmus | Luracasmus | v1.5.1 | Optimised models | `models/block/` |

### Dependency Management

- **Update mechanism**: Manual re-vendoring
- **Frequency**: As needed (typically with Minecraft version updates)
- **Process**:
  1. Download latest community pack release
  2. Extract relevant assets
  3. Copy to appropriate paths under `assets/minecraft/`
  4. Resolve conflicts (NuciCraft originals take priority)
  5. Test in-game
  6. Commit with attribution in README.md

### Attribution

All community pack authors credited in [README.md](../README.md#attributions). Licenses vary by pack; most are permissive (MIT, CC-BY, custom permissive).

## Platform Dependencies

| Dependency | Version | Type | Purpose |
|------------|---------|------|---------|
| Minecraft Java Edition | 1.21.x | Runtime host | Loads and interprets resource pack |
| Java Runtime | 21+ | Client requirement | Runs Minecraft client |

### Pack Format Compatibility

| Minecraft Version | Pack Format | Supported |
|-------------------|-------------|-----------|
| 1.21.x | 107.1 | ✅ Target |
| 1.20.5 - 1.20.6 | 48 | ✅ Via `supported_formats` |
| 1.20.3 - 1.20.4 | 41 | ✅ Via `supported_formats` |
| 1.20 - 1.20.2 | 15 | ✅ Via `supported_formats` |
| 1.19.3 - 1.19.4 | 12 | ✅ Via `supported_formats` |
| 1.18 - 1.19.2 | 8-9 | ✅ Via `supported_formats` |
| < 1.18 | < 8 | ❌ Below `min_inclusive` |

## Internal Dependencies (Asset Graph)

### Dependency Direction

```
pack.mcmeta
    │
    ▼
assets/minecraft/
    │
    ├── blockstates/ ──────┐
    │                      │ (references)
    ├── models/block/ ◄────┤
    │                      │
    ├── models/item/ ◄─────┤
    │                      │
    ├── textures/ ─────────┘ (referenced by models)
    │
    ├── atlases/ ──────────► textures/ (sources)
    │
    └── lang/ (independent)
```

### Critical Path Dependencies

| Consumer | Provider | Failure Mode |
|----------|----------|--------------|
| `blockstates/*.json` | `models/block/*.json` | Purple/black checkerboard |
| `models/block/*.json` | `textures/block/*.png` | Magenta texture |
| `models/item/*.json` | `models/block/*.json` OR `textures/item/*.png` | Missing model/texture |
| `atlases/blocks.json` | `textures/**/*.png` | Missing atlas entries |
| `lang/*.json` | (vanilla keys) | Fallback to vanilla text |

### Dependency Validation

No automated validation currently. Manual verification required:
- All blockstate model references exist
- All model texture references resolve
- All atlas sources exist
- No circular model parent chains

## Build/Release Dependencies

| Tool | Version | Purpose |
|------|---------|---------|
| Git | Any | Version control |
| zip | Any | Package creation |
| sha1sum | Any | Checksum computation |
| GitHub Actions | ubuntu-latest | CI/CD runner |
| actions/checkout | v4 | Repository checkout |
| actions/upload-release-asset | v1 | Release asset upload |

## Development Dependencies

None required for development — only a text editor and Minecraft client for testing.

### Recommended Tools

| Tool | Purpose |
|------|---------|
| VS Code + JSON extension | Edit JSON models/blockstates |
| Blockbench | Visual 3D model editing |
| GIMP / Photoshop | Texture editing |
| Minecraft 1.21.x | Testing |

## Dependency Risk Assessment

| Dependency | Risk | Mitigation |
|------------|------|------------|
| Minecraft version upgrade | Breaking pack_format changes | Test on snapshots, update pack_format |
| Community pack updates | Asset conflicts, license changes | Pin versions, review before re-vendoring |
| Texture atlas overflow | Too many unique textures | Monitor texture count, consolidate variations |
| Model complexity | Render performance degradation | Vet community models, limit poly count |
| Vanilla translation key changes | Custom names revert to vanilla | Track vanilla changes, update lang files |

## Dependency Update Procedure

### For Community Pack Updates

1. Identify updated pack
2. Download release
3. Compare assets with current vendored version
4. Copy new/changed assets to `assets/minecraft/`
5. Resolve conflicts (keep NuciCraft customisations)
6. Update attribution in README.md if author/license changed
7. Test thoroughly in-game
8. Commit with message: `chore: update <pack-name> to <version>`

### For Minecraft Version Upgrade

1. Wait for stable release
2. Check pack_format for new version
3. Update `pack.mcmeta`:
   - `pack_format` to new version
   - `supported_formats.max_inclusive` to 999 (or new max)
4. Check for asset schema changes (blockstates, models, atlases)
5. Update any changed vanilla assets
6. Test on new version
7. Release as new minor version

## License Compatibility

| Pack | License | Compatible with GPL v3.0? |
|------|---------|---------------------------|
| VanillaTweaks | Custom permissive | Yes |
| Faithful 32x | Custom (non-commercial) | Verify |
| MCjossic | MIT | Yes |
| Saplistic | MIT | Yes |
| Ecorn | Custom permissive | Yes |
| Ivango | MIT | Yes |
| JMHMirror | MIT | Yes |
| D_Dimensional | MIT | Yes |
| ZzartySt | MIT | Yes |
| PiXeLGaMeS_or_not | Custom permissive | Yes |
| Ensis | MIT | Yes |
| Vexcenot | MIT | Yes |
| Hecco | MIT | Yes |
| Relidium | MIT | Yes |
| Infina | MIT | Yes |
| Oslypsis | MIT | Yes |
| Amethyst Geode | MIT | Yes |
| PatchWolf | MIT | Yes |
| Mexorax | MIT | Yes |
| PixalThePixeled | MIT | Yes |
| DudeAbide | MIT | Yes |
| Luracasmus | MIT | Yes |

**Note**: Faithful 32x has non-commercial restriction. NuciCraft is non-commercial (free distribution), so compatible. If commercial use ever considered, Faithful 32x textures must be replaced.

## Dependency Visualisation

```
┌─────────────────────────────────────────────────────────────┐
│                    NUCICRAFT RESOURCE PACK                  │
├─────────────────────────────────────────────────────────────┤
│  pack.mcmeta                                                │
│       │                                                     │
│       ▼                                                     │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              assets/minecraft/                       │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────┐  │   │
│  │  │blockstates│ │ models/  │ │ models/  │ │textures│  │   │
│  │  │          │ │ block/   │ │ item/    │ │        │  │   │
│  │  └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬───┘  │   │
│  │       │            │            │            │       │   │
│  │       └────────────┴────────────┴────────────┘       │   │
│  │                    │                                   │   │
│  │  ┌──────────┐ ┌────┴────┐ ┌──────────┐ ┌──────────┐  │   │
│  │  │ atlases/ │ │ lang/   │ │ shaders/ │ │ sounds/  │  │   │
│  │  └──────────┘ └─────────┘ └──────────┘ └──────────┘  │   │
│  └─────────────────────────────────────────────────────┘   │
│       │                                                     │
│       ▼                                                     │
│  ┌─────────────────────────────────────────────────────┐   │
│  │           VENDORED COMMUNITY ASSETS                  │   │
│  │  VanillaTweaks, Faithful 32x, MCjossic, Saplistic,  │   │
│  │  Ecorn, Ivango, JMHMirror, D_Dimensional, ZzartySt, │   │
│  │  PiXeLGaMeS_or_not, Ensis, Vexcenot, Hecco,         │   │
│  │  Relidium, Infina, Oslypsis, Amethyst Geode,        │   │
│  │  PatchWolf, Mexorax, PixalThePixeled, DudeAbide,    │   │
│  │  Luracasmus                                          │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│              MINECRAFT JAVA EDITION CLIENT                  │
│  Resource Pack Loader → Asset Manager → Rendering Engine   │
└─────────────────────────────────────────────────────────────┘
```