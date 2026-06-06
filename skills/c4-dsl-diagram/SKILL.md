---
name: c4-dsl-diagram
description: Generate C4 model architecture diagrams using Structurizr DSL with a modular !include structure. Supports Structurizr CLI, Structurizr Lite, and Structurizr vNext. Triggers include "architecture diagram", "C4 diagram", "structurizr", "C1", "C2", "C3".
---

# C4 DSL — Structurizr Modular

Generate C4 architecture diagrams using **Structurizr DSL** with a clean modular `!include` structure. Each container, component group, relationship, and view is split into its own file for maintainability.

**Supported tools:** Structurizr CLI, Structurizr Lite, Structurizr vNext, any Structurizr-compatible renderer.

---

## When to Use

This skill should be used when:

- Creating or updating C4 architecture diagrams for a project
- The project already has a `docs/architecture/` folder with Structurizr DSL
- You need to document system context (C1), containers (C2), components (C3), or flows
- You need to visualize module structure, IPC handlers, services, and repositories

Do **not** use this skill for:
- Mermaid or PlantUML diagrams (use appropriate skill instead)
- Ad-hoc architecture sketches (use excalidraw or similar)

## Instructions

Follow these steps in order:

### 1. Explore Existing Structure

If `docs/architecture/` already exists:
- Read `docs/architecture/workspace.dsl` to understand the current model
- List `docs/architecture/modules/containers/` to identify existing container definitions
- List `docs/architecture/modules/containers/<name>/components/` for component groups
- List `docs/architecture/modules/relationship/` for existing relationships
- List `docs/architecture/modules/views/` for existing views

