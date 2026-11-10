# Troubleshooting

This document provides solutions for common issues with the NuciCraft Resource Pack.

## Quick Diagnosis

| Symptom | Likely Cause | Quick Fix |
|---------|--------------|-----------|
| Pack not in list | Wrong folder / version | Check `resourcepacks/`, Minecraft 1.18+ |
| Purple/black blocks | Missing model | Report bug / check blockstate |
| Magenta textures | Missing texture | Report bug / check model |
| Names not custom | Wrong language | Set to "Română (România)" |
| Low FPS | Complex models | Reduce render distance, try OptiFine |
| Server pack fails | Bad URL / SHA1 | Verify server.properties |

## Installation Issues

### Pack Not Appearing in Resource Pack Menu

**Causes**:
- ZIP in wrong folder
- Minecraft version too old (< 1.18)
- Corrupted download

**Solutions**:
1. Verify folder: `.minecraft/resourcepacks/` (Windows: `%APPDATA%\.minecraft\resourcepacks\`)
2. Check Minecraft version: 1.18 or newer required
3. Re-download from GitHub Releases
4. Verify SHA1 matches release notes

### "Invalid Pack Format" Error

**Cause**: `pack_format` in `pack.mcmeta` doesn't match client version.

**Solutions**:
- Update Minecraft to 1.21.x (target version)
- Or use older pack version compatible with your Minecraft version

### Pack Loads But Shows Errors in Log

**Check**: `logs/latest.log` for `[ResourcePackLoader]` errors.

**Common**:
- JSON syntax error → Fix `pack.mcmeta` or asset JSON
- Missing file → Asset reference broken

## Visual Issues

### Purple/Black Checkerboard (Missing Model)

**What it means**: Blockstate references a model that doesn't exist.

**Diagnosis**:
1. Note the block name (F3+H for advanced tooltips)
2. Find blockstate: `assets/minecraft/blockstates/<block>.json`
3. Check all `"model"` references
4. Verify model files exist in `assets/minecraft/models/block/`

**Fix**: Add missing model or correct reference.

### Magenta / Bright Pink Texture (Missing Texture)

**What it means**: Model references a texture that doesn't exist.

**Diagnosis**:
1. Note the block/item
2. Find model: `assets/minecraft/models/block/<model>.json` or `models/item/<model>.json`
3. Check `textures` section for all `#variable` references
4. Verify texture files exist in `assets/minecraft/textures/block/` or `textures/item/`

**Fix**: Add missing texture or correct reference.

### Z-Fighting / Flickering Faces

**Cause**: Coplanar faces in model (two faces at same position).

**Fix**: Adjust model element coordinates to avoid overlap.

### Models Look Wrong (Stretched, Rotated)

**Cause**: Incorrect UV mapping or element coordinates.

**Fix**: Check model JSON:
- `from`/`to` coordinates in [0,16]
- UV coordinates in [0,16]
- Face definitions correct

## Language Issues

### Custom Names Not Showing

**Cause**: Language not set to Romanian, or key mismatch.

**Solutions**:
1. Options → Language → "Română (România)"
2. Restart Minecraft (required for language change)
3. If still vanilla names: key may have changed in vanilla

### English Names Not Custom

**Expected**: English variants (`en_us.json`, etc.) have minimal overrides. Most names remain vanilla.

**Reason**: Romanian is primary custom language; English only overrides problematic vanilla names.

### Language Menu Doesn't Show "Română (România)"

**Cause**: `languages.json` missing or malformed.

**Fix**: Verify `assets/minecraft/lang/languages.json` exists and lists `ro_ro`.

## Performance Issues

### Low FPS with Pack Enabled

**Diagnosis**:
1. Open F3 debug screen
2. Check frame time (ms) — should be < 16.67 for 60 FPS
3. Disable pack → compare

**Common Causes**:
- Too many high-poly models in view
- Texture atlas near capacity
- Many texture variations

**Solutions**:
- Reduce render distance
- Install OptiFine or Sodium
- Disable other resource packs
- Report specific blocks causing issues

### Long Reload Time (F3+T)

**Cause**: Large pack (many textures/models).

**Normal**: 2-10 seconds depending on hardware.

**Excessive**: > 30 seconds → may indicate atlas overflow or corrupt files.

### "Out of Memory" / Crashes

**Cause**: Pack too large for allocated memory.

**Solutions**:
- Increase Minecraft memory: `-Xmx4G` or more
- Reduce texture resolution (not applicable — already 16x/32x)
- Report if persistent

## Server Issues

### Server Resource Pack Prompt Not Appearing

**Server Config Check** (`server.properties`):
```properties
resource-pack=https://github.com/.../nucicraft_v6.3.5.zip
resource-pack-sha1=abcdef123456...
resource-pack-prompt="NuciCraft Resource Pack"
resource-pack-enforce=false
```

**Common Issues**:
- URL not HTTPS → Must be HTTPS
- SHA1 mismatch → Verify against release notes
- URL requires auth → Must be public
- Server restarted after config change → Required

### Pack Downloads But Fails to Apply

**Causes**:
- SHA1 mismatch (corrupted download)
- Pack format incompatible with client
- Client cache issue

**Solutions**:
- Clear client resource pack cache: delete `resourcepacks/` contents, re-download
- Verify client version matches pack_format
- Check client logs for errors

### Players Kicked When Joining

**Cause**: `resource-pack-enforce=true` and player declines.

**Solutions**:
- Set `resource-pack-enforce=false` (optional)
- Communicate pack requirement to players
- Provide direct download link as backup

## Specific Feature Issues

### Texture Variations Not Working

**Check**:
1. Blockstate uses `variants` with weighted array
2. Each variant model exists
3. Each model references correct variation texture
4. Textures exist in `textures/block/variated/`

**Debug**: Place 20+ blocks — should see visual differences.

### Magic Compass Not Working

**Requirements**:
- Datapack that sets `CustomModelData: 1001` on compass
- Model override in `models/item/magic_compass.json`

**Without datapack**: Shows as normal compass.

### Musket Not Showing / Sounds Missing

**Check**:
- Crossbow replaced — must use crossbow slot
- Model: `models/item/crossbow.json` (or musket model)
- Sounds: `sounds/item/musket_*.ogg`

### Animated Textures Not Animating

**Check**:
- `.mcmeta` file exists alongside `.png`
- `.mcmeta` has valid `animation` object
- Frame count = height / width (vertical strip)

## Development Issues

### JSON Syntax Errors

**Tool**: `jq empty file.json` — validates syntax.

**Common**:
- Trailing commas (not allowed in JSON)
- Unquoted keys
- Mismatched braces/brackets

### Model Parent Chain Too Deep / Circular

**Symptom**: Model load error, stack overflow.

**Fix**: Flatten parent chain, max depth ~10, no cycles.

### Texture Not Power of Two

**Symptom**: Texture doesn't load, magenta in-game.

**Fix**: Resize to 16, 32, 64, 128, 256, 512, 1024...

### Atlas Overflow

**Symptom**: "Atlas overflow" in logs, magenta textures.

**Fix**: Reduce unique textures (fewer variations, consolidate).

## Getting Help

### Before Reporting

1. Check this troubleshooting guide
2. Check `logs/latest.log` for errors
3. Note Minecraft version, pack version, OS
4. Try with only NuciCraft enabled
5. Try F3+T reload

### Report Template

```markdown
**Minecraft Version**: 1.21.1
**Pack Version**: v6.3.5
**OS**: Linux/Windows/macOS
**Other Packs**: [list or "none"]

**Issue**: [description]

**Steps to Reproduce**:
1. ...
2. ...
3. ...

**Expected**: ...
**Actual**: ...

**Logs**: (relevant excerpt from latest.log)

**Screenshots**: (if visual)
```

### Where to Report

- GitHub Issues: Bug reports, feature requests
- GitHub Discussions: Questions, help

## Related Documents

- [Error Handling](./error-handling.md) — Error categories
- [Logging](./logging.md) — Log analysis
- [FAQ](./faq.md) — Common questions
- [Change Guide](./change-guide.md) — Fixing issues