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
- Rules vs Skills vs Instructions decision framework
- Setting up Rules (`.cursor/rules/*.md`)
- Creating Skills for reusable workflows (`.cursor/skills/`)
- Configuring Hooks for lifecycle control (`.cursor/hooks.json`)
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
├── rules/                    # Project rules (always-on constraints)
│   ├── typescript.md         # TS patterns (any .md file works)
│   ├── react-patterns.md     # Component structure
│   ├── api-conventions.md    # REST/GraphQL standards
│   ├── database.md           # Prisma, migrations
│   └── testing.md            # Jest, Vitest, Playwright
├── skills/                   # Optional reusable workflows (NEW)
│   ├── test-loop/
│   │   └── SKILL.md          # TDD test-fix-iterate workflow
│   ├── scaffold-package/
│   │   └── SKILL.md          # Create new workspace package
│   ├── api-migration/
│   │   └── SKILL.md          # Migrate REST to GraphQL
│   └── code-review/
│       └── SKILL.md          # Automated review checklist
├── commands/                 # Reusable team prompts
│   ├── pr-review.md          # PR review checklist
│   ├── new-feature.md        # Feature scaffold template
│   └── debug-prod.md         # Production debugging steps
├── hooks.json                # Lifecycle hooks configuration (NEW)
├── notepads/                 # Persistent notes (Beta)
│   ├── architecture.md       # System design decisions
│   ├── troubleshooting.md    # Common issues + solutions
│   └── sprint-context.md     # Current sprint goals
└── mcp.json                  # MCP server configuration

instructions.md               # Project context (RECOMMENDED)
INSTRUCTIONS.md               # Alternative: uppercase variant also works
AGENTS.md                     # Simple agent instructions (alternative to rules)
.cursorignore                 # Indexing exclusions
.cursorrules                  # LEGACY: Will be deprecated, migrate to rules/
```

### Understanding the Directory Structure

#### Rules Directory (`.cursor/rules/`)

**Purpose:** Always-on constraints and guardrails

- Any `.md` file in this directory is loaded as a rule
- Cursor recursively scans for all `.md` files
- No specific naming required (`typescript.md`, `style.md`, `build.md` all work)
- Use frontmatter for conditional activation

**Example:** `.cursor/rules/typescript.md`

#### Skills Directory (`.cursor/skills/`)

**Purpose:** Optional reusable workflows

- Each skill is a folder with a `SKILL.md` file
- Skills activate conditionally based on relevance
- Can be invoked via slash commands (`/test-loop`)
- Agent auto-selects when goal matches

**Example:** `.cursor/skills/test-loop/SKILL.md`

#### Hooks Configuration (`.cursor/hooks.json`)

**Purpose:** Control agent lifecycle and iteration

- Defines when agent should stop or continue
- Enables verification loops
- Hooks run scripts at specific lifecycle points

**Example:** Stop hook that runs until tests pass

---

## 🎯 Rules vs Skills vs Instructions: Decision Framework

Before creating any guidance file in `.cursor/`, use this decision framework:

### The Three-Question Method

| Question | If YES → | Create | Location |
|----------|----------|--------|----------|
| **Must this ALWAYS hold?** | → Rule | `.cursor/rules/*.md` | Always enforced |
| **Is this a reusable workflow?** | → Skill | `.cursor/skills/*/SKILL.md` | Used when relevant |
| **Does this explain context?** | → Instructions | `INSTRUCTIONS.md` or `instructions.md` | Background info |

### Decision Examples

| Scenario | Type | Reasoning | File |
|----------|------|-----------|------|
| "Always use TypeScript strict mode" | **Rule** | Must always apply | `.cursor/rules/typescript.md` |
| "Refactor API endpoints workflow" | **Skill** | Useful sometimes | `.cursor/skills/api-migration/SKILL.md` |
| "Why we chose Fastify over Express" | **Instructions** | Context/rationale | `INSTRUCTIONS.md` |
| "Never commit secrets to git" | **Rule** | Critical constraint | `.cursor/rules/security.md` |
| "Test-fix-iterate loop pattern" | **Skill** | Productivity workflow | `.cursor/skills/test-loop/SKILL.md` |
| "Current sprint goals and priorities" | **Instructions** | Project context | `INSTRUCTIONS.md` |
| "All API responses use {data, error}" | **Rule** | Standard pattern | `.cursor/rules/api-conventions.md` |
| "Code review checklist procedure" | **Skill** | Optional workflow | `.cursor/skills/code-review/SKILL.md` |
| "Team coding style preferences" | **Rule** | Consistency standard | `.cursor/rules/style.md` |
| "Database migration strategy" | **Instructions** | Understanding | `INSTRUCTIONS.md` |

### Detailed Decision Tree

```mermaid
graph TD
    START["Need to add<br/>guidance?"] --> Q1{"Must it ALWAYS<br/>hold true?"}
    
    Q1 -->|Yes| RULE["📜 Rule<br/>.cursor/rules/*.md"]
    Q1 -->|No| Q2{"Is it a reusable<br/>workflow?"}
    
    Q2 -->|Yes| SKILL["🎨 Skill<br/>.cursor/skills/*/SKILL.md"]
    Q2 -->|No| Q3{"Does it explain<br/>context or intent?"}
    
    Q3 -->|Yes| INSTR["📖 Instructions<br/>INSTRUCTIONS.md"]
    Q3 -->|No| PROMPT["💬 Use inline<br/>in prompts"]
    
    style START fill:#e1f5ff,stroke:#0066cc,stroke-width:2px
    style RULE fill:#ffe4e1
    style SKILL fill:#fff4e1
    style INSTR fill:#d4edda
    style PROMPT fill:#f0f0f0
