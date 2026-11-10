# Frequently Asked Questions

## General

### What is NuciCraft Resource Pack?

A Minecraft Java Edition resource pack that adds 3D models, texture variations, custom names (Romanian + English), custom items (magic compass, musket), retro sounds, and performance optimisations. It combines curated community packs with original additions.

### Which Minecraft versions are supported?

**Primary**: 1.21.x (pack_format 107.1)
**Also loads on**: 1.18 - 1.20.6 (pack_format 8-48) via `supported_formats` range
**Best experience**: 1.21.x (all features present)

### Is this a mod? Do I need Forge/Fabric?

**No.** It's a vanilla resource pack — drag and drop into `resourcepacks/` folder. No mod loader required.

### Does it work on servers?

Yes. Two ways:
1. **Client-side**: Install locally, works on any server
2. **Server-prompted**: Server admin configures `resource-pack` in `server.properties`, clients auto-download

### Is it compatible with OptiFine / Sodium / shaders?

Yes. Tested with OptiFine, Sodium, Iris, Canvas. No conflicts.

## Installation

### How do I install it?

1. Download `nucicraft_<version>.zip` from GitHub Releases
2. Open Minecraft → Options → Resource Packs → Open Pack Folder
3. Copy ZIP to that folder
4. Enable in Resource Packs menu
5. Move to top of list for priority

### Do I need to unzip it?

**No.** Minecraft reads ZIP files directly. Do not extract.

### Why doesn't it show in my resource pack list?

- Wrong folder (must be `resourcepacks/` in `.minecraft`)
- Wrong Minecraft version (needs 1.18+)
- Corrupted download (re-download, verify SHA1)

### Can I use it with other resource packs?

Yes. Enable multiple packs; order matters (top = highest priority). Place NuciCraft at top for full effect.

## Features

### What 3D models are included?

Fences, gates, doors, trapdoors, chains, lanterns, candles, crops (beetroot, carrot, potato, melon, pumpkin, sweet berry), flowers, grass, ferns, redstone dust, buckets, goat horn, trident, amethyst clusters, iron bars, dripstone, armor trims, and more.

### What are texture variations?

Randomised textures for blocks like cobblestone, ores, stone variants, bricks, logs, obsidian — each placed block gets a random variant for natural look.

### Why are some blocks/items renamed in Romanian?

Romanian (`ro_ro`) is the primary custom language. Switch language to "Română (România)" in options to see all custom names. English variants have minimal overrides.

### What is the magic compass?

A custom item using compass + custom model data (1001). Requires a datapack to set `CustomModelData: 1001` on a compass. Shows advanced resource pack capability.

### What is the musket?

Crossbow replacement with custom 3D model and sounds (from PiXeLGaMeS_or_not). Functions as crossbow but looks/sounds like musket.

## Troubleshooting

### I see purple/black checkerboard blocks

Missing model. Check blockstate references valid model. Report if in released pack.

### I see bright pink (magenta) textures

Missing texture. Model references non-existent texture. Report if in released pack.

### Custom names not showing

- Ensure language set to "Română (România)" for Romanian names
- English names mostly vanilla (few overrides)
- Restart Minecraft after language change

### Low FPS / lag

- Reduce render distance
- Disable other resource packs
- Some 3D models are complex; try OptiFine/Sodium
- Report specific blocks causing issues

### Server pack prompt not working

- Server must use HTTPS URL to ZIP
- SHA1 must match exactly (check release notes)
- `resource-pack-enforce=true` kicks if declined

## Development

### Can I contribute?

Yes. See [CONTRIBUTING.md](../CONTRIBUTING.md) and [Change Guide](./change-guide.md).

### How do I add a new block model?

See [Change Guide](./change-guide.md#adding-a-new-block).

### How do I update a community pack?

See [Change Guide](./change-guide.md#updating-community-pack-assets) and [Dependencies](./dependencies.md).

### Where are the source files?

All in `assets/minecraft/` — blockstates, models, textures, lang, etc.

### Is there a development environment?

No build step. Edit JSON/PNG directly, test with F3+T reload in Minecraft.

## Licensing

### What license?

GPL v3.0. See [LICENSE](../LICENSE).

### Can I use it in my modpack/server?

Yes, GPL v3.0 allows redistribution with source. Include attribution (README.md has full list).

### Faithful 32x is non-commercial — is that OK?

NuciCraft is non-commercial (free). If you use commercially, replace Faithful 32x textures.

## Support

### Where do I report bugs?

GitHub Issues on the repository.

### Where do I ask questions?

GitHub Discussions or Issues.

### Is there a Discord?

Not currently. Use GitHub.