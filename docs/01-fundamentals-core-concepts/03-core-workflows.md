---
title: "Section 3: Core Workflows"
parent: "Part 1: Fundamentals & Core Concepts"
nav_order: 3
---

# Part 1, Section 3: Core Workflows

**Part of**: [Cursor IDE: Complete Technical Guide](../../README.md)  
**Estimated reading time**: 45 minutes  
**Prerequisites**: [Section 1](./01-mental-models-architecture.md), [Section 2](./02-environment-project-setup.md)

---

## 📋 Overview

This section covers essential daily development workflows that will transform how you code with Cursor IDE.

**What you'll learn:**
- TDD workflow with Agent (test-first pattern)
- Refactoring pattern (multi-step, incremental)
- Debug Mode workflow (hypothesis-driven)
- Parallel agents (exploring multiple approaches)
- Infrastructure-as-Code workflows
- Progressive enhancement pattern

**Why this matters:** These workflows are battle-tested patterns that maximize productivity while maintaining code quality.

---

## 🧪 TDD with Agent (Test-First Pattern)

### The Pattern

Write tests first, then implementation, then run tests and fix until passing.

### Prompt Template

```
Write tests first, then implementation, then run tests and fix until passing.

Context:
- Test framework: Vitest
- Coverage target: >80%
- Edge cases: [null, undefined, empty array, large inputs, special characters]
- YOLO mode: enabled (auto-run tests)

Task: Implement markdown-to-HTML converter

Requirements:
1. Tests must cover:
   - Basic markdown (headers, bold, italic, links)
   - Edge cases (malformed markdown, empty input, XSS attempts)
   - Performance (handle 10MB input)

2. Implementation:
   - Use marked library (not regex)
   - Sanitize output (DOMPurify)
   - Handle errors gracefully

3. Agent workflow:
   - Generate comprehensive test suite
   - Wait for test approval
   - Implement converter
   - Run tests automatically
   - Fix failures iteratively
   - Report when all tests pass

DO NOT PROCEED to implementation until tests are reviewed.
```

### Why This Works

- Agent generates tests → you verify correctness
- Agent implements → tests validate behavior
- Agent iterates → no manual debugging loop
- YOLO mode (auto-run commands) closes the loop

### YOLO Mode Setup

```
Settings > Agent > YOLO Mode
Prompt: "Any tests (vitest, npm test, nr test), build commands (tsc, build), and file ops (mkdir, touch) are always allowed."

Allow list: test, build, tsc, mkdir, touch
Deny list: rm -rf, drop database, curl (unless specified)
```

### Example Flow

1. Agent creates `markdown.test.ts` with 15 test cases
2. You review tests → approve
3. Agent creates `markdown.ts` implementation
4. Agent runs `npm test`
5. If failures: Agent reads errors → fixes → re-runs
6. If pass: Agent reports success

---

## 🔄 Refactoring Pattern (Multi-Step)

### Bad Approach (Single-Shot)

```
Refactor this 500-line file to use modern patterns.
```

**Result:** Massive diff, unverifiable, likely breaks edge cases.

### Good Approach (Incremental)

```
Step 1: Analyze this file. Identify code smells, tech debt, performance issues.

Step 2: Plan refactoring strategy. Break into 3-5 atomic steps.

Step 3: Execute step 1. Show diff. Wait for approval.

Step 4: Run tests. Fix failures.

Step 5: Repeat for remaining steps.
```

### Real-World Example: Express → Fastify Migration

```
@instructions.md @src/routes/

Task: Migrate Express API to Fastify

Step 1: Analysis
Analyze Express routes:
- How many routes? (list them)
- What middleware used? (auth, cors, validation)
- What's the request/response pattern?
- Any custom error handling?
- Dependencies on Express-specific features?

Identify migration challenges:
- Breaking changes (req/res API differences)
- Incompatible middleware
- Performance considerations

Wait for review before proceeding.

Step 2: Migration Plan
Based on analysis, propose 5-step plan:
1. Set up Fastify server (parallel to Express)
2. Migrate authentication middleware
3. Convert /api/users routes (3 endpoints)
4. Convert /api/posts routes (5 endpoints)
5. Remove Express, promote Fastify

For each step:
- Estimated effort
- Risks
- Rollback plan
- Testing strategy

Wait for approval.

Step 3: Execute Step 1
Create src/fastify-server.ts
- Port over base config (cors, helmet, rate-limiting)
- Add health check route
- Run on port 3001 (Express stays on 3000)

After completion:
- Run: npm run dev (both servers)
- Test: curl localhost:3001/health
- Wait for approval before step 4

[Continue for remaining steps...]
```

### Composer Mode Recipe

