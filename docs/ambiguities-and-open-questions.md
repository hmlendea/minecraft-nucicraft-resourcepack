# Ambiguities and Open Questions

This document tracks known ambiguities, unresolved decisions, and open questions for the NuciCraft Resource Pack.

## Open Questions

### Q1: Pack Format Versioning Strategy

**Question**: How to handle Minecraft version upgrades that change pack_format?

**Current State**: Target 1.21.x (pack_format 107.1), `supported_formats` 84-999.

**Options**:
- Single pack with wide `supported_formats` (current) — simpler, but older clients miss new assets
- Separate packs per major version — more work, but optimal per version
- Datapack companion for version-specific logic — adds complexity

**Decision Needed**: Before 1.22 release.

---

### Q2: Faithful 32x License Compliance for Commercial Use

**Question**: Faithful 32x has non-commercial license. NuciCraft is GPL v3.0 (allows commercial). Is this compatible?

**Current State**: NuciCraft distributed free (non-commercial), so compatible.

**Risk**: If NuciCraft ever used commercially, Faithful 32x textures must be replaced.

**Decision Needed**: Clarify project commercial intent; find 32x alternative if needed.

---

### Q3: Texture Atlas Capacity Limits

**Question**: How many unique textures before atlas overflow?

**Current State**: ~2000+ textures, no overflow reported on 1.21.x.

**Unknowns**:
- Exact limit varies by GPU/driver
- No automated monitoring
- Community pack updates add textures

**Decision Needed**: Implement atlas size estimation in CI; set variation budget.

---

### Q4: Model Complexity Budget

**Question**: What poly count / element count is acceptable per model?

**Current State**: Community models vetted manually, no hard limits.

**Unknowns**:
- Performance impact on low-end hardware
- No automated measurement
- Some models (doors, chains) inherently complex

**Decision Needed**: Define budget (e.g., max 50 elements per model); add validation.

---

### Q5: Language Maintenance Burden

**Question**: How to keep Romanian translations in sync with vanilla key changes?

**Current State**: Manual updates when vanilla changes noticed.

**Options**:
- Script to diff against vanilla translation files
- Crowdsourced translations (Weblate, Crowdin)
- Accept drift, fix on request

**Decision Needed**: Before next major Minecraft version with many key changes.

---

### Q6: Custom Item Expansion

**Question**: Add more custom items beyond magic compass and musket?

**Current State**: Two custom items demonstrating capabilities.

**Options**:
- More crossbow replacements (bow, trident)
- Custom model data items (require datapack)
- Custom armor (requires datapack for attributes)

**Constraints**: Cannot add new item types without datapack/mod.

**Decision Needed**: Scope of custom items vs. datapack dependency.

---

### Q7: Shader Pack Inclusion

**Question**: Include shaders in the pack?

**Current State**: `shaders/core/` directory exists, empty.

**Options**:
- Include popular shader (BSL, Complementary) — licensing, size
- Write custom lightweight shader — maintenance
- Leave empty, document how to add — current

**Decision Needed**: If shaders desired, choose approach.

---

### Q8: Automated Testing Investment

**Question**: How much to invest in automated testing?

**Current State**: Manual only, documented in testing.md.

**Options**:
- Minimal: JSON syntax + texture dimensions in CI (low effort, high value)
- Medium: Reference integrity checker (medium effort, high value)
- Full: Headless Minecraft + screenshot diff (high effort, flaky)

**Decision Needed**: Prioritise first two; defer full visual regression.

---

### Q9: Community Pack Update Automation

**Question**: Automate community pack vendoring?

**Current State**: Fully manual.

**Options**:
- Script to fetch releases, extract, diff, apply
- GitHub Action on schedule
- Stay manual (infrequent updates)

**Decision Needed**: If update frequency increases, automate.

---

### Q10: Documentation Completeness vs. Maintenance

**Question**: 30+ doc files — sustainable to maintain?

**Current State**: Generated initially, must stay in sync.

**Risk**: Docs drift from reality.

**Options**:
- Reduce scope (fewer files, more reference)
- Automate generation from assets
- Accept drift, fix on major releases

**Decision Needed**: After first few release cycles.

---

## Ambiguities

