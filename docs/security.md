# Security

This document describes the security model, threat model, and security considerations for the NuciCraft Resource Pack.

## Security Model

### Trust Boundaries

```
┌─────────────────────────────────────────────────────────────┐
│                    MINECRAFT CLIENT                         │
│  ┌─────────────────────────────────────────────────────┐   │
│  │           RESOURCE PACK LOADER (Trusted)            │   │
│  │  - Validates pack.mcmeta                            │   │
│  │  - Enforces pack_format compatibility               │   │
│  │  - Sandboxes asset loading                          │   │
│  └─────────────────────────────────────────────────────┘   │
│                           │                                 │
│                           ▼                                 │
│  ┌─────────────────────────────────────────────────────┐   │
│  │           ASSET MANAGER (Trusted)                   │   │
│  │  - Parses JSON (no code execution)                  │   │
│  │  - Loads PNG/OGG/GLSL (validated parsers)           │   │
│  │  - Builds texture atlases                           │   │
│  └─────────────────────────────────────────────────────┘   │
│                           │                                 │
│                           ▼                                 │
│  ┌─────────────────────────────────────────────────────┐   │
│  │         RENDERING ENGINE (Trusted)                  │   │
│  │  - Renders baked models                             │   │
│  │  - Plays sounds                                     │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                  NUCICRAFT RESOURCE PACK                    │
│  (Untrusted Data - Validated by Client)                     │
│  - pack.mcmeta                                              │
│  - assets/minecraft/** (JSON, PNG, OGG, GLSL)              │
└─────────────────────────────────────────────────────────────┘
```

### Key Principle

**The resource pack is untrusted data.** The Minecraft client validates all inputs. The pack cannot execute code, access files outside its ZIP, or communicate over network.

## Threat Model

### Assets We Protect

| Asset | Sensitivity | Protection |
|-------|-------------|------------|
| Pack integrity | Medium | SHA1 checksum on release |
| User's Minecraft worlds | High | Not accessible by pack |
| User's credentials | High | Not accessible by pack |
| User's system | High | Not accessible by pack |

### Threat Actors

| Actor | Capability | Motivation |
|-------|------------|------------|
| Malicious pack creator | Craft malicious assets | Exploit client parsers |
| Compromised community pack | Inject malicious assets | Supply chain attack |
| MITM on download | Modify ZIP in transit | Deliver malicious pack |

### Attack Vectors

| Vector | Likelihood | Impact | Mitigation |
|--------|------------|--------|------------|
| Malformed JSON parser exploit | Low | High | Client uses hardened parsers |
| PNG decompression exploit | Low | High | Client uses hardened image libs |
| OGG parsing exploit | Low | Medium | Client uses hardened audio libs |
| GLSL shader exploit | Low | Medium | Client validates shaders |
| ZIP slip (path traversal) | Low | High | Client validates paths |
| Supply chain (community pack) | Medium | Medium | Manual review, pin versions |
| MITM download | Low | High | HTTPS, SHA1 verification |

## Security Controls

### Client-Side (Minecraft)

- JSON parsing with size/depth limits
- PNG loading with dimension limits
- OGG decoding with duration/size limits
- GLSL compilation with sandboxing
- Path validation (no `..`, absolute paths)
- Texture atlas size limits
- Model complexity limits (implicit via geometry)

### Pack-Side (NuciCraft)

- **No executable code** — Fundamental protection
- **HTTPS distribution** — GitHub Releases over TLS
- **SHA1 checksums** — Published in release notes
- **Manual community pack review** — Before vendoring
- **Version pinning** — Community packs at specific releases
- **No external references** — All assets self-contained

### Release Process Security

1. **Tag creation** — Only maintainers with write access
2. **Release publish** — Only maintainers
3. **Workflow execution** — GitHub Actions (isolated runner)
4. **Asset upload** — GitHub-signed release assets
5. **SHA1 publication** — In release notes for verification

## Vulnerability Management

### Reporting

Report security issues via GitHub Security Advisories (private) or SECURITY.md contact.

### Response Process

1. Acknowledge within 48 hours
2. Assess impact (client-side vs pack-side)
3. If client-side: Report to Mojang (upstream)
4. If pack-side: Fix, test, release patch
5. Disclose after fix released

### Known Vulnerability Classes

| Class | Status | Notes |
|-------|--------|-------|
| JSON parser DoS | Mitigated | Client limits |
| PNG decompression bomb | Mitigated | Client limits |
| Shader injection | Mitigated | Client validates |
| Path traversal | Mitigated | Client validates |
| Supply chain | Monitored | Manual review |

## Data Handling

### Personal Data

**None collected, processed, or stored by the pack.**

### Telemetry

**None.** The pack has no network capability.

### Secrets

**No secrets in pack.** No API keys, tokens, passwords.

## Compliance

### Licenses

- **NuciCraft**: GPL v3.0
- **Community packs**: Various (MIT, custom permissive, Faithful 32x non-commercial)
- **Compatibility**: Verified for non-commercial distribution

### Export Controls

Not applicable — no encryption, no restricted technology.

## Security Checklist for Contributors

- [ ] No executable code added (no `.class`, `.jar`, `.js`, `.py`, etc.)
- [ ] No external URLs in assets (no `http://`, `https://` in JSON)
- [ ] No absolute paths or `..` in references
- [ ] Community pack updates reviewed manually
- [ ] SHA1 verified on release download
- [ ] Release published by authorised maintainer only

## Incident Response

If a security issue is discovered in a released pack:

1. **Immediate**: Delete release asset from GitHub
2. **Short-term**: Publish fixed version with incremented patch
3. **Communication**: Release notes mention security fix (no details until users updated)
4. **Post-mortem**: Document root cause, improve review process

## Related Documents

- [SECURITY.md](../SECURITY.md) — Public security policy
- [Error Handling](./error-handling.md) — Failure modes
- [Dependencies](./dependencies.md) — Supply chain
- [Build and Deployment](./build-and-deployment.md) — Release process