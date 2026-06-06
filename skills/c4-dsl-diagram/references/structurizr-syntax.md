# Structurizr DSL Syntax Reference

Complete syntax reference for Structurizr DSL (C4 model). Render via `structurizr/cli` Docker image.

## Table of Contents

1. [Workspace Structure](#workspace-structure)
2. [Model Elements](#model-elements)
3. [Relationships](#relationships)
4. [Views](#views)
5. [Deployment Nodes](#deployment-nodes)
6. [Animation & Layout](#animation--layout)
7. [Styling](#styling)
8. [Tags](#tags)
9. [Complete Examples](#complete-examples)

## Workspace Structure

Every Structurizr DSL file follows this structure:

```
workspace {
    description "..."
    
    model {
        // elements and relationships
    }
    
    views {
        // diagram views
    }
}
```

## Model Elements

### Person
```
<alias> = person "<name>" "<description>"
```

### Software System
```
<alias> = softwareSystem "<name>" "<description>"
```

### Container (inside a software system)
```
<alias> = container "<name>" "<technology>" "<description>"
// or shorthand for databases/queues:
<alias> = containerDb "<name>" "<technology>" "<description>"
<alias> = containerQueue "<name>" "<technology>" "<description>"
```

### Component (inside a container)
```
<alias> = component "<name>" "<technology>" "<description>"
```

### Group (logical grouping)
```
group "<name>" {
    // elements
}
```

### External Systems
Add `tags "External"` to mark external elements:
```
browser = softwareSystem "Web Browser" "Description" {
    tags "External"
}
```

## Relationships

### Basic
```
<from> -> <to> "<description>"
<from> -> <to> "<description>" "<technology>"
```

### With technology
```
frontend -> api "Fetches products" "JSON/HTTPS"
```

## Views

### System Context View
```
views {
    systemContext <softwareSystemAlias> {
        include *
        autoLayout
    }
}
```

### Container View
```
views {
    container <softwareSystemAlias> {
        include *
        autoLayout
    }
}
```

### Component View
```
views {
    component <containerAlias> {
        include *
        autoLayout
    }
}
```

### Dynamic View (numbered sequence)
```
views {
    dynamic "<title>" {
        include *
        autoLayout

        element1 -> element2 "1. Step description"
        element2 -> element3 "2. Step description"
    }
}
```

### Deployment View
```
views {
    deployment "<title>" {
        include *
        autoLayout
    }
}
```

### Filtering Elements
```
include *
exclude <elementAlias>
```

## Deployment Nodes

```
<alias> = deploymentNode "<name>" "<type>" "<description>" {
    // nested deployment nodes or containers
}
```

Example:
```
aws = deploymentNode "AWS Cloud" "us-east-1" {
    ecs = deploymentNode "ECS Cluster" "Fargate" {
        api = container "API Service" "Node.js" "REST API"
    }
    rds = deploymentNode "RDS" "db.r5.large" {
        db = containerDb "Primary DB" "PostgreSQL" "Application data"
    }
}
```

## Animation & Layout

### Auto Layout
```
autoLayout
```

Default auto layout. Can be placed in any view block.

## Styling

### Element Styles
```
styles {
    element "<tag>" {
        background "#color"
        color "#color"
        shape RoundedBox
        // or: shape Box, shape Cylinder (for DB), shape Pipe (for queue)
    }
}
```

### Relationship Styles
```
styles {
    relationship "<tag>" {
        color "#color"
        dashed true
        routing Orthogonal
        // or: routing Curved
    }
}
```

### Default Styles
```
styles {
    element "Element" {
        background "#ffffff"
        color "#000000"
        shape RoundedBox
    }
    element "Person" {
        shape Person
        background "#08427b"
        color "#ffffff"
    }
    element "Container" {
        background "#438dd5"
        color "#ffffff"
    }
    element "Database" {
        shape Cylinder
        background "#438dd5"
        color "#ffffff"
    }
    element "External" {
        background "#999999"
        color "#ffffff"
    }
    relationship "Relationship" {
        color "#707070"
    }
}
```

## Tags

Tags are used for styling and filtering:

### Adding tags
```
containerAlias = container "Name" "Tech" "Description" {
    tags "Tag1,Tag2"
}
```

### Built-in tags
- `Element` - All elements
- `Person` - People
- `SoftwareSystem` - Software systems
- `Container` - Containers
- `Component` - Components
- `External` - External elements (manual)

## Complete Examples

### C4Context Example
```dsl
workspace {
    description "System Context diagram for Internet Banking System"

    model {
        customer = person "Banking Customer" "A customer with bank accounts"
        bankingSystem = softwareSystem "Internet Banking System" "View accounts and make payments"
        mainframe = softwareSystem "Mainframe Banking System" "Core banking data" {
            tags "External"
        }
        email = softwareSystem "E-mail System" "Microsoft Exchange" {
            tags "External"
        }

        customer -> bankingSystem "Uses"
        bankingSystem -> mainframe "Reads/writes" "JDBC"
        bankingSystem -> email "Sends emails" "SMTP"
        email -> customer "Sends emails to"
    }

    views {
        systemContext bankingSystem {
            include *
            autoLayout
        }
    }
}
```

### C4Container Example
```dsl
workspace {
    description "Container diagram for Internet Banking System"

    model {
        customer = person "Customer" "Bank customer with accounts"
        email = softwareSystem "E-Mail System" "Microsoft Exchange" {
            tags "External"
        }
        mainframe = softwareSystem "Mainframe Banking System" "Core banking" {
            tags "External"
        }

        internetBanking = softwareSystem "Internet Banking" {
            spa = container "Single-Page App" "JavaScript, Angular" "Banking UI"
            mobile = container "Mobile App" "C#, Xamarin" "Mobile banking"
            api = container "API Application" "Java, Spring MVC" "Banking API"
            db = containerDb "Database" "SQL Server" "User data, logs"

            spa -> api "Uses" "JSON/HTTPS"
            mobile -> api "Uses" "JSON/HTTPS"
            api -> db "Reads/writes" "JDBC"
            api -> mainframe "Uses" "XML/HTTPS"
            api -> email "Sends emails" "SMTP"
        }

        customer -> internetBanking.spa "Uses" "HTTPS"
        customer -> internetBanking.mobile "Uses"
    }

    views {
        container internetBanking {
            include *
            autoLayout
        }
    }
}
```

### C4Component Example
```dsl
workspace {
    description "Component diagram for API Application"

    model {
        spa = container "Single Page App" "Angular" "Banking UI"
        db = containerDb "Database" "SQL Server" "User data"
        mainframe = softwareSystem "Mainframe" "Core banking" {
            tags "External"
        }

        apiApp = container "API Application" {
            signIn = component "Sign In Controller" "Spring MVC" "User authentication"
            accounts = component "Accounts Controller" "Spring MVC" "Account operations"
            security = component "Security Component" "Spring Bean" "Auth logic"
            facade = component "Mainframe Facade" "Spring Bean" "Mainframe integration"

            signIn -> security "Uses"
            accounts -> facade "Uses"
            security -> db "Reads/writes" "JDBC"
            facade -> mainframe "Uses" "XML/HTTPS"
        }

        spa -> apiApp.signIn "Uses" "JSON/HTTPS"
        spa -> apiApp.accounts "Uses" "JSON/HTTPS"
    }

    views {
        component apiApp {
            include *
            autoLayout
        }
    }
}
```

### C4Dynamic Example
```dsl
workspace {
    description "Dynamic diagram - User Sign In Flow"

    model {
        db = containerDb "Database" "SQL Server" "User credentials"
        spa = container "Single-Page App" "Angular" "Banking UI"

        apiApp = container "API Application" {
            signIn = component "Sign In Controller" "Spring MVC" "Authentication endpoint"
            security = component "Security Component" "Spring Bean" "Validates credentials"

            signIn -> security "Validates"
            security -> db "Query user" "JDBC"
        }

        spa -> apiApp.signIn "Submit credentials" "JSON/HTTPS"
    }

    views {
        dynamic "User Sign In Flow" {
            include *
            autoLayout

            spa -> apiApp.signIn "1. Submit credentials" "JSON/HTTPS"
            apiApp.signIn -> apiApp.security "2. Validate credentials"
            apiApp.security -> db "3. Query user" "JDBC"
        }
    }
}
```

### C4Deployment Example
```dsl
workspace {
    description "Deployment Diagram - Production"

    model {
        mobile = deploymentNode "Customer's Mobile" "iOS/Android" {
            mobileApp = container "Mobile App" "Xamarin" "Mobile banking"
        }

        browser = deploymentNode "Customer's Browser" "Chrome/Firefox" {
            spa = container "SPA" "Angular" "Web banking"
        }

        dc = deploymentNode "Data Center" "AWS" {
            web = deploymentNode "Web Tier" "EC2" {
                api = container "API" "Spring Boot" "Banking API"
            }
            data = deploymentNode "Data Tier" "RDS" {
                db = containerDb "Database" "PostgreSQL" "Banking data"
            }
        }

        mobileApp -> api "API calls" "HTTPS"
        spa -> api "API calls" "HTTPS"
        api -> db "Reads/writes" "JDBC"
    }

    views {
        deployment "Production" {
            include *
            autoLayout
        }
    }
}
```

## Structurizr CLI Commands

```bash
# Export to PlantUML
docker run --rm -v $PWD:/usr/local/structurizr structurizr/cli export \
  -workspace workspace.dsl \
  -format plantuml \
  -output diagrams/

# Validate DSL
docker run --rm -v $PWD:/usr/local/structurizr structurizr/cli validate \
  -workspace workspace.dsl
```

<!-- Note: This skill exclusively uses Structurizr DSL rendered to PlantUML via structurizr/cli Docker image. -->