### A1: "NuciCraft" Brand Scope

**Ambiguity**: Does "NuciCraft" refer to just this resource pack, or a broader project (server, datapacks, mods)?

**Impact**: Naming, licensing, future expansion.

**Resolution**: Currently resource pack only. Document if expands.

---

### A2: Version Numbering Scheme

**Ambiguity**: `v6.3.5` — what do major/minor/patch mean?

**Current**: Major = big overhaul, Minor = feature pack, Patch = fixes.

**Not Formalised**: No written policy.

**Resolution**: Document in change-guide.md or CONTRIBUTING.md.

---

### A3: Attribution Completeness

**Ambiguity**: Are all community pack assets properly attributed?

**Current**: README.md has attribution section.

**Risk**: Missed assets, license changes upstream.

**Resolution**: Audit script to cross-reference assets vs. attributions.

---

### A4: Custom Namespace Usage

**Ambiguity**: When to use `nucicraft:` vs `minecraft:` namespace?

**Current**: `minecraft:` for overrides, `nucicraft:` for new custom items.

**Grey Area**: Modified vanilla blocks — override or new?

**Resolution**: Override vanilla (use `minecraft:`) for replacements; `nucicraft:` only for truly new items.

---

### A5: Variation Weight Distribution

**Ambiguity**: Are variation weights balanced for natural look?

**Current**: Equal weights (1:1:1) for most variations.

**Unknown**: Perceptual uniformity — some textures more noticeable.

**Resolution**: Visual audit; adjust weights if patterns visible.

---

### A6: Pack Icon (pack.png) Requirements

**Ambiguity**: Optimal pack.png specifications.

**Current**: 128x128 PNG, exists.

**Unknown**: Animated? 256x256 for high-DPI? Transparency?

**Resolution**: Follow Minecraft wiki recommendations; test in menu.

---

### A7: Sound Event Naming

**Ambiguity**: Custom sound event naming convention.

**Current**: `nucicraft:item.musket_shoot`, `nucicraft:damage.retro_hurt`

**Not Formalised**: No documented convention.

**Resolution**: Document in configuration.md or sound component doc.

---

### A8: Release Note Format

**Ambiguity**: Standardised release note template?

**Current**: Free-form in GitHub Releases.

**Benefit**: Consistency, changelog generation.

**Resolution**: Add template to release workflow or CONTRIBUTING.md.

---

## Tracking

| ID | Title | Priority | Status | Owner |
|----|-------|----------|--------|-------|
| Q1 | Pack format versioning | High | Open | Maintainer |
| Q2 | Faithful 32x license | Medium | Open | Maintainer |
| Q3 | Atlas capacity | Medium | Open | Maintainer |
| Q4 | Model complexity budget | Medium | Open | Maintainer |
| Q5 | Language maintenance | Low | Open | Maintainer |
| Q6 | Custom item expansion | Low | Open | Maintainer |
| Q7 | Shader inclusion | Low | Open | Maintainer |
| Q8 | Automated testing | High | Planned | Maintainer |
| Q9 | Community pack automation | Low | Open | Maintainer |
| Q10 | Documentation sustainability | Medium | Open | Maintainer |
| A1 | Brand scope | Low | Documented | Maintainer |
| A2 | Version numbering | Low | Open | Maintainer |
| A3 | Attribution audit | Medium | Open | Maintainer |
| A4 | Namespace convention | Low | Documented | Maintainer |
| A5 | Variation weights | Low | Open | Maintainer |
| A6 | Pack icon specs | Low | Open | Maintainer |
| A7 | Sound naming | Low | Open | Maintainer |
| A8 | Release note template | Low | Open | Maintainer |

## Resolution Process

1. **Discuss** in GitHub Issue/Discussion
2. **Decide** with maintainer consensus
3. **Document** in relevant guide (change-guide, design-decisions, etc.)
4. **Implement** if code change needed
5. **Close** question/ambiguity

## Related Documents

- [Design Decisions](./design-decisions.md) — Resolved decisions (ADRs)
- [Change Guide](./change-guide.md) — Procedures
- [Testing](./testing.md) — Test strategy
- [Dependencies](./dependencies.md) — Community packs