# Build and Deployment

This document describes the build process, release pipeline, and deployment for the NuciCraft Resource Pack.

## Build Process

### Overview

The NuciCraft Resource Pack is a **data-only** pack with no compilation step. The "build" is simply packaging the asset files into a ZIP archive.

### Source Files

All source files are committed to the repository:

```
assets/
pack.mcmeta
pack.png
```

### Packaging

The release package includes:

| Path | Included | Purpose |
|------|----------|---------|
| `assets/` | Yes | All Minecraft assets |
| `pack.mcmeta` | Yes | Pack metadata |
| `pack.png` | Yes | Pack icon (128x128) |
| `.github/` | No | CI configuration |
| `docs/` | No | Documentation |
| `ARCHITECTURE.md` | No | Architecture doc |
| `README.md` | No | Project readme |
| `LICENSE` | No | License |
| `CODE_OF_CONDUCT.md` | No | Code of conduct |
| `CONTRIBUTING.md` | No | Contributing guide |
| `SECURITY.md` | No | Security policy |
| `PRIVACY.md` | No | Privacy policy |

### Package Structure

```
nucicraft_<version>.zip
├── assets/
│   └── minecraft/
│       ├── atlases/
│       ├── blockstates/
│       ├── lang/
│       ├── models/
│       ├── shaders/
│       ├── sounds/
│       └── textures/
├── pack.mcmeta
└── pack.png
```

## Release Pipeline

### GitHub Actions Workflow

**File**: `.github/workflows/release-asset.yml`

**Trigger**: `release.published` (when a GitHub Release is published, not draft)

### Workflow Steps

```yaml
name: Release Asset
on:
  release:
    types: [published]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Create release package
        run: |
          zip -r nucicraft_${{ github.ref_name }}.zip assets/ pack.mcmeta pack.png

      - name: Compute SHA1
        run: echo "SHA1: $(sha1sum nucicraft_${{ github.ref_name }}.zip | cut -d' ' -f1)"

      - name: Upload release asset
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ release.upload_url }}
          asset_path: ./nucicraft_${{ github.ref_name }}.zip
          asset_name: nucicraft_${{ github.ref_name }}.zip
          asset_content_type: application/zip
```

### Release Artifact

- **Filename**: `nucicraft_<tag>.zip` (e.g., `nucicraft_v6.3.5.zip`)
- **Format**: ZIP (deflate compression)
- **Contents**: `assets/`, `pack.mcmeta`, `pack.png`
- **Checksum**: SHA1 published in release notes

### Versioning

| Component | Format | Example |
|-----------|--------|---------|
| Git tag | `v<major>.<minor>.<patch>` | `v6.3.5` |
| Pack description | `NuciCraft v<major>.<minor>.<patch>` | `NuciCraft v6.3.5` |
| ZIP filename | `nucicraft_<tag>.zip` | `nucicraft_v6.3.5.zip` |
| Pack format | Integer (Minecraft version) | `107.1` |

### Release Process

1. **Prepare release**
   - Update `pack.mcmeta` description to new version
   - Commit changes
   - Create and push Git tag: `git tag v6.3.5 && git push origin v6.3.5`

2. **Create GitHub Release**
   - Go to GitHub Releases
   - Click "Draft a new release"
   - Select tag `v6.3.5`
   - Fill release notes
   - Click "Publish release"

3. **Automatic packaging**
   - Workflow triggers on `release.published`
   - Creates ZIP with current repository state
   - Uploads as release asset
   - Computes and publishes SHA1

4. **Verification**
   - Download ZIP from release assets
   - Verify contents: `unzip -l nucicraft_v6.3.5.zip`
   - Verify SHA1: `sha1sum nucicraft_v6.3.5.zip`
   - Test in Minecraft client

## Deployment

### Client Installation (Manual)

1. Download `nucicraft_<version>.zip` from GitHub Releases
2. Open Minecraft → Options → Resource Packs → Open Pack Folder
3. Copy ZIP to `resourcepacks/` folder
4. Enable pack in Resource Packs menu
5. Move to top of list for priority

