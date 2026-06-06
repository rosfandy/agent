# Advanced C4 Architecture Patterns

This guide covers advanced patterns for documenting complex architectures including microservices, event-driven systems, deployments, and API documentation.

## Microservices Architecture

### Single Team Ownership

When one team owns all microservices, model them as **containers** within a single system:

```dsl
workspace {

    model {
        customer = person "Customer" "Online shopper"

        platform = softwareSystem "E-commerce Platform" {
            gateway = container "API Gateway" "Kong" "Routing, auth, rate limiting"
            orderSvc = container "Order Service" "Node.js" "Order processing"
            orderDb = containerDb "Order DB" "PostgreSQL" "Orders"
            productSvc = container "Product Service" "Go" "Product catalog"
            productDb = containerDb "Product DB" "MongoDB" "Products"
            userSvc = container "User Service" "Java" "Authentication"
            userDb = containerDb "User DB" "PostgreSQL" "Users"
            cache = containerDb "Cache" "Redis" "Sessions"

            gateway -> orderSvc "Routes" "HTTP"
            gateway -> productSvc "Routes" "HTTP"
            gateway -> userSvc "Routes" "HTTP"

            orderSvc -> orderDb "Persists" "SQL"
            productSvc -> productDb "Persists" "MongoDB"
            userSvc -> userDb "Persists" "SQL"
            userSvc -> cache "Caches" "Redis"
        }

        payment = softwareSystem "Stripe" "Payments" {
            tags "External"
        }
        shipping = softwareSystem "FedEx" "Shipping" {
            tags "External"
        }

        customer -> platform.gateway "API requests" "HTTPS"
        platform.orderSvc -> payment "Charges" "REST"
        platform.orderSvc -> shipping "Ships" "REST"
    }

    views {
        container platform {
            include *
            autoLayout
        }
    }
}
```

### Multi-Team Ownership

When separate teams own microservices, **promote each to a software system**:

```dsl
workspace {

    model {
        customer = person "Customer" "Online shopper"
        admin = person "Admin" "Store manager"

        group "Acme Corp" {
            orderSystem = softwareSystem "Order System" "Team Alpha - Order lifecycle"
            productSystem = softwareSystem "Product System" "Team Beta - Catalog management"
            userSystem = softwareSystem "User System" "Team Gamma - Identity & auth"
            analyticsSystem = softwareSystem "Analytics System" "Team Delta - Business intelligence"
        }

        payment = softwareSystem "Stripe" "Payment processing" {
            tags "External"
        }
        warehouse = softwareSystem "Warehouse System" "Fulfillment partner" {
            tags "External"
        }

        customer -> orderSystem "Places orders"
        customer -> productSystem "Browses products"
        admin -> productSystem "Manages catalog"
        admin -> analyticsSystem "Views reports"

        orderSystem -> userSystem "Authenticates"
        orderSystem -> productSystem "Checks inventory"
        orderSystem -> payment "Processes payments"
        orderSystem -> warehouse "Fulfills orders"
        analyticsSystem -> orderSystem "Aggregates data"
    }

    views {
        systemContext orderSystem {
            include *
            autoLayout
        }
    }
}
```

Each team then creates their own Container diagram:

```dsl
workspace {

    model {
        orderSystem = softwareSystem "Order System" {
            orderApi = container "Order API" "Spring Boot" "REST endpoints"
            orderWorker = container "Order Worker" "Spring Boot" "Async processing"
            orderDb = containerDb "Order DB" "PostgreSQL" "Order data"
            orderQueue = containerQueue "Order Queue" "RabbitMQ" "Processing queue"

            orderApi -> orderDb "Reads/writes" "JDBC"
            orderApi -> orderQueue "Publishes" "AMQP"
            orderWorker -> orderQueue "Consumes" "AMQP"
            orderWorker -> orderDb "Updates" "JDBC"
        }

        productSystem = softwareSystem "Product System" "Inventory checks" {
            tags "External"
        }
        userSystem = softwareSystem "User System" "Authentication" {
            tags "External"
        }
        payment = softwareSystem "Stripe" "Payments" {
            tags "External"
        }

        orderSystem.orderApi -> userSystem "Validates tokens" "REST"
        orderSystem.orderApi -> productSystem "Reserves stock" "REST"
        orderSystem.orderWorker -> payment "Charges" "REST"
    }

    views {
        container orderSystem {
            include *
            autoLayout
        }
    }
}
```

