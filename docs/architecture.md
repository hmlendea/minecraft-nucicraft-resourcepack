# Architecture

This document elaborates on the root [ARCHITECTURE.md](../ARCHITECTURE.md) with implementation-level detail for maintainers and contributors.

## System Context (Expanded)

The NuciCraft Resource Pack is a **data-only** resource pack loaded by the Minecraft Java Edition client. It declares assets that the client's rendering engine interprets; it contains no executable code.

### External Actors and Systems

| Actor/System | Role | Interface |
|--------------|------|-----------|
| Minecraft Java Edition Client | Host platform | Resource Pack Loader, Asset Manager, Rendering Engine |
| NuciCraft Server (optional) | Distribution | `server-resource-packs` in `server.properties` |
| Pack Maintainer | Author | Git repository, GitHub Releases |
| Community Pack Authors | Upstream sources | Vendored assets under `assets/minecraft/**` |

### Trust Boundaries

- **Client ↔ Pack**: Client validates `pack_format` and asset schema; malformed assets are ignored or cause missing model/texture errors (purple/black checkerboard, magenta)
- **Server ↔ Client**: Server may prompt pack download; client user must accept
- **Maintainer ↔ Community Packs**: Vendored assets are copied at release time; no runtime dependency

## Architectural Style (Expanded)

**Declarative Data-Driven Architecture** conforming to Minecraft resource pack specification v107.1 (pack_format 84-999).

### Key Characteristics

1. **No executable logic** — All behaviour is data interpreted by the client
2. **Schema-driven** — JSON schemas for blockstates, models, atlases, language files
3. **Asset-graph dependencies** — Blockstates → Models → Textures (directed acyclic graph)
4. **Vendor integration via copy** — Community assets vendored under standard paths
5. **Version coupling** — `pack_format` ties pack to Minecraft version range

### Asset Dependency Graph

```
pack.mcmeta (root)
    └── assets/minecraft/
        ├── blockstates/ ──────┐
        │                      ▼
        ├── models/block/ ────► textures/block/
        │                      ▲
        ├── models/item/ ──────┘
        │
        ├── atlases/ ──────────► textures/ (sources)
        │
        └── lang/ (independent)
```

## Runtime Flow (Expanded)

### Load Sequence

1. **Client Startup / Resource Pack Reload** (F3+T)
2. **Loader** reads `pack.mcmeta`
   - Validates `pack_format` compatibility
   - Reads description for UI display
3. **Asset Manager** indexes `assets/minecraft/**`
   - Scans all directories for JSON, PNG, OGG, GLSL files
   - Builds internal asset registry
4. **Texture Atlas Construction**
   - Reads `atlases/blocks.json`
   - Collects sources from `textures/` directories
   - Packs into GPU texture atlases
5. **Model Baking**
   - Parses all `blockstates/*.json`
   - Resolves variant → model references
   - Parses `models/block/*.json` and `models/item/*.json`
   - Bakes geometry, resolves texture references
   - Validates parent chains (no cycles)
6. **Language Registration**
   - Loads `lang/*.json`
   - Registers translation overrides
7. **Render Loop**
   - For each block/item state in world
   - Renderer requests baked model from Asset Manager
   - Asset Manager returns baked model + resolved textures
   - Renderer draws frame

### Failure Modes

| Stage | Failure | Client Behaviour |
|-------|---------|------------------|
| Loader | Invalid `pack_format` | Pack ignored, fallback to vanilla |
| Loader | Missing `pack.mcmeta` | Pack ignored |
| Asset Manager | Malformed JSON | Asset skipped, error logged |
| Atlas | Missing texture source | Magenta placeholder texture |
| Model Baking | Missing variant model | Purple/black checkerboard |
| Model Baking | Circular parent | Error logged, model skipped |
| Language | Unknown key | Vanilla translation used |

## Architectural Areas (Expanded)

### Blockstates (`assets/minecraft/blockstates/`)

**Responsibility**: Map block state properties to model variants.

**Format Types**:
- **Variants** — Explicit property combination → model mapping (used for fence gates, doors, etc.)
- **Multipart** — Conditional model application (used for fences, walls, glass panes)

**Boundary Rules**:
- All referenced models must exist under `models/block/`
- Property names must match Minecraft's block state definitions
- Variant keys must be valid state property combinations

**Ownership**: Block authors (one file per block)