```
@codebase @instructions.md

Task: Migrate all API routes from Express to Fastify

Use Plan Mode:
1. Create migration plan (Mermaid diagram)
2. For each route group:
   - Create Fastify version
   - Update tests
   - Run tests
   - Wait for approval
3. Update documentation
4. Remove Express dependencies

Constraints:
- Maintain existing error handling patterns
- Keep middleware compatible
- No breaking changes to API contract
- Backward compatibility for 2 weeks (both servers running)

Execute one route group at a time.
```

---

## 🐛 Debug Mode Workflow (NEW)

### When to Use

Elusive bugs, race conditions, runtime issues that are hard to reproduce.

### How It Works

1. Describe the bug
2. Agent generates hypotheses
3. Agent instruments code with debug logs
4. You reproduce the bug
5. Agent analyzes logs
6. Agent identifies root cause
7. Agent proposes fix
8. Agent applies fix + verifies

### Example

```
Bug: User sessions randomly expire after ~5 minutes (expected: 24 hours)

Debug this issue:
- Framework: Express + express-session
- Session store: Redis
- Auth: JWT + refresh tokens
- Environment: Production only (dev works fine)

Hypotheses to investigate:
- Redis connection issues
- Session cookie config (SameSite, Secure)
- Clock drift between servers
- Race condition in token refresh

Instrument code and analyze.
```

### Agent Response

```
Hypothesis 1: Session cookie not being set correctly
Instrumenting:
- Log session creation (timestamp, maxAge)
- Log cookie headers sent to client
- Log incoming cookie in subsequent requests

Hypothesis 2: Redis connection timeout
Instrumenting:
- Log Redis connection events
- Log session read/write operations
- Log Redis errors

[Generates instrumented code]

Reproduce the bug now, then share the logs.
```

### After Reproducing

```
Here are the logs: [paste logs]
```

**Agent:** Analyzes, identifies root cause (e.g., missing `secure: true` in production), proposes fix.

---

## 🔀 Parallel Agents (NEW in 2.0)

### When to Use

- Exploring multiple architectural approaches
- Want best-of-N solutions
- Complex refactors that can be parallelized
- Experimenting with different implementations

### Setup Git Worktrees

```bash
# Create worktree for parallel agent execution
git worktree add ../cursor-agent-1 HEAD
git worktree add ../cursor-agent-2 HEAD
git worktree add ../cursor-agent-3 HEAD
```

### Prompt

```
Run 3 parallel agents to implement user authentication:

Approach 1: JWT with refresh tokens
Approach 2: Session-based with Redis
Approach 3: Passkey (WebAuthn)

For each approach:
- Implement auth flow
- Write tests
- Document pros/cons
- Estimate implementation time

Evaluate and recommend best approach for our use case.
```

**Result:** Cursor runs 3 agents simultaneously, evaluates results, presents best option.

### Script for Automated Worktree Management

```bash
#!/bin/bash
# setup-parallel-agents.sh

AGENTS=3
BASE_DIR=$(pwd)

for i in $(seq 1 $AGENTS); do
  WORKTREE_DIR="../cursor-agent-$i"
  if [ ! -d "$WORKTREE_DIR" ]; then
    git worktree add "$WORKTREE_DIR" HEAD
    echo "Created worktree: $WORKTREE_DIR"
  fi
done

echo "Parallel agent worktrees ready"
```

---

## 🏗️ Infrastructure-as-Code (Terraform, CDK)

### Pattern: Context7-Grounded IaC

```
@instructions.md

Create a Terraform module using latest AWS provider syntax:
- VPC with 3 subnets (1 public, 2 private)
- NAT Gateway in public subnet
- Route tables for private subnets
- Security groups for web tier (port 443)

Use Context7 for current AWS provider best practices.

Generate:
- main.tf
- variables.tf
- outputs.tf
- examples/basic/main.tf
- README.md
```

Agent uses Context7 to fetch latest Terraform AWS provider docs (updates weekly).

### Alternative: Static Internal Docs

For static internal docs, use @Docs:
```
Settings > Indexing & Docs > Add documentation
URL: https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/
```

---

## ⚡ Lambda/Serverless Recipe

### Context Setup

```markdown
# .cursor/rules/lambda-patterns/RULE.md
---
description: "AWS Lambda patterns and best practices"
globs:
  - "src/lambdas/**/*.js"
  - "functions/**/*.ts"
alwaysApply: false
---

# AWS Lambda Patterns

## Handler structure
export const handler = async (event, context) => {
  // Validation
  // Business logic
  // Error handling with proper status codes
  return { statusCode: 200, body: JSON.stringify(result) }
}

## Dependencies
- Use layers for shared libs (aws-sdk, lodash)
- Keep deployment package <50MB
- Use environment variables for config

## Testing
- Mock AWS SDK calls with aws-sdk-mock
- Test locally with SAM CLI
- Integration tests hit real AWS resources (sandbox account)
```