If it does not exist, scaffold the folder structure first (see [Folder Structure](#folder-structure)).

### 2. Map Project to DSL

Explore the actual project codebase to identify architecture layers:

- List top-level source directories (e.g., `src/`, `backend/`, `electron/`) to understand project layout
- Identify **modules/domains** — each subdirectory or namespace that represents a business domain (e.g., auth, product, order, payment)
- For each module, determine its **layer structure** — does it use handlers/controllers, services, repositories? (common patterns: `controller.ts`, `service.ts`, `repo.ts`, `handler.ts`, `route.ts`)
- Identify **containers** — deployable runtime units (e.g., web frontend, API server, worker processes, databases)
- Read handler/controller files to understand available endpoints or IPC channels
- Read service files for business logic
- Read repository/data files for data access patterns

### 3. Add or Update Components

- Place component groups in the correct container's `components/` folder
- Each module gets its own `.dsl` file
- **C3 maps to function level** — each component must represent a specific exported function from the actual source code. Do NOT generalize:
  - ✅ `loginPin = component "loginPin" "auth/service.ts" "Validates PIN login"`
  - ✅ `loginPassword = component "loginPassword" "auth/service.ts" "Validates email/password"`
  - ❌ `authService = component "authService" "auth/service.ts" "Auth service functions"`
- Component alias should match the exported function name from the source file
- File path in the description must be the actual relative path from project root

### 4. Add or Update Relationships

- Create a `.dsl` file in `relationship/` per domain module
- Relationships connect components within & between containers
- Label with action verbs and technology/protocol

### 5. Add or Update Views

- C1-C2: `views/system.dsl` (only if not exists)
- **C3: one file per module** — do NOT combine multiple modules into a single view file. Each module gets its own `.dsl` file under the container's views folder:
  - ✅ `views/react/login.dsl`, `views/react/products.dsl`, `views/react/shift.dsl` (separate files)
  - ❌ `views/react/all-pages.dsl` (single file for everything)
- Same rule applies to Electron views: `views/electron/auth.dsl`, `views/electron/pos.dsl`, etc.
- C3 views scope to a container — no actors allowed
- Flow views: `views/flow/<module>.dsl` — include numbered steps, actors allowed

### 6. Validate

```bash
tools/structurizr.bat validate -w docs/architecture/workspace.dsl
```

Fix any identifier conflicts or syntax errors before committing.

---

## Project-Aware Diagram Generation

When creating C4 diagrams for an **existing project**, the DSL must reflect the actual codebase structure, not a generic template.

### Pre-flight Checklist

Before writing any DSL:

1. **Explore the codebase** — read the project's source tree to identify:
   - Actual file/folder layout (e.g., `electron/ipc/<module>/route.ts`, `src/features/<module>/`)
   - Module boundaries: are services separated from repos? Are handlers in `route.ts`?
   - Technology stack (Electron? React? Flutter? Express?)
2. **Map modules** — list every domain module (auth, product, transaction, etc.) and note:
   - Does it have IPC handlers? (`route.ts`)
   - Does it have business logic? (`service.ts`)
   - Does it have data access? (`repo.ts`)
3. **Identify containers** — determine what the running processes/containers are:
   - Electron main process (backend services + IPC)
   - React/Flutter/Vue app (frontend UI)
   - Database, cache, etc.
4. **Check existing DSL** — if `docs/architecture/workspace.dsl` already exists, read it thoroughly before making changes. Follow the same conventions.

### Alignment Rules

| DSL Element | Must Match Project |
|-------------|-------------------|
| **Container names** | Actual technology/runtime name (electron, react, api, etc.) |
| **Component aliases** | Module/feature names (auth, pos, product, etc.) |
| **Component file paths** | Real relative paths from project root |
| **Group names** | Actual team/domain boundaries |
| **Relationships** | Real IPC calls, service dependencies, DB queries |
| **Folder structure** | `containers/<name>/` folders mirror actual architecture layers |

### Example: Mapping Project Files to DSL

```
Project files:                          DSL representation:
electron/ipc/auth/route.ts             → authLogin component (IPC handler)
electron/ipc/auth/service.ts           → authService component (business logic)
electron/ipc/auth/repo.ts              → authRepo component (data access)
```

---

## Folder Structure

```
docs/architecture/
├── workspace.dsl              ← entry point (only !include, ~100 lines)
│
└── modules/
    ├── actors.dsl             ← Person definitions
    ├── external.dsl           ← External systems
    ├── container.dsl          ← aggregator: !include all containers
    ├── relationship.dsl       ← aggregator: !include all relationships
    ├── views.dsl              ← aggregator: !include all views
    │
    ├── containers/            ← one folder per container
    │   └── <name>/
    │       ├── <name>.dsl           ← container definition
    │       └── components/          ← component groups owned by this container
    │           └── *.dsl
    │
    ├── relationship/          ← relationships per domain module
    │   └── *.dsl
    │
    └── views/
        ├── system.dsl         ← C1-C2 views
        ├── deployment.dsl     ← D view
        ├── styles.dsl         ← element styling
        ├── <container>/       ← C3 component views per container
        │   └── <module>.dsl   ← one file per module (NOT combined into one)
        └── flow/              ← Dynamic flow views
            └── <module>.dsl   ← one file per flow
```

### File Examples

**workspace.dsl** — pure aggregator:
```dsl
workspace "Project Name" "Description" {
    model {
        !include modules/actors.dsl
        !include modules/external.dsl
        system = softwareSystem "System Name" "" {
            !include modules/container.dsl
        }
        !include modules/relationship.dsl
    }
    views {
        !include modules/views.dsl
    }
}
```

**containers/<name>/<name>.dsl** — container + its components:
```dsl
myContainer = container "Container Name" "Tech" "Description" {
    tags "Container"
    inlineComponent = component "inline" "path/file.ts" "desc"
    !include components/module.dsl
}
```

**containers/<name>/components/module.dsl** — component group:
```dsl
group "[Group] module" {
    comp1 = component "comp:name" "path/file.ts" "description"
    comp2 = component "comp:other" "path/other.ts" "description"
}
```

---

## C4 Model Levels

| Label | View Type | Scope | Content | Actors allowed? |
|-------|-----------|-------|---------|----------------|
| **C1** Context | `systemContext` | softwareSystem | System + actors + external systems | ✅ Yes |
| **C2** Container | `container` | softwareSystem | All containers inside the system | ✅ Yes |
| **C3** Component | `component` | **container** | Internal functions + their source files | ❌ No |
| **Flow** (Dynamic) | `dynamic` | **container** | Numbered step sequence | ✅ Yes |
| **D** Deployment | `deployment` | `*` | Deployment nodes + environments | ❌ No |

### C3 Rules (Important)

- Shows **functions/methods and their related source files** inside a container
- **Actors (Person) must NOT appear** in C3 views — actors interact at C1/C2 level only
- Scope must be a **container**, not a softwareSystem
- Focus on **internal component relationships**

### View Key Examples

```dsl
// C1
systemContext system "SystemContext" {
    title "[C1] - System Context"
    include *
    autoLayout
}

// C2
container system "ContainerDiagram" {
    title "[C2] - Container Diagram"
    include *
    autoLayout
}

// C3 — no actors
component myContainer "ModuleName" {
    title "[C3] - ModuleName"
    include comp1 comp2 comp3 dbTable
    autoLayout
}

// Flow
dynamic myContainer "TransactionFlow" {
    title "[Flow] - Transaction Flow"
    autoLayout
    actor -> page "1. Action"
    page -> handler "2. IPC"
    handler -> service "3. Process"
}

// Deployment
deployment * Production "DeploymentDiagram" {
    title "[D] - Deployment Diagram"
    include *
    autoLayout
}
```

---

## Modular Rules

1. **1 container = 1 folder** at `containers/<name>/`
2. **Component groups** go in `containers/<name>/components/`
3. **Views** are separated per container at `views/<name>/`
4. **Relationships** are separated per domain module at `relationship/`
5. **Folder name = actual container name** (e.g., `electron`, `react`, `api`, `mobile`)
6. **`!include` paths are relative** to the including file
7. **C3 views** exist only in `views{}`, not in `model{}`
8. **Identifier names must be unique** across the entire `model{}`
9. **Dynamic views** scope must be a container, not a softwareSystem

---

## Structurizr Usage

### Preview (Structurizr Lite)
```bash
java -jar structurizr-lite.war docs/architecture/
# Open http://localhost:8080
```

### Validate
```bash
java -cp "tools;tools/lib/*" com.structurizr.cli.StructurizrCliApplication \
  validate -w docs/architecture/workspace.dsl
```

### Export
```bash
# PlantUML
java -cp "tools;tools/lib/*" com.structurizr.cli.StructurizrCliApplication \
  export -w docs/architecture/workspace.dsl -f plantuml -o docs/diagrams/

# Mermaid
java -cp "tools;tools/lib/*" com.structurizr.cli.StructurizrCliApplication \
  export -w docs/architecture/workspace.dsl -f mermaid -o docs/diagrams/
```

### Structurizr vNext (Docker)
```bash
docker run -it --rm -p 8080:8080 \
  -v "$PWD/docs/architecture:/usr/local/structurizr" \
  structurizr/structurizr local
```

---

## Structurizr DSL Syntax Notes

- **View key** is the second string argument (not a `key "..."` property)
  - ✅ `systemContext app "KeyName" {`
- **`styles {}`** must be inside `views {}`, not at workspace level
- **Dynamic view scope** — use container to access its components
  - ✅ `dynamic myContainer "Flow" {`
  - ❌ `dynamic mySystem "Flow" {` → error: "Components can't be added"
- **Deployment view** — `deployment <*|softwareSystem> <environment> <key>`
  - ✅ `deployment * Production "DeploymentDiagram" {`
- **Deployment environment** — `deploymentEnvironment "<name>"` inside `model {}`
- **`containerInstance`** — reference containers inside deployment nodes
- **Windows path** — Git Bash converts `C:/` to `/c/`; use `winpty` or absolute path
- **No built-in C4 Code view** — use `component` view + tag `"Code"` instead

---

## Auto-generated Files

### Makefile

Generate a `Makefile` at the same level as `workspace.dsl` (inside `docs/architecture/`) with targets for common Structurizr operations:

```makefile
STRUCTURIZR := java -cp "../tools;../tools/lib/*" com.structurizr.cli.StructurizrCliApplication

.PHONY: validate export clean

validate:
	$(STRUCTURIZR) validate -w workspace.dsl

export:
	$(STRUCTURIZR) export -w workspace.dsl -f plantuml -o ../diagrams/

clean:
	rm -rf diagrams/
```

Include this Makefile generation in the scaffolding step when the project does not yet have one. Use relative paths so it works regardless of project root.

### .gitignore

Add `.structurizr/` to `.gitignore` — this directory is created by Structurizr Lite at runtime (contains cache, logs, and generated thumbnails). It should not be committed:

```gitignore
# Structurizr runtime data
docs/architecture/.structurizr/
```

---

## References

| File | Content |
|------|---------|
| `references/structurizr-syntax.md` | Complete Structurizr DSL syntax (elements, relationships, views, deployment) |
| `references/anti-patterns.md` | Anti-patterns to avoid when creating C4 diagrams |
| `references/architecture-patterns.md` | Microservices, event-driven, CQRS, deployment patterns |