## Event-Driven Architecture

### Showing Individual Topics

Always model message topics/queues as separate containers:

```dsl
workspace {

    model {
        orderSvc = container "Order Service" "Java" "Creates orders"
        inventorySvc = container "Inventory Service" "Go" "Manages stock"
        paymentSvc = container "Payment Service" "Node.js" "Processes payments"
        shippingSvc = container "Shipping Service" "Python" "Creates shipments"
        notificationSvc = container "Notification Service" "Python" "Sends alerts"

        orderCreated = containerQueue "order.created" "Kafka" "New order events"
        stockReserved = containerQueue "inventory.reserved" "Kafka" "Stock reservation events"
        paymentComplete = containerQueue "payment.completed" "Kafka" "Payment events"
        orderShipped = containerQueue "order.shipped" "Kafka" "Shipment events"

        orderSvc -> orderCreated "Publishes" "Avro"

        inventorySvc -> orderCreated "Consumes" "Avro"
        inventorySvc -> stockReserved "Publishes" "Avro"

        paymentSvc -> stockReserved "Consumes" "Avro"
        paymentSvc -> paymentComplete "Publishes" "Avro"

        shippingSvc -> paymentComplete "Consumes" "Avro"
        shippingSvc -> orderShipped "Publishes" "Avro"

        notificationSvc -> orderCreated "Consumes" "Avro"
        notificationSvc -> paymentComplete "Consumes" "Avro"
        notificationSvc -> orderShipped "Consumes" "Avro"

        orderSvc -> paymentComplete "Consumes" "Avro"
        orderSvc -> orderShipped "Consumes" "Avro"
    }

    views {
        systemContext orderSvc {
            include *
            autoLayout
        }
    }
}
```

### Event Flow with Dynamic Diagram

Use Dynamic diagrams to show the sequence of events:

```dsl
workspace {

    model {
        orderSvc = container "Order Service" "Java"
        inventorySvc = container "Inventory Service" "Go"
        paymentSvc = container "Payment Service" "Node.js"
        shippingSvc = container "Shipping Service" "Python"

        orderCreated = containerQueue "order.created" "Kafka"
        stockReserved = containerQueue "inventory.reserved" "Kafka"
        paymentComplete = containerQueue "payment.completed" "Kafka"

        orderSvc -> orderCreated "1. Publishes order" "Avro"
        inventorySvc -> orderCreated "2. Consumes order" "Avro"
        inventorySvc -> stockReserved "3. Publishes reservation" "Avro"
        paymentSvc -> stockReserved "4. Consumes reservation" "Avro"
        paymentSvc -> paymentComplete "5. Publishes payment" "Avro"
        shippingSvc -> paymentComplete "6. Consumes payment" "Avro"
    }

    views {
        dynamic "Order Processing Flow" {
            include *
            autoLayout

            orderSvc -> orderCreated "1. Publishes order" "Avro"
            inventorySvc -> orderCreated "2. Consumes order" "Avro"
            inventorySvc -> stockReserved "3. Publishes reservation" "Avro"
            paymentSvc -> stockReserved "4. Consumes reservation" "Avro"
            paymentSvc -> paymentComplete "5. Publishes payment" "Avro"
            shippingSvc -> paymentComplete "6. Consumes payment" "Avro"
        }
    }
}
```

### CQRS Pattern

```dsl
workspace {

    model {
        user = person "User" "Application user"

        commandApi = container "Command API" "Java" "Write operations"
        queryApi = container "Query API" "Node.js" "Read operations"

        writeDb = containerDb "Write DB" "PostgreSQL" "Source of truth"
        readDb = containerDb "Read DB" "Elasticsearch" "Query-optimized"

        events = containerQueue "Domain Events" "Kafka" "State changes"
        projector = container "Projector" "Java" "Updates read model"

        user -> commandApi "Commands" "HTTPS"
        user -> queryApi "Queries" "HTTPS"

        commandApi -> writeDb "Writes" "JDBC"
        commandApi -> events "Publishes" "Avro"

        projector -> events "Consumes" "Avro"
        projector -> readDb "Updates" "REST"

        queryApi -> readDb "Queries" "REST"
    }

    views {
        container commandApi {
            include *
            autoLayout
        }
    }
}
```

## Deployment Patterns

### AWS Production Deployment

