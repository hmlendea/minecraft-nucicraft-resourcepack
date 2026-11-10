# Testing

This document describes the testing strategy, procedures, and verification methods for the NuciCraft Resource Pack.

## Current State

**No automated test suite exists.** All verification is manual.

## Testing Strategy

### Test Levels

| Level | Scope | Automation | Frequency |
|-------|-------|------------|-----------|
| Unit | Individual asset validity (JSON syntax, PNG format) | None (manual) | Per change |
| Integration | Asset reference integrity (blockstate to model to texture) | None (manual) | Per release |
| System | Full pack load in Minecraft client | None (manual) | Per release |
| Regression | Visual/functional comparison vs previous version | None (manual) | Per release |
| Performance | Render performance, atlas size | None (manual) | Per release |
| Compatibility | Multiple Minecraft versions | None (manual) | Per Minecraft update |

### Test Environment

- **Primary**: Minecraft Java Edition 1.21.x (target version)
- **Secondary**: Minecraft 1.20.5, 1.20.6 (pack_format 48)
- **Minimum**: Minecraft 1.18 (pack_format 8, min_inclusive)
- **OS**: Linux, Windows, macOS (client behaviour identical)

## Manual Test Procedures

### 1. Pack Load Test

**Objective**: Verify pack loads without errors.

**Steps**:
1. Install pack ZIP in `resourcepacks/`
2. Launch Minecraft
3. Open Resource Packs menu
4. Enable NuciCraft
5. Verify no error toast appears
6. Verify description shows "NuciCraft v<version>"

**Pass Criteria**: Pack appears in list, enables without error, description correct.

### 2. Block Model Rendering Test

**Objective**: Verify all custom 3D block models render correctly.

**Test Blocks** (priority):
- Fences/gates: all wood types + nether brick
- Doors/trapdoors: all wood types + iron
- Chains, lanterns, candles
- Crops: beetroot, carrot, potato, melon, pumpkin, sweet berry
- Flowers, grass, ferns
- Redstone dust, repeaters, comparators
- Buckets, goat horn, trident
- Custom: magic compass, musket

**Steps**:
1. Create creative world
2. Place each test block
3. View from multiple angles
4. Check for:
   - Purple/black checkerboard (missing model)
   - Magenta texture (missing texture)
   - Z-fighting, clipping
   - Incorrect rotation/scale

**Pass Criteria**: All blocks render correctly, no missing models/textures.

### 3. Texture Variation Test

**Objective**: Verify randomised texture variations appear.

**Test Blocks**:
- Birch logs (variated bark)
- Cobblestone (variated)
- Ores: coal, iron, gold, diamond, emerald, copper, redstone, lapis
- Stone variants: andesite, diorite, granite
- Bricks, stone bricks
- Obsidian (variated)
- Prismarine bricks

**Steps**:
1. Place multiple instances of each block
2. Verify visual differences between instances
3. Verify variations are subtle (not jarring)

**Pass Criteria**: Variations visible, natural-looking, no identical adjacent blocks.

### 4. Item Model and Display Test

**Objective**: Verify item models and display transforms.

**Test Items**:
- All tools/weapons/armor (3D models)
- Custom: magic compass, musket
- Food, blocks, materials
- Items in: hand (1st/3rd person), inventory, item frame, ground, head

**Steps**:
1. Obtain each item in creative
2. Cycle through perspectives (F5)
3. Open inventory, creative menu
4. Place in item frame
5. Drop on ground
6. Wear on head (if applicable)

**Pass Criteria**: Correct model in all perspectives, no missing textures, proper scale/rotation.

### 5. Language/Translation Test

**Objective**: Verify custom names appear in Romanian and English.

**Test Keys** (sample):
- `block.minecraft.bamboo` to "Bambus" (ro_ro)
- `block.minecraft.cherry_planks` to "Placi de cireș"
- `item.minecraft.crossbow` to "Muskete" (musket replacement)
- Custom: `item.nucicraft.magic_compass` to "Busolă Magică"

**Steps**:
1. Set language to "Română (România)"
2. Check creative menu, tooltips, inventory
3. Switch to English (US)
4. Verify English variants where overridden

**Pass Criteria**: Custom names appear in Romanian, English fallbacks work.

### 6. Animation Test

**Objective**: Verify animated textures play correctly.

**Test Animations**:
- Candles (flicker)
- Crying obsidian
- Animated terrain (water, lava if modified)
- Custom animated textures

**Steps**:
1. Place animated blocks
2. Observe for several seconds
3. Verify smooth animation, correct timing

**Pass Criteria**: Animations play smoothly, no frozen frames.

### 7. Shader Test (if applicable)

**Objective**: Verify shaders load and render.

**Steps**:
1. Enable shaders in video settings (if pack provides)
2. Verify no console errors
3. Check visual effects

**Pass Criteria**: Shaders apply without errors.

### 8. Sound Test

**Objective**: Verify custom sounds play.

**Test Sounds**:
- Damage sounds (retro)
- Item sounds (musket, magic compass)

**Steps**:
1. Take damage, use items
2. Listen for custom sounds