### Prompt

```
@rules/lambda-patterns @instructions.md

Create a Lambda function using latest AWS SDK v3:
- Event: S3 upload to bucket `user-uploads`
- Action: Resize image, upload to `user-uploads-resized`
- Dependencies: sharp (image processing)
- Error handling: Log to CloudWatch, send SNS alert on failure

Use Context7 for current @aws-sdk/client-s3 and @aws-sdk/client-sns APIs.

Include:
- handler.js (main function)
- package.json (with sharp in layers)
- serverless.yml (deployment config)
- tests/ (unit + integration)

Follow @lambda-patterns
```

If agent shows exact SDK v3 module imports (e.g., `@aws-sdk/client-s3`), Context7 was used.

---

## 📈 Progressive Enhancement Pattern (NEW)

### Build Features Incrementally

Build features incrementally with validation at each stage:

```
@instructions.md

Implement search feature in 4 phases:

Phase 1: Core functionality (MVP)
- Input field
- Search button  
- Results list
- No filters, no sorting, no pagination

Generate code + tests.
Run tests.
Wait for approval.

Phase 2: Add features
- Filters (category, price range)
- Sorting (relevance, date, price)
- Show result count

Generate code + tests.
Run tests.
Wait for approval.

Phase 3: Optimize
- Debounced input (300ms)
- Loading states
- Empty states
- Error handling
- Keyboard shortcuts (⌘K to focus)

Generate code + tests.
Run tests.
Wait for approval.

Phase 4: Polish
- Pagination (infinite scroll)
- Highlight search terms
- Recent searches (localStorage)
- Search suggestions (API)

Generate final version + tests.
```

### Why This Works

- Smaller iterations = easier to validate
- Less risk of cascading errors
- Early feedback on approach
- Can ship Phase 1 while building Phase 2

---

## ⌨️ Essential Keyboard Shortcuts

### Core Features

| Action | Windows/Linux | macOS |
|--------|---------------|-------|
| Tab autocomplete | `Tab` | `Tab` |
| Inline Edit | `Ctrl+K` | `Cmd+K` |
| Composer/Agent | `Ctrl+I` | `Cmd+I` |
| Chat | `Ctrl+L` | `Cmd+L` |
| New chat | `Ctrl+Shift+L` | `Cmd+Shift+L` |
| Comment line | `Ctrl+/` | `Cmd+/` |
| Quick fix | `Ctrl+.` | `Cmd+.` |

### Agent Controls

| Action | Windows/Linux | macOS |
|--------|---------------|-------|
| Command palette | `Ctrl+Shift+P` | `Cmd+Shift+P` |
| Commands menu | Type `/` | Type `/` |
| Mention context | Type `@` | Type `@` |
| Submit prompt | `Ctrl+Enter` | `Cmd+Enter` |
| Cancel agent | `Esc` | `Esc` |

### Navigation

| Action | Windows/Linux | macOS |
|--------|---------------|-------|
| Quick file open | `Ctrl+P` | `Cmd+P` |
| Symbol search | `Ctrl+T` | `Cmd+T` |
| Find in file | `Ctrl+F` | `Cmd+F` |
| Find in project | `Ctrl+Shift+F` | `Cmd+Shift+F` |

---

## 🎯 Key Takeaways

### TDD Pattern
- Write tests first
- Agent implements
- Auto-run tests with YOLO mode
- Iterate until passing

### Refactoring Pattern
- Analyze first
- Plan incremental steps
- Execute one step at a time
- Run tests after each step
- Wait for approval before next step

### Debug Mode
- Describe the bug
- Agent generates hypotheses
- Agent instruments code
- Reproduce bug
- Agent analyzes logs
- Agent proposes fix

### Parallel Agents
- Use git worktrees
- Run multiple approaches simultaneously
- Agent evaluates and recommends best
- Great for exploration

### Infrastructure-as-Code
- Use Context7 for latest provider docs
- Generate complete modules
- Include examples and tests

### Progressive Enhancement
- Build in phases
- Validate each phase
- Ship early, iterate fast

---

## 📖 Next Steps

Congratulations! You've completed Part 1: Fundamentals & Core Concepts.

**Next:** Proceed to [Part 2: Advanced Features & Visual Development](../../02-advanced-features-visual-dev/) to learn:
- Advanced recipes (API migrations, database migrations)
- Visual development patterns (screenshot-to-code)
- Complex multi-phase workflows

Or return to [Part 1 Index](./README.md) to review any section.

---

**Part 1, Section 3** | [Back to Part 1 Index](./README.md) | [← Section 2](./02-environment-project-setup.md) | [Next: Part 2 →](../../02-advanced-features-visual-dev/)
