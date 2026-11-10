# Design Decisions

This document records key architectural and design decisions for the NuciCraft Resource Pack, their rationale, and alternatives considered.

## Decision Records

### ADR-001: Data-Only Resource Pack (No Code)

**Date**: 2024 (project inception)
**Status**: Accepted

**Context**: Need to provide custom 3D models, textures, and translations for Minecraft Java Edition.

**Decision**: Implement as a pure data-only resource pack (JSON, PNG, OGG, GLSL) with no executable code, datapacks, or mods.

**Rationale**:
- Maximum compatibility — works on any Minecraft Java Edition client
- No installation complexity — drag-and-drop ZIP
- No version coupling beyond pack_format
- Server-side distribution via `resource-pack` property
- No security concerns (no code execution)
- Leverages Minecraft's built-in resource pack system

**Alternatives Considered**:
- **Forge/Fabric mod** — Rejected: requires mod loader, server must run same mod, version-locked
- **Datapack + resource pack** — Rejected: datapacks add complexity, not needed for visual changes
- **External launcher/profile** — Rejected: user friction, not portable

**Consequences**:
- Limited to declarative asset definitions
- Cannot add new block/item types (only replace existing)
- Cannot add custom logic (particles, sounds triggered by events beyond vanilla)
- Dependent on Minecraft's resource pack capabilities

---

### ADR-002: Vendoring Community Pack Assets

**Date**: 2024
**Status**: Accepted

**Context**: Many high-quality community resource packs exist (VanillaTweaks, Faithful 32x, MCjossic, etc.) with assets we want to include.

**Decision**: Copy (vendor) community pack assets directly into our repository under standard Minecraft asset paths, rather than using them as external dependencies.

**Rationale**:
- Zero runtime dependencies — pack works standalone
- No external download required by users
- Full control over asset versions
- Can modify/cherry-pick individual assets
- No broken links if upstream disappears
- Simpler distribution (single ZIP)

**Alternatives Considered**:
- **Git submodules** — Rejected: adds complexity, users must clone recursively
- **Runtime download** — Rejected: requires internet, breaks offline play, security concern
- **CurseForge/Modrinth dependencies** — Rejected: platform-specific, not universal

**Consequences**:
- Manual update process required
- Repository size larger (~50MB+)
- License compliance must be tracked per asset
- Potential conflicts between packs (resolved manually)
- Attribution maintenance burden

---

### ADR-003: Romanian as Primary Custom Language

**Date**: 2024
**Status**: Accepted

**Context**: Need custom names for blocks/items (renamed vanilla + new custom items).

**Decision**: Use Romanian (`ro_ro.json`) as the primary custom translation language, with minimal English overrides.

**Rationale**:
- Project author's native language
- Demonstrates full translation capability
- English variants only override where vanilla name is undesirable
- `languages.json` registers Romanian for client language selector

**Alternatives Considered**:
- **English-only custom names** — Rejected: misses translation showcase opportunity
- **Multiple full translations** — Rejected: maintenance burden, author not fluent

**Consequences**:
- Romanian users get fully customised experience
- English users see mostly vanilla names (few overrides)
- Translation maintenance tied to vanilla key changes
- Custom keys use `nucicraft` namespace for new items

---

### ADR-004: Texture Variations via Blockstate Weighted Variants

**Date**: 2024
**Status**: Accepted

**Context**: Want natural-looking variation for blocks like cobblestone, ores, logs without CTM (Connected Textures Mod).

**Decision**: Use blockstate `variants` with weighted random selection pointing to different models, each referencing a unique texture.

**Rationale**:
- Works in vanilla — no mod required
- Simple to understand and maintain
- Per-block control over variation count/weights
- Compatible with all Minecraft versions supporting pack_format

**Alternatives Considered**:
- **CTM (Connected Textures Mod)** — Rejected: requires mod, not vanilla compatible
- **Model predicates with custom model data** — Rejected: items only, not blocks
- **Animated textures cycling variants** — Rejected: looks artificial, not random

**Consequences**:
- Each variation needs separate model file (file count scales)
- Blockstate files grow with variant count
- Atlas space increases (each variation = unique texture)
- Random distribution per block placement (not per chunk)

---

### ADR-005: Pack Format 107.1 Targeting Minecraft 1.21.x

**Date**: 2024
**Status**: Accepted

**Context**: Minecraft 1.21 is current stable; need to choose target version.

**Decision**: Target pack_format 107.1 (Minecraft 1.21.x) with `supported_formats` range 84-999.

**Rationale**:
- 1.21.x is current stable with long-term support
- Wide `supported_formats` range allows older clients to load
- New features (trial chambers, copper bulbs, etc.) available
- Community packs mostly updated to 1.21

**Alternatives Considered**:
- **Target 1.20.6 (pack_format 48)** — Rejected: older, missing new blocks
- **Target snapshot/preview** — Rejected: unstable, breaking changes

**Consequences**:
- Users on 1.20.x can load but miss 1.21-specific assets
- Must update pack_format on each major Minecraft release
- `max_inclusive: 999` is speculative but safe

---

### ADR-006: 3D Models for Fences, Doors, Chains, Crops

**Date**: 2024
**Status**: Accepted

**Context**: Vanilla models are flat/2D for many blocks; community has created 3D variants.

**Decision**: Include 3D models for fences, gates, doors, trapdoors, chains, lanterns, candles, crops, flowers, and more.

**Rationale**:
- Major visual improvement
- Community models already exist (Ecorn, Saplistic, MCjossic, etc.)
- No performance impact on modern hardware
- Enhances immersion significantly

