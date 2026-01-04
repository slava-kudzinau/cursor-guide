---
title: "Appendix A: Complete instructions.md Guide"
parent: "Appendices"
nav_order: 1
---

# Appendix A: Complete instructions.md Guide

**Part of**: [Cursor IDE: Complete Technical Guide](../../README.md)  
**Referenced from**: [Environment & Project Setup](../01-fundamentals-core-concepts/02-environment-project-setup.md)

---

## 📋 Overview

This appendix provides comprehensive guidelines for creating and maintaining an effective `instructions.md` file for your Cursor IDE projects.

The `instructions.md` file serves as your project's single source of truth, providing consistent context to the AI agent across all interactions.

---

## Core Sections (Recommended)

### 1. **Project Title & Description**
Start with a clear project name and 1-2 sentence description.

```markdown
# Project: E-commerce Platform

A modern multi-tenant e-commerce platform with real-time inventory,
payment processing, and order fulfillment.
```

### 2. **Architecture**
List your tech stack, frameworks, and infrastructure. Be specific with versions.

**What to include:**
- Programming languages and versions
- Frameworks and major libraries
- Database(s) and caching layers
- Message queues, job processors
- Hosting/deployment platform
- Monorepo vs multi-repo structure

**Example:**
```markdown
## Architecture
- **Monorepo:** pnpm workspaces
- **Backend:** Node.js 20 + Fastify 4.x + Prisma ORM
- **Frontend:** Next.js 14 (App Router) + React 18 + Tailwind CSS
- **Database:** PostgreSQL 15 (primary), Redis 7 (cache/sessions)
- **Queue:** BullMQ for async jobs
- **Deployment:** Vercel (frontend), Railway (backend)
- **Storage:** AWS S3 for media uploads
```

### 3. **Current Sprint/Goals**
What you're actively working on RIGHT NOW. Update this frequently (weekly/sprint).

**What to include:**
- Current features being built
- Immediate priorities
- Refactoring efforts
- Migration tasks

**Example:**
```markdown
## Current Sprint Goals (Week of Jan 6, 2026)
1. ✅ Implement checkout flow with cart persistence
2. 🚧 Add Stripe payment processing (webhooks in progress)
3. ⏳ Email notifications (SendGrid integration)
4. 📋 Admin dashboard for order management
5. 🔧 Migrate from REST to tRPC for type safety
```

**Pro tip:** Use emojis or markers (✅ done, 🚧 in progress, ⏳ next, 📋 planned)

### 4. **Conventions & Standards**
Your project's "way of doing things." This prevents the AI from guessing.

**What to include:**
- API design patterns (REST/GraphQL/tRPC/gRPC)
- Error handling format
- Authentication/authorization approach
- Testing requirements and coverage goals
- Code style and linting rules
- Commit message format
- Branch naming conventions
- Documentation standards

**Example:**
```markdown
## Conventions

### API
- **Style:** REST, all routes prefixed with `/api/v1/`
- **Response format:** 
  - Success: `{ data: {...}, meta?: {...} }`
  - Error: `{ error: { code: string, message: string, details?: {} } }`
- **Status codes:** 200 (success), 400 (bad request), 401 (unauthorized), 403 (forbidden), 404 (not found), 500 (server error)

### Authentication
- JWT tokens (access + refresh pattern)
- Access token: 15 min expiry
- Refresh token: 7 days, stored in httpOnly cookie
- All protected routes require `Authorization: Bearer <token>` header

### Testing
- **Unit tests:** Vitest (aim for >80% coverage)
- **E2E tests:** Playwright for critical user flows
- **API tests:** Supertest for all endpoints
- Run tests before committing: `pnpm test`

### Code Style
- **Linting:** ESLint + Prettier (auto-format on save)
- **TypeScript:** Strict mode enabled, no `any` types
- **Imports:** Use absolute imports (`@/components/...`)
- **Components:** Functional components with hooks (no class components)

### Git Workflow
- **Commits:** Conventional commits (feat:, fix:, docs:, refactor:, test:, chore:)
- **Branches:** `feature/add-checkout`, `fix/payment-webhook`, `refactor/user-auth`
- **PRs:** Require 1 approval, all tests passing
```

### 5. **Project Structure & Key Files**
Help the AI navigate your codebase quickly.

**What to include:**
- Directory structure overview
- Most important files and their purposes
- Where to find specific functionality
- Entry points for different services

**Example:**
```markdown
## Project Structure

### Key Directories
```
/apps
  /web              # Next.js frontend
  /api              # Fastify backend
/packages
  /ui               # Shared React components
  /db               # Prisma schema & migrations
  /types            # Shared TypeScript types
  /config           # Shared configs (ESLint, TS, etc.)