```

### Common Mistakes to Avoid

#### ❌ Mistake 1: Rules That Should Be Skills

**Bad:**
```markdown
# .cursor/rules/refactoring.md
When refactoring components:
1. Create tests first
2. Refactor incrementally
3. Run tests after each change
```

**Why wrong:** This is a workflow, not a constraint. It doesn't always apply.

**Fix:**
```markdown
# .cursor/skills/refactor-component/SKILL.md
---
name: "refactor-component"
description: "Safe component refactoring workflow"
---
# [Skill definition]
```

---

#### ❌ Mistake 2: Instructions That Should Be Rules

**Bad:**
```markdown
# INSTRUCTIONS.md
We prefer TypeScript strict mode.
Try to avoid using `any` types.
```

**Why wrong:** These are constraints that must always hold, not context.

**Fix:**
```markdown
# .cursor/rules/typescript.md
---
alwaysApply: true
---
# TypeScript Standards
- Use strict mode
- Avoid `any` types (use `unknown`)
```

---

#### ❌ Mistake 3: Skills That Should Be Rules

**Bad:**
```markdown
# .cursor/skills/no-secrets/SKILL.md
Check for committed secrets
```

**Why wrong:** Security checks aren't optional; they must always run.

**Fix:**
```markdown
# .cursor/rules/security.md
---
alwaysApply: true
---
# Security Rules
- Never commit secrets
- Scan for API keys before commit
```

### Quick Reference Table

| Content Type | Rules | Skills | Instructions |
|--------------|-------|--------|--------------|
| **Constraints** | ✅ Primary use | ❌ No | ❌ No |
| **Workflows** | ❌ No | ✅ Primary use | ❌ No |
| **Context** | ❌ No | ❌ No | ✅ Primary use |
| **Always on** | ✅ Yes | ❌ No | ✅ Yes |
| **Optional** | ❌ No | ✅ Yes | ❌ No |
| **Enforced** | ✅ Yes | ❌ No | ❌ No |
| **Examples** | Code style, security | TDD loop, migrations | Why decisions made |

### Best Practices

✅ **Do:**
- Start with Instructions for new projects
- Add Rules only when patterns become clear
- Create Skills for recurring multi-step tasks
- Keep each type focused on its purpose

❌ **Don't:**
- Mix concerns in one file type
- Create Rules for optional workflows
- Put constraints in Instructions
- Create Skills for simple one-step actions

---

### 🤖 Automated Project Initialization

For a systematic approach to setting up Rules, Skills, and Instructions in a new project, see:

**📖 [Appendix B: /project-init System Prompt](../appendices/project-init-prompt.md)**

This appendix provides a complete system prompt that can be used as a Cursor command to automatically extract and organize project constraints from an architecture specification.

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

Project rules live in `.cursor/rules/`. Each rule is simply a `.md` file - any markdown file in this directory is loaded as a rule. Cursor recursively scans for all `.md` files.

#### Creating a Rule

Use the `New Cursor Rule` command or go to `Cursor Settings > Rules, Commands`, or simply create a `.md` file in `.cursor/rules/`.

#### Rule Structure (Flexible Naming)

```
.cursor/rules/
  typescript.md       # Any .md file works
  react-patterns.md   # No specific naming required
  testing.md          # Simple flat files
  build.md
  security.md
  
  # OR organize in subdirectories:
  frontend/
    react.md
    styling.md
  backend/
    api.md
    database.md
