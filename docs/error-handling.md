# Error Handling

This document describes error handling patterns, failure modes, and recovery procedures for the NuciCraft Resource Pack.

## Error Categories

### 1. Pack Load Errors

**Trigger**: Client reads `pack.mcmeta` on resource pack enable.

| Error | Cause | Client Behaviour | Resolution |
|-------|-------|------------------|------------|
| Invalid JSON | Syntax error in `pack.mcmeta` | Pack ignored, error in log | Fix JSON syntax |
| Missing `pack_format` | Field absent | Pack ignored | Add `pack_format` |
| `pack_format` too high | > client's supported format | Pack ignored | Lower `pack_format` or update client |
| `pack_format` too low | < `min_inclusive` | Pack ignored | Raise `min_inclusive` or update client |
| Missing `description` | Field absent | Pack loads, no description | Add description |

### 2. Asset Load Errors

**Trigger**: Asset Manager indexes `assets/minecraft/**`.

| Error | Cause | Client Behaviour | Resolution |
|-------|-------|------------------|------------|
| Malformed JSON | Syntax error in any `.json` | Asset skipped, error logged | Fix JSON syntax |
| Missing file | Referenced asset doesn't exist | Depends on reference type | Add missing file or fix reference |
| Invalid schema | JSON doesn't match expected schema | Asset skipped, error logged | Fix schema compliance |

### 3. Model Baking Errors

**Trigger**: Client bakes models from blockstates + model definitions.

| Error | Cause | Client Behaviour | Resolution |
|-------|-------|------------------|------------|
| Missing variant model | Blockstate references non-existent model | Purple/black checkerboard | Create model or fix reference |
| Circular parent chain | Model A → Model B → Model A | Model skipped, error logged | Break cycle |
| Invalid element coordinates | `from`/`to` outside [0,16] | Model skipped, error logged | Fix coordinates |
| Invalid UV coordinates | UV outside atlas bounds | Texture distortion | Fix UV mapping |
| Missing texture variable | `#texture` not defined | Magenta texture | Define texture variable |

### 4. Texture Errors

**Trigger**: Client loads textures for atlas building.

| Error | Cause | Client Behaviour | Resolution |
|-------|-------|------------------|------------|
| Non-power-of-two dimensions | Width/height not 16, 32, 64... | Texture may not load | Resize to power of two |
| Corrupted PNG | Invalid PNG data | Texture skipped, error logged | Re-export PNG |
| Missing animation .mcmeta | Animated texture without metadata | Static texture (first frame) | Add .mcmeta or remove animation |
| Atlas overflow | Too many unique textures | Some textures become magenta | Reduce texture count/variations |

### 5. Language Errors

**Trigger**: Client registers translation overrides.

| Error | Cause | Client Behaviour | Resolution |
|-------|-------|------------------|------------|
| Invalid JSON | Syntax error in `.json` | File skipped, vanilla used | Fix JSON syntax |
| Unknown key | Key doesn't exist in vanilla | Key ignored (no effect) | Use correct vanilla key |
| Duplicate key | Same key twice in file | Last value wins | Remove duplicate |

### 6. Runtime Render Errors

**Trigger**: During frame rendering.

| Error | Cause | Client Behaviour | Resolution |
|-------|-------|------------------|------------|
| Missing baked model | Block state has no valid variant | Purple/black checkerboard | Fix blockstate variants |
| Magenta texture | Model references missing texture | Magenta placeholder | Add texture or fix reference |
| Z-fighting | Coplanar faces | Flickering | Adjust model geometry |
| Performance drop | Too many complex models | Low FPS | Simplify models, reduce variations |

## Error Detection

### Client Logs

Check `logs/latest.log` in Minecraft directory for:
- `[ResourcePackLoader]` — Pack load errors
- `[ModelBakery]` — Model baking errors
- `[TextureAtlas]` — Texture atlas errors
- `[Language]` — Language load errors

### In-Game Indicators

