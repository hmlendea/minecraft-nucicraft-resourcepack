# NuciCraft Resource Pack Documentation

Master index and navigation for the NuciCraft Resource Pack documentation.

## Repository Summary

NuciCraft Resource Pack is a Minecraft Java Edition resource pack for the NuciCraft private server. It provides custom 3D block and item models, texture tweaks, variations, renamed elements, and performance optimisations, combining curated community packs with original additions.

## Root Document Map

- [ARCHITECTURE.md](../ARCHITECTURE.md) — High-level architecture, system context, runtime flows, and design constraints
- [SECURITY.md](../SECURITY.md) — Security policy, supported versions, vulnerability reporting
- [ROADMAP.md](../ROADMAP.md) — Project roadmap and planned features
- [PRIVACY.md](../PRIVACY.md) — Data handling and personal data practices
- [LICENSE](../LICENSE) — GNU General Public License v3.0 terms
- [CONTRIBUTING.md](../CONTRIBUTING.md) — Contribution guidelines and processes
- [CODE_OF_CONDUCT.md](../CODE_OF_CONDUCT.md) — Community standards and moderation policy

## Documentation Catalogue

### Architecture & Structure
- [Architecture](./architecture.md) — Detailed architecture complementing root ARCHITECTURE.md
- [Repository Overview](./repository-overview.md) — Repository purpose, scope, and entry points
- [Repository Structure](./repository-structure.md) — Source tree layout and module organisation
- [Data Model](./data-model.md) — Domain entities, relationships, and persistence
- [Components](./components/) — Per-component deep dives
- [Flows](./flows/) — End-to-end execution flows

### Configuration & Operations
- [Configuration](./configuration.md) — Configuration schema, sources, and precedence
- [Build and Deployment](./build-and-deployment.md) — Build pipeline, deployment, environments
- [Dependencies](./dependencies.md) — External and internal dependencies
- [Testing](./testing.md) — Test strategy, organisation, coverage

### Design & Quality
- [Design Decisions](./design-decisions.md) — Key architectural/design choices and rationale
- [Error Handling](./error-handling.md) — Error taxonomy, handling patterns, recovery
- [Logging](./logging.md) — Logging framework, levels, structured fields, correlation, sinks
- [Security](./security.md) — Security model, threats, mitigations (complements root SECURITY.md)
- [State and Persistence](./state-and-persistence.md) — State management, stores, caches, migrations
- [Invariants](./invariants.md) — System-wide invariants and contracts
- [Concurrency and Scheduling](./concurrency-and-scheduling.md) — Threading, async, schedulers, locks

### User-Facing
- [Behaviour](./behaviour/) — User-facing behaviours
- [API Reference](./api-reference/) — API reference (one file per controller)
- [API Usage Examples](./api-usage-examples.md) — Example requests/responses for API usage
- [Quick Start](./quick-start.md) — Getting started guide for newcomers
- [FAQ](./faq.md) — Frequently asked questions
- [Troubleshooting](./troubleshooting.md) — Common issues and solutions

### Maintenance
- [Documentation Maintenance](./documentation-maintenance.md) — How to keep docs current, ownership
- [Change Guide](./change-guide.md) — How to modify common areas safely
- [Ambiguities and Open Questions](./ambiguities-and-open-questions.md) — Unresolved items, TODOs, known gaps

## Navigation Aids

- **Start here** (newcomers): [Quick Start](./quick-start.md) → [Repository Overview](./repository-overview.md) → [Architecture](./architecture.md)
- **Deep dive** (component owners): [Components](./components/) → [Data Model](./data-model.md) → [Design Decisions](./design-decisions.md)
- **Flows** (debuggers): [Flows](./flows/) → [Error Handling](./error-handling.md) → [Troubleshooting](./troubleshooting.md)

## Maintenance Metadata

- Last reviewed: 2026-11-10
- Owner: NuciCraft maintainers
- Coverage status: Initial generation — comprehensive review pending