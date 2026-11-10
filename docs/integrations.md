# Integrations

This document describes all integration points for the NuciCraft Resource Pack.

## Integration Overview

The NuciCraft Resource Pack integrates with Minecraft Java Edition through the standard resource pack system. It has no direct integrations with other software.

```
┌─────────────────────────────────────────────────────────────┐
│                    EXTERNAL SYSTEMS                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────┐    ┌─────────────────┐                │
│  │  Minecraft      │    │  NuciCraft      │                │
│  │  Java Edition   │◄───│  Server         │                │
│  │  Client         │    │  (Optional)     │                │
│  └────────┬────────┘    └────────┬────────┘                │
│           │                      │                         │
│           │  resource-pack       │                         │
│           │  prompt              │                         │
│           ▼                      ▼                         │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              RESOURCE PACK LOADER                   │   │
│  │  (Built into Minecraft Client)                      │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Minecraft Java Edition Client (Primary Integration)

### Interface

**Resource Pack Loader** — Built into Minecraft client.

### Contract

| Aspect | Specification |
|--------|---------------|
| Pack Format | 107.1 (Minecraft 1.21.x) |
| Supported Formats | 84-999 (1.18 - future) |
| Asset Namespace | `minecraft:` (override vanilla) |
| Custom Namespace | `nucicraft:` (for new items) |
| File Formats | JSON, PNG, OGG, GLSL |
| Max Pack Size | Client-dependent (~100MB practical) |

### Integration Points

| Point | Mechanism | Pack Responsibility |
|-------|-----------|---------------------|
| Discovery | `resourcepacks/` folder scan | Place ZIP in folder |
| Validation | `pack.mcmeta` parsing | Valid JSON, correct format |
| Asset Loading | `assets/minecraft/**` scan | Standard paths, valid schemas |
| Model Baking | Blockstate → Model → Texture | Valid references, no cycles |
| Texture Atlas | `atlases/blocks.json` sources | Sources exist, fit in atlas |
| Language | `lang/*.json` registration | Keys match vanilla |
| Rendering | Baked models + atlases | Valid geometry, textures |
| Sound | `sounds/**/*.ogg` events | Valid OGG, correct names |

### Version Compatibility

| Client Version | Pack Format | Status |
|----------------|-------------|--------|
| 1.21.x | 107.1 | ✅ Full support |
| 1.20.5-1.20.6 | 48 | ⚠️ Loads, missing 1.21 assets |
| 1.20.3-1.20.4 | 41 | ⚠️ Loads, missing 1.21 assets |
| 1.20-1.20.2 | 15 | ⚠️ Loads, missing 1.21 assets |
| 1.19.3-1.19.4 | 12 | ⚠️ Loads, missing 1.21 assets |
| 1.18-1.19.2 | 8-9 | ⚠️ Loads, missing 1.21 assets |
| < 1.18 | < 8 | ❌ Rejected (below min_inclusive) |

## NuciCraft Server (Optional Integration)

### Interface

**Server Resource Pack Prompt** — `server.properties` configuration.

### Configuration

```properties
# server.properties
resource-pack=https://github.com/<owner>/nucicraft-resourcepack/releases/download/v6.3.5/nucicraft_v6.3.5.zip
resource-pack-sha1=<sha1-checksum-from-release>
resource-pack-prompt="NuciCraft Resource Pack v6.3.5 - Required for custom models and textures"
resource-pack-enforce=true
```

### Contract

| Field | Required | Description |
|-------|----------|-------------|
| `resource-pack` | Yes | Direct HTTPS URL to ZIP |
| `resource-pack-sha1` | Yes | SHA1 of ZIP (lowercase hex) |
| `resource-pack-prompt` | No | User-facing message |
| `resource-pack-enforce` | No | `true` = kick if declined |

### Flow

1. Client connects to server
2. Server sends pack URL + SHA1 + prompt
3. Client shows prompt to user
4. User accepts → Client downloads, verifies SHA1, enables pack
5. User declines → If `enforce=true`, disconnect; else vanilla

### Requirements

- HTTPS URL (GitHub Releases works)
- SHA1 must match exactly (case-sensitive)
- ZIP must be accessible without auth
- Server must not modify pack after SHA1 published

## Community Pack Integration (Build-Time)

### Interface

**Manual vendoring** — Copy assets at release preparation.

### Process

1. Maintainer downloads community pack release
2. Extracts relevant assets
3. Copies to `assets/minecraft/**` under standard paths
4. Resolves conflicts (NuciCraft priority)
5. Updates README.md attributions
6. Commits and releases

### Integrated Packs

| Pack | Assets | Path |
|------|--------|------|
| VanillaTweaks | Texture tweaks, variations | `textures/block/`, `models/block/` |
| Faithful 32x | 32x item textures | `textures/item/` |
| MCjossic | 3D amethyst, iron bars, dripstone | `models/block/`, `textures/block/` |
| Saplistic | 3D chains | `models/block/`, `textures/block/` |
| Ecorn | 3D doors, trapdoors | `models/block/`, `textures/block/` |
| Ivango | 3D buckets | `models/block/`, `textures/block/` |
| JMHMirror | 3D goat horn | `models/item/`, `textures/item/` |
| D_Dimensional | 3D torches, lanterns | `models/block/`, `textures/block/` |
| ZzartySt | 3D armor trims | `textures/trims/`, `models/item/` |
| PiXeLGaMeS_or_not | Musket | `models/item/`, `textures/item/` |
| Ensis | 3D beetroots, carrots | `models/block/`, `textures/block/` |
| Vexcenot | 3D crops | `models/block/`, `textures/block/` |
| Hecco | 3D flowers | `models/block/`, `textures/block/` |
| Relidium | Variated obsidian | `textures/block/variated/`, `models/block/variated/` |
| Infina | Variated bricks | `textures/block/variated/`, `models/block/variated/` |
| Oslypsis | Colorful cobblestone | `textures/block/variated/`, `models/block/variated/` |
| Amethyst Geode | Glowstone variations | `textures/block/variated/`, `models/block/variated/` |
| PatchWolf | Prismarine bricks | `textures/block/`, `models/block/` |
| Mexorax | Animated candles | `textures/block/`, `models/block/` |
| PixalThePixeled | Animated crying obsidian | `textures/block/`, `models/block/` |
| DudeAbide | 3D redstone dust | `models/block/`, `textures/block/` |
| Luracasmus | Optimised models | `models/block/` |

### Conflict Resolution

| Conflict | Resolution |
|----------|------------|
| Same file, different content | NuciCraft customisation wins |
| Upstream adds files | Add if compatible |
| Upstream removes files | Remove if not customised |
| Namespace collision | Use `nucicraft:` for custom |

## Datapack Integration (Magic Compass)

### Interface

**Custom Model Data** — Item predicate in model overrides.

### Magic Compass Implementation

**Item Model** (`models/item/magic_compass.json`):
```json
{
  "parent": "minecraft:item/compass",
  "overrides": [
    { "predicate": { "custom_model_data": 1001 }, "model": "minecraft:item/magic_compass_active" }
  ]
}
```

**Datapack Required** (separate, not in this pack):
- Sets `CustomModelData: 1001` on compass via commands/loot tables
- Defines compass behaviour (tracking, particles, etc.)

### Contract

| Aspect | Value |
|--------|-------|
| Custom Model Data | 1001 |
| Base Item | Compass |
| Active Model | `minecraft:item/magic_compass_active` |
| Datapack | External (user-provided) |

## Shader Pack Integration (Future)

### Interface

**Minecraft Shader System** — `shaders/core/*.glsl`

### Current State

- Directory exists: `assets/minecraft/shaders/core/`
- No shaders currently included
- Ready for future addition

### Contract

| Aspect | Specification |
|--------|---------------|
| Language | GLSL (version client-dependent) |
| Location | `assets/minecraft/shaders/core/` |
| Activation | Video Settings → Shaders |
| Compatibility | Client shader version |

## Mod Compatibility

### Known Compatible

| Mod | Compatibility | Notes |
|-----|---------------|-------|
| OptiFine | ✅ Full | Enhances textures, connected textures |
| Sodium | ✅ Full | Performance, no visual changes |
| Iris | ✅ Full | Shader support |
| Canvas | ✅ Full | Shader support |
| Fabric/Forge | ✅ Full | No conflicts (data-only) |

### Known Issues

| Mod | Issue | Workaround |
|-----|-------|------------|
| CTM (Connected Textures) | May conflict with variations | Disable CTM or pack variations |
| Custom Block Models (mod) | May override pack models | Load pack after mod |

## CI/CD Integration

### GitHub Actions

**Workflow**: `.github/workflows/release-asset.yml`

**Triggers**: `release.published`

**Integration Points**:
- GitHub Releases (asset upload)
- GitHub Token (authentication)
- Ubuntu runner (build environment)

### Release Artifact

- **Name**: `nucicraft_<tag>.zip`
- **Content**: `assets/`, `pack.mcmeta`, `pack.png`
- **Checksum**: SHA1 in release notes

## Development Tool Integration

### Recommended Tools

| Tool | Purpose | Integration |
|------|---------|-------------|
| Blockbench | 3D model editing | Export JSON to `models/` |
| VS Code + JSON | JSON editing | Schema validation |
| GIMP/Photoshop | Texture editing | Export PNG to `textures/` |
| ImageMagick | Batch texture processing | CLI automation |
| jq | JSON validation | `jq empty file.json` |

### No Required Tools

- No build tools required
- No compilation step
- No package manager
- Plain text editor sufficient

## Integration Testing

### Manual Test Matrix

| Integration | Test Procedure |
|-------------|----------------|
| Client Load | Install ZIP, enable, verify no errors |
| Server Prompt | Configure server, connect, accept prompt |
| OptiFine | Install OptiFine, load pack, verify CTM |
| Shader Pack | Add shader, enable in video settings |
| Datapack (compass) | Install datapack, give compass, verify model |

### Automated Tests (Planned)

- JSON schema validation in CI
- Reference integrity check in CI
- Texture dimension validation in CI

## Related Documents

- [Architecture](./architecture.md) — Runtime flow
- [Build and Deployment](./build-and-deployment.md) — Release process
- [Dependencies](./dependencies.md) — Community packs
- [Configuration](./configuration.md) — Pack format, server config
- [Change Guide](./change-guide.md) — Updating community packs