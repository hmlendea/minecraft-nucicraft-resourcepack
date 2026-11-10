# State and Persistence

This document describes state management and persistence in the NuciCraft Resource Pack.

## Overview

**The NuciCraft Resource Pack has no persistent state.** It is a pure data-only resource pack containing only static assets (JSON, PNG, OGG, GLSL) that are loaded by the Minecraft client at runtime.

## State Categories

### 1. Pack Metadata (Static)

**Location**: `pack.mcmeta`

| Field | Persistence | Description |
|-------|-------------|-------------|
| `pack_format` | Immutable per release | Target Minecraft version |
| `description` | Immutable per release | Display name with version |
| `supported_formats` | Immutable per release | Compatibility range |

**Lifecycle**: Set at release, never changes at runtime.

### 2. Asset Files (Static)

**Location**: `assets/minecraft/**`

| Asset Type | Files | Persistence |
|------------|-------|-------------|
| Blockstates | `blockstates/*.json` | Immutable per release |
| Block Models | `models/block/**/*.json` | Immutable per release |
| Item Models | `models/item/*.json` | Immutable per release |
| Textures | `textures/**/*.png` | Immutable per release |
| Animations | `textures/**/*.mcmeta` | Immutable per release |
| Atlases | `atlases/*.json` | Immutable per release |
| Languages | `lang/*.json` | Immutable per release |
| Shaders | `shaders/**/*.glsl` | Immutable per release |
| Sounds | `sounds/**/*.ogg` | Immutable per release |

**Lifecycle**: Committed to repository, packaged in release ZIP, loaded by client.

### 3. Client Runtime State (Ephemeral)

**Managed by Minecraft Client**, not the pack:

| State | Location | Lifetime |
|-------|----------|----------|
| Loaded asset registry | Client memory | Session |
| Baked models | Client memory | Session |
| Texture atlases (GPU) | GPU memory | Session |
| Language translations | Client memory | Session |
| Sound buffers | Client memory | Session |

**Lifecycle**: Created on pack load, destroyed on pack unload or client exit.

### 4. User Configuration (External)

**Not managed by pack**:

| Config | Location | Controlled By |
|--------|----------|---------------|
| Enabled resource packs | `options.txt` | User |
| Resource pack order | `options.txt` | User |
| Language selection | `options.txt` | User |
| Shader settings | `options.txt` | User |
| Video settings | `options.txt` | User |

## Data Flow

```
┌─────────────────┐     Load      ┌──────────────────┐
│  Release ZIP    │ ─────────────► │  Minecraft Client │
│  (Static Files) │                │  (Runtime State)  │
└─────────────────┘                └──────────────────┘
                                          │
                                          ▼
                                   ┌──────────────────┐
                                   │   Render Loop    │
                                   │  (Read-only)     │
                                   └──────────────────┘
```

## Persistence Operations

### Pack Installation

1. User copies ZIP to `resourcepacks/`
2. Client detects on startup or menu open
3. No pack-side action

### Pack Enable

1. User enables in Resource Pack menu
2. Client calls `ResourcePackLoader.loadPack()`
3. Client builds runtime state from static files
4. No pack-side action

### Pack Reload (F3+T)

1. User presses F3+T
2. Client discards runtime state
3. Client rebuilds from static files
4. No pack-side action

### Pack Update

1. User replaces ZIP in `resourcepacks/`
2. User reloads (F3+T) or restarts
3. Client rebuilds from new static files
4. No migration needed (stateless)

### Pack Disable

1. User disables in Resource Pack menu
2. Client discards runtime state
3. Vanilla assets used
4. No pack-side action

## State Invariants

| Invariant | Enforcement |
|-----------|-------------|
| Static files never modified at runtime | File system (read-only in ZIP) |
| No pack-side persistent storage | No code capability |
| Client runtime state derived purely from static files | Client architecture |
| No cross-session persistence | Client architecture |
| No user data stored by pack | No code capability |

## Migration

**No migration needed.** Pack updates are stateless replacements.

### Version Upgrade

1. User downloads new ZIP
2. Replaces old ZIP
3. Reloads (F3+T) or restarts
4. Client loads new static files
5. Done

### Version Downgrade

Same process — replace ZIP, reload.

## Backup and Recovery

### Pack Files

- Source of truth: Git repository
- Release artifacts: GitHub Releases
- User copy: `resourcepacks/` folder

### Client State

- Not backed up (ephemeral)
- Rebuilt on each load

## Concurrency

**No concurrency concerns** — pack is read-only data.

- Multiple clients can use same ZIP simultaneously
- No locking, no conflicts
- Server-prompted download: each client gets own copy

## Related Documents

- [Data Model](./data-model.md) — Static asset definitions
- [Build and Deployment](./build-and-deployment.md) — Release packaging
- [Error Handling](./error-handling.md) — Load failures
- [Architecture](./architecture.md) — Runtime flow