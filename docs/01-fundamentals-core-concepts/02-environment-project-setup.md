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

### What to Include

Your `instructions.md` should contain:

1. **Project Title & Description** - Clear 1-2 sentence overview
2. **Architecture** - Tech stack with specific versions
3. **Current Sprint/Goals** - What you're actively working on RIGHT NOW
4. **Conventions & Standards** - API patterns, testing, code style, git workflow
5. **Project Structure & Key Files** - Directory layout and critical files
6. **Known Issues & Workarounds** - Active bugs and temporary solutions
7. **Off-Limits / Do NOT** - Critical guardrails to prevent breaking changes
8. **Development Commands** - Quick reference for common tasks
9. **Environment Variables** - Required and optional env vars
10. **Team Context** (Optional) - Domain experts and communication channels

### Usage in Prompts

Always reference it when working with the AI:

```
@instructions.md

Implement checkout endpoint for shopping cart.
Follow existing patterns and conventions.
```

**Result:** Agent has full project context without you repeating it every time.

### Complete Guide

For detailed guidelines, templates, examples, and best practices for each section, see:

**📖 [Appendix A: Complete instructions.md Guide](../appendices/instructions-md-guide)**

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
