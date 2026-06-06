# Common C4 Model Mistakes to Avoid (Structurizr DSL)

This guide documents frequent anti-patterns and errors when creating C4 architecture diagrams in Structurizr DSL, with examples of what to do instead.

## Abstraction Level Mistakes

### 1. Confusing Containers and Components

**The Problem:**
Containers are **deployable units** (applications, services, databases). Components are **non-deployable elements inside a container** (modules, classes, packages).

**Wrong - Java class shown as container:**
```dsl
model {
    userController = container "UserController" "Java Class" "Handles user requests"
    userService = container "UserService" "Java Class" "Business logic"
    db = containerDb "Database" "PostgreSQL" "User data"

    userController -> userService "Calls"
    userService -> db "Queries"
}
```

**Correct - Classes as components inside a container:**
```dsl
model {
    db = containerDb "Database" "PostgreSQL" "User data"

    api = container "User API Service" {
        userController = component "UserController" "Spring MVC" "REST endpoints"
        userService = component "UserService" "Spring Bean" "Business logic"
        userRepo = component "UserRepository" "JPA" "Data access"

        userController -> userService "Calls"
        userService -> userRepo "Uses"
        userRepo -> db "Queries" "JDBC"
    }
}
```

### 2. Adding Undefined Abstraction Levels

**The Problem:**
C4 defines exactly four levels. Don't invent "subcomponents", "modules", or other arbitrary levels.

**Wrong:**
- Level 3.5: "Subcomponents"
- Level 2.5: "Microservice groups"
- Custom levels like "packages" or "modules"

**Correct:**
Stick to Person, Software System, Container, Component. If you need more detail, you're at Level 4 (Code) which should use UML class diagrams.

## Shared Libraries Mistake

**The Problem:**
Modeling a shared library as a container implies it's an independently running service. Libraries are copied into applications, not deployed separately.

**Wrong - Library as separate container:**
```dsl
model {
    serviceA = container "Service A" "Java"
    serviceB = container "Service B" "Java"
    sharedLib = container "Shared Utils Library" "Java" "Common utilities"

    serviceA -> sharedLib "Uses"
    serviceB -> sharedLib "Uses"
}
```

**Correct - Show library within each service:**
```dsl
model {
    serviceA = container "Service A" {
        controllerA = component "Controller" "Spring MVC"
        utilsA = component "Shared Utils" "Java Library" "Bundled copy"
    }
    serviceB = container "Service B" {
        controllerB = component "Controller" "Spring MVC"
        utilsB = component "Shared Utils" "Java Library" "Bundled copy"
    }
}
```

Or simply omit the library from architecture diagrams since it's an implementation detail.

## Message Broker Mistakes

### Single Message Bus Anti-Pattern

**The Problem:**
Showing Kafka/RabbitMQ as a single container creates a misleading "hub and spoke" diagram that hides actual data flows.

**Wrong - Central message bus:**
```dsl
model {
    orderSvc = container "Order Service" "Java"
    inventorySvc = container "Inventory Service" "Java"
    paymentSvc = container "Payment Service" "Java"
    kafka = containerQueue "Kafka" "Event Streaming" "Message bus"

    orderSvc -> kafka "Publishes/Subscribes"
    inventorySvc -> kafka "Publishes/Subscribes"
    paymentSvc -> kafka "Publishes/Subscribes"
}
```

**Correct - Individual topics:**
```dsl
model {
    orderSvc = container "Order Service" "Java" "Creates orders"
    inventorySvc = container "Inventory Service" "Java" "Manages stock"
    paymentSvc = container "Payment Service" "Java" "Processes payments"

    orderCreated = containerQueue "order.created" "Kafka" "New orders"
    stockReserved = containerQueue "stock.reserved" "Kafka" "Stock events"
    paymentComplete = containerQueue "payment.complete" "Kafka" "Payment events"

    orderSvc -> orderCreated "Publishes"
    inventorySvc -> orderCreated "Consumes"
    inventorySvc -> stockReserved "Publishes"
    paymentSvc -> stockReserved "Consumes"
    paymentSvc -> paymentComplete "Publishes"
    orderSvc -> paymentComplete "Consumes"
}
```

## External Systems Mistakes

### Showing Internal Details of External Systems

**The Problem:**
You don't control external systems. Showing their internals creates coupling and becomes stale quickly.

**Wrong - External system internals:**
```dsl
model {
    myApp = container "My App" "Node.js"

    stripe = softwareSystem "Stripe" {
        stripeApi = container "Stripe API" "Ruby"
        stripeWorker = container "Payment Worker" "Java"
        stripeDb = containerDb "Payment DB" "MySQL"
    }

    myApp -> stripe.stripeApi "Charges cards"
}
```

**Correct - External system as black box:**
```dsl
model {
    myApp = container "My App" "Node.js" "E-commerce backend"
    stripe = softwareSystem "Stripe" "Payment processing platform" {
        tags "External"
    }

    myApp -> stripe "Processes payments" "REST API"
}
```

## Metadata and Documentation Mistakes

### 1. Missing Descriptions

**The Problem:**
Elements without descriptions force viewers to guess their purpose.

**Wrong:**
```dsl
svc = container "Service" "Java"
```

**Correct:**
```dsl
orderSvc = container "Order Service" "Spring Boot" "Manages order lifecycle and fulfillment"
```

### 2. Generic Relationship Labels

**The Problem:**
Labels like "uses" or "communicates with" don't explain what data flows or why.

**Wrong:**
```dsl
frontend -> api "Uses"
api -> db "Accesses"
```

