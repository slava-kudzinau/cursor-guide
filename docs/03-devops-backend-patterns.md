---
title: "Part 3: DevOps & Backend Architecture"
nav_order: 4
---

# Cursor IDE: DevOps & Backend Architecture
## Part 3 of 6: Infrastructure and Backend Patterns

**Maintainer**: Viachaslau Kudzinau (viachaslau_kudzinau@epam.com)  
**Version**: 2.0  
**Last Updated**: December 2025

> **📚 Navigation**: [Index](00-index.md) | [← Part 2](02-advanced-recipes-visual-dev.md) | **Part 3** | [Part 4 →](04-context-prompting-strategies.md)

---

## Table of Contents

6. [DevOps & Infrastructure](#6-devops--infrastructure)
7. [Backend Architecture Patterns](#7-backend-architecture-patterns)

---

## 6. DevOps & Infrastructure (NEW SECTION)

### 6.1 Kubernetes with MCP Integration

**Setup MCP server:**
```json
// .cursor/mcp.json
{
  "mcpServers": {
    "kubernetes": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-kubernetes"],
      "env": {
        "KUBECONFIG": "${HOME}/.kube/config"
      }
    }
  }
}
```

**Live infrastructure queries:**
```
Check logs from pod orders-api-78d8 in prod namespace.
Why are payment-service pods crashing?
List all services in staging namespace with external IPs.
Show resource usage for all nodes in prod cluster.
Scale deployment orders-api to 5 replicas in prod.
```

**Agent workflow:**
1. Uses MCP to query live Kubernetes cluster
2. Analyzes logs/errors
3. Suggests fixes
4. Can apply fixes with kubectl commands (YOLO mode)

**Real-world debugging scenario:**
```
I'm seeing 500 errors in production for /api/orders endpoint.

[Agent uses MCP to:]
1. Check pod logs: `kubectl logs -l app=orders-api --tail=100`
2. Find error: "connection timeout to Redis"
3. Check Redis service: `kubectl get svc redis`
4. Identify issue: Redis service endpoint missing
5. Review deployment: `kubectl get deploy orders-api -o yaml`
6. Propose fix: Add Redis service endpoint env var
7. Apply fix: `kubectl set env deployment/orders-api REDIS_URL=redis://redis:6379`
8. Verify: Check logs again, errors gone

Full resolution: 2 minutes
```

### 6.2 Terraform Patterns

**Multi-stage infrastructure:**
```
@instructions.md @terraform-docs

Create Terraform module for 3-tier web application:

Architecture:
- VPC with 3 subnets (1 public, 2 private)
- NAT Gateway in public subnet
- Application Load Balancer (public subnet)
- ECS Fargate tasks (private subnets)
- RDS PostgreSQL (private subnet with Multi-AZ)
- ElastiCache Redis (private subnet)
- S3 bucket for static assets
- CloudFront distribution

Security:
- Security groups (ALB → ECS → RDS, principle of least privilege)
- IAM roles for ECS tasks
- Secrets in AWS Secrets Manager
- Enable encryption at rest (RDS, S3)
- Enable encryption in transit (ALB)

Networking:
- Route tables for public/private subnets
- Internet Gateway for public subnet
- NAT Gateway for private subnet outbound
- VPC endpoints for AWS services (S3, ECR)

Observability:
- CloudWatch logs for ECS tasks
- CloudWatch alarms (CPU > 80%, error rate > 5%)
- VPC flow logs
- ALB access logs to S3

Cost optimization:
- Use smallest viable instance types
- Enable autoscaling (target tracking, CPU 70%)
- Use Spot instances for non-critical workloads

Generate:
- modules/vpc/main.tf
- modules/ecs/main.tf
- modules/rds/main.tf
- modules/alb/main.tf
- environments/dev/main.tf
- environments/prod/main.tf
- variables.tf (all configurable parameters)
- outputs.tf (important resource IDs)
- README.md (usage, architecture diagram)

Include Terraform best practices:
- Use remote state (S3 + DynamoDB locking)
- Separate modules for reusability
- Consistent naming (environment-service-resource)
- Tag all resources (Environment, Project, Owner, CostCenter)
```

### 6.3 Docker Multi-Stage Builds

**Pattern: Optimized production images**

```
@instructions.md

Create Dockerfile for Node.js API:

Requirements:
- Base: node:20-alpine
- Multi-stage build (deps, build, production)
- Final image < 200MB
- Run as non-root user (node)
- Include healthcheck
- Optimize layer caching

Stages:
1. deps: Install all dependencies (including devDependencies)
2. build: Copy source, build TypeScript, run tests
3. production: Copy only built files + production dependencies

Security:
- No secrets in layers
- Minimal attack surface (alpine)
- Non-root user
- Read-only root filesystem

Performance:
- Cache node_modules (copy package.json first)
- Parallel stages where possible
- Minimize layer count

Generate:
- Dockerfile
- .dockerignore
- docker-compose.yml (dev environment)
- docker-compose.prod.yml (production)

Validate with: docker build --no-cache -t app:test .
```

**Expected output:**
```dockerfile
# Stage 1: Dependencies
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

# Stage 2: Build
FROM node:20-alpine AS build
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build && npm run test

# Stage 3: Production
FROM node:20-alpine AS production
WORKDIR /app

# Create non-root user
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001

# Copy built assets and production dependencies
COPY --from=deps /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
COPY package*.json ./

# Switch to non-root user
USER nodejs

# Healthcheck
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"

EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### 6.4 CI/CD Pipeline Templates

**GitHub Actions with Matrix Strategy:**

```
@instructions.md

Create GitHub Actions workflow for monorepo:

Structure:
- packages/api (Node.js)
- packages/web (Next.js)
- packages/shared (TypeScript lib)

Workflow:
1. Detect changed packages (only run jobs for changed code)
2. Install dependencies (pnpm)
3. Run linting (ESLint)
4. Run type checking (TypeScript)
5. Run tests (Vitest with coverage)
6. Build packages
7. Build Docker images (only for api/web)
8. Push to AWS ECR
9. Deploy to ECS (staging on main, production on tags)

Optimization:
- Cache pnpm store
- Cache Docker layers
- Matrix strategy for parallel jobs
- Skip unchanged packages

Security:
- Use OIDC for AWS credentials (no long-lived keys)
- Scan Docker images (Trivy)
- Check dependencies for vulnerabilities (npm audit)

Generate:
- .github/workflows/ci.yml
- .github/workflows/deploy-staging.yml
- .github/workflows/deploy-production.yml
- scripts/detect-changes.sh (detect changed packages)
- scripts/docker-build.sh (build + push)
```

### 6.5 Helm Chart Generation

```
@instructions.md

Create Helm chart for microservice deployment:

Service: order-processor
Base image: myregistry/order-processor:latest

Chart structure:
- Chart.yaml (version 1.0.0)
- values.yaml (all configurable parameters)
- templates/
  - deployment.yaml (with rolling update strategy)
  - service.yaml (ClusterIP)
  - ingress.yaml (path: /api/orders)
  - configmap.yaml (env variables)
  - secret.yaml (template only, actual secrets via CI)
  - hpa.yaml (CPU > 70% → scale to max 10)
  - servicemonitor.yaml (Prometheus scraping)

Features:
- Resource limits (500m CPU, 512Mi memory)
- Health checks (liveness: /health, readiness: /ready)
- Pod anti-affinity (spread across nodes)
- Pod disruption budget (min 2 available)
- Init container to wait for DB
- Graceful shutdown (SIGTERM handling)

Support multiple environments:
- values-dev.yaml (1 replica, lower resources)
- values-staging.yaml (2 replicas, moderate resources)
- values-prod.yaml (5 replicas, full resources, autoscaling)

Generate:
- Full chart structure
- README.md with installation instructions
- Example values for each environment

Installation command:
helm install order-processor ./charts/order-processor -f values-prod.yaml -n production
```

---

## 7. Backend Architecture Patterns (NEW SECTION)

### 7.1 Microservices Consistency Pattern

**Pattern:** "Follow existing patterns" is most effective for microservices.

```
@instructions.md @services/user-service/ @services/order-service/

Create new payment-service microservice.

Analysis first:
1. Review user-service structure:
   - What's the folder layout?
   - How are routes organized?
   - What's the error handling pattern?
   - How are tests structured?

2. Review order-service patterns:
   - How does it handle async operations?
   - What's the logging format?
   - How are database transactions handled?

3. Identify common patterns to follow

After analysis, generate payment-service with:
- Same structure as user-service
- Same error handling as order-service
- Same testing patterns as both

Requirements:
- Port: 3003
- Database: PostgreSQL (like user-service)
- API: REST with OpenAPI spec (like order-service)
- Message queue: RabbitMQ for async events

Generate:
- src/index.ts (entry point)
- src/routes/ (API routes)
- src/services/ (business logic)
- src/models/ (DB models with Prisma)
- src/middleware/ (auth, validation, error handling)
- src/lib/ (shared utilities)
- tests/ (unit + integration)
- Dockerfile
- docker-compose.yml
- README.md
- .env.example

Follow patterns, don't reinvent.
```

**Why this works:**
- Maintains consistency across services
- Reduces architectural drift
- Easier onboarding for new developers
- Predictable debugging

### 7.2 Event Sourcing Implementation

```
@instructions.md

Implement event sourcing for Order aggregate:

Architecture:
- Event Store: PostgreSQL (events table)
- Event types: OrderCreated, OrderPaid, OrderShipped, OrderCancelled
- Aggregate: Order (rebuilt from events)
- Snapshots: Every 50 events for performance
- Projections: Read models for queries

Domain-Driven Design patterns:
- Aggregates enforce invariants
- Commands create events (Command → Events)
- Events are immutable (append-only)
- Use optimistic locking (version field)
- Domain events trigger side effects

Components:
1. Event definitions (TypeScript types)
2. Event store (append + replay)
3. Aggregate root (Order class)
4. Command handlers (validate + persist events)
5. Event handlers (build read models)
6. Snapshot mechanism (optimize replays)

Business rules:
- Order can't be paid if cancelled
- Order can't be shipped if not paid
- Cancellation after shipping requires refund
- All state changes must go through events

Generate:
- src/domain/Order.ts (aggregate root)
- src/domain/events.ts (event types)
- src/eventStore.ts (storage + replay)
- src/commands/ (command handlers)
- src/projections/ (read model builders)
- src/snapshots.ts (snapshot mechanism)
- tests/ (event replay, invariants, concurrency)
- migration.sql (events table schema)
- README.md (event sourcing explanation)

Include comprehensive tests:
- Event replay produces correct state
- Business rules enforced
- Concurrent updates handled (optimistic lock)
- Snapshots work correctly
```

### 7.3 CQRS (Command Query Responsibility Segregation)

```
@instructions.md

Implement CQRS for Product catalog:

Architecture separation:

Command side (writes):
- Handles: CreateProduct, UpdatePrice, UpdateStock, DeleteProduct
- Validates business rules
- Persists to write DB (PostgreSQL)
- Publishes events to message bus (RabbitMQ)
- Optimized for consistency

Query side (reads):
- Denormalized views (Elasticsearch)
- Subscribes to events from command side
- Updates search index
- Serves queries (fast reads)
- Eventually consistent

Tech stack:
- Commands: Fastify + Prisma + RabbitMQ
- Queries: Fastify + Elasticsearch client
- Events: RabbitMQ
- Language: TypeScript

Components:
1. Command API (POST /products, PUT /products/:id)
2. Command handlers (validation + persistence)
3. Event publisher (RabbitMQ)
4. Event subscriber (consume from RabbitMQ)
5. Query updater (sync to Elasticsearch)
6. Query API (GET /products, GET /products/search)

Eventual consistency handling:
- Return command result immediately (202 Accepted)
- Poll query side for updates (with timeout)
- Provide "freshness" indicator in responses

Generate:
- src/commands/ (write side)
- src/queries/ (read side)
- src/events/ (event definitions + publisher)
- src/sync/ (event handlers for read model)
- src/config/ (RabbitMQ + Elasticsearch setup)
- tests/ (consistency, event handling, search)
- docker-compose.yml (all services)
- README.md (architecture diagram)

Test scenarios:
- Create product → verify in write DB → verify in search
- Update price → verify event published → verify search updated
- Handle event replay (idempotency)
- Measure sync lag (write → read)
```

### 7.4 API Gateway Pattern

```
@instructions.md

Create API Gateway for microservices:

Services to aggregate:
- user-service (port 3001)
- order-service (port 3002)
- payment-service (port 3003)
- notification-service (port 3004)

Gateway features:
- Routing (path-based to services)
- Authentication (JWT validation)
- Rate limiting (per user, per IP)
- Request/response transformation
- Circuit breaker (fail fast on downstream errors)
- Request aggregation (combine multiple service calls)
- Caching (Redis for GET requests)
- Logging (structured logs with correlation ID)
- Metrics (Prometheus format)

Tech stack:
- Framework: Fastify
- Cache: Redis
- Rate limiting: redis-backed
- Circuit breaker: opossum
- Metrics: prom-client

Example routes:
- GET /api/users/:id → user-service
- GET /api/orders/:id → order-service
- POST /api/orders → order-service + payment-service (aggregated)
- GET /api/users/:id/orders → user-service + order-service (aggregated)

Generate:
- src/index.ts (gateway server)
- src/routes/ (route definitions)
- src/middleware/ (auth, rate-limit, logging)
- src/services/ (HTTP clients for each service)
- src/aggregators/ (combine multiple service calls)
- src/cache/ (Redis caching layer)
- src/circuit-breaker/ (resilience patterns)
- config/ (service endpoints, timeouts)
- tests/ (integration tests with mocked services)
- docker-compose.yml
- README.md

Include:
- Graceful degradation (return partial data if service down)
- Timeout handling (5s timeout per service)
- Retry logic (exponential backoff)
- Health check endpoint (aggregate service health)
```

---

## Best Practices Summary

### DevOps Checklist
- [ ] Use MCP for live infrastructure access
- [ ] Document infrastructure as code (Terraform/CDK)
- [ ] Multi-stage Docker builds for optimization
- [ ] Implement CI/CD with caching and parallelization
- [ ] Use Helm for Kubernetes deployments
- [ ] Enable monitoring and alerting
- [ ] Implement graceful degradation

### Backend Architecture Checklist
- [ ] Follow existing service patterns for consistency
- [ ] Use event sourcing for audit trails and temporal queries
- [ ] Separate read/write with CQRS for scalability
- [ ] Implement API gateway for cross-cutting concerns
- [ ] Use message queues for async communication
- [ ] Handle eventual consistency explicitly
- [ ] Add comprehensive integration tests

---

## Next Steps

Continue to [Part 4: Context Management & Prompting →](04-context-prompting-strategies.md)

Or return to the [Index](00-index.md) for the complete guide navigation.

---

**Part 3 of 6** | [← Part 2](02-advanced-recipes-visual-dev.md) | [Index](00-index.md) | [Part 4 →](04-context-prompting-strategies.md)