| Visual | Meaning |
|--------|---------|
| Purple/black checkerboard | Missing block/item model |
| Magenta (bright pink) texture | Missing texture |
| Missing text (key shown) | Missing translation |
| Flickering faces | Z-fighting |

### Debug Screen (F3)

- **Right side**: Resource pack status, errors
- **Frame time**: Performance issues

## Recovery Procedures

### Pack Won't Load

1. Check `pack.mcmeta` JSON syntax: `jq empty pack.mcmeta`
2. Verify `pack_format` matches client version
3. Check client log for specific error
4. Fix and reload (F3+T)

### Missing Models (Purple/Black)

1. Identify block/item from creative menu or world
2. Find its blockstate file: `assets/minecraft/blockstates/<name>.json`
3. Check all referenced models exist in `models/block/` or `models/item/`
4. Create missing models or fix references
5. Reload (F3+T)

### Missing Textures (Magenta)

1. Identify block/item with magenta texture
2. Find its model: `assets/minecraft/models/block/<name>.json` or `models/item/<name>.json`
3. Check `textures` section for all referenced textures
4. Verify texture files exist in `assets/minecraft/textures/`
5. Add missing textures or fix references
6. Reload (F3+T)

### Language Not Working

1. Check `assets/minecraft/lang/<lang>.json` syntax: `jq empty file.json`
2. Verify keys match vanilla exactly (case-sensitive)
3. Check `languages.json` includes the language
4. Restart Minecraft (language changes require restart)

### Performance Issues

1. Open F3 debug screen
2. Check frame time > 16.67ms (60 FPS)
3. Identify high-poly models: look for complex custom blocks
4. Reduce model complexity or variation count
5. Check texture atlas size (if visible in logs)
6. Consider removing least-impact variations

## Validation Tools

### JSON Syntax

```bash
# Validate all JSON files
find assets -name "*.json" -exec jq empty {} \;

# Validate specific file
jq empty assets/minecraft/blockstates/oak_fence.json
```

### Texture Dimensions

```bash
# Check all PNGs are power of two
find assets -name "*.png" | while read f; do
  dims=$(identify -format "%w %h" "$f")
  w=$(echo $dims | cut -d' ' -f1)
  h=$(echo $dims | cut -d' ' -f2)
  if ! (( (w & (w-1)) == 0 && w > 0 )); then
    echo "WARN: $f width $w not power of 2"
  fi
  if ! (( (h & (h-1)) == 0 && h > 0 )); then
    echo "WARN: $f height $h not power of 2"
  fi
done
```

### Reference Integrity (Manual)

```bash
# Check blockstate model references
grep -r '"model"' assets/minecraft/blockstates/ | \
  sed 's/.*"model": "\([^"]*\)".*/\1/' | \
  while read model; do
    if [ ! -f "assets/minecraft/models/${model#minecraft:}.json" ]; then
      echo "MISSING: $model"
    fi
  done

# Check model texture references
grep -r '#[a-z_]*":' assets/minecraft/models/ | \
  sed 's/.*#\([a-z_]*\)": "\([^"]*\)".*/\1 \2/' | \
  while read var texture; do
    if [ ! -f "assets/minecraft/textures/${texture#minecraft:}.png" ]; then
      echo "MISSING TEXTURE: $texture (var: $var)"
    fi
  done
```

## Error Prevention Checklist

Before committing changes:

- [ ] All JSON files pass `jq empty`
- [ ] All blockstate model references resolve
- [ ] All model texture references resolve
- [ ] All texture PNGs are power-of-two dimensions
- [ ] No circular model parent chains
- [ ] Language keys match vanilla (spot check)
- [ ] Tested in Minecraft (F3+T reload)
- [ ] No purple/black checkerboard
- [ ] No magenta textures
- [ ] Performance acceptable

## Escalation

If error cannot be resolved:

1. Document error with steps to reproduce
2. Check Minecraft version compatibility
3. Check community pack source for similar issues
4. Create GitHub issue with:
   - Minecraft version
   - Error log excerpt
   - Steps to reproduce
   - Screenshots if visual
5. Consider reverting recent changes if regression