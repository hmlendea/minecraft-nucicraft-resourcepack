# Concurrency and Scheduling

This document describes concurrency and scheduling considerations for the NuciCraft Resource Pack.

## Overview

**The NuciCraft Resource Pack has no internal concurrency or scheduling.** It is a static data-only resource pack loaded by the Minecraft client. All concurrency is managed by the Minecraft client's rendering engine and asset loading system.

## Client-Side Concurrency (Minecraft Managed)

### Asset Loading

The Minecraft client loads resource packs using a multi-threaded asset loading system:

```
Main Thread                    Worker Threads
    │                              │
    ├─ ResourcePackLoader ────────►│ (parse pack.mcmeta)
    │                              │
    ├─ AssetManager ──────────────►│ (scan assets/minecraft/**)
    │                              │  ├─ JSON parsing (blockstates, models, lang)
    │                              │  ├─ PNG decoding (textures)
    │                              │  ├─ OGG decoding (sounds)
    │                              │  └─ GLSL compilation (shaders)
    │                              │
    ├─ TextureAtlasBuilder ───────►│ (stitch textures to atlas)
    │                              │
    ├─ ModelBakery ───────────────►│ (bake models, resolve refs)
    │                              │
    ▼                              ▼
Render Thread ◄────────────────────┘ (consumes baked models, atlases)
```

### Thread Safety

- **Pack files**: Read-only, thread-safe by immutability
- **Asset parsing**: Worker threads, isolated per file
- **Atlas building**: Single-threaded (GPU texture upload)
- **Model baking**: Worker threads, dependency-ordered
- **Rendering**: Main render thread only

### Synchronisation Points

1. **Pack load complete** — All assets parsed, atlases built, models baked
2. **Frame render** — Render thread consumes immutable baked data
3. **Reload (F3+T)** — Full pipeline restart, old data discarded

## Pack-Side Concurrency

**None.** The pack contains no:
- Background threads
- Async operations
- Scheduled tasks
- Event handlers
- Timers
- Network communication

## Scheduling

### Load-Time Schedule

| Phase | Trigger | Duration | Parallelism |
|-------|---------|----------|-------------|
| Pack discovery | Client startup / menu open | ~ms | Single |
| pack.mcmeta parse | Pack enable | ~ms | Single |
| Asset scan | Pack enable | ~10-100ms | Multi (per file) |
| JSON parse | Asset scan | ~ms/file | Multi |
| PNG decode | Asset scan | ~ms/file | Multi |
| Atlas build | After scan | ~10-500ms | Single (GPU) |
| Model bake | After scan | ~10-200ms | Multi (dep-ordered) |
| Language register | After scan | ~ms | Single |
| Ready | All complete | — | — |

### Runtime Schedule

| Activity | Frequency | Thread |
|----------|-----------|--------|
| Frame render | Every frame (60+ Hz) | Render |
| Chunk render | As needed | Render |
| Entity render | As needed | Render |
| Sound play | Event-driven | Audio |
| Resource reload | F3+T / menu | Main + Workers |

## Resource Contention

### Texture Atlas

- **Shared resource**: GPU texture atlas
- **Contention**: Multiple packs → single atlas
- **Resolution**: Client merges all enabled packs' atlas sources
- **Limit**: Atlas size (typically 8192x8192 or 16384x16384)

### Model Registry

- **Shared resource**: Baked model registry
- **Contention**: Multiple packs → same block/item
- **Resolution**: Pack priority order (top wins)
- **NuciCraft**: Place at top for full effect

### Language Registry

- **Shared resource**: Translation map
- **Contention**: Multiple packs → same key
- **Resolution**: Pack priority order (top wins)
- **NuciCraft**: Romanian overrides at top

## Performance Implications

### Pack Size Impact

| Factor | Impact | Mitigation |
|--------|--------|------------|
| Texture count | Atlas build time, GPU memory | Limit variations |
| Texture resolution | Atlas size, VRAM | Use 16x, 32x max |
| Model complexity | Bake time, render time | Vet community models |
| JSON file count | Scan/parse time | Acceptable (<2000 files) |

### Concurrency-Related Issues

| Issue | Cause | Symptom | Fix |
|-------|-------|---------|-----|
| Atlas overflow | Too many textures | Magenta textures | Reduce variations |
| Model bake timeout | Circular refs, complex models | Pack load stall | Fix refs, simplify |
| Frame drops | High-poly models in view | Low FPS | Reduce poly count |
| Reload lag | Large pack | F3+T takes seconds | Acceptable |

## Thread Safety for Contributors

### Do Not

- Add any executable code (scripts, mods, datapacks with logic)
- Reference external resources (URLs, file paths outside pack)
- Use mutable state in JSON (not possible anyway)

### Safe Practices

- All assets static and immutable
- References use relative paths within `minecraft:` namespace
- No side effects in asset definitions
- Test with multiple packs enabled (priority order)

## Future Considerations

If the project evolves to include:
- **Datapacks** — Still no concurrency (event-driven, single-threaded)
- **Mods** — Would introduce concurrency (separate project)
- **External tools** — Build-time only, not runtime

## Related Documents

- [Architecture](./architecture.md) — Runtime flow
- [Performance](./testing.md#9-performance-test) — Testing
- [Error Handling](./error-handling.md) — Load failures
- [State and Persistence](./state-and-persistence.md) — No state