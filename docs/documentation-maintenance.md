# Documentation Maintenance

This document describes how to maintain the documentation in the `/docs` directory.

## Documentation Structure

```
/docs/
├── INDEX.md                          # Master index
├── architecture.md                   # Architecture deep-dive
├── repository-overview.md            # Purpose, scope, entry points
├── repository-structure.md           # Source tree, modules
├── data-model.md                     # Domain entities, relationships
├── configuration.md                  # Configuration schemas
├── build-and-deployment.md           # Release pipeline
├── dependencies.md                   # External/internal dependencies
├── testing.md                        # Test strategy
├── change-guide.md                   # Safe modification patterns
├── design-decisions.md               # ADRs
├── error-handling.md                 # Failure modes, recovery
├── logging.md                        # Log sources, analysis
├── security.md                       # Security model
├── state-and-persistence.md          # State management
├── integrations.md                   # Integration points
├── concurrency-and-scheduling.md     # Concurrency model
├── documentation-maintenance.md      # This file
├── faq.md                            # Frequently asked questions
├── troubleshooting.md                # Common issues
├── ambiguities-and-open-questions.md # Known unknowns
├── api-usage-examples.md             # Usage examples
├── quick-start.md                    # Getting started
├── invariants.md                     # System invariants
├── api-reference/
│   └── INDEX.md                      # API reference index
├── components/
│   ├── blockstates.md                # Blockstate component
│   ├── block-models.md               # Block model component
│   ├── item-models.md                # Item model component
│   ├── textures.md                   # Texture component
│   ├── atlases.md                    # Atlas component
│   └── language.md                   # Language component
├── flows/
│   ├── pack-load.md                  # Pack load flow
│   ├── model-baking.md               # Model baking flow
│   └── render-frame.md               # Render frame flow
└── behaviour/
    ├── variation-rendering.md        # Texture variation behaviour
    └── custom-item-behaviour.md      # Custom item behaviour
```

## Maintenance Principles

1. **Documentation lives with code** — Update docs when assets change
2. **Single source of truth** — Don't duplicate asset content in docs
3. **Reference, don't copy** — Link to asset files, don't inline them
4. **Version with releases** — Docs match the released pack version
5. **Automate where possible** — Scripts for reference validation

## Update Triggers

| Change | Docs to Update |
|--------|----------------|
| New block/item added | `repository-structure.md`, `data-model.md`, `components/*.md` |
| Texture variation added | `repository-structure.md`, `components/textures.md`, `behaviour/variation-rendering.md` |
| Community pack updated | `dependencies.md`, `design-decisions.md` (if process changes) |
| Pack format changed | `configuration.md`, `architecture.md`, `build-and-deployment.md` |
| New custom item | `data-model.md`, `components/item-models.md`, `behaviour/custom-item-behaviour.md` |
| Release process changed | `build-and-deployment.md` |
| Test procedure changed | `testing.md` |
| New integration | `integrations.md` |

## Documentation Standards

### Markdown Style

- Headings: `#`, `##`, `###` (no deeper than `###`)
- Code blocks: Fenced with language (`json`, `bash`, `yaml`)
- Tables: GitHub-flavored markdown
- Links: Relative paths (`./file.md`), verified before commit
- Images: Not used (text-only for git diff friendliness)

### Content Standards

- **No placeholder text** — Every section has real content
- **No "TODO" in committed docs** — Resolve before commit
- **Cross-reference** — Link to related docs
- **Examples from reality** — Use actual asset paths/names
- **Version-agnostic** — Don't hardcode version numbers (use `<version>`)

### File Headers

Each file should have:
```markdown
# Title

Brief description (1-2 sentences).

## Table of Contents (optional for long files)

## Section 1
...
```

## Validation

### Pre-Commit Checks

```bash
# Check all internal links resolve
find docs -name "*.md" -exec grep -h '\[.*\](\./.*)' {} \; | \
  sed 's/.*(\.\/\([^)]*\)).*/\1/' | \
  while read link; do
    if [ ! -f "docs/$link" ] && [ ! -f "docs/$link.md" ]; then
      echo "BROKEN LINK: $link"
    fi
  done

# Check no absolute paths to repo root
grep -r '](/home/horatiu/' docs/ && echo "ABSOLUTE PATH FOUND"

# Check no placeholder text
grep -r 'TODO\|FIXME\|XXX\|PLACEHOLDER' docs/ && echo "PLACEHOLDER FOUND"
```

### Link Verification

All `./` links must resolve to existing files in `/docs` or repository root.

## Automation Opportunities

### Scripts to Build

| Script | Purpose | Trigger |
|--------|---------|---------|
| `scripts/gen_repo_structure.py` | Generate `repository-structure.md` tree | On asset change |
| `scripts/check_refs.py` | Validate blockstate→model→texture refs | CI / pre-commit |
| `scripts/gen_api_ref.py` | Generate `api-reference/INDEX.md` | On schema change |
| `scripts/check_links.py` | Verify all doc links | CI |

### CI Integration

```yaml
# .github/workflows/docs.yml
name: Documentation
on: [push, pull_request]
jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Check doc links
        run: python3 scripts/check_links.py
      - name: Check references
        run: python3 scripts/check_refs.py
```

## Review Process

### For Documentation Changes

1. **Author** updates relevant docs with code change
2. **Reviewer** checks:
   - Accuracy vs actual assets
   - Links resolve
   - No placeholder text
   - Style consistency
3. **Merge** with code change

### For Documentation-Only Changes

1. **Author** creates PR with doc updates
2. **Reviewer** validates accuracy
3. **Merge**

## Versioning

- Documentation version = Pack version (from `pack.mcmeta` description)
- No separate versioning
- `INDEX.md` shows current version

## Archival

- Old versions in Git history
- No separate archive needed
- Release tags capture docs at release

## Related Documents

- [INDEX.md](./INDEX.md) — Master index
- [Change Guide](./change-guide.md) — Asset change procedures
- [Build and Deployment](./build-and-deployment.md) — Release process