```

**Key point:** Cursor loads **all `.md` files** in `.cursor/rules/`, not just files named `RULE.md`.

#### Rule Types

Control how rules are applied via frontmatter metadata:

| Rule Type | Description | Frontmatter |
|-----------|-------------|-------------|
| **Always Apply** | Apply to every chat session | `alwaysApply: true` |
| **Apply Intelligently** | Agent decides based on description | `alwaysApply: false` + `description` |
| **Apply to Specific Files** | When file matches pattern | `globs: ["*.ts", "src/**"]` |
| **Apply Manually** | When @-mentioned (e.g., `@typescript`) | No special config |

#### Example: TypeScript Rule

**`.cursor/rules/typescript.md`:**
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

**`.cursor/rules/api-conventions.md`:**
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

**New format (`.cursor/rules/typescript.md`):**
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
1. Create rules directory: `mkdir -p .cursor/rules`
2. Move content to `typescript.md` with frontmatter
3. Choose rule type (Always Apply, Apply Intelligently, etc.)
4. Test rule in `Cursor Settings > Rules, Commands`
5. Delete old `.cursorrules` file
6. Commit changes to git

#### From `.mdc` Files to `.md` Files

**Old format (`.cursor/rules/typescript.mdc`):**
```markdown
---
description: TypeScript standards
---
# Content here
```

**New format (`.cursor/rules/typescript.md`):**
```markdown
---
description: "TypeScript standards"
alwaysApply: false
---
# Content here
```

**Migration steps:**
1. Rename files: `.mdc` → `.md`
2. Update frontmatter if needed
3. Restart Cursor

#### Quick Migration Script

```bash
#!/bin/bash
# migrate-rules.sh - Rename .mdc files to .md

cd .cursor/rules
for file in *.mdc; do
  if [ -f "$file" ]; then
    newname="${file%.mdc}.md"
    mv "$file" "$newname"
    echo "Migrated: $file → $newname"
  fi
done
```

**Note:** No need for folder structure - flat `.md` files work perfectly!

---

## 🎨 Skills Configuration

**Skills** are optional, reusable workflows stored in `.cursor/skills/`. Unlike rules (always on), skills activate conditionally when relevant.

### Creating a Skill

**Structure:**
```
.cursor/skills/
  <skill-name>/
    SKILL.md        # Skill definition with frontmatter
```

### Example: Test-Loop Skill

**`.cursor/skills/test-loop/SKILL.md`:**
```markdown
---
name: "test-loop"
description: "Run tests, fix failures, iterate until all pass"
commands:
  - "test-loop"
  - "tdd"
  - "test-fix"
---

# Test-Fix-Iterate Loop

## Purpose
Automate the TDD workflow: write tests, run tests, fix failures, repeat until green.

## When to Use
- Implementing TDD workflow
- Fixing failing tests
- Ensuring test coverage before commit

## Workflow Steps

1. **Run test suite**
   ```bash
   npm test
   ```

2. **Analyze failures**
   - Read error messages
   - Identify root cause
   - Determine fix strategy

3. **Apply fix**
   - Edit source code
   - Address specific failure

4. **Re-run tests**
   ```bash
   npm test
   ```

5. **Repeat steps 2-4 until all tests pass**

