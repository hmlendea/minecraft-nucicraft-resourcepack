# Repository Structure

## Source Tree Layout

```
nucicraft-resourcepack/
├── .github/
│   └── workflows/
│       └── release-asset.yml          # Release packaging and publishing
├── assets/
│   └── minecraft/
│       ├── atlases/
│       │   └── blocks.json            # Texture atlas configuration
│       ├── blockstates/               # Block state to model mappings
│       │   ├── *.json                 # One file per block with variants
│       ├── items/                     # (Empty - legacy, models under models/item/)
│       ├── lang/                      # Translation overrides
│       │   ├── en_au.json
│       │   ├── en_ca.json
│       │   ├── en_gb.json
│       │   ├── en_nz.json
│       │   ├── en_ud.json
│       │   ├── en_us.json
│       │   ├── languages.json
│       │   └── ro_ro.json             # Romanian translations (primary)
│       ├── models/
│       │   ├── block/                 # Block 3D models
│       │   │   ├── *.json             # Standard block models
│       │   │   ├── andesite/          # Andesite variants
│       │   │   ├── birch_log/         # Birch log variations
│       │   │   └── variated/          # Randomised texture models
│       │   └── item/                  # Item 3D models and display transforms
│       │       ├── *.json             # Standard item models
│       │       └── magic_compass.json # Custom datapack item
│       ├── shaders/
│       │   └── core/                  # Post-processing shaders
│       ├── sounds/
│       │   ├── damage/                # Custom damage sounds
│       │   └── item/                  # Custom item sounds
│       └── textures/
│           ├── block/                 # Block textures
│           │   ├── 3d/                # 3D model textures
│           │   ├── colormap/          # Biome colormaps
│           │   ├── entity/            # Entity textures
│           │   ├── environment/       # Environment textures
│           │   ├── font/              # Font textures
│           │   ├── gui/               # GUI textures
│           │   ├── item/              # Item textures
│           │   ├── misc/              # Miscellaneous textures
│           │   ├── models/            # Model-specific textures
│           │   ├── particle/          # Particle textures
│           │   ├── trims/             # Armor trim textures
│           │   └── variated/          # Variation textures
│           ├── colormap/
│           ├── entity/
│           ├── environment/
│           ├── font/
│           ├── gui/
│           ├── item/
│           ├── misc/
│           ├── models/
│           ├── particle/
│           └── trims/
├── pack.mcmeta                        # Pack metadata (format, description)
├── pack.png                           # Pack icon
├── ARCHITECTURE.md                    # Architecture documentation
├── CODE_OF_CONDUCT.md                 # Code of conduct
├── CONTRIBUTING.md                    # Contribution guidelines
├── LICENSE                            # GPL v3.0 license
├── PRIVACY.md                         # Privacy policy
├── README.md                          # Project overview and features
├── SECURITY.md                        # Security policy
└── docs/                              # Generated documentation
    ├── INDEX.md
    ├── architecture.md
    ├── repository-overview.md
    ├── repository-structure.md
    ├── data-model.md
    ├── configuration.md
    ├── build-and-deployment.md
    ├── dependencies.md
    ├── testing.md
    ├── change-guide.md
    ├── design-decisions.md
    ├── error-handling.md
    ├── logging.md
    ├── security.md
    ├── state-and-persistence.md
    ├── integrations.md
    ├── concurrency-and-scheduling.md
    ├── documentation-maintenance.md
    ├── faq.md
    ├── troubleshooting.md
    ├── ambiguities-and-open-questions.md
    ├── api-usage-examples.md
    ├── quick-start.md
    ├── invariants.md
    ├── api-reference/
    │   └── INDEX.md
    ├── behaviour/
    ├── components/
    └── flows/
```

## Module Organisation

### Asset Categories (Minecraft Standard)

| Category | Path | Purpose |
|----------|------|---------|
| Blockstates | `assets/minecraft/blockstates/` | Maps block state properties to model variants |
| Block Models | `assets/minecraft/models/block/` | 3D geometry definitions for blocks |
| Item Models | `assets/minecraft/models/item/` | 3D geometry and display transforms for items |
| Textures | `assets/minecraft/textures/` | Pixel data for models (PNG + optional .mcmeta) |
| Atlases | `assets/minecraft/atlases/` | Texture atlas source configuration |
| Language | `assets/minecraft/lang/` | Translation key overrides |
| Shaders | `assets/minecraft/shaders/` | Post-processing shader programs |
| Sounds | `assets/minecraft/sounds/` | Custom sound events (OGG) |

### Blockstates

Each block with custom state handling has a JSON file under `blockstates/`. Files use either:
- **Variants format** — Direct mapping of state property combinations to models (e.g., `poplar_fence_gate.json`)
- **Multipart format** — Conditional model application based on state properties (e.g., `poplar_fence.json`)

### Block Models

Organised under `models/block/` with subdirectories for:
- **Standard models** — One JSON per block variant
- **Variation groups** — Subdirectories like `andesite/`, `birch_log/`, `variated/` for randomised textures
- **Template models** — Reusable base models (e.g., `custom_fence_post.json`, `template_fence_gate.json`)

### Item Models

Under `models/item/` with:
- Standard item models referencing block models or custom geometry
- Display transforms for firstperson, thirdperson, gui, head, ground, fixed perspectives
- Custom items like `magic_compass.json` for datapack integration

### Textures

Organised by category under `textures/`:
- `block/` — Block textures including 3D model textures, variations, animations
- `item/` — Item textures (some 32x from Faithful 32x)
- `entity/`, `environment/`, `gui/`, `particle/`, etc. — Other categories

### Language Files

Multiple English variants plus Romanian (`ro_ro.json`) as primary custom language. Keys override vanilla translations for renamed blocks/items.

## Configuration Files

| File | Purpose |
|------|---------|
| `pack.mcmeta` | Pack format version (min 84, max 999), description |
| `pack.png` | Pack icon (128x128 PNG) |
| `.github/workflows/release-asset.yml` | Release automation |

## Documentation Structure

Root documentation (required at repository root):
- `README.md` — Project overview, features, usage, installation
- `ARCHITECTURE.md` — High-level architecture
- `CONTRIBUTING.md` — Contribution guidelines
- `CODE_OF_CONDUCT.md` — Community standards
- `SECURITY.md` — Security policy
- `PRIVACY.md` — Privacy policy
- `LICENSE` — GPL v3.0

Generated documentation under `docs/`:
- See [INDEX.md](./INDEX.md) for full catalogue

## Build Artifacts

- Release ZIP: `nucicraft_<version>.zip` (contains `assets/`, `pack.mcmeta`, `pack.png`)
- Published via GitHub Releases on tag push