```dsl
workspace {

    model {
        route53 = deploymentNode "Route 53" "DNS" {
            dns = container "DNS" "AWS" "api.example.com"
        }

        cloudfront = deploymentNode "CloudFront" "CDN" {
            cdn = container "CDN" "AWS" "Static asset caching"
        }

        vpc = deploymentNode "VPC" "10.0.0.0/16" {

            public = deploymentNode "Public Subnets" "Multi-AZ" {
                alb = deploymentNode "ALB" "Application LB" {
                    lb = container "Load Balancer" "AWS ALB" "TLS termination, routing"
                }
            }

            private = deploymentNode "Private Subnets" "Multi-AZ" {

                ecs = deploymentNode "ECS Cluster" "Fargate" {
                    api1 = container "API" "Node.js" "Instance 1"
                    api2 = container "API" "Node.js" "Instance 2"
                    worker1 = container "Worker" "Python" "Instance 1"
                }

                rds = deploymentNode "RDS" "db.r5.xlarge" {
                    primary = containerDb "Primary" "PostgreSQL 14" "Multi-AZ"
                }

                elasticache = deploymentNode "ElastiCache" "cache.r5.large" {
                    redis = containerDb "Redis" "Redis 7" "Cluster mode"
                }
            }
        }

        dns -> cdn "Routes to" "HTTPS"
        cdn -> lb "Forwards" "HTTPS"
        lb -> api1 "Routes" "HTTP"
        lb -> api2 "Routes" "HTTP"
        api1 -> primary "Queries" "JDBC"
        api2 -> primary "Queries" "JDBC"
        api1 -> redis "Caches" "Redis"
        worker1 -> primary "Processes" "JDBC"
    }

    views {
        deployment "Production - AWS us-east-1" {
            include *
            autoLayout
        }
    }
}
```

### Kubernetes Deployment

```dsl
workspace {

    model {
        ingress = deploymentNode "Ingress Controller" "nginx" {
            nginx = container "Nginx" "nginx-ingress" "TLS, routing"
        }

        cluster = deploymentNode "Kubernetes Cluster" "EKS 1.28" {

            nsApp = deploymentNode "app namespace" "Application" {

                apiDeploy = deploymentNode "api-deployment" "3 replicas" {
                    api = container "API Pod" "Node.js 20" "REST API"
                }

                workerDeploy = deploymentNode "worker-deployment" "2 replicas" {
                    worker = container "Worker Pod" "Python 3.11" "Background jobs"
                }
            }

            nsData = deploymentNode "data namespace" "Databases" {

                pgStateful = deploymentNode "postgres-statefulset" "HA" {
                    pg = containerDb "PostgreSQL" "PostgreSQL 15" "Primary + Replica"
                }

                redisStateful = deploymentNode "redis-statefulset" "Cluster" {
                    redis = containerDb "Redis" "Redis 7" "3 node cluster"
                }
            }
        }

        nginx -> api "Routes /api/*" "HTTP"
        api -> pg "Queries" "JDBC"
        api -> redis "Caches" "Redis"
        worker -> pg "Processes" "JDBC"
    }

    views {
        deployment "Production - Kubernetes" {
            include *
            autoLayout
        }
    }
}
```

### Multi-Region Deployment

```dsl
workspace {

    model {
        globalLB = deploymentNode "Global Load Balancer" "AWS Global Accelerator" {
            glb = container "GLB" "AWS" "Geographic routing"
        }

        usEast = deploymentNode "US-East-1" "Primary Region" {
            usEcs = deploymentNode "ECS Cluster" "Fargate" {
                usApi = container "API" "Node.js" "US instances"
            }
            usRds = deploymentNode "RDS" "Multi-AZ" {
                usPrimary = containerDb "Primary DB" "PostgreSQL" "Write leader"
            }
        }

        euWest = deploymentNode "EU-West-1" "Secondary Region" {
            euEcs = deploymentNode "ECS Cluster" "Fargate" {
                euApi = container "API" "Node.js" "EU instances"
            }
            euRds = deploymentNode "RDS" "Read Replica" {
                euReplica = containerDb "Replica DB" "PostgreSQL" "Read replica"
            }
        }

        glb -> usApi "US traffic" "HTTPS"
        glb -> euApi "EU traffic" "HTTPS"
        usApi -> usPrimary "Reads/writes" "JDBC"
        euApi -> euReplica "Reads" "JDBC"
        euApi -> usPrimary "Writes" "JDBC"
        usPrimary -> euReplica "Replicates" "Streaming"
    }

    views {
        deployment "Multi-Region Active-Active" {
            include *
            autoLayout
        }
    }
}
```

