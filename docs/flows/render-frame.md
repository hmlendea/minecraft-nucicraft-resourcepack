# Render Frame Flow

End-to-end flow: chunk data → GPU → pixels per frame.

## Overview

```
Chunk Rebuild → Block State Lookup → Model Lookup → Vertex Buffer → Draw Calls → GPU → Display
```

## Phase 1: Chunk Rebuild

### Trigger

- Chunk loaded
- Block changed
- Light updated
- Resource pack reload

### Process

1. **ChunkCache** rebuilds chunk sections (16×16×16)
2. **Iterate** all block positions in section
3. **Get** `BlockState` for each position
4. **Lookup** `BlockStateModel` from registry
5. **Collect** `BakedModel` quads
6. **Transform** quads to chunk coordinates
7. **Batch** by render layer
8. **Upload** to VBO (Vertex Buffer Object)

### Render Layers

| Layer | Blocks | Transparency |
|-------|--------|--------------|
| `solid` | Stone, dirt, wood | Opaque |
| `translucent` | Glass, water, leaves | Alpha blend |
| `cutout` | Flowers, grass | Alpha test |
| `cutout_mipped` | Leaves (mipmapped) | Alpha test + mipmap |
| `translucent_no_crumbling` | Particles | Additive |

## Phase 2: Block State → Model Lookup

### BlockStateModel Registry

Built during model baking (see model-baking.md).

```
BlockState → BlockStateModel → List<BakedModel>
```

### Variants Resolution

For variants blockstates:
1. **Match** block state properties to variant key
2. **Select** model (weighted random if array)
3. **Return** BakedModel

### Multipart Resolution

For multipart blockstates:
1. **Evaluate** each part's `when` condition
2. **Collect** matching part models
3. **Compose** into combined BakedModel (at bake time)
4. **Return** combined model

### Caching

- Chunk rebuild caches model references
- Invalidate on block change or resource reload

## Phase 3: Vertex Buffer Generation

### Per Chunk Section

For each render layer:

1. **Allocate** VBO (or reuse)
2. **For each block** in section:
   - Get BakedModel
   - Get quads (general + face-culled)
   - **Transform** vertices: local → chunk coordinates
   - **Apply** block entity transforms (if any)
   - **Write** to buffer
3. **Upload** to GPU (glBufferData)

### Vertex Data

Per vertex (from BakedModel):
```
Position (x,y,z)        - chunk-relative
Normal (nx,ny,nz)       - face normal
UV (u,v)                - atlas coordinates
Color (r,g,b,a)         - tint + AO
Lightmap (u,v)          - block + sky light
```

### Face Culling

At bake time: faces with `cullface` removed if neighbor solid.
At render time: GPU backface culling (CCW winding).

## Phase 4: Draw Calls

### Per Frame

For each render layer:

1. **Bind** shader program
2. **Bind** texture atlas
3. **Set** uniforms (projection, view, lightmap)
4. **For each chunk** in view frustum:
   - **Frustum cull** (chunk AABB vs view frustum)
   - **Bind** chunk VBO
   - **Draw** (glDrawArrays / glDrawElements)
5. **Unbind**

### Shader Uniforms

| Uniform | Description |
|---------|-------------|
| `Projection` | Projection matrix |
| `ModelView` | View matrix |
| `Lightmap` | Lightmap texture |
| `Texture` | Texture atlas |
| `FogColor` | Fog color |
| `FogStart/End` | Fog distances |

## Phase 5: Item Rendering

### Per Item Stack

1. **Resolve** model:
   - Base model from item
   - Check overrides (custom_model_data, pulling, etc.)
   - Get override BakedModel or base
2. **Get** display transform for view:
   - `gui`, `ground`, `fixed`, `firstperson_righthand`, etc.
3. **Apply** transform to model vertices
4. **Render** with item shader

### Item Shader

Separate from block shader:
- No lightmap (uses fixed brightness)
- Different fog handling
- Enchantment glint effect (layer1)

## Phase 6: Entity Rendering

### Entity Models

- Separate model system (not resource pack models)
- Resource pack can override textures only
- Geometry defined in code (EntityModel classes)

### Texture Binding

- Entity texture from `textures/entity/`
- Bound per entity type

## Phase 7: Post-Processing

### Effects

1. **Weather** (rain/snow) — particle systems
2. **Fog** — shader-based
3. **Bloom** — if shader pack (Iris/OptiFine)
4. **Shadows** — if shader pack

### GUI Overlay

- Hotbar, inventory, HUD
- Separate render pass
- Orthographic projection

## Performance

### Bottlenecks

| Stage | Typical Cost | Optimization |
|-------|--------------|--------------|
| Chunk rebuild | High (on change) | Incremental updates |
| Vertex upload | Medium | Buffer orphaning |
| Draw calls | High (many chunks) | Frustum culling, batching |
| State changes | Medium | Sort by layer/texture |
| Shader switches | Low | Minimize programs |

### Metrics (F3)

- **FPS**: Frames per second
- **Frame time**: ms per frame (target < 16.67 for 60 FPS)
- **Chunks rendered**: Visible chunks
- **Entities rendered**: Visible entities
- **Draw calls**: Per frame

## Debugging

### Visual Debug

- F3+B: Hitboxes
- F3+G: Chunk borders
- F3+T: Reload resources

### Logs

```
[ChunkCache] Rebuilt chunk 12,34 in 45ms
[RenderSystem] 1234 draw calls, 567890 vertices
```

### Wireframe

Debug option: render wireframe (shows geometry).

## Shader Pack Integration (Iris/OptiFine)

### Pipeline Changes

1. **Deferred** rendering (G-buffer)
2. **Shadow maps** (cascaded)
3. **Post-process** chain
4. **Custom uniforms** for shaders

### Resource Pack Compatibility

- Vanilla shaders still work
- Custom shaders in `shaders/core/` ignored by Iris
- Use Iris shader packs instead

## Related Documents

- [Flows: Pack Load](./pack-load.md)
- [Flows: Model Baking](./model-baking.md)
- [Components: Block Models](../components/block-models.md)
- [Components: Blockstates](../components/blockstates.md)
- [Behaviour: Variation Rendering](../behaviour/variation-rendering.md)
- [Troubleshooting](../troubleshooting.md#performance-issues)