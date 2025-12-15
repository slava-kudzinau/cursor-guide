---
title: "Part 4: Context Management & Prompting"
nav_order: 5
---

# Cursor IDE: Context Management & Prompting
## Part 4 of 6: Mastering Context and Advanced Prompts

**Maintainer**: Viachaslau Kudzinau (viachaslau_kudzinau@epam.com)  
**Version**: 2.0  
**Last Updated**: December 2025

> **📚 Navigation**: [Index](../) | [← Part 3](03-devops-backend-patterns) | **Part 4** | [Part 5 →](05-team-domain-patterns)

---

## Table of Contents

8. [Context Management](#8-context-management)
9. [Advanced Prompting Strategies](#9-advanced-prompting-strategies)

---

## 8. Context Management (EXPANDED)

### 8.1 The "Instructions.md" Pattern (UNIVERSAL)

**Covered in Part 1, Section 2.2** - This is your single source of truth for project context. Reference it in every significant prompt with `@instructions.md`.

### 8.2 Context-First Approach (Anti-Hallucination)

**Pattern:** Always analyze before implementing to prevent hallucination.

```
@instructions.md @relevant-files

1. CONTEXT FIRST — NO GUESSWORK
   List files in target directory
   Identify existing patterns (error handling, validation, tests)
   Review relevant documentation
   
2. CLARIFY REQUIREMENTS
   Ask only necessary questions
   Confirm assumptions
   
3. DETECT PATTERNS
   Match style, structure, logic of existing code
   Follow project conventions
   Use Context7 for library-specific best practices (auto-invoked)
   
4. IDENTIFY DEPENDENCIES
   List environment variables needed
   Check config files
   Find system dependencies
   Review package.json for available libraries

5. THEN IMPLEMENT
   Generate code following discovered patterns
   Include tests
   Add documentation

Example task: Add wishlist feature

@src/features/cart/ @src/features/products/

1. First, analyze cart implementation:
   - What state management? (Zustand/Redux/Context)
   - What API patterns? (REST endpoints, request/response format)
   - What data structure? (how items stored)
   - What tests exist? (unit/integration coverage)

2. Ask clarifying questions:
   - Should wishlist persist across sessions?
   - Max items per wishlist?
   - Can items be in both cart and wishlist?
   - Should moving to cart remove from wishlist?

3. Follow same patterns:
   - Same state management as cart
   - Same API structure as cart
   - Same validation patterns
   - Same test coverage

4. Check dependencies:
   - Does API support wishlist endpoints?
   - Does DB schema need updates?
   - Do we need new environment variables?

5. Generate wishlist feature:
   - src/features/wishlist/ (following cart structure)
   - API routes
   - Tests
   - Documentation

DO NOT write code until step 5.
```

**Why this works:**
- Prevents hallucination (agent confirms patterns first)
- Ensures consistency with existing code
- Catches missing dependencies early
- Produces code that actually works

### 8.3 Focused Context Pattern

**Problem:** Open tabs pollute context window.

**Solution: Context Window Hygiene**

**Before starting agent task:**

1. **Close irrelevant tabs**
   - Agent pulls open files into context
   - Extra tabs = wasted context, slower responses
   - Close everything except files you'll reference

2. **Use @mentions strategically**
   ```
   # BAD (too broad, pulls entire codebase)
   @codebase Refactor authentication

   # GOOD (focused on specific files)
   @src/auth/login.ts @src/auth/jwt.ts 
   Refactor JWT token generation to use RS256 instead of HS256
   
   # ALSO GOOD (folder when appropriate)
   @src/components/ui/
   Ensure all UI components follow accessibility guidelines
   ```

3. **Start narrow, expand if needed**
   ```
   # First attempt: narrow scope
   @src/auth/jwt.ts
   Implement JWT refresh token rotation
   
   # If agent needs more context:
   @src/auth/jwt.ts @src/auth/middleware.ts @src/auth/types.ts
   Implement JWT refresh token rotation
   Include middleware changes and type updates
   ```

**Measured impact:** Clean context = 30% faster responses

### 8.4 Conversation Reset Pattern

**When to reset conversation:**

**Signals:**
- Same error appears 3+ times
- Agent proposes same failed solution repeatedly
- Context window cluttered with failed attempts
- Agent "forgets" earlier decisions
- Conversation thread has >20 messages

**Action:**
1. Stop current conversation
2. Extract learnings from failed attempt
3. Start new chat with updated requirements

**Example:**
```
# New conversation prompt

Previous attempt to implement WebSocket client failed because:
- Assumed synchronous API, but it's async (WebSockets)
- Didn't handle connection timeouts
- Used wrong event payload structure
- Missed reconnection logic

Implement WebSocket client for real-time notifications:

Context:
- Server: wss://api.example.com/notifications
- Protocol: Socket.IO v4 (not raw WebSocket)
- Authentication: JWT in connection query params
- Events to handle: connect, disconnect, error, message, reconnect

Requirements:
- Handle connection events properly
- Reconnect with exponential backoff (max 5 attempts, starting at 1s)
- Parse message format: { type, data, timestamp }
- Store notifications in Zustand store
- Display connection status to user

Learnings from failed attempt:
- Use Socket.IO client library (npm install socket.io-client)
- Pass auth token: io(url, { query: { token } })
- Handle reconnection_attempt, reconnection_failed events
- Test with mock server first

@src/lib/socket.ts (if exists, review existing patterns)
@src/stores/notifications.ts (Zustand store)

Generate implementation with proper error handling and tests.
```

**Result:** Fresh start with corrected assumptions = faster resolution.

### 8.5 @ Mention Strategy Reference

**File-level:**
```
@src/components/Button.tsx "Add loading state"
```
- Loads full file into context
- Use for: Single-file edits, explanations

**Folder-level:**
```
@src/components/ "Find all uses of useState"
```
- Scans all files in folder
- Use for: Cross-component refactors, pattern analysis

**Codebase-level:**
```
@codebase "Where is authentication handled?"
```
- Semantic search across entire repo
- Use for: Discovering patterns, finding usage

**Docs:**
```
@react-docs.org "How do I implement suspense?"
```
- Searches indexed documentation
- Use for: Learning new APIs, best practices

**Web (NEW):**
```
@web "Latest Next.js 15 features"
```
- Real-time web search
- Use for: Recent updates, current best practices

**Recommended (NEW in 2.0):**
```
@Recommended
```
- Auto-fetches relevant context in Agent mode
- Cursor determines what's relevant
- Use for: Quick starts without manual context

**Trade-offs:**
- More @ mentions = larger context = slower responses
- Start narrow (@file), expand if needed (@folder, @codebase)

**Context Prioritization (what agent sees first):**
1. **Current file + cursor position** (highest relevance)
2. **@mentions** (explicitly requested)
3. **Open files in tabs** (active working set)
4. **Recently viewed files** (temporal relevance)
5. **Linter errors** (fixing mode)
6. **Codebase search results** (when @codebase used)
7. **MCP data** (when MCP tools used)

---

## 9. Advanced Prompting Strategies (EXPANDED)

### 9.1 Chain-of-Thought for Complex Features

**Pattern:** Force reasoning before implementation.

```
@instructions.md

Before implementing user authentication system, analyze:

1. **Requirements Analysis**
   What user flows are needed?
   - Signup (email + password)
   - Login (with remember me)
   - Logout (invalidate session)
   - Password reset (email flow)
   - Email verification
   - OAuth (Google, GitHub)
   
   What data needs to be stored?
   - Users table: email, password_hash, email_verified, created_at
   - Sessions table: user_id, token, expires_at
   - Reset tokens: user_id, token, expires_at (1 hour)
   
   What security requirements?
   - Password: min 8 chars, mix of letters/numbers/symbols
   - Rate limiting: 5 login attempts per 15 minutes
   - Session management: JWT + refresh tokens
   - CSRF protection: SameSite cookies

2. **Edge Cases**
   - Duplicate email signup → 409 Conflict
   - Login with unverified email → 403 Forbidden (allow with warning?)
   - Expired password reset tokens → 400 Bad Request, send new email
   - Concurrent login sessions → allow or invalidate old sessions?
   - Account enumeration attacks → generic error messages
   - Brute force → rate limiting + account lockout after 10 failed attempts

3. **Dependencies**
   - Email service: SendGrid (already configured? check env vars)
   - Token signing: jsonwebtoken (check package.json)
   - Password hashing: bcrypt (check package.json)
   - Database: PostgreSQL + Prisma (check existing schema)
   - Rate limiting: Redis (is it available? check docker-compose)

4. **Architecture**
   - Routes: src/routes/auth/ (new directory)
   - Middleware: src/middleware/auth.ts (validateJWT, checkEmailVerified)
   - Services: src/services/auth.ts (business logic)
   - Models: extend existing Prisma schema
   - Tests: tests/auth/ (unit + integration)
   - Email templates: src/templates/email/

5. **Error Handling**
   - 400: Validation errors (weak password, invalid email)
   - 401: Unauthorized (invalid credentials)
   - 403: Forbidden (email not verified)
   - 409: Conflict (email already exists)
   - 429: Too many requests (rate limit exceeded)
   - 500: Internal error (DB connection, email service down)

After analysis, propose implementation plan with:
- File structure
- Database migrations needed
- Environment variables required
- Estimated complexity (S/M/L)
- Risks and mitigations

Wait for approval before generating code.
```

**Result:** Agent thinks through problem completely, you validate approach before wasting time on code.

### 9.2 Multi-Tool Orchestration

**Pattern:** Complex workflows spanning multiple systems

```
@instructions.md @notepads/architecture

Implement feature flag system:

ORCHESTRATION PLAN:
1. Database schema (Prisma migration)
2. Feature flag service (TypeScript)
3. Admin UI (React)
4. Middleware (check flags in requests)
5. CI/CD sync (flags from config file)
6. Monitoring (log flag checks)

DETAILED STEPS:

Step 1: Database
- Create feature_flags table (name, enabled, description, rules)
- Create feature_flag_overrides table (flag_name, user_id, enabled)
- Generate Prisma migration
- Run migration in dev DB
- Verify schema with: npx prisma studio
- Commit migration

Step 2: Service layer
- Create src/services/featureFlags.ts
- Methods: isEnabled(flagName, userId?), getAllFlags(), updateFlag()
- Cache flags in Redis (TTL: 60s)
- Support environment-based overrides (env vars)
- Add tests
- Commit service

Step 3: Middleware
- Create src/middleware/featureFlag.ts
- Decorator: @requireFeature('new-checkout')
- Block request if flag disabled (return 404)
- Log flag checks to analytics
- Add tests
- Commit middleware

Step 4: Admin UI
- Create pages/admin/feature-flags.tsx
- Table view (all flags with toggle switches)
- User overrides (search user, set flag)
- Audit log (who changed what, when)
- Use Next.js + shadcn/ui
- Add auth requirement (admin role only)
- Commit UI

Step 5: CI/CD integration
- Create config/feature-flags.yaml (source of truth)
- Create script: scripts/sync-flags.ts
- On deploy: sync YAML → database
- Add to CI pipeline (.github/workflows/deploy.yml)
- Test sync in staging
- Commit CI changes

Step 6: Monitoring
- Add Datadog logging for flag checks
- Create dashboard (flag usage, toggle frequency)
- Set up alerts (flag check errors)
- Commit monitoring

EXECUTION:
- Run each step sequentially
- After each step: run tests, commit
- Use YOLO mode for: migrations, tests, file operations
- Stop if any step fails, await manual intervention
- Final verification: deploy to staging, test all flags

Begin with Step 1. Wait for approval after each step.
```

**Why this works:**
- Clear plan with validation points
- Automated execution where safe
- Human oversight at critical points
- Easy to rollback (git commits per step)

### 9.3 The "Over-Specification" Anti-Pattern

**BAD (Micro-managing):**
```
Create button component.
Use const Button = () => {}
Import React from 'react'
Use background color #3B82F6
Add className prop
Use Tailwind classes bg-blue-500 hover:bg-blue-600
Make it rounded with rounded-lg
Add padding px-4 py-2
Font size text-sm
Font weight font-medium
Export default Button
Add TypeScript types: interface ButtonProps {}
Props: children (ReactNode), onClick (function), disabled (boolean)
```

**Result:** Agent has no room for intelligent decisions, produces brittle code.

**GOOD (Intent + Constraints):**
```
@instructions.md

Create reusable Button component:
- React + TypeScript
- Tailwind styling (primary blue theme)
- Variants: primary, secondary, outline, ghost
- Sizes: sm, md, lg
- Props: variant, size, onClick, disabled, loading
- Accessible (ARIA label, keyboard navigation, focus indicators)
- Loading state (spinner icon + disabled)

Follow existing component patterns:
@components/Input.tsx (use as reference for structure)

Generate:
- components/Button.tsx
- components/Button.stories.tsx (Storybook)
- components/Button.test.tsx (Vitest)
```

**Result:** Agent produces well-designed component following best practices.

### 9.4 Model Ensemble Strategy

**Pattern:** Use multiple models for different strengths

```
# Complex refactoring task

Phase 1: Planning (Claude Sonnet)
@codebase

Analyze this codebase and create a detailed refactoring plan:
- Identify code smells
- Prioritize by impact
- Break into phases
- Estimate effort
- Identify risks

[Switch to Cursor Model for execution]

Phase 2: Execution (Cursor Model - faster)
@refactoring-plan.md

Execute Phase 1 of the refactoring plan.
Focus on speed while maintaining quality.

[Switch to Gemini for final review]

Phase 3: Review (Gemini - large context)
@codebase @all-changes

Review all changes made in this refactoring:
- Check for regressions
- Verify patterns followed consistently
- Identify any missed optimizations
- Review test coverage
```

**Model Selection Guide:**
- **Claude Sonnet**: Deep analysis, complex instructions, legacy code
- **Cursor Model**: Fast execution, multi-file edits
- **GPT-4o**: Boilerplate, speed-critical tasks
- **Gemini**: Large context review, monorepo analysis

---

## Best Practices Summary

### Context Management Checklist
- [ ] Use `@instructions.md` in all significant prompts
- [ ] Close irrelevant tabs before starting agent tasks
- [ ] Start with narrow @ mentions, expand only if needed
- [ ] Reset conversation after 3 failed attempts or >20 messages
- [ ] Analyze before implementing (context-first approach)
- [ ] Clean context = 30% faster responses

### Prompting Strategies Checklist
- [ ] Use chain-of-thought for complex features
- [ ] Break multi-system workflows into orchestrated steps
- [ ] Avoid over-specification (intent + constraints, not micro-managing)
- [ ] Use model ensemble for different phases (plan/execute/review)
- [ ] Force reasoning before implementation
- [ ] Include wait-for-approval checkpoints in complex workflows

---

## Next Steps

Continue to [Part 5: Team Collaboration & Domain Patterns →](05-team-domain-patterns.md)

Or return to the [Index](../) for the complete guide navigation.

---

**Part 4 of 6** | [← Part 3](03-devops-backend-patterns) | [Index](../) | [Part 5 →](05-team-domain-patterns)

