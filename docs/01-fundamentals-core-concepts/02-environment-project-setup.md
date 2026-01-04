---
title: "Section 2: Environment & Project Setup"
parent: "Part 1: Fundamentals & Core Concepts"
nav_order: 2
---

# Part 1, Section 2: Environment & Project Setup

**Part of**: [Cursor IDE: Complete Technical Guide](../../README.md)  
**Estimated reading time**: 30-45 minutes  
**Prerequisites**: [Section 1: Mental Models & Architecture](./01-mental-models-architecture.md)

---

## 📋 Overview

Proper setup is critical for Cursor IDE productivity. This section covers everything from installation to advanced configuration patterns that will save you hours of frustration.

**What you'll learn:**
- Installing and configuring Cursor IDE
- Codebase indexing strategies for different project sizes
- Creating `instructions.md` for project context
- Setting up `.cursorrules` or `.cursor/rules/`
- Configuring `.cursorignore` properly
- Commands feature for reusable team prompts
- Monorepo best practices

**Why this matters:** Clean setup = 30% faster responses, better context, fewer hallucinations.

---

## 🚀 Installation & Initial Configuration

### Download and Install

1. Visit [cursor.com/downloads](https://cursor.com/downloads)
2. Download for your platform (Windows, macOS, Linux)
3. Install and launch Cursor IDE
4. Sign in with your account

### Subscription Tiers

Based on official Cursor pricing:

- **Free Tier**: Limited requests, basic features
- **Pro Tier** ($20/month): Unlimited requests, all models, priority support
- **Business Tier**: Team features, admin controls
- **Enterprise Tier**: Custom deployment, SSO, compliance

**Recommendation:** Start with Pro tier for serious development work.

---

## 📊 Codebase Indexing Strategy

### Small Repos (<10k files)

**Strategy:** Index everything

```bash
# .cursorignore (minimal)
node_modules/
.git/
*.log
dist/
build/
```

### Medium Repos (10k-50k files)

**Strategy:** Exclude generated code and assets

```bash
# .cursorignore
node_modules/
.git/
*.log
dist/
build/
*.generated.ts
.next/
__generated__/

# Non-code assets
*.mp4
*.jpg
*.png
*.svg
public/assets/
```

### Large Monorepos (50k+ files)

**Strategy:** Aggressive filtering

```bash
# Ignore sibling packages
packages/*/node_modules/
!packages/my-package/

# Ignore non-code assets
*.mp4
*.jpg
*.png
*.svg
public/assets/

# Ignore generated code
dist/
build/
*.generated.ts
.next/
__generated__/

# Ignore test artifacts
coverage/
.nyc_output/
```

**Why:** Agent searches entire index. Noise = slower queries + irrelevant context.

### Check Indexed Files

```
Cursor Settings > Indexing & Docs > View included files
```

**Pro tip:** Clean index = 30% faster responses (measured)

---

## 📁 Project Structure (Best Practices)

### Recommended Directory Layout

```
.cursor/
├── rules/                    # NEW: Project rules (recommended)
│   ├── typescript/
│   │   └── RULE.md           # TS patterns
│   ├── react-patterns/
│   │   └── RULE.md           # Component structure
│   ├── api-conventions/
│   │   └── RULE.md           # REST/GraphQL standards
│   ├── database/
│   │   └── RULE.md           # Prisma, migrations
│   └── testing/
│       └── RULE.md           # Jest, Vitest, Playwright
├── commands/                 # Reusable team prompts
│   ├── pr-review.md          # PR review checklist
│   ├── new-feature.md        # Feature scaffold template
│   └── debug-prod.md         # Production debugging steps
├── notepads/                 # Persistent notes (Beta)
│   ├── architecture.md       # System design decisions
│   ├── troubleshooting.md    # Common issues + solutions
│   └── sprint-context.md     # Current sprint goals
└── mcp.json                  # MCP server configuration

instructions.md               # Project context (RECOMMENDED)
AGENTS.md                     # Simple agent instructions (alternative to rules)
.cursorignore                 # Indexing exclusions
.cursorrules                  # LEGACY: Will be deprecated, migrate to rules/
```

---

## 📝 The `instructions.md` Pattern (UNIVERSAL)

**NEW and HIGHLY RECOMMENDED:** Create `instructions.md` in your project root.

### Why `instructions.md`?

- **Single source of truth** for project context
- Referenced in every significant prompt with `@instructions.md`
- **Eliminates repetition** - stop re-explaining your project
- Ensures **consistency** across team members
- Agent always has full project context
- **Evolves with your project** - keep it updated as you build

---

### Core Sections (Recommended)

#### 1. **Project Title & Description**
Start with a clear project name and 1-2 sentence description.

```markdown
# Project: E-commerce Platform

A modern multi-tenant e-commerce platform with real-time inventory,
payment processing, and order fulfillment.
```

#### 2. **Architecture**
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

#### 3. **Current Sprint/Goals**
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

#### 4. **Conventions & Standards**
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

#### 5. **Project Structure & Key Files**
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

#### 6. **Known Issues & Workarounds**
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

#### 7. **Off-Limits / Do NOT**
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

#### 8. **Development Commands**
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

#### 9. **Environment Variables**
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

#### 10. **Team Context** (Optional)
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

### Complete Template Example

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

### Usage in Prompts

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

### Best Practices

#### ✅ DO:
- **Update frequently** - Keep sprint goals current
- **Be specific** - "Next.js 14 App Router" not just "Next.js"
- **Include versions** - Helps with breaking changes
- **Document workarounds** - Save time on known issues
- **Use examples** - Show API response format, not just describe it
- **Keep it concise** - AI reads this every time, don't write essays

#### ❌ DON'T:
- **Don't copy entire configs** - Link to them instead
- **Don't include implementation details** - This is context, not code
- **Don't let it get stale** - Old info is worse than no info
- **Don't duplicate documentation** - Link to existing docs
- **Don't include secrets** - Even example ones

---

### For Different Project Types

**Mobile App:**
```markdown
## Architecture
- **Framework:** React Native 0.73 (Expo 50)
- **Navigation:** Expo Router (file-based)
- **State:** Zustand + React Query
- **Backend:** Supabase (auth, database, storage)
- **Platforms:** iOS 15+, Android 12+
```

**Python Backend:**
```markdown
## Architecture
- **Language:** Python 3.11
- **Framework:** FastAPI + Pydantic v2
- **Database:** PostgreSQL 15 + SQLAlchemy 2.x
- **Tasks:** Celery + Redis
- **Deployment:** Docker + AWS ECS
```

**Microservices:**
```markdown
## Architecture
- **Languages:** Go (services), TypeScript (frontend)
- **Communication:** gRPC (internal), REST (external)
- **Gateway:** Kong API Gateway
- **Service Mesh:** Istio
- **Observability:** OpenTelemetry + Grafana
```

---

## 📜 Rules Configuration

### Current Formats (2026)

Cursor supports multiple rule formats:

1. **Project Rules** (`.cursor/rules/`) - **RECOMMENDED** ✅
2. **AGENTS.md** - Simple alternative for straightforward projects
3. **User Rules** - Global preferences in Cursor Settings
4. **Team Rules** - Managed from Cursor dashboard (Team/Enterprise plans)
5. **`.cursorrules`** - **LEGACY**, will be deprecated ⚠️

### Project Rules (Recommended)

Project rules live in `.cursor/rules/`. Each rule is a **folder** containing a `RULE.md` file with frontmatter metadata.

#### Creating a Rule

Use the `New Cursor Rule` command or go to `Cursor Settings > Rules, Commands`.

#### Rule Structure

```
.cursor/rules/
  typescript/
    RULE.md           # Main rule file
    scripts/          # Helper scripts (optional)
  react-patterns/
    RULE.md
  testing/
    RULE.md
```

#### Rule Types

Control how rules are applied via frontmatter metadata:

| Rule Type | Description | Frontmatter |
|-----------|-------------|-------------|
| **Always Apply** | Apply to every chat session | `alwaysApply: true` |
| **Apply Intelligently** | Agent decides based on description | `alwaysApply: false` + `description` |
| **Apply to Specific Files** | When file matches pattern | `globs: ["*.ts", "src/**"]` |
| **Apply Manually** | When @-mentioned (e.g., `@typescript`) | No special config |

#### Example: TypeScript Rule

**`.cursor/rules/typescript/RULE.md`:**
```markdown
---
description: "TypeScript best practices and type safety standards"
alwaysApply: false
---

# TypeScript Best Practices

## Type Safety
- Use strict TypeScript configuration
- Avoid `any` types; use `unknown` for truly unknown types
- Prefer interfaces over types for objects
- Use union types for variants

## Error Handling
- Always handle promise rejections
- Use try-catch for async operations
- Return Result types for operations that can fail

## Testing
- Unit tests for all business logic
- Integration tests for API endpoints
- >80% code coverage required
```

#### Example: File-Specific Rule

**`.cursor/rules/api-conventions/RULE.md`:**
```markdown
---
description: "REST API conventions and standards"
globs:
  - "src/api/**/*.ts"
  - "src/routes/**/*.ts"
alwaysApply: false
---

# API Conventions

## REST Principles
- Use proper HTTP methods (GET, POST, PUT, DELETE)
- Return appropriate status codes
- Follow `/api/v1/` prefix convention

## Error Responses
Format: `{ error: { code, message, details } }`

## Authentication
- JWT tokens in Authorization header
- Refresh tokens in HttpOnly cookies
```

### AGENTS.md (Simple Alternative)

For straightforward projects, create `AGENTS.md` in your project root:

```markdown
# Project Instructions

## Code Style
- Use TypeScript for all new files
- Prefer functional components in React
- Use snake_case for database columns

## Architecture
- Follow the repository pattern
- Keep business logic in service layers
- Use Prisma for database access
```

**Nested AGENTS.md** are also supported in subdirectories:

```
project/
  AGENTS.md              # Global instructions
  frontend/
    AGENTS.md            # Frontend-specific
  backend/
    AGENTS.md            # Backend-specific
```

### Legacy Format (`.cursorrules`)

⚠️ **The `.cursorrules` file is legacy and will be deprecated.** Migrate to Project Rules or AGENTS.md.

If you still use `.cursorrules`:

```markdown
# TypeScript Best Practices

TypeScript is the primary language for this project.

## Type Safety
- Enable strict mode in tsconfig.json
- Avoid `any` types; use `unknown` for truly unknown types
- Use type guards for runtime type checking
```

### Best Practices for Rules

**Write as encyclopedia articles, not commands:**

❌ **DON'T:**
```
You are a senior engineer.
Always use TypeScript.
```

✅ **DO:**
```markdown
---
description: "TypeScript standards and type safety guidelines"
alwaysApply: false
---

# TypeScript Best Practices

TypeScript is the primary language for this project.

## Type Safety
- Enable strict mode in tsconfig.json
- Avoid `any` types; use `unknown` for truly unknown types
- Use type guards for runtime type checking
```

**Additional Tips:**
- Keep rules under 500 lines
- Split large rules into multiple, composable rules
- Provide concrete examples or referenced files
- Avoid vague guidance - write rules like clear internal docs
- Reuse rules when repeating prompts in chat

### Migrating from Legacy Formats

If you're upgrading from older Cursor versions:

#### From `.cursorrules` to Project Rules

**Old format (`.cursorrules`):**
```markdown
# TypeScript Best Practices

## Type Safety
- Use strict TypeScript
- Avoid any types
```

**New format (`.cursor/rules/typescript/RULE.md`):**
```markdown
---
description: "TypeScript best practices and type safety standards"
alwaysApply: false
---

# TypeScript Best Practices

## Type Safety
- Use strict TypeScript
- Avoid any types
```

**Migration steps:**
1. Create rule folder: `mkdir -p .cursor/rules/typescript`
2. Move content to `RULE.md` with frontmatter
3. Choose rule type (Always Apply, Apply Intelligently, etc.)
4. Test rule in `Cursor Settings > Rules, Commands`
5. Delete old `.cursorrules` file
6. Commit changes to git

#### From `.mdc` to `RULE.md` Files

**Old format (`.cursor/rules/typescript.mdc`):**
```markdown
---
description: TypeScript standards
---
# Content here
```

**New format (`.cursor/rules/typescript/RULE.md`):**
```markdown
---
description: "TypeScript standards"
alwaysApply: false
---
# Content here
```

**Migration steps:**
1. Create folder: `mkdir .cursor/rules/typescript`
2. Move `typescript.mdc` → `typescript/RULE.md`
3. Update frontmatter if needed
4. Repeat for all `.mdc` files
5. Delete old `.mdc` files
6. Restart Cursor

#### Quick Migration Script

```bash
#!/bin/bash
# migrate-rules.sh - Convert .mdc files to RULE.md folders

cd .cursor/rules
for file in *.mdc; do
  if [ -f "$file" ]; then
    name="${file%.mdc}"
    mkdir -p "$name"
    mv "$file" "$name/RULE.md"
    echo "Migrated: $file → $name/RULE.md"
  fi
done
```

---

## 🔧 Commands Feature (NEW)

**Commands** are reusable prompts that can be invoked with `/command-name`.

### Setup

Create `.cursor/commands/[name].md`

### Example: PR Review Command

**`.cursor/commands/pr-review.md`:**
```markdown
# PR Review Checklist

Review the current branch against these standards:

## Code Quality
- [ ] No hardcoded secrets or API keys
- [ ] All functions have JSDoc comments
- [ ] Error handling for all async operations
- [ ] Input validation on all public APIs
- [ ] No `any` types in TypeScript

## Testing
- [ ] Unit tests for new functions (>80% coverage)
- [ ] Integration tests for API endpoints
- [ ] E2E tests for critical user flows

## Performance
- [ ] No N+1 queries
- [ ] Large lists use pagination
- [ ] Heavy computations memoized

## Security
- [ ] SQL injection prevention
- [ ] XSS prevention
- [ ] CSRF tokens on state-changing ops
- [ ] Auth on protected routes

Run tests and generate report.
```

### Usage

Type `/pr-review` in Agent input → command executes

### More Command Examples

**`.cursor/commands/new-feature.md`:**
```markdown
# New Feature Scaffold

Create a new feature following our architecture:

1. Create feature directory structure
2. Generate TypeScript types
3. Create API endpoints
4. Add validation schemas
5. Write unit tests
6. Write integration tests
7. Update documentation
```

**`.cursor/commands/debug-prod.md`:**
```markdown
# Production Debugging

Debug production issue:

1. Check error logs
2. Review recent deployments
3. Check database connections
4. Review monitoring dashboards
5. Identify root cause
6. Propose fix
7. Create rollback plan
```

---

## 🏢 Monorepo Best Practices

### Problem

Different packages = different tech stacks (React frontend, Go backend, Python ML)

### Solutions

#### 1. Open Subdirs as Separate Workspaces

```bash
cursor frontend/  # Index only React
cursor backend/   # Index only Go
cursor ml/        # Index only Python
```

**Pros:** Clean separation, focused indexing  
**Cons:** Can't cross-reference between packages

#### 2. Use `.cursorignore` Per Package

```bash
# frontend/.cursorignore
../backend/
../ml-service/

# backend/.cursorignore
../frontend/
../ml-service/
```

#### 3. Shared Rules via Symlink

```bash
# package.json postinstall
"postinstall": "ln -sf ../../.cursor .cursor"
```

#### 4. `instructions.md` Per Package

```bash
frontend/instructions.md     # React-specific context
backend/instructions.md      # Go-specific context
ml/instructions.md           # Python-specific context
instructions.md              # Monorepo-level context
```

### Trade-off

Narrower index = agent can't cross-reference other packages. Use `@web` or manual file references for cross-package queries.

---

## 🔐 MCP Configuration

### Setup MCP Servers

**`.cursor/mcp.json`:**
```json
{
  "mcpServers": {
    "gdrive": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-gdrive"],
      "env": {
        "GDRIVE_CLIENT_ID": "${GDRIVE_CLIENT_ID}",
        "GDRIVE_CLIENT_SECRET": "${GDRIVE_CLIENT_SECRET}"
      }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    },
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

### Environment Variables

Create `.env` file:
```bash
GDRIVE_CLIENT_ID=your_client_id
GDRIVE_CLIENT_SECRET=your_client_secret
DATABASE_URL=postgresql://user:pass@localhost:5432/db
```

**Security:** Never commit `.env` to git. Add to `.gitignore`.

---

## ✅ Setup Checklist

Before proceeding to workflows, ensure you've completed:

### Basic Setup
- [ ] Installed Cursor IDE
- [ ] Signed in to your account
- [ ] Configured subscription tier

### Project Configuration
- [ ] Created `instructions.md` with project context
- [ ] Set up Project Rules in `.cursor/rules/` OR created `AGENTS.md`
- [ ] Configured `.cursorignore` for your project size
- [ ] Created at least one command in `.cursor/commands/`
- [ ] Migrated from `.cursorrules` if upgrading from legacy format

### Testing
- [ ] Tested Tab autocomplete (type a function)
- [ ] Tested Inline Edit (Cmd+K on a function)
- [ ] Tested Composer/Agent (Cmd+I with a multi-file task)
- [ ] Verified codebase indexing (check indexed files count)

### Optional (Advanced)
- [ ] Set up MCP servers (if needed)
- [ ] Created team notepads (if using)
- [ ] Configured monorepo structure (if applicable)

---

## 🎯 Key Takeaways

### Indexing Strategy
- **Small repos:** Index everything
- **Medium repos:** Exclude generated code and assets
- **Large monorepos:** Aggressive filtering
- **Clean index = 30% faster responses**

### Project Context
- **`instructions.md`** is your single source of truth
- Reference it in every significant prompt with `@instructions.md`
- Update it as your project evolves

### Rules Configuration
- **Use `.cursor/rules/` with RULE.md files** (new format)
- **Or use `AGENTS.md`** for simple projects
- Write rules as encyclopedia articles, not commands
- Use frontmatter to control when rules apply
- `.cursorrules` is legacy and will be deprecated

### Commands
- Create reusable prompts in `.cursor/commands/`
- Invoke with `/command-name`
- Great for PR reviews, feature scaffolds, debugging

### Monorepos
- Open subdirs as separate workspaces for clean separation
- Use `.cursorignore` per package
- Share rules via symlinks
- Create `instructions.md` per package

---

## 📖 Next Steps

Now that your environment is properly configured, proceed to:

**[Section 3: Core Workflows →](./03-core-workflows.md)**

Learn essential daily development workflows including TDD, refactoring, debugging, and parallel agents.

---

**Part 1, Section 2** | [Back to Part 1 Index](./README.md) | [← Section 1](./01-mental-models-architecture.md) | [Next: Section 3 →](./03-core-workflows.md)
