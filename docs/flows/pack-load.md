# Pack Load Flow

End-to-end flow: resource pack loading from file to active assets.

## Overview

```
ZIP File → ResourcePackLoader → ResourceManager → AssetManager → Rendering Engine
```

## Phase 1: Pack Discovery

### Trigger

- Game start
- Resource pack menu open
- F3+T reload
- Server resource pack download

### Process

1. **Scan** `resourcepacks/` folder
2. **Read** each ZIP/root folder
3. **Parse** `pack.mcmeta`
4. **Validate** `pack_format` compatibility
5. **Register** pack in `ResourcePackRepository`

### pack.mcmeta Validation

```json
{
  "pack": {
    "pack_format": 107.1,
    "supported_formats": { "min_inclusive": 84, "max_inclusive": 999 },
    "description": "NuciCraft v6.3.5"
  }
}
```

- `pack_format` ≤ client format → compatible
- `supported_formats` range includes client format → compatible
- Invalid JSON → pack rejected
- Missing file → pack rejected

## Phase 2: Resource Pack Loading

### ResourcePackLoader

For each enabled pack (priority order):

1. **Open** ZIP or directory
2. **Index** all assets under `assets/`
3. **Build** resource map: `namespace:type/path` → input stream
4. **Register** resource pack in `ResourceManager`

### Namespace Handling

- `minecraft:` — overrides vanilla
- `nucicraft:` — custom additions
- Other namespaces — community pack assets (vendored under minecraft:)

### Asset Types Indexed

| Type | Path Pattern |
|------|--------------|
| Blockstates | `assets/minecraft/blockstates/*.json` |
| Block Models | `assets/minecraft/models/block/**/*.json` |
| Item Models | `assets/minecraft/models/item/**/*.json` |
| Textures | `assets/minecraft/textures/**/*.png` |
| Texture Meta | `assets/minecraft/textures/**/*.png.mcmeta` |
| Languages | `assets/minecraft/lang/*.json` |
| Atlases | `assets/minecraft/atlases/*.json` |
| Shaders | `assets/minecraft/shaders/**/*` |
| Sounds | `assets/minecraft/sounds/**/*.ogg` |
| Sound Events | `assets/minecraft/sounds.json` |

## Phase 3: Resource Manager Merging

### Priority Order

1. **Vanilla** (built-in, lowest)
2. **Enabled packs** (first enabled = highest priority)
3. **Server pack** (if prompted, highest)

### Merge Strategy

- **Replace**: Same namespace:type/path → higher priority wins
- **Merge**: Language files — keys merged, higher priority wins
- **Append**: Atlas sources — all sources combined

### Conflict Resolution

```
Vanilla: block/cobblestone → vanilla model
NuciCraft: block/cobblestone → NuciCraft model (wins)
```

## Phase 4: Asset Manager Processing

### Blockstate Loading

1. **Parse** all `blockstates/*.json`
2. **Validate** JSON syntax
3. **Resolve** model references
4. **Build** `BlockStateModel` map

### Model Baking (Deferred)

Models not baked yet — references stored for Phase 5.

### Texture Loading

1. **Collect** all texture references from models
2. **Load** PNG files
3. **Parse** `.mcmeta` for animations
4. **Stitch** into atlases (see Atlas Stitching)

### Language Loading

1. **Parse** all `lang/*.json`
2. **Merge** by priority
3. **Register** in `LanguageManager`

### Atlas Stitching

1. **Read** `atlases/blocks.json`
2. **Collect** textures from sources
3. **Sort** by size (largest first)
4. **Pack** into atlas pages
5. **Generate** UV coordinates
6. **Upload** to GPU

## Phase 5: Model Baking

### Trigger

- After all resources loaded
- On F3+T reload
- When model references resolved

### Process

1. **Topological sort** model dependencies (parent chain)
2. **Bake** each model:
   - Resolve parent
   - Merge textures
   - Process elements
   - Apply transforms
   - Generate `BakedModel`
3. **Cache** baked models
4. **Register** in `ModelManager`

### BakedModel Output

- Vertex data (position, UV, normal, color)
- Face culling info
- Tint indices
- Transformation matrices per view

## Phase 6: Rendering Engine Integration

### Block Rendering

1. **Chunk rebuild** → collect block states
2. **Lookup** `BlockStateModel` for each state
3. **Get** `BakedModel`
4. **Generate** vertex buffer (VBO)
5. **Upload** to GPU
6. **Draw** calls per chunk

### Item Rendering

1. **Get** item stack
2. **Resolve** model (check overrides)
3. **Get** `BakedModel`
4. **Apply** display transform for view
5. **Render**

## Error Handling

### Load-Time Errors

| Error | Handling |
|-------|----------|
| Invalid pack.mcmeta | Pack disabled, logged |
| Missing model reference | Purple/black block, logged |
| Missing texture | Magenta texture, logged |
| JSON syntax error | Asset skipped, logged |
| Atlas overflow | Magenta textures, logged |

### Runtime Errors

| Error | Handling |
|-------|----------|
| Model bake failure | Fallback to missing model |
| Texture missing at draw | Magenta |
| Shader compile error | Fallback to default |

## Logging

### Key Loggers

- `ResourcePackLoader` — pack discovery
- `ResourceManager` — resource merging
- `ModelBakery` — model baking
- `TextureAtlas` — atlas stitching
- `LanguageManager` — language loading

### Log Levels

- **INFO**: Pack loaded, atlas stitched
- **WARN**: Missing optional asset
- **ERROR**: Critical failure (pack disabled)

## Performance

### Timing (Typical)

| Phase | Time |
|-------|------|
| Pack discovery | < 100ms |
| Resource indexing | 100-500ms |
| Texture loading | 500-2000ms |
| Atlas stitching | 200-1000ms |
| Model baking | 500-3000ms |
| **Total (F3+T)** | **2-10 seconds** |

### Optimization

- Fewer unique textures → faster atlas
- Simpler models → faster baking
- Fewer packs → less merging

## Debugging

### Commands

```
/reload                    # Full reload (server)
F3+T                       # Client resource reload
F3+H                       # Advanced tooltips (show keys)
```

### Logs

```
logs/latest.log
```

Search for:
- `ResourcePackLoader`
- `ModelBakery`
- `TextureAtlas`
- `Atlas overflow`

### In-Game Debug

F3 → "Texture Atlas" section shows:
- Atlas dimensions
- Texture count
- Usage percentage

## Related Documents

- [Flows: Model Baking](./model-baking.md)
- [Flows: Render Frame](./render-frame.md)
- [Components: Blockstates](../components/blockstates.md)
- [Components: Block Models](../components/block-models.md)
- [Components: Textures](../components/textures.md)
- [Components: Atlases](../components/atlases.md)
- [Troubleshooting](../troubleshooting.md)