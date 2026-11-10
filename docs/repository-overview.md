# Repository Overview

## Purpose

This repository contains the NuciCraft Resource Pack, a Minecraft Java Edition resource pack for the NuciCraft private server. The pack provides custom 3D block and item models, texture tweaks, variations, renamed elements, and performance optimisations, combining curated community packs with original additions.

## Scope

The repository owns:
- All asset files under `assets/minecraft/` (blockstates, models, textures, language files, shaders, sounds, atlases)
- Pack metadata (`pack.mcmeta`, `pack.png`)
- Build and release automation (GitHub Actions workflow)
- Documentation (root markdown files and `docs/` directory)

The repository does not own:
- Minecraft client or server code
- Community pack source repositories (assets are vendored)
- Server-side datapacks or mods (separate repositories)

## Entry Points

| Entry Point | Location | Description |
|-------------|----------|-------------|
| Pack metadata | `pack.mcmeta` | Declares `pack_format` compatibility (min 84, max 999) and description |
| Pack icon | `pack.png` | Displayed in Minecraft resource pack selection screen |
| Asset root | `assets/minecraft/` | Root of all Minecraft asset categories |
| Release workflow | `.github/workflows/release-asset.yml` | Packages and publishes release ZIP on GitHub release |

## Principal Capabilities

1. **3D Block Models** — Custom models for doors, trapdoors, fences, chains, rails, amethyst crystals, pointed dripstone, iron bars, ladders, bookshelves
2. **3D Item Models** — Custom models for buckets, goat horn, mace, torches, lanterns, trims, various VanillaTweaks items
3. **Texture Variations** — Randomised textures for birch logs, bricks, cobblestone, dirt, glowstone, grass, gravel, planks, red mushroom, moss, obsidian, ores, terracotta, stone
4. **Texture Tweaks** — Borderless glass, brighter nether bricks, centered chiseled stone bricks, directional dispensers/droppers/hoppers/observers, fencier fences, lower sides for nylium/dirt path/grass/mycelium/podzol/snow, polished andesite recolour, improved prismarine bricks, invisible tripwire, smoother diorite, sticky piston sides, suspicious sand/gravel, unlit redstone ore
5. **Renamed Elements** — Custom names for blocks/items (Bamboo→Bambus, Cherry→Sakura, Concrete→Cement, etc.) with Romanian language support
6. **Custom Items** — Magic compass (from magic-compass datapack), fried potatoes, musket (crossbow replacement), custom goat horns
7. **Mob Tweaks** — Iron golem with red flowers, villager variations, darker dark oak leaves, blaze feet fix
8. **Environment** — Golden savanna biome, 3D crops/flowers/mushrooms/sculk vein/sugar cane/vines, bushy leaves, moss carpet overhang, visual sapling growth, cloud fog fix, 3D redstone dust, 3D sun/moon, brighter water, colourful enchanting particles, twinkling stars, unobtrusive rain/snow
9. **GUI** — Coloured ping bars, cooked porkchop hunger bar, clearer wither hearts, HD font, lowered shield, numbered hotbar, rainbow experience bar, Romanian language, unobtrusive enchantment glint
10. **Sounds** — Retro damage sound from older Minecraft versions
11. **Performance** — Optimised models from Luracasmus

## Integration Boundaries

- **Minecraft Client** — Loads pack via resource pack loader; consumes all asset categories
- **NuciCraft Server** — Optional: prompts clients to download pack via `server-resource-packs` in `server.properties`
- **Community Packs** — Vendored assets from VanillaTweaks, Faithful 32x, MCjossic, Saplistic, Ecorn, Ivango, JMHMirror, D_Dimensional, ZzartySt, PiXeLGaMeS_or_not, Ensis, Vexcenot, Hecco, Relidium, Infina, Oslypsis, Amethyst Geode, PatchWolf, Mexorax, PixalThePixeled, DudeAbide, Luracasmus

## Runtime Topology

```
Minecraft Java Edition Client
  Loader -> Manager -> Renderer
                    ^
                    | File system / ZIP
                    v
NuciCraft Resource Pack
  pack.mcmeta + assets/minecraft/**
```

## Major State

The pack owns no persistent runtime state. All data is static asset files read by the client at load time.

## System-Wide Invariants

- `pack_format` in `pack.mcmeta` must be compatible with target Minecraft version
- All blockstate variant models must exist under `models/block/`
- All model texture references must resolve under `textures/`
- Language keys must match vanilla translation keys
- Texture dimensions must be powers of two
- No circular model parent references