### Block Models (`assets/minecraft/models/block/`)

**Responsibility**: Define 3D geometry for blocks.

**Structure**:
- Elements with `from`/`to` coordinates (0-16 cube space)
- Faces with UV mapping, texture references, culling
- Parent inheritance for reuse
- Texture variables (`#texture`, `#particle`, etc.)

**Subdirectories**:
- `andesite/` — Andesite slab/stair variants
- `birch_log/` — Birch log variation models
- `variated/` — Randomised texture models using `variated/` textures

**Boundary Rules**:
- Texture references must resolve under `textures/block/`
- Parent references must not create cycles
- Element coordinates within 0-16 range
- UV coordinates within texture atlas bounds

**Ownership**: Block authors

### Item Models (`assets/minecraft/models/item/`)

**Responsibility**: Define 3D geometry and display transforms for items.

**Key Differences from Block Models**:
- Display transforms: `firstperson_righthand`, `firstperson_lefthand`, `thirdperson_righthand`, `thirdperson_lefthand`, `gui`, `head`, `ground`, `fixed`
- Often reference block models via `"model": "minecraft:block/..."`
- Custom items (e.g., `magic_compass.json`) define unique geometry

**Boundary Rules**:
- Texture references under `textures/item/`
- Display transform values follow Minecraft conventions
- Parent references valid

**Ownership**: Item authors

### Textures (`assets/minecraft/textures/`)

**Responsibility**: Provide pixel data for models.

**Organisation**:
- `block/` — Block textures (16x, 32x, animated via .mcmeta)
- `item/` — Item textures (some 32x from Faithful 32x)
- `entity/`, `environment/`, `gui/`, `particle/`, `font/`, `trims/`, `colormap/`, `misc/`, `models/`

**Animation**: `.mcmeta` files with `animation` key for frame timing/interpolation

**Boundary Rules**:
- Dimensions: powers of two (16, 32, 64, 128...)
- PNG format
- Animation metadata follows Minecraft schema

**Ownership**: Texture artists

### Atlases (`assets/minecraft/atlases/`)

**Responsibility**: Configure texture atlas sources.

**Current Configuration** (`blocks.json`):
- Entity sources: chest_boat, boat, chest, minecart, trident
- Directory and single-file sources

**Boundary Rules**:
- All sources must exist under `textures/`
- Atlas size limits apply (client-dependent)

**Ownership**: Pack maintainer

### Language (`assets/minecraft/lang/`)

**Responsibility**: Override vanilla translation keys.

**Files**:
- `ro_ro.json` — Romanian (primary custom language, ~5000+ keys)
- `en_*.json` — English variants (minimal overrides)
- `languages.json` — Language metadata

**Boundary Rules**:
- Keys must match vanilla translation keys exactly
- Values are free-form text
- No client-side validation beyond key existence

**Ownership**: Translators

### Shaders (`assets/minecraft/shaders/core/`)

**Responsibility**: Post-processing shader programs (GLSL).

**Boundary Rules**:
- Referenced by client shader system
- Must be compatible with client's shader version

**Ownership**: Shader authors (if any)

### Sounds (`assets/minecraft/sounds/`)

**Responsibility**: Custom sound events.

**Structure**:
- `damage/` — Retro damage sounds
- `item/` — Custom item sounds

**Boundary Rules**:
- OGG format
- Referenced by sound events in client

**Ownership**: Sound designers

## Data Architecture (Expanded)

### Static Asset Files

All data is static files read at load time. No persistent runtime state.

| Data Type | Format | Location | Owner | Lifecycle |
|-----------|--------|----------|-------|-----------|
| Pack metadata | JSON | `pack.mcmeta` | Maintainer | Immutable per release |
| Blockstates | JSON | `blockstates/*.json` | Block authors | Immutable per release |
| Block models | JSON | `models/block/**/*.json` | Block authors | Immutable per release |
| Item models | JSON | `models/item/*.json` | Item authors | Immutable per release |
| Textures | PNG + .mcmeta | `textures/**/*.png` | Texture artists | Immutable per release |
| Atlases | JSON | `atlases/*.json` | Maintainer | Immutable per release |
| Language | JSON | `lang/*.json` | Translators | Immutable per release |
| Shaders | GLSL | `shaders/**/*.glsl` | Shader authors | Immutable per release |
| Sounds | OGG | `sounds/**/*.ogg` | Sound designers | Immutable per release |

