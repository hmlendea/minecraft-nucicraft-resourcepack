# System Invariants

Guaranteed properties that hold across all versions and configurations of the NuciCraft Resource Pack.

## Pack-Level Invariants

### I1: Pack Format Compatibility

**Invariant**: Pack loads on Minecraft 1.18+ (pack_format 8-999)

**Guarantee**:
- `pack.mcmeta` specifies `supported_formats.min_inclusive: 84` (1.18)
- `pack.mcmeta` specifies `supported_formats.max_inclusive: 999` (future)
- Target `pack_format: 107.1` (1.21.x)
- No version-specific assets outside supported range

**Verification**: Install on 1.18, 1.19, 1.20, 1.21 — pack appears and loads.

---

### I2: No Code Execution

**Invariant**: Pack contains zero executable code

**Guarantee**:
- Only JSON, PNG, OGG, GLSL files
- No `.class`, `.jar`, `.js`, `.py`, `.lua`, `.dll`, `.so`
- No scripts in any form
- Safe to distribute and run

**Verification**: `find . -type f ! -name "*.json" ! -name "*.png" ! -name "*.ogg" ! -name "*.glsl" ! -name "*.mcmeta" ! -name "*.md" ! -name "*.txt" ! -name "*.yml" ! -name "*.yaml" ! -name "*.toml" ! -name "LICENSE" ! -name "pack.mcmeta" | grep -v ".git"`

---

### I3: Namespace Isolation

**Invariant**: All custom assets use `minecraft:` or `nucicraft:` namespace

**Guarantee**:
- Overrides use `minecraft:` (replace vanilla)
- New custom items use `nucicraft:` (magic_compass, musket sounds)
- No other namespaces introduced
- No conflicts with other packs

**Verification**: `grep -r '"namespace":' assets/ | grep -v "minecraft:" | grep -v "nucicraft:"`

---

### I4: Deterministic Asset Loading

**Invariant**: Same pack version → same asset resolution

**Guarantee**:
- No randomness in pack structure
- Variant selection deterministic per block position (seeded by position)
- Language fallback deterministic (priority order)
- Model baking order deterministic (topological sort)

**Verification**: Load same world twice → identical block variations.

---

## Asset-Level Invariants

### I5: Valid JSON Syntax

**Invariant**: All `.json` files parse without error

**Guarantee**:
- No trailing commas
- All keys quoted
- No comments
- UTF-8 encoding

**Verification**: `find assets -name "*.json" -exec jq empty {} \;`

---

### I6: Texture Dimensions Valid

**Invariant**: All PNG textures are square powers of two

**Guarantee**:
- Width = Height
- Dimensions ∈ {1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024}
- No exceptions

**Verification**: `python3 -c "from PIL import Image; import os; [print(f) for r,d,f in os.walk('assets/minecraft/textures') for f in f if f.endswith('.png') and (lambda p: (lambda w,h: w!=h or (w&(w-1))!=0)(*Image.open(p).size))(os.path.join(r,f))]"`

---

### I7: Model References Resolve

**Invariant**: Every model reference in blockstates/items points to existing model file

**Guarantee**:
- Blockstate `model` → `models/block/*.json` exists
- Item model `parent` → exists
- Item model `overrides[].model` → exists
- No missing model references

**Verification**: Custom script to cross-reference.

---

### I8: Texture References Resolve

**Invariant**: Every texture reference in models points to existing PNG

**Guarantee**:
- Model `textures.*` → `textures/block/*.png` or `textures/item/*.png` exists
- Blockstate multipart models → textures exist
- No magenta textures from missing files

**Verification**: Custom script to cross-reference.

---

### I9: Language Keys Valid Format

**Invariant**: All language keys follow `category.namespace.key` pattern

**Guarantee**:
- Categories: block, item, entity, effect, enchantment, potion, death, gui, commands, stat
- Namespace: minecraft (vanilla) or nucicraft (custom)
- Key: lowercase with underscores

**Verification**: `jq 'keys[]' assets/minecraft/lang/ro_ro.json | grep -vE '^(block|item|entity|effect|enchantment|potion|death|gui|commands|stat)\.(minecraft|nucicraft)\.[a-z0-9_]+$'`

---

### I10: Animation Meta Valid

**Invariant**: All `.mcmeta` files valid JSON with correct animation structure

**Guarantee**:
- `animation` object present
- `frametime` positive integer
- `frames` array of integers/objects
- `interpolate` boolean
- PNG height = width × frame count

**Verification**: `find assets -name "*.mcmeta" -exec jq empty {} \;` + custom frame count check.