6. **Verify final state**
   - All tests green ✅
   - No warnings
   - Code coverage maintained/improved

## Success Criteria
- ✅ All tests passing
- ✅ No test warnings
- ✅ Code coverage ≥ target threshold
- ✅ Build succeeds

## Example Usage

**Manual invocation:**
```
/test-loop

Implement user authentication with tests
```

**Automatic activation:**
```
@instructions.md Implement password reset feature.
Use TDD approach: tests first, then implementation.
```

Agent will automatically activate test-loop skill when it sees "TDD approach".
```

### Example: Scaffold-Package Skill

**`.cursor/skills/scaffold-package/SKILL.md`:**
```markdown
---
name: "scaffold-package"
description: "Create new workspace package with standard structure"
commands:
  - "scaffold-package"
  - "new-package"
---

# Scaffold New Package

## Purpose
Generate complete package structure following project conventions.

## When to Use
- Adding new package to monorepo
- Creating standalone library
- Bootstrapping microservice

## Workflow Steps

1. **Create directory structure**
   ```bash
   mkdir -p packages/<package-name>/{src,tests,dist}
   ```

2. **Generate package.json**
   - Set name, version, dependencies
   - Add build scripts
   - Configure exports

3. **Create tsconfig.json**
   - Extend root config
   - Set output directory
   - Configure source maps

4. **Add initial tests**
   - Create test setup file
   - Add sample test case

5. **Generate README.md**
   - Package purpose
   - Installation
   - Usage examples
   - API documentation

6. **Verify build**
   ```bash
   cd packages/<package-name>
   npm run build
   npm test
   ```

## Success Criteria
- ✅ Package compiles without errors
- ✅ Tests run and pass
- ✅ README contains essential info
- ✅ Follows monorepo conventions
```

### Skills Best Practices

✅ **Do:**
- Create skills for multi-step workflows (3+ steps)
- Include clear success criteria
- Provide usage examples
- Document when to use the skill

❌ **Don't:**
- Create skills for single commands
- Make skills too specific (not reusable)
- Skip the frontmatter metadata
- Forget to test skill activation

### Skill Activation Patterns

**1. Automatic selection** - Agent matches goal to description
```
"Implement checkout feature using TDD"
→ Activates test-loop skill automatically
```

**2. Slash command** - Explicit invocation
```
/test-loop
"Fix failing authentication tests"
```

**3. Inline mention** - Direct reference
```
"Use the test-loop skill to implement user registration"
```

---

## 🪝 Hooks Configuration

**Hooks** control agent lifecycle, enabling verification loops and stop conditions.

### Setting Up Hooks

**Configuration file:** `.cursor/hooks.json`

### Example: Stop Hook for Test Loop

**`.cursor/hooks.json`:**
```json
{
  "hooks": {
    "stop": {
      "script": "./scripts/stop-when-tests-pass.sh",
      "description": "Continue iterating until all tests pass"
    }
  }
}
```

**`./scripts/stop-when-tests-pass.sh`:**
```bash
#!/bin/bash
# Exit 0 = continue, Exit 1 = stop

# Run tests silently
npm test --silent

# If tests pass, stop iterating
if [ $? -eq 0 ]; then
  echo "✅ All tests passing - stopping iteration"
  exit 1  # Stop
else
  echo "❌ Tests failing - continuing to fix"
  exit 0  # Continue
fi
```

**Make script executable:**
```bash
chmod +x scripts/stop-when-tests-pass.sh
```

### Hook Use Cases

#### 1. Quality Gate Hook

**Stop when code coverage reaches threshold:**

```bash
#!/bin/bash
# scripts/coverage-gate.sh

coverage=$(npm test --coverage --silent | grep "Statements" | awk '{print $3}' | tr -d '%')

if [ "$coverage" -ge 80 ]; then
  echo "✅ Coverage ${coverage}% ≥ 80% - stopping"
  exit 1  # Stop
else
  echo "❌ Coverage ${coverage}% < 80% - continuing"
  exit 0  # Continue
fi
```

#### 2. Build Verification Hook

**Continue until build succeeds:**

```bash
#!/bin/bash
# scripts/build-gate.sh