## API Documentation Patterns

### API Gateway Pattern

```dsl
workspace {

    model {
        mobile = person "Mobile User" "iOS/Android app user"
        web = person "Web User" "Browser user"
        partner = person "Partner" "Third-party integration"

        mobileApp = container "Mobile App" "React Native" "Native mobile client"
        webApp = container "Web App" "React" "SPA client"

        gateway = container "API Gateway" "Kong" "Auth, rate limit, routing"
        bff = container "BFF" "Node.js" "Backend for frontend"

        userApi = container "User API" "Java" "User management"
        orderApi = container "Order API" "Go" "Order processing"
        productApi = container "Product API" "Python" "Product catalog"

        auth0 = softwareSystem "Auth0" "Identity provider" {
            tags "External"
        }

        mobile -> mobileApp "Uses"
        web -> webApp "Uses"
        partner -> gateway "API calls" "REST/HTTPS"

        mobileApp -> bff "GraphQL" "HTTPS"
        webApp -> bff "GraphQL" "HTTPS"

        bff -> gateway "REST calls" "HTTP"
        gateway -> auth0 "Validates tokens" "HTTPS"

        gateway -> userApi "Routes /users/*" "HTTP"
        gateway -> orderApi "Routes /orders/*" "HTTP"
        gateway -> productApi "Routes /products/*" "HTTP"
    }

    views {
        container gateway {
            include *
            autoLayout
        }
    }
}
```

### API Component Detail

```dsl
workspace {

    model {
        gateway = container "API Gateway" "Kong"
        db = containerDb "Order DB" "PostgreSQL"
        events = containerQueue "Order Events" "Kafka"
        payment = softwareSystem "Payment Service" "Stripe" {
            tags "External"
        }

        orderApi = container "Order API" {
            controller = component "Order Controller" "Spring MVC" "REST endpoints"
            validator = component "Request Validator" "Bean Validation" "Input validation"
            service = component "Order Service" "Spring Service" "Business logic"
            paymentClient = component "Payment Client" "Feign" "Stripe integration"
            repository = component "Order Repository" "Spring Data JPA" "Data access"
            publisher = component "Event Publisher" "Spring Kafka" "Event publishing"

            gateway -> controller "HTTP requests" "JSON"
            controller -> validator "Validates"
            controller -> service "Delegates"
            service -> paymentClient "Charges"
            service -> repository "Persists"
            service -> publisher "Publishes events"

            paymentClient -> payment "REST" "HTTPS"
            repository -> db "JDBC" "SQL"
            publisher -> events "Produces" "Avro"
        }
    }

    views {
        component orderApi {
            include *
            autoLayout
        }
    }
}
```

## Supplementary Diagram Patterns

### Authentication Flow (Dynamic)

```dsl
workspace {

    model {
        spa = container "SPA" "React" "Web application"
        api = container "API" "Node.js" "Resource server"
        auth0 = softwareSystem "Auth0" "Authorization server" {
            tags "External"
        }
        db = containerDb "User DB" "PostgreSQL" "User data"

        spa -> auth0 "1. Redirect to /authorize"
        auth0 -> spa "2. Redirect with auth code"
        spa -> api "3. Exchange code for tokens" "HTTPS"
        api -> auth0 "4. POST /oauth/token" "HTTPS"
        api -> spa "5. Return access + refresh tokens"
        spa -> api "6. API request with access token" "HTTPS"
        api -> db "7. Fetch user data" "SQL"
    }

    views {
        dynamic "OAuth2 Authorization Code Flow" {
            include *
            autoLayout

            spa -> auth0 "1. Redirect to /authorize"
            auth0 -> spa "2. Redirect with auth code"
            spa -> api "3. Exchange code for tokens" "HTTPS"
            api -> auth0 "4. POST /oauth/token" "HTTPS"
            api -> spa "5. Return access + refresh tokens"
            spa -> api "6. API request with access token" "HTTPS"
            api -> db "7. Fetch user data" "SQL"
        }
    }
}
```

### Error Handling Flow