### Data Flow

```
Static Files (JSON, PNG, OGG, GLSL)
         │
         ▼
Asset Manager (Client)
         │
         ├──► Baked Models (geometry + texture refs)
         │
         └──► GPU Texture Atlases
                  │
                  ▼
            Renderer → Frame
```

### Consistency Requirements

- **Referential integrity**: All cross-asset references must resolve
- **Schema compliance**: All JSON files must match Minecraft schemas
- **Format compatibility**: `pack_format` must match client version
- **Atlas capacity**: Total unique textures must fit in atlas

## Interfaces and Integrations (Expanded)

### Minecraft Resource Pack Loader (Inbound)

**Contract**: `pack.mcmeta` + `assets/minecraft/**`

**Validation**:
- `pack_format` range check
- Required directories exist
- JSON syntax valid

**Failure**: Pack ignored, vanilla assets used

### Minecraft Rendering Engine (Inbound)

**Contract**: Block/item model JSON schema, texture atlas

**Consumption**:
- Baked models for each block/item state
- Resolved textures from atlases

**Failure**: Missing model → purple/black checkerboard; missing texture → magenta

### Minecraft Asset Manager (Inbound)

**Contract**: Texture atlas config (`atlases/blocks.json`)

**Consumption**: Builds atlases from declared sources

**Failure**: Missing source → magenta texture

### NuciCraft Server (Outbound, Optional)

**Contract**: `server-resource-packs` in `server.properties`

**Behaviour**: Server sends pack URL + SHA1; client prompts download

**Failure**: Client declines or download fails → vanilla

## Dependency Direction and Rules (Expanded)

### Dependency Graph

```
pack.mcmeta
    │
    ▼
assets/minecraft/
    │
    ├── blockstates/ ──────┐
    │                      │
    ├── models/block/ ◄────┤ (references)
    │                      │
    ├── models/item/ ◄─────┤ (references)
    │                      │
    ├── textures/ ─────────┘ (referenced by models)
    │
    ├── atlases/ ──────────► textures/ (sources)
    │
    └── lang/ (independent)
```

### Rules