**Alternatives Considered**:
- **Only high-impact models (doors, fences)** — Rejected: inconsistent feel
- **No 3D models** — Rejected: misses core value proposition

**Consequences**:
- More model files to maintain
- Atlas space for additional textures
- Must vet community models for poly count
- Some models may conflict (resolved by priority)

---

### ADR-007: Custom Items via Crossbow Replacement (Musket) and New Item (Magic Compass)

**Date**: 2024
**Status**: Accepted

**Context**: Want custom items but cannot add new item types without datapack/mod.

**Decision**:
- Replace crossbow model/texture with musket (PiXeLGaMeS_or_not)
- Add magic compass as custom model using existing compass item with custom model data predicate

**Rationale**:
- Musket: crossbow is underused, good replacement candidate
- Magic compass: uses custom model data (datapack-compatible), shows advanced capability
- Both demonstrate item model override system

**Alternatives Considered**:
- **Replace multiple vanilla items** — Rejected: breaks vanilla gameplay expectations
- **Datapack for custom items** — Rejected: adds complexity, out of scope

**Consequences**:
- Crossbow users see musket (intentional)
- Magic compass requires datapack for custom model data (documented)
- Limited to item replacement/override patterns

---

### ADR-008: Retro Sound Pack for Damage/Items

**Date**: 2024
**Status**: Accepted

**Context**: Audio customisation enhances atmosphere.

**Decision**: Include retro-style damage sounds and custom item sounds (musket, magic compass).

**Rationale**:
- Nostalgic appeal
- Differentiates from vanilla
- OGG format, vanilla-compatible
- Low file size impact

**Alternatives Considered**:
- **Full sound overhaul** — Rejected: too large, licensing issues
- **No sounds** — Rejected: misses sensory dimension

**Consequences**:
- Sound files in `assets/minecraft/sounds/`
- Must match vanilla sound event names
- Client plays custom sounds automatically

---

### ADR-009: GitHub Actions for Release Packaging

**Date**: 2024
**Status**: Accepted

**Context**: Need automated, reproducible release artifact creation.

**Decision**: Use GitHub Actions workflow triggered on release publish to create ZIP and upload as asset.

**Rationale**:
- Zero infrastructure — runs on GitHub
- Reproducible builds
- SHA1 checksum computed automatically
- Integrated with GitHub Releases
- Free for public repos

**Alternatives Considered**:
- **Local packaging script** — Rejected: not reproducible, manual step
- **External CI (GitLab, CircleCI)** — Rejected: extra account, no benefit
- **Manual ZIP upload** — Rejected: error-prone, no checksum verification

**Consequences**:
- Release process: tag → GitHub Release → auto-package
- ZIP naming: `nucicraft_<tag>.zip`
- Workflow file must be maintained

---

### ADR-010: No Automated Testing (Manual Only)

**Date**: 2024
**Status**: Accepted (with intent to improve)

**Context**: Resource packs are visual/data-driven; testing is inherently visual.

**Decision**: Rely on manual testing for now; document procedures; plan automation for future.

**Rationale**:
- Visual verification requires human eye
- Minecraft client not easily headless-testable
- Current scope small enough for manual testing
- Automation investment high for current value

**Alternatives Considered**:
- **Headless Minecraft + screenshot diff** — Rejected: complex setup, flaky
- **JSON schema validation only** — Accepted as first step (planned)
- **Reference integrity checker** — Accepted as first step (planned)

**Consequences**:
- Release testing takes 30-60 minutes
- Human error possible
- No CI gate on asset validity
- Documented in testing.md with improvement roadmap

---

### ADR-011: GPL v3.0 License

**Date**: 2024
**Status**: Accepted

**Context**: Need license for distribution.

**Decision**: License under GPL v3.0.

**Rationale**:
- Copyleft ensures derivatives remain free
- Compatible with most community pack licenses (MIT, permissive)
- Standard for Minecraft resource packs
- Allows commercial use (with source distribution)

**Alternatives Considered**:
- **MIT** — Rejected: allows proprietary derivatives
- **CC-BY-SA** — Rejected: not ideal for code/assets mix
- **Custom** — Rejected: legal uncertainty

**Consequences**:
- Faithful 32x (non-commercial) compatible only for non-commercial use
- Derivatives must be GPL v3.0
- Source must be available

---

### ADR-012: Documentation in /docs Directory (Generated)

**Date**: 2025
**Status**: Accepted

**Context**: User requested exhaustive documentation for institutional knowledge preservation.

**Decision**: Generate comprehensive documentation under `/docs` with structure: `api-reference/`, `behaviour/`, `components/`, `flows/` plus root-level guides.

**Rationale**:
- Separates documentation from assets
- Structured for maintainability
- Supports the 16-aspect quality gate requirement
- Version-controlled with code

**Alternatives Considered**:
- **Wiki** — Rejected: separate from repo, not versioned with code
- **README only** — Rejected: insufficient for exhaustive docs
- **External site** — Rejected: extra maintenance

**Consequences**:
- 30+ markdown files to maintain
- Must stay in sync with asset changes
- Generated initially, then maintained manually

---

## Decision Template (for future decisions)

```markdown
### ADR-XXX: <Title>

**Date**: YYYY-MM-DD
**Status**: Proposed | Accepted | Superseded | Rejected

**Context**: <What problem are we solving?>

**Decision**: <What did we decide?>

**Rationale**: <Why this decision?>

**Alternatives Considered**:
- <Alternative 1> — <Why rejected>
- <Alternative 2> — <Why rejected>

**Consequences**: <What are the implications?>
```