```dsl
workspace {

    model {
        api = container "API" "Node.js"
        circuitBreaker = container "Circuit Breaker" "Resilience4j"
        payment = softwareSystem "Payment Service" "Stripe" {
            tags "External"
        }
        fallback = containerDb "Fallback Cache" "Redis"

        api -> circuitBreaker "1. Request payment"
        circuitBreaker -> payment "2. Forward request" "HTTPS"
        payment -> circuitBreaker "3a. Success response"
        circuitBreaker -> api "4a. Return success"
        payment -> circuitBreaker "3b. Timeout/Error"
        circuitBreaker -> fallback "4b. Check cached response"
        circuitBreaker -> api "5b. Return fallback or error"
    }

    views {
        dynamic "Error Handling - Circuit Breaker" {
            include *
            autoLayout

            api -> circuitBreaker "1. Request payment"
            circuitBreaker -> payment "2. Forward request" "HTTPS"
            payment -> circuitBreaker "3a. Success response"
            circuitBreaker -> api "4a. Return success"
            payment -> circuitBreaker "3b. Timeout/Error"
            circuitBreaker -> fallback "4b. Check cached response"
            circuitBreaker -> api "5b. Return fallback or error"
        }
    }
}
```

## Architecture Decision Record Integration

Link C4 diagrams to Architecture Decision Records (ADRs):

### ADR Reference in Diagrams

```dsl
workspace {

    model {
        gateway = container "API Gateway" "Kong" "ADR-001: Selected for plugin ecosystem"
        api = container "Order API" "Spring Boot" "Order processing"
        db = containerDb "Order DB" "PostgreSQL" "ADR-002: ACID compliance required"
        events = containerQueue "Events" "Kafka" "ADR-003: Event sourcing pattern"

        gateway -> api "Routes" "HTTP"
        api -> db "Persists" "JDBC"
        api -> events "Publishes" "Avro"
    }

    views {
        container gateway {
            include *
            autoLayout
        }
    }
}
```

### Directory Structure

Organize C4 diagrams with ADRs:

```
docs/
├── architecture/
│   ├── c4-context.dsl
│   ├── c4-containers.dsl
│   ├── c4-components-order-api.dsl
│   ├── c4-deployment-production.dsl
│   └── c4-dynamic-auth-flow.dsl
└── decisions/
    ├── 001-api-gateway-selection.md
    ├── 002-database-selection.md
    ├── 003-event-driven-architecture.md
    └── template.md
```

## System Landscape Diagram

For enterprise-level views showing multiple systems:

```dsl
workspace {

    model {
        customer = person "Customer" "External customer"
        employee = person "Employee" "Internal staff"
        partner = person "Partner" "Business partner"

        group "Customer-Facing" {
            ecommerce = softwareSystem "E-commerce Platform" "Online store"
            mobile = softwareSystem "Mobile App" "Customer mobile experience"
            support = softwareSystem "Support Portal" "Customer service"
        }

        group "Internal Systems" {
            erp = softwareSystem "ERP System" "SAP - Finance & operations"
            crm = softwareSystem "CRM System" "Salesforce - Customer data"
            analytics = softwareSystem "Analytics Platform" "Business intelligence"
        }

        group "Integration Layer" {
            esb = softwareSystem "Integration Hub" "MuleSoft - API management"
            etl = softwareSystem "Data Pipeline" "Airflow - Data processing"
        }

        payment = softwareSystem "Payment Gateway" "Stripe" {
            tags "External"
        }
        shipping = softwareSystem "Shipping Provider" "FedEx" {
            tags "External"
        }
        warehouse = softwareSystem "Warehouse System" "3PL Partner" {
            tags "External"
        }

        customer -> ecommerce "Shops online"
        customer -> mobile "Uses app"
        customer -> support "Gets help"
        employee -> erp "Manages operations"
        employee -> crm "Manages customers"
        partner -> esb "API integration"

        ecommerce -> esb "API calls"
        esb -> erp "Syncs orders"
        esb -> crm "Syncs customers"
        esb -> payment "Processes payments"
        esb -> shipping "Creates shipments"
        etl -> analytics "Feeds data"
    }

    views {
        systemLandscape {
            include *
            autoLayout
        }
    }
}
```

## Best Practices Summary

1. **Choose abstraction based on ownership**: Single team = containers, Multi-team = systems
2. **Show individual message topics**: Not a single "Kafka" or "RabbitMQ" box
3. **Use deployment diagrams for infrastructure**: Keep container diagrams logical
4. **Create dynamic diagrams for complex flows**: Authentication, payment, error handling
5. **Link to ADRs**: Document why decisions were made
6. **Use system landscape for enterprise views**: Show all systems and their relationships
7. **Keep diagrams focused**: One concern per diagram, split when complex