1. **pack.mcmeta** is root; no asset depends on it
2. **blockstates/** depends on **models/block/**
3. **models/block/** and **models/item/** depend on **textures/**
4. **atlases/** depends on **textures/**
5. **lang/** is independent (referenced by client only)
6. **shaders/** and **sounds/** independent unless referenced
7. Community pack assets vendored under standard paths; no external runtime dependency

## External Dependencies (Expanded)

| Dependency | Version/Source | Integration | Consequence |
|------------|----------------|-------------|-------------|
| Minecraft Java Edition | 1.21.x (pack_format 107.1) | Host platform | Pack must target compatible version |
| VanillaTweaks | Community pack | Vendored assets | Manual re-vendoring for updates |
| Faithful 32x | Community pack | Vendored 32x item textures | Manual re-vendoring |
| MCjossic | Community pack | 3D models (amethyst, iron bars, dripstone) | Manual integration |
| Saplistic | Community pack | 3D chains | Manual integration |
| Ecorn | Community pack | 3D doors/trapdoors v2.3.1 | Manual integration |
| Ivango | Community pack | 3D buckets | Manual integration |
| JMHMirror | Community pack | 3D goat horn | Manual integration |
| D_Dimensional | Community pack | 3D torches/lanterns v1.3.4 | Manual integration |
| ZzartySt | Community pack | 3D trims | Manual integration |
| PiXeLGaMeS_or_not | Community pack | Musket (crossbow replacement) | Manual integration |
| Ensis | Community pack | 3D beetroots/carrots | Manual integration |
| Vexcenot | Community pack | 3D crops (melon, potato, pumpkin, sweet berry) | Manual integration |
| Hecco | Community pack | 3D flowers | Manual integration |
| Relidium | Community pack | Variated obsidian v2 | Manual integration |
| Infina | Community pack | Variated bricks | Manual integration |
| Oslypsis | Community pack | Colorful cobblestone | Manual integration |
| Amethyst Geode | Community pack | Glowstone variations | Manual integration |
| PatchWolf | Community pack | Improved prismarine bricks v1.1 | Manual integration |
| Mexorax | Community pack | Animated candles | Manual integration |
| PixalThePixeled | Community pack | Animated crying obsidian | Manual integration |
| DudeAbide | Community pack | 3D redstone dust | Manual integration |
| Luracasmus | Community pack | Optimised models v1.5.1 | Manual integration |

## Deployment and Operations (Expanded)

### Distribution

- **Primary**: GitHub Releases (ZIP: `nucicraft_<version>.zip`)
- **Installation**: Manual copy to `resourcepacks/` folder
- **Server Prompt**: Optional via `server-resource-packs`

### Versioning

- `pack.mcmeta` description: `NuciCraft v<version>`
- Git tags: `v<version>`
- ZIP filename: `nucicraft_<version>.zip`
- `pack_format` ties to Minecraft version

### Release Process

1. Create Git tag `v<version>`
2. GitHub Action `release-asset.yml` triggers on published release
3. Action packages `assets/`, `pack.mcmeta`, `pack.png` into ZIP
4. Computes SHA1 checksum
5. Uploads ZIP as release asset with release notes + checksum

### Updates

- Replace ZIP in `resourcepacks/`
- Client reload via F3+T or restart
- No hot-reload of individual assets

### Persistent State

None — no migration needed

## Compatibility Contracts (Expanded)

| Contract | Invariant | Verification | Change Policy |
|----------|-----------|--------------|---------------|
| `pack_format` | Matches target Minecraft version | Client loads without error | Update on Minecraft version upgrade |
| Blockstate → Model | All variant models exist | Client logs missing model errors | Add missing models before release |
| Model → Texture | All texture paths resolve | Client logs missing texture errors | Add missing textures before release |
| Language keys | Match vanilla translation keys | In-game text appears correctly | Update keys on vanilla changes |
| Atlas capacity | Total textures fit | Client builds atlas successfully | Limit variations if exceeded |
| Model complexity | Render performance acceptable | Manual testing | Vet community models for poly count |

## Testing and Verification (Expanded)

### Current State

**No automated test suite exists.** Verification is entirely manual.

### Manual Verification Steps

```bash
# 1. Install pack in Minecraft client
# 2. Load a world with the pack enabled
# 3. Verify:
#    - Custom 3D models render correctly (doors, fences, chains, etc.)
#    - Texture variations appear (birch log, cobblestone, ores, etc.)
#    - Renamed items/blocks show custom names (Bambus, Sakura, Cement, etc.)
#    - No purple/black checkerboard (missing model) or magenta (missing texture)
#    - Performance acceptable (no excessive model complexity)
# 4. Test with server resource pack prompt (if configured)
```

### Recommended Automated Checks (Future)

- JSON schema validation for all `.json` files
- Reference integrity checker (blockstate→model, model→texture)
- Texture dimension validator (powers of two)
- `pack_format` compatibility checker
- Atlas size estimator

## Design Constraints (Expanded)

1. **Minecraft pack_format coupling** — Bound to pack_format 107.1 (Minecraft 1.21.x); upgrading Minecraft may require pack format changes
2. **No runtime logic** — All behaviour declarative; dynamic effects require client-side mods (out of scope)
3. **Texture atlas limits** — Excessive unique textures can exceed atlas capacity; variations must be balanced
4. **Model complexity budget** — High-poly models impact render performance; community models vetted for complexity
5. **Vendor lock-in** — Community pack assets vendored; upstream updates require manual integration
6. **Single-player / server parity** — Pack works in single-player and on servers that prompt it; no server-side component
7. **Language maintenance** — Custom translations must be kept in sync with vanilla key changes
8. **File system case sensitivity** — Asset paths case-sensitive on Linux; must match exactly
9. **JSON formatting** — Minecraft parser strict; trailing commas, comments invalid (though some clients tolerate)
10. **Texture animation limits** — Frame count and timing affect memory/performance

## Related Documentation

- [Repository Overview](./repository-overview.md) — Purpose, scope, entry points
- [Repository Structure](./repository-structure.md) — Source tree, module organisation
- [Data Model](./data-model.md) — Domain entities, relationships
- [Components](./components/) — Per-component deep dives
- [Flows](./flows/) — End-to-end execution flows
- [Configuration](./configuration.md) — Configuration schema
- [Build and Deployment](./build-and-deployment.md) — Release pipeline
- [Dependencies](./dependencies.md) — External/internal dependencies
- [Design Decisions](./design-decisions.md) — Key choices and rationale
- [Testing](./testing.md) — Test strategy
- [Change Guide](./change-guide.md) — Safe modification patterns