### Client Installation (Server-Prompted)

**Server Configuration** (`server.properties`):
```properties
resource-pack=https://github.com/<owner>/nucicraft-resourcepack/releases/download/v6.3.5/nucicraft_v6.3.5.zip
resource-pack-sha1=<sha1-from-release-notes>
resource-pack-prompt="NuciCraft Resource Pack v6.3.5 - Required for custom models and textures"
```

**Client Behaviour**:
1. Client connects to server
2. Server sends pack URL + SHA1 + prompt
3. Client prompts user to download
4. User accepts → downloads and enables
5. User declines → uses vanilla (may see missing models)

### Server-Side Requirements

- Server must host or proxy the ZIP (GitHub Releases works)
- SHA1 must match exactly (case-sensitive)
- `resource-pack-enforce=true` to require pack (optional)

### Update Process

1. User downloads new ZIP
2. Replaces old ZIP in `resourcepacks/`
3. Reloads resources: F3+T or restart Minecraft
4. No world migration needed (data-only pack)

## Verification

### Pre-Release Checklist

- [ ] `pack.mcmeta` description updated to new version
- [ ] `pack_format` matches target Minecraft version
- [ ] All blockstate model references resolve
- [ ] All model texture references resolve
- [ ] No purple/black checkerboard in test world
- [ ] No magenta textures in test world
- [ ] Performance acceptable (F3 debug screen)
- [ ] Custom names appear in Romanian/English
- [ ] 3D models render correctly (doors, fences, chains, etc.)
- [ ] Texture variations work (birch log, cobblestone, ores)
- [ ] ZIP creates without errors
- [ ] SHA1 computed and recorded

### Post-Release Verification

- [ ] Release asset uploaded to GitHub
- [ ] SHA1 in release notes matches asset
- [ ] ZIP downloads correctly
- [ ] Pack loads in Minecraft 1.21.x
- [ ] Server prompt works (if configured)

## Rollback

If a release has critical issues:

1. **Delete release asset** from GitHub Releases
2. **Delete Git tag** locally and remotely:
   ```bash
   git tag -d v6.3.5
   git push origin :refs/tags/v6.3.5
   ```
3. **Fix issues** in repository
4. **Create new tag** with patch version: `v6.3.6`
5. **Publish new release**

## Local Testing

### Quick Test

```bash
# Create test ZIP
zip -r test_pack.zip assets/ pack.mcmeta pack.png

# Verify structure
unzip -l test_pack.zip | head -30

# Check pack.mcmeta
unzip -p test_pack.zip pack.mcmeta | jq .
```

### Minecraft Test

1. Copy `test_pack.zip` to `resourcepacks/`
2. Launch Minecraft 1.21.x
3. Enable pack
4. Create/test world
5. Verify all features

## CI/CD Extensions (Future)

Potential additions:

- **JSON schema validation** — Validate all `.json` files against Minecraft schemas
- **Reference integrity check** — Verify blockstate→model, model→texture references
- **Texture dimension check** — Ensure all PNGs are powers of two
- **Atlas size estimation** — Warn if texture count/size may exceed limits
- **Performance benchmark** — Automated render time measurement
- **Multi-version test** — Test pack_format compatibility range

## Troubleshooting

| Issue | Cause | Resolution |
|-------|-------|------------|
| Pack not loading | Wrong `pack_format` | Update `pack.mcmeta` for target version |
| Missing models (purple/black) | Blockstate references missing model | Add model or fix reference |
| Missing textures (magenta) | Model references missing texture | Add texture or fix reference |
| SHA1 mismatch | ZIP corrupted or wrong file | Recreate ZIP, verify checksum |
| Server prompt fails | URL/SAH1 incorrect | Verify release asset URL and SHA1 |
| Performance issues | Too many high-poly models | Reduce model complexity or variations |
| Atlas overflow | Too many unique textures | Consolidate textures, reduce variations |