**Pass Criteria**: Custom sounds play, no missing sound errors.

### 9. Performance Test

**Objective**: Verify acceptable render performance.

**Steps**:
1. Open debug screen (F3)
2. Create world with many custom blocks
3. Monitor:
   - Frame time (ms)
   - Entity count
   - Texture atlas size (if visible)
4. Compare with vanilla

**Pass Criteria**: Frame time < 16.67ms (60 FPS) on mid-range hardware, no significant regression vs vanilla.

### 10. Server Resource Pack Test

**Objective**: Verify server-prompted installation works.

**Steps**:
1. Configure server with `resource-pack` URL and SHA1
2. Connect with vanilla client
3. Accept prompt
4. Verify pack downloads and enables
5. Verify all features work on server

**Pass Criteria**: Pack downloads, enables, functions identically to local install.

## Regression Testing

### Visual Regression

No automated visual regression. Manual comparison:
1. Screenshot key scenes in previous version
2. Screenshot same scenes in new version
3. Compare side-by-side
4. Check for unintended changes

### Functional Regression

Test all features from previous release still work:
- All 3D models
- All texture variations
- All custom names
- All animations
- All sounds

## Compatibility Testing

### Minecraft Version Matrix

| Version | Pack Format | Test Status |
|---------|-------------|-------------|
| 1.21.x | 107.1 | Primary |
| 1.20.5-1.20.6 | 48 | Secondary |
| 1.20.3-1.20.4 | 41 | Secondary |
| 1.20-1.20.2 | 15 | Secondary |
| 1.19.3-1.19.4 | 12 | Secondary |
| 1.18-1.19.2 | 8-9 | Secondary |

**Note**: Only 1.21.x is fully tested. Older versions may have missing features or errors due to schema changes.

### Platform Testing

| Platform | Test Status |
|----------|-------------|
| Linux | Primary |
| Windows | Untested |
| macOS | Untested |

## Test Data

### Test World

Create a test world with:
- All custom blocks placed
- All custom items in chests
- Redstone contraptions with custom dust
- Farms with custom crops
- Decorative builds with variations

### Test Resource Pack

Use the release ZIP for testing (not loose files) to match user experience.

## Automated Checks (Future)

### Recommended Automated Validation

| Check | Tool | Integration |
|-------|------|-------------|
| JSON syntax validation | `jq` or JSON Schema validator | Pre-commit / CI |
| JSON schema compliance | Minecraft schema files | CI |
| Reference integrity | Custom script | CI |
| Texture dimensions | `file` + `identify` (ImageMagick) | CI |
| Atlas size estimation | Custom script | CI |
| Duplicate texture detection | Perceptual hash | CI |
| Language key completeness | Diff vs vanilla keys | CI |

### Example CI Job

```yaml
name: Validate Assets
on: [push, pull_request]
jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Validate JSON syntax
        run: find assets -name "*.json" -exec jq empty {} \;
      - name: Validate texture dimensions
        run: |
          find assets -name "*.png" | while read f; do
            dims=$(identify -format "%w %h" "$f")
            w=$(echo $dims | cut -d' ' -f1)
            h=$(echo $dims | cut -d' ' -f2)
            if ! (( (w & (w-1)) == 0 && w > 0 )); then
              echo "ERROR: $f width $w not power of 2"
              exit 1
            fi
            if ! (( (h & (h-1)) == 0 && h > 0 )); then
              echo "ERROR: $f height $h not power of 2"
              exit 1
            fi
          done
      - name: Check reference integrity
        run: python3 scripts/check_references.py
```

## Test Reporting

### Release Test Report Template

```markdown
# NuciCraft v<version> Test Report

## Environment
- Minecraft: 1.21.x
- OS: Linux
- Java: 21
- GPU: <model>

## Test Results

| Test | Status | Notes |
|------|--------|-------|
| Pack Load | Pass/Fail | |
| Block Models | Pass/Fail | List failures |
| Texture Variations | Pass/Fail | |
| Item Models | Pass/Fail | |
| Language (ro_ro) | Pass/Fail | |
| Language (en_*) | Pass/Fail | |
| Animations | Pass/Fail | |
| Sounds | Pass/Fail | |
| Performance | Pass/Fail | Frame time: X ms |
| Server Pack | Pass/Fail | |

## Issues Found
- [ ] Issue 1
- [ ] Issue 2

## Sign-off
Tested by: <name>
Date: <date>
```

## Known Test Gaps

1. **No automated testing** — All manual, error-prone, slow
2. **Limited version matrix** — Only 1.21.x thoroughly tested
3. **No performance benchmarks** — Subjective "feels fast"
4. **No visual regression** — Manual screenshot comparison only
5. **No accessibility testing** — Color blindness, etc.
6. **No multiplayer stress testing** — Server with many players

## Test Improvement Roadmap

1. Add JSON syntax validation to CI
2. Add texture dimension validation to CI
3. Build reference integrity checker
4. Create automated test world schematic
5. Add performance benchmark script
6. Implement visual regression with screenshot diff
7. Test on Windows/macOS in CI (GitHub Actions matrix)