npm run build 2>/dev/null

if [ $? -eq 0 ]; then
  echo "✅ Build successful - stopping"
  exit 1  # Stop
else
  echo "❌ Build failed - continuing to fix"
  exit 0  # Continue
fi
```

#### 3. Production Safety Hook

**Stop immediately if production detected:**

```bash
#!/bin/bash
# scripts/production-safety.sh

if [ "$ENVIRONMENT" = "production" ]; then
  echo "🚨 Production environment detected - stopping immediately"
  exit 1  # Stop immediately
fi

echo "✅ Safe environment - continuing"
exit 0  # Continue
```

### Hooks + YOLO Mode = Autonomous Iteration

**Without hooks:** Agent stops after each step, waits for approval.

**With hooks:** Agent iterates automatically based on hook conditions.

**Example prompt with hooks:**
```
@instructions.md

Implement user authentication with comprehensive tests.

Use TDD workflow:
1. Write tests
2. Implement features
3. Fix any failures
4. Iterate until all tests pass

YOLO mode: Enabled
Hook: Continue until tests pass
```

**Agent behavior:**
1. Writes authentication tests
2. Runs tests (fail) → Hook says "continue"
3. Implements auth logic
4. Runs tests (fail) → Hook says "continue"
5. Fixes bugs
6. Runs tests (pass) → Hook says "stop"
7. Reports completion

### Hooks Best Practices

✅ **Do:**
- Keep hook scripts fast (< 5 seconds)
- Use clear exit codes (0 = continue, 1 = stop)
- Log hook decisions for debugging
- Test hooks independently before using

❌ **Don't:**
- Make hooks slow (blocks agent)
- Use complex logic (keep simple)
- Forget error handling
- Hardcode values (use env vars)

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
- [ ] Set up Project Rules in `.cursor/rules/*.md` OR created `AGENTS.md`
- [ ] Configured `.cursorignore` for your project size
- [ ] Created at least one command in `.cursor/commands/`
- [ ] Migrated from `.cursorrules` if upgrading from legacy format

### Skills & Hooks (Optional but Recommended)
- [ ] Created `.cursor/skills/` directory
- [ ] Added at least one skill (e.g., test-loop)
- [ ] Configured `.cursor/hooks.json` for verification loops
- [ ] Tested skill activation with slash command
- [ ] Verified hook behavior with a simple test

### Testing
- [ ] Tested Tab autocomplete (type a function)
- [ ] Tested Inline Edit (Cmd+K on a function)
- [ ] Tested Composer/Agent (Cmd+I with a multi-file task)
- [ ] Verified codebase indexing (check indexed files count)
- [ ] Tested a skill with `/skill-name` command

### Optional (Advanced)
- [ ] Set up MCP servers (if needed)
- [ ] Created team notepads (if using)
- [ ] Configured monorepo structure (if applicable)
- [ ] Set up multiple hooks for different workflows

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

### Rules, Skills, Instructions Decision Framework
- **Rules** (`.cursor/rules/*.md`) - Always-on constraints ("Must always hold?")
- **Skills** (`.cursor/skills/*/SKILL.md`) - Optional workflows ("Reusable workflow?")
- **Instructions** (`INSTRUCTIONS.md`) - Project context ("Explains intent?")
- Any `.md` file in rules directory works - no need for specific naming

### Rules Configuration
- **Use `.cursor/rules/*.md`** - Any `.md` file works, not just `RULE.md`
- **Or use `AGENTS.md`** for simple projects
- Write rules as encyclopedia articles, not commands
- Use frontmatter to control when rules apply
- `.cursorrules` is legacy and will be deprecated

### Skills Configuration
- Create skills for multi-step workflows (3+ steps)
- Agent selects automatically or via slash commands
- Include clear success criteria and usage examples
- Examples: test-loop, scaffold-package, api-migration

### Hooks Configuration
- Hooks control agent iteration lifecycle
- Stop hooks determine when to stop (tests pass, coverage met)
- Combine with YOLO mode for autonomous iteration
- Keep hooks fast (< 5 seconds) and simple

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
