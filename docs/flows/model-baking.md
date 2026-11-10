# Model Baking Flow

End-to-end flow: model JSON → baked GPU-ready model.

## Overview

```
Model JSON → Dependency Resolution → Geometry Processing → BakedModel → ModelManager
```

## Phase 1: Dependency Collection

### Trigger

- Resource pack load complete
- F3+T reload
- New model reference encountered

### Process

1. **Scan** all `models/block/*.json` and `models/item/*.json`
2. **Parse** each model JSON
3. **Extract** `parent` references
4. **Build** dependency graph

### Dependency Graph

```
cobblestone.json
  └─ parent: minecraft:block/block
     └─ (vanilla built-in)

oak_fence_post.json
  └─ parent: minecraft:block/template_fence_post
     └─ (vanilla built-in)

magic_compass.json
  └─ parent: minecraft:item/compass
     └─ (vanilla built-in)
```

## Phase 2: Topological Sort

### Algorithm

1. **Identify** root models (no parent or vanilla parent)
2. **Sort** by dependency depth (parents before children)
3. **Detect** cycles (error if found)

### Order Example

1. `minecraft:block/block` (vanilla)
2. `minecraft:block/cube_column` (vanilla)
3. `birch_log/birch_log.json` (parent: cube_column)
4. `cobblestone.json` (parent: block)
5. `variated/cobblestone_variated_1.json` (parent: block)

## Phase 3: Model Processing (Per Model)

### Input

Raw model JSON + resolved parent baked data

### Steps

#### 3.1 Texture Resolution

```json
"textures": {
  "particle": "minecraft:block/cobblestone",
  "all": "minecraft:block/cobblestone"
}
```

- Resolve all `#variable` references
- Look up in atlas → get UV coordinates
- Particle texture → breaking particles, map icon

#### 3.2 Element Processing

For each element in `elements[]`:

1. **Vertices**: Generate 8 corners from `from`/`to`
2. **Faces**: For each of 6 faces:
   - Check `cullface` — skip if neighbor block present
   - Compute UV from `uv` + texture atlas coords
   - Apply face `rotation` (0, 90, 180, 270)
   - Apply element `rotation` if present
   - Compute normal
   - Assign `tintindex`
3. **Culling**: Remove faces with `cullface` matching solid neighbor

#### 3.3 Transform Application

- **GUI transform**: From `display.gui`
- **Ground transform**: From `display.ground`
- **Fixed transform**: From `display.fixed`
- **Hand transforms**: From `display.firstperson_*`, `thirdperson_*`

#### 3.4 Vertex Format

Output per vertex:
```
Position (x,y,z)     - 3 floats
Normal (nx,ny,nz)    - 3 floats
UV (u,v)             - 2 floats
Color (r,g,b,a)      - 4 bytes (tint)
Lightmap UV          - 2 shorts
```

### Output: BakedModel

```java
class BakedModel {
  List<BakedQuad> quads;           // General quads
  List<BakedQuad>[] faceQuads;     // Per-face culling
  boolean ambientOcclusion;
  boolean gui3d;
  TextureAtlasSprite particleIcon;
  ItemCameraTransforms transforms;
  ItemOverrides overrides;
}
```

## Phase 4: Item Override Processing

### For Item Models with Overrides

```json
"overrides": [
  { "predicate": { "custom_model_data": 1001 }, "model": "magic_compass" }
]
```

### Processing

1. **Collect** all override predicates
2. **Sort** by specificity (more predicates = higher priority)
3. **Create** `ItemOverrides` handler
4. **At render time**: Evaluate predicates → return override model

### Predicate Evaluation Order

1. Most specific first (most predicate fields)
2. First match wins
3. Fallback to base model

## Phase 5: Caching & Registration

### ModelManager Cache

- Key: `ResourceLocation` (e.g., `minecraft:block/cobblestone`)
- Value: `BakedModel`
- Invalidate on: Resource reload, pack change

### BlockStateModel Registry

Maps block state → BakedModel:

```
minecraft:cobblestone[default] → BakedModel(cobblestone)
minecraft:oak_fence[north=true,east=false,...] → BakedModel(composed)
```

## Special Model Types

### Compass/Clock

- Parent: `minecraft:item/compass` or `minecraft:item/clock`
- Special: `TextureAtlasSprite` animated by game logic
- BakedModel includes custom `IBakedModel` implementation

### Crossbow/Bow

- Parent: `minecraft:item/crossbow` or `minecraft:item/bow`
- Overrides for pulling animation
- Special: `pulling` and `pull` predicates

### Multipart (Blockstates)

- Not a model type — composed at blockstate level
- Each part baked separately
- Combined at render time

## Error Handling

| Error | Result |
|-------|--------|
| Parent missing | Fallback to `missing` model (purple/black) |
| Circular parent | Error logged, fallback to `missing` |
| Texture missing | Magenta UV coords |
| Element coords invalid | Element skipped |
| JSON syntax error | Model skipped |

## Performance

### Baking Time Factors

| Factor | Impact |
|--------|--------|
| Model count | Linear |
| Element count per model | Linear |
| Face count per element | Linear |
| Override count | Per-frame evaluation |
| Parent chain depth | One-time resolution |

### Optimization

- Flatten parent chains
- Reduce element count
- Use cullface aggressively
- Minimize overrides

## Debugging

### Logs

```
[ModelBakery] Baking model: minecraft:block/cobblestone
[ModelBakery] Loaded 1234 models in 456ms
```

### In-Game

F3+T → watch log for baking messages.

### Missing Model

Purple/black checkerboard = `missing` model fallback.

## Related Documents

- [Flows: Pack Load](./pack-load.md)
- [Flows: Render Frame](./render-frame.md)
- [Components: Block Models](../components/block-models.md)
- [Components: Item Models](../components/item-models.md)
- [Components: Blockstates](../components/blockstates.md)
- [API Reference](../api-reference/INDEX.md)