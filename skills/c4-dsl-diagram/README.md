# C4 DSL — Structurizr Modular

Generate C4 architecture diagrams using **Structurizr DSL** with a clean modular `!include` structure. Each container, component group, relationship, and view is split into its own file.

**Supported tools:** Structurizr CLI, Structurizr Lite, Structurizr vNext

## Folder Layout

```
docs/architecture/
├── workspace.dsl
│
└── modules/
    ├── actors.dsl               ← Person definitions
    ├── external.dsl             ← External systems
    ├── container.dsl            ← container aggregator
    ├── relationship.dsl         ← relationship aggregator
    ├── views.dsl                ← views aggregator
    │
    ├── containers/
    │   ├── <name>/<name>.dsl    ← container definition
    │   └── <name>/components/   ← component groups per container
    │
    ├── relationship/            ← relationships per module
    │
    └── views/
        ├── system.dsl           ← C1-C2
        ├── deployment.dsl
        ├── styles.dsl
        ├── <container>/         ← C3 component views
        └── flow/                ← Dynamic flow views
```

## Levels

| Label | View Type | Scope | Actors allowed? |
|-------|-----------|-------|----------------|
| C1 | `systemContext` | softwareSystem | ✅ |
| C2 | `container` | softwareSystem | ✅ |
| C3 | `component` | container | ❌ |
| Flow | `dynamic` | container | ✅ |
| D | `deployment` | `*` | ❌ |

## Quick Start

```bash
# Preview
java -jar structurizr-lite.war docs/architecture/

# Validate
tools/structurizr.sh validate -w docs/architecture/workspace.dsl

# Export
tools/structurizr.sh export -w docs/architecture/workspace.dsl -f plantuml -o docs/diagrams/
```

## References

- [`references/structurizr-syntax.md`](references/structurizr-syntax.md) — Complete DSL syntax
- [`references/anti-patterns.md`](references/anti-patterns.md) — Anti-patterns to avoid
- [`references/architecture-patterns.md`](references/architecture-patterns.md) — Advanced architecture patterns
