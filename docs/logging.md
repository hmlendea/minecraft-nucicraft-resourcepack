# Logging

This document describes logging behaviour, log sources, and diagnostic information for the NuciCraft Resource Pack.

## Log Sources

The NuciCraft Resource Pack itself produces **no logs** — it is a data-only pack with no executable code. All logging comes from the Minecraft client when loading and using the pack.

### Minecraft Client Logs

**Location**: `.minecraft/logs/latest.log` (Windows: `%APPDATA%\.minecraft\logs\latest.log`)

**Relevant Loggers**:
- `ResourcePackLoader` — Pack loading and validation
- `ModelBakery` — Model baking and validation
- `TextureAtlas` — Texture atlas building
- `LanguageManager` — Language file loading
- `SoundSystem` — Sound loading

## Log Events by Category

### Pack Loading

```
[ResourcePackLoader] Loading resource pack: nucicraft_v6.3.5.zip
[ResourcePackLoader] Pack format: 107.1, supported: 84-999
[ResourcePackLoader] Description: NuciCraft v6.3.5
[ResourcePackLoader] Successfully loaded resource pack
```

**Error Example**:
```
[ResourcePackLoader] Failed to load resource pack: nucicraft_v6.3.5.zip
com.google.gson.JsonSyntaxException: Expected a com.google.gson.JsonObject but was com.google.gson.JsonPrimitive
```

### Model Baking

```
[ModelBakery] Loading blockstate: minecraft:oak_fence
[ModelBakery] Loading model: minecraft:block/oak_fence_post
[ModelBakery] Loading model: minecraft:block/oak_fence_side
[ModelBakery] Baked 1234 models
```

**Error Examples**:
```
[ModelBakery] Exception loading blockstate: minecraft:custom_block
java.io.FileNotFoundException: minecraft:blockstates/custom_block.json

[ModelBakery] Circular reference detected: minecraft:block/model_a -> minecraft:block/model_b -> minecraft:block/model_a

[ModelBakery] Invalid element coordinates: from=[-1,0,0] to=[16,16,16]
```

### Texture Atlas

```
[TextureAtlas] Created texture atlas: minecraft:textures/atlas/blocks.png (8192x8192)
[TextureAtlas] Stitched 2345 textures
```

**Error Examples**:
```
[TextureAtlas] Missing texture: minecraft:block/missing_texture
[TextureAtlas] Texture too large: minecraft:block/huge_texture (16384x16384 > 8192)
[TextureAtlas] Atlas overflow: 123 textures could not fit
```

### Language Loading

```
[LanguageManager] Loaded language: ro_ro (Română (România))
[LanguageManager] Loaded 5432 translations for ro_ro
```

**Error Example**:
```
[LanguageManager] Failed to load language: ro_ro
com.google.gson.JsonSyntaxException: Unexpected character at line 123
```

### Sound Loading

```
[SoundSystem] Loaded sound: nucicraft:item.musket_shoot
[SoundSystem] Loaded sound: nucicraft:damage.retro_hurt
```

## Diagnostic Information

### Debug Screen (F3)

The debug screen shows resource pack status:

```
Resource Packs: 1/1 (NuciCraft v6.3.5)
```

### Resource Pack Menu

Shows pack description and status:
- Green checkmark = loaded successfully
- Red X = failed to load
- Yellow warning = loaded with errors (check logs)

### Model/Texture Debugging

**F3 + T**: Reload resource packs (shows errors in chat/log)

**F3 + H**: Advanced tooltips (shows model/texture paths in creative menu)

## Log Analysis Patterns

### Successful Load Pattern

```
[ResourcePackLoader] Loading resource pack: nucicraft_v6.3.5.zip
[ResourcePackLoader] Pack format: 107.1
[ResourcePackLoader] Description: NuciCraft v6.3.5
[ModelBakery] Baked 1234 models
[TextureAtlas] Stitched 2345 textures
[LanguageManager] Loaded language: ro_ro
```

### Common Failure Patterns

| Pattern | Meaning | Action |
|---------|---------|--------|
| `JsonSyntaxException` in ResourcePackLoader | Invalid `pack.mcmeta` | Fix JSON |
| `FileNotFoundException` in ModelBakery | Missing blockstate/model | Add file or fix reference |
| `Circular reference` in ModelBakery | Model parent cycle | Break cycle |
| `Missing texture` in TextureAtlas | Texture file not found | Add texture or fix reference |
| `Atlas overflow` in TextureAtlas | Too many textures | Reduce variations |
| `JsonSyntaxException` in LanguageManager | Invalid lang JSON | Fix JSON |

## Log Filtering

### Grep for Resource Pack Errors

```bash
# Pack load errors
grep -i "resourcepackloader" logs/latest.log | grep -i error

# Model errors
grep -i "modelbakery" logs/latest.log | grep -iE "error|exception|fail"

# Texture errors
grep -i "textureatlas" logs/latest.log | grep -iE "error|exception|fail|missing|overflow"

# Language errors
grep -i "languagemanager" logs/latest.log | grep -iE "error|exception|fail"
```

### Real-Time Monitoring

```bash
# Follow log while testing
tail -f logs/latest.log | grep -iE "resourcepack|modelbakery|textureatlas|languagemanager"
```

## Log Retention

- Minecraft keeps `latest.log` and `latest.log.1` (previous session)
- Older logs compressed in `logs/` directory
- No pack-specific log retention needed

## Custom Logging (Not Applicable)

The pack cannot produce custom logs. For debugging:
1. Use Minecraft client logs
2. Use F3+T reload to trigger fresh log entries
3. Use F3+H for advanced tooltips
4. Create test worlds to isolate issues

## Log-Based Verification Checklist

After loading pack, verify logs show:

- [ ] `ResourcePackLoader` — Pack loaded successfully
- [ ] `ModelBakery` — No "Exception loading" or "Circular reference"
- [ ] `TextureAtlas` — No "Missing texture" or "Atlas overflow"
- [ ] `LanguageManager` — Target language loaded
- [ ] No `JsonSyntaxException` anywhere

If any check fails, see [Error Handling](./error-handling.md) for resolution.