---

## Runtime Invariants

### I11: No Purple/Black Blocks (Missing Models)

**Invariant**: No block renders as purple/black checkerboard in normal gameplay

**Guarantee**:
- All vanilla blocks have blockstate entries
- All blockstate variants reference existing models
- All models bake successfully
- Fallback only for truly missing vanilla blocks (none)

**Verification**: Creative mode inventory scan — no purple/black.

---

### I12: No Magenta Textures (Missing Textures)

**Invariant**: No texture renders as bright magenta in normal gameplay

**Guarantee**:
- All model texture references resolve
- All animation PNGs have correct dimensions
- Atlas stitching succeeds (no overflow)
- Fallback only for truly missing vanilla textures (none)

**Verification**: Creative mode inventory scan — no magenta.

---

### I13: Variation Distribution Stable

**Invariant**: Block variation selection deterministic per position

**Guarantee**:
- Same world seed + same position = same variation
- World save/load preserves variations
- Resource pack reload (F3+T) preserves variations
- No re-randomisation on chunk reload

**Verification**: Place block, save, quit, load, check same block → same variation.

---

### I14: Custom Items Functional

**Invariant**: Magic compass and musket work as specified

**Guarantee**:
- Magic compass: CustomModelData 1001 → custom model
- Musket: Crossbow → musket model + sounds + pulling animation
- Both work in all views (GUI, hand, ground, frame)
- Enchantment glint works on both

**Verification**: Give items, test all views, test enchantments.

---

### I15: Language Fallback Works

**Invariant**: Untranslated keys fall back to vanilla English

**Guarantee**:
- `ro_ro.json` overrides Romanian
- `en_us.json` minimal overrides
- Missing keys → vanilla translation
- No key shows as raw key (e.g., `block.minecraft.cobblestone`)

**Verification**: Switch languages, check untranslated blocks.

---

### I16: Atlas No Overflow

**Invariant**: Texture atlas stitching succeeds without overflow

**Guarantee**:
- Total unique textures < atlas capacity
- Atlas size ≤ 8192×8192 (or 16384×16384)
- No "Atlas overflow" in logs
- All textures accessible

**Verification**: Check logs, F3 texture atlas info.

---

## Development Invariants

### I17: Git History Linear on Main

**Invariant**: Main branch has linear history (no merge commits)

**Guarantee**:
- PRs merged via squash or rebase
- No direct pushes to main
- Clean bisectable history

**Verification**: `git log --first-parent --oneline main | grep -c "Merge"`

---

### I18: Release Tags Match Version

**Invariant**: Git tag `vX.Y.Z` matches `pack.mcmeta` description

**Guarantee**:
- Release workflow extracts version from tag
- ZIP named `nucicraft_vX.Y.Z.zip`
- `pack.mcmeta` description = "NuciCraft vX.Y.Z"

**Verification**: `git tag -l "v*"` vs `jq -r .pack.description pack.mcmeta`

---

### I19: Attribution Complete

**Invariant**: All community pack assets attributed in README.md

**Guarantee**:
- Every vendored pack listed
- Version/source noted
- License compatible (GPL v3.0 or compatible)

**Verification**: Manual audit of README.md attributions vs assets.

---

### I20: Documentation Sync

**Invariant**: Documentation reflects current asset structure

**Guarantee**:
- Component docs match actual files
- API reference matches schemas
- Flows match implementation
- Change guide procedures work

**Verification**: Periodic review, automated checks where possible.

---

## Invariant Violation Response

| Invariant | Severity | Response |
|-----------|----------|----------|
| I1-I4 (Pack) | Critical | Block release, fix immediately |
| I5-I10 (Asset) | High | Fix before merge, CI check |
| I11-I16 (Runtime) | High | Fix before release, test check |
| I17-I20 (Dev) | Medium | Fix in next PR, process improvement |

---

## Automated Checks (CI)

### Current

- GitHub Actions: Release packaging only

### Planned

```yaml
# .github/workflows/validate.yml
jobs:
  validate:
    steps:
      - jq empty pack.mcmeta
      - find assets -name "*.json" -exec jq empty {} \;
      - python validate_textures.py
      - python validate_references.py
      - python validate_language.py
```

---

## Related Documents

- [Testing](../testing.md) — Test procedures
- [Change Guide](../change-guide.md) — Procedures maintaining invariants
- [Error Handling](../error-handling.md) — Handling violations
- [Troubleshooting](../troubleshooting.md) — Diagnosing violations
- [Architecture](../architecture.md) — System context