```

### Critical Files
- **Backend:**
  - `apps/api/src/server.ts` - API server entry point
  - `apps/api/src/lib/auth.ts` - JWT authentication logic
  - `apps/api/src/lib/db.ts` - Database client initialization
  - `apps/api/src/routes/` - All API route handlers
  
- **Frontend:**
  - `apps/web/src/app/layout.tsx` - Root layout (App Router)
  - `apps/web/src/lib/api-client.ts` - API fetch wrapper
  - `apps/web/src/hooks/` - Shared React hooks
  
- **Database:**
  - `packages/db/prisma/schema.prisma` - Database schema (source of truth)
  - `packages/db/prisma/migrations/` - Migration history

- **Configuration:**
  - `.env.example` - Required environment variables
  - `turbo.json` - Monorepo build configuration
```

### 6. **Known Issues & Workarounds**
Document current problems so the AI doesn't suggest broken approaches.

**What to include:**
- Active bugs and their workarounds
- Performance bottlenecks
- Integration issues
- Dependencies with known problems
- Temporary hacks that need cleanup

**Example:**
```markdown
## Known Issues

### Active Bugs
- **Payment webhooks fail in staging:** Stripe webhook secret mismatch
  - Workaround: Test payments in dev with Stripe CLI
  - Tracking: Issue #142
  
- **Search slow for >10k products:** Full-text search on PostgreSQL struggles
  - Plan: Migrate to Elasticsearch (Q1 2026)
  - Current: Added pagination limit of 100 items

### Technical Debt
- User sessions stored in Redis without TTL (memory leak risk)
- Frontend has mix of Pages Router and App Router (mid-migration)
- Some API routes missing input validation (legacy code)

### Environment Issues
- Local Redis sometimes crashes on Windows (use Docker instead)
- Next.js dev server occasionally needs restart when changing .env
```

### 7. **Off-Limits / Do NOT**
Critical guardrails to prevent the AI from breaking things.

**What to include:**
- Files/code that should never be modified
- Dangerous operations
- Breaking changes that need approval
- Security-sensitive areas

**Example:**
```markdown
## Off-Limits / Do NOT

### Database
- ❌ Do NOT modify migration files in `prisma/migrations/` manually
- ❌ Do NOT use `prisma db push` in production (use migrations only)
- ❌ Do NOT expose database credentials in code or logs

### API
- ❌ Do NOT change API response structure without versioning
- ❌ Do NOT remove or rename existing API endpoints (deprecate instead)
- ❌ Do NOT expose internal IDs directly (use UUIDs or hashids)

### Security
- ❌ Do NOT commit API keys, secrets, or credentials
- ❌ Do NOT disable CORS without understanding implications
- ❌ Do NOT store passwords in plain text (always hash with bcrypt)

### Dependencies
- ❌ Do NOT upgrade major versions without testing
- ❌ Do NOT install packages without checking licenses

### Code Quality
- ❌ Do NOT bypass TypeScript errors with `@ts-ignore`
- ❌ Do NOT commit commented-out code (use git history)
- ❌ Do NOT skip writing tests for new features
```

### 8. **Development Commands**
Quick reference for common tasks.

**Example:**
```markdown
## Development Commands

### Setup
```bash
pnpm install              # Install dependencies
pnpm db:setup            # Initialize database + run migrations
cp .env.example .env     # Set up environment variables
```

### Running Locally
```bash
pnpm dev                 # Start all services (turbo)
pnpm dev:web             # Frontend only
pnpm dev:api             # Backend only
```

### Testing
```bash
pnpm test                # Run all tests
pnpm test:watch          # Watch mode
pnpm test:e2e            # E2E tests
pnpm test:coverage       # Coverage report
```

### Database
```bash
pnpm db:migrate          # Run pending migrations
pnpm db:seed             # Seed with test data
pnpm db:studio           # Open Prisma Studio
```

### Building
```bash
pnpm build               # Build all packages
pnpm lint                # Run linter
pnpm format              # Format with Prettier
```
```

### 9. **Environment Variables**
Document required and optional environment variables.

**Example:**
```markdown
## Environment Variables

### Required
```
DATABASE_URL=postgresql://user:pass@localhost:5432/mydb
REDIS_URL=redis://localhost:6379
JWT_SECRET=your-secret-key-here
STRIPE_SECRET_KEY=sk_test_...
```

### Optional
```
SENDGRID_API_KEY=SG...           # For email notifications
AWS_S3_BUCKET=my-bucket           # For file uploads
SENTRY_DSN=https://...            # Error tracking
NODE_ENV=development              # development | production
```

### Getting API Keys
- Stripe: https://dashboard.stripe.com/apikeys
- SendGrid: https://app.sendgrid.com/settings/api_keys
- AWS: https://console.aws.amazon.com/iam/
```

### 10. **Team Context** (Optional)
Helpful for team projects.

**Example:**
```markdown
## Team Context

### Domain Experts
- **Payments:** @alice (Stripe integration expert)
- **DevOps:** @bob (Railway, CI/CD)
- **Frontend:** @carol (Next.js, React)

### Meeting Schedule
- Daily standup: 9:30 AM EST
- Sprint planning: Mondays 2 PM EST
- Code review: Async, 24hr SLA

### Communication
- Slack: #ecommerce-dev
- Urgent issues: Tag @on-call in Slack
- Docs: Notion workspace
```