**Correct:**
```dsl
frontend -> api "Fetches products, submits orders" "JSON/HTTPS"
api -> db "Reads/writes order data" "JDBC"
```

## Diagram Scope Mistakes

### 1. Not Tailoring to Audience

**The Problem:**
Showing Level 4 code diagrams to executives, or only Level 1 to developers who need implementation details.

| Audience | Appropriate Levels |
|----------|-------------------|
| Executives | Level 1 (Context) only |
| Product Managers | Levels 1-2 |
| Architects | Levels 1-3 |
| Developers | All levels as needed |
| DevOps | Levels 2 + Deployment |

### 2. Creating All Four Levels by Default

**The Problem:**
Not every system needs all four levels. Level 3 (Component) and Level 4 (Code) often add no value.

**Guidance:**
- **Always create:** Context (L1) and Container (L2)
- **Create if valuable:** Component (L3) for complex containers
- **Rarely create:** Code (L4) - let IDEs generate these

### 3. Too Many Elements Per Diagram

**The Problem:**
Diagrams with 20+ elements become unreadable.

**Simon Brown's advice:** "If a diagram with a dozen boxes is hard to understand, don't draw a diagram with a dozen boxes!"

**Solutions:**
- Split by bounded context or domain
- Create separate diagrams per service
- Show one service + its direct dependencies
- Use multiple focused diagrams instead of one comprehensive diagram

## Arrow Mistakes

### 1. Bidirectional Arrows

**The Problem:**
Bidirectional arrows are ambiguous. Who initiates the call? What flows each direction?

**Wrong:**
```dsl
frontend <-> api "Data"  // Not valid DSL, but the concept is wrong
```

**Correct:**
```dsl
frontend -> api "Requests products" "JSON/HTTPS"
api -> frontend "Returns product data" "JSON/HTTPS"
```

Or show the initiator's perspective:
```dsl
frontend -> api "Fetches products" "JSON/HTTPS"
```

### 2. Unlabeled Arrows

**The Problem:**
Arrows without labels force readers to guess what flows between elements.

**Wrong:**
```dsl
orderSvc -> paymentSvc
```

**Correct:**
```dsl
orderSvc -> paymentSvc "Requests payment authorization" "gRPC"
```

## Deployment Diagram Mistakes

### 1. Deployment Details in Container Diagrams

**The Problem:**
Container diagrams should show logical architecture, not infrastructure details.

**Wrong - Infrastructure in container diagram:**
```dsl
model {
    api1 = container "API (Instance 1)" "Java" "Primary"
    api2 = container "API (Instance 2)" "Java" "Replica"
    api3 = container "API (Instance 3)" "Java" "Replica"
    lb = container "Load Balancer" "HAProxy" "Distributes traffic"
    primary = containerDb "Primary DB" "PostgreSQL" "Write"
    replica = containerDb "Read Replica" "PostgreSQL" "Read"
}
```

**Correct - Use Deployment diagram for infrastructure:**
```dsl
model {
    lb = deploymentNode "Load Balancer" "AWS ALB" {
        alb = container "ALB" "AWS" "Traffic distribution"
    }
    ecs = deploymentNode "ECS Cluster" "Fargate" {
        api1 = container "API Instance 1" "Spring Boot"
        api2 = container "API Instance 2" "Spring Boot"
        api3 = container "API Instance 3" "Spring Boot"
    }
    rds = deploymentNode "RDS" "Multi-AZ" {
        primary = containerDb "Primary" "PostgreSQL"
        replica = containerDb "Replica" "PostgreSQL"
    }
}
```

### 2. Missing Environment Context

**The Problem:**
Deployment diagrams should specify which environment (production, staging, dev).

**Wrong:**
```dsl
views {
    deployment "Deployment" {  // Which environment?
```

**Correct:**
```dsl
views {
    deployment "Production (AWS us-east-1)" {
```

## Consistency Mistakes

### 1. Inconsistent Notation Across Diagrams

**The Problem:**
Using different naming for the same elements across diagrams.

**Wrong:**
- Context diagram: "Payment System"
- Container diagram: "Payment Service"
- Component diagram: "Payment Module"

**Correct:**
Use consistent naming across all diagrams and views.

### 2. No Legend/Key

**The Problem:**
Assuming viewers understand your notation without explanation.

**Solution:**
Always include a legend explaining colors, shapes, and line styles. Define in `styles` block:
```dsl
styles {
    element "Person" { shape Person }
    element "Database" { shape Cylinder }
    element "External" { background "#999999" }
}
```

## Decision Documentation Mistakes

### Showing Decision Process in Diagrams

**The Problem:**
Architecture diagrams show **outcomes** of decisions, not the decision-making process.

**Correct approach:**
- Document decisions separately in Architecture Decision Records (ADRs)
- Link ADRs to relevant diagrams
- Diagrams show the chosen architecture, ADRs explain why

## Quick Reference: Checklist

Before finalizing any C4 diagram, verify:

- [ ] Every element has: name, type, technology (if applicable), description
- [ ] All arrows are unidirectional with action verb labels
- [ ] Technology/protocol included on relationships
- [ ] View has a clear, specific title
- [ ] Under 20 elements (ideally under 15)
- [ ] Appropriate level for the target audience
- [ ] Containers are deployable, components are not
- [ ] External systems shown as black boxes
- [ ] Message topics shown individually (not as single broker)
- [ ] No infrastructure details in container diagrams
- [ ] Consistent with other diagrams in the set
- [ ] Workspace is valid (`structurizr/cli validate`)