---

## Complete Template Example

```markdown
# Project: E-commerce Platform

A modern multi-tenant e-commerce platform with real-time inventory,
payment processing, and order fulfillment.

## Architecture
- **Monorepo:** pnpm workspaces
- **Backend:** Node.js 20 + Fastify 4.x + Prisma ORM
- **Frontend:** Next.js 14 (App Router) + React 18 + Tailwind CSS
- **Database:** PostgreSQL 15 (primary), Redis 7 (cache/sessions)
- **Queue:** BullMQ for async jobs
- **Deployment:** Vercel (frontend), Railway (backend)
- **Storage:** AWS S3 for media uploads

## Current Sprint Goals (Week of Jan 6, 2026)
1. ✅ Implement checkout flow with cart persistence
2. 🚧 Add Stripe payment processing (webhooks in progress)
3. ⏳ Email notifications (SendGrid integration)
4. 📋 Admin dashboard for order management

## Conventions

### API
- REST, `/api/v1/` prefix
- Success: `{ data: {...} }`, Error: `{ error: { code, message } }`
- Status codes: 200, 400, 401, 403, 404, 500

### Auth
- JWT (access 15min + refresh 7d)
- Header: `Authorization: Bearer <token>`

### Testing
- Vitest (>80% coverage)
- Playwright E2E for critical flows

### Code Style
- TypeScript strict mode, no `any`
- Conventional commits: feat:, fix:, docs:, etc.
- ESLint + Prettier on save

## Project Structure
```
/apps/web         # Next.js frontend
/apps/api         # Fastify backend
/packages/db      # Prisma schema
```

### Key Files
- `apps/api/src/lib/auth.ts` - JWT authentication
- `packages/db/prisma/schema.prisma` - Database schema
- `apps/web/src/lib/api-client.ts` - API client

## Known Issues
- Payment webhooks fail in staging (Stripe secret mismatch)
- Search slow >10k products (needs Elasticsearch)

## Off-Limits
- Do NOT modify migration files manually
- Do NOT change API contract without versioning
- Do NOT expose internal IDs in responses
- Do NOT commit secrets or credentials

## Development Commands
```bash
pnpm install && pnpm db:setup    # First-time setup
pnpm dev                          # Start all services
pnpm test                         # Run tests
pnpm db:migrate                   # Run migrations
```

## Environment Variables (Required)
```
DATABASE_URL=postgresql://...
REDIS_URL=redis://...
JWT_SECRET=...
STRIPE_SECRET_KEY=sk_test_...
```
```

---

## Usage in Prompts

**Always reference it:**
```
@instructions.md

Implement checkout endpoint for shopping cart.
Follow existing patterns and conventions.
```

**For specific features:**
```
@instructions.md @src/routes/

Add new API endpoint for product reviews.
Use same error handling and auth patterns.
```

**Result:** Agent has full project context without you repeating it every time.

---

## Best Practices

### ✅ DO:
- **Update frequently** - Keep sprint goals current
- **Be specific** - "Next.js 14 App Router" not just "Next.js"
- **Include versions** - Helps with breaking changes
- **Document workarounds** - Save time on known issues
- **Use examples** - Show API response format, not just describe it
- **Keep it concise** - AI reads this every time, don't write essays

### ❌ DON'T:
- **Don't copy entire configs** - Link to them instead
- **Don't include implementation details** - This is context, not code
- **Don't let it get stale** - Old info is worse than no info
- **Don't duplicate documentation** - Link to existing docs
- **Don't include secrets** - Even example ones

---

## For Different Project Types

### Mobile App:
```markdown
## Architecture
- **Framework:** React Native 0.73 (Expo 50)
- **Navigation:** Expo Router (file-based)
- **State:** Zustand + React Query
- **Backend:** Supabase (auth, database, storage)
- **Platforms:** iOS 15+, Android 12+
```

### Python Backend:
```markdown
## Architecture
- **Language:** Python 3.11
- **Framework:** FastAPI + Pydantic v2
- **Database:** PostgreSQL 15 + SQLAlchemy 2.x
- **Tasks:** Celery + Redis
- **Deployment:** Docker + AWS ECS
```

### Microservices:
```markdown
## Architecture
- **Languages:** Go (services), TypeScript (frontend)
- **Communication:** gRPC (internal), REST (external)
- **Gateway:** Kong API Gateway
- **Service Mesh:** Istio
- **Observability:** OpenTelemetry + Grafana
```

---

## Maintenance Tips

### Weekly Review
- Update sprint goals with current priorities
- Mark completed items with ✅
- Add new known issues as they're discovered
- Remove resolved issues

### Monthly Review
- Review architecture section for outdated versions
- Update conventions if patterns have changed
- Clean up stale information
- Add new sections as project grows

### When Onboarding New Developers
- Walk through the entire `instructions.md`
- Update team context section
- Add their domain expertise
- Ensure all commands still work

---

**[← Back to Environment & Project Setup](../01-fundamentals-core-concepts/02-environment-project-setup.md)**
