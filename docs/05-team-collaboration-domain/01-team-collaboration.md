---
title: "Team Collaboration"
nav_order: 1
parent: "Part 5: Team Collaboration & Domain Patterns"
permalink: /docs/05-team-collaboration-domain/team-collaboration/
---

# Section 1: Team Collaboration

**Target Audience:** Team leads, engineering managers, senior developers  
**Time to Complete:** 45-60 minutes  
**Prerequisites:** [Part 1: Fundamentals](../../01-fundamentals-core-concepts/)

---

## 📋 Overview

Learn how to scale Cursor IDE across your team with shared configurations, collaborative workflows, and consistent development practices.

**What you'll learn:**
- Shared rules repository setup
- Team notepads for collective knowledge
- Pair programming patterns with AI agents
- PR review workflows
- Onboarding automation

---

## 1. Shared Rules Repository

### The Problem
Teams without shared Cursor configurations experience:
- Inconsistent AI behavior across developers
- Repeated setup for each team member
- Divergent coding standards
- Knowledge silos

### Solution: Git-Committed Configuration

**Repository Structure:**
```
your-repo/
├── .cursor/
│   └── rules/
│       ├── 00-project-context.mdrule
│       ├── 01-architecture.mdrule
│       ├── 02-code-standards.mdrule
│       ├── 03-testing.mdrule
│       └── 04-security.mdrule
├── .cursorignore
├── instructions.md          # Single source of truth
└── AGENTS.md               # Alternative to .cursor/rules/
```

**Best Practice:** Commit all configuration files to git so new team members inherit the setup automatically.

---

### Example: Team Project Rules

**File: `.cursor/rules/00-project-context.mdrule`**
```markdown
# Project Context

## Application Overview
- **Name:** PaymentPlatform
- **Stack:** React 19, TypeScript 5.8, Node.js 22
- **Architecture:** Event-driven microservices
- **Team size:** 12 engineers across 3 squads

## Key Directories
- `/apps/api/` - Backend microservices
- `/apps/web/` - Frontend React app
- `/libs/shared/` - Shared utilities and types
- `/infra/` - Kubernetes manifests and Terraform

## Development Principles
1. Test-first development (TDD)
2. Type safety everywhere (strict TypeScript)
3. Minimal external dependencies
4. Comprehensive error handling
```

**File: `.cursor/rules/02-code-standards.mdrule`**
```markdown
# Code Standards

## Naming Conventions
- **Components:** PascalCase (e.g., `PaymentButton.tsx`)
- **Functions:** camelCase (e.g., `calculateTotal`)
- **Constants:** UPPER_SNAKE_CASE (e.g., `MAX_RETRY_ATTEMPTS`)
- **Types/Interfaces:** PascalCase with `I` prefix (e.g., `IPaymentRequest`)

## File Organization
- One component per file
- Co-locate tests with implementation: `Button.tsx` + `Button.test.tsx`
- Barrel exports in `index.ts` for public API

## Error Handling Pattern
Always use structured errors:

```typescript
class PaymentError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly context?: Record<string, unknown>
  ) {
    super(message);
    this.name = 'PaymentError';
  }
}
```

## Code Review Checklist
- [ ] Tests included (unit + integration)
- [ ] Error handling implemented
- [ ] TypeScript strict mode satisfied
- [ ] Documentation updated
- [ ] No console.logs (use logger)
```

---

### Example: `instructions.md` (Single Source of Truth)

```markdown
# PaymentPlatform - Development Guide

## Quick Start
```bash
npm install
npm run dev:api    # Start backend on :3000
npm run dev:web    # Start frontend on :5173
```

## AI Assistant Context

When working with this codebase, follow these guidelines:

### Architecture Patterns
- **Event Sourcing:** All payment events stored in EventStore
- **CQRS:** Separate read/write models for payment data
- **API Gateway:** Kong routes requests to microservices
- **Authentication:** JWT tokens with 15-minute expiry

### Common Tasks

#### Adding a New Payment Method
1. Create event handler in `apps/api/src/events/`
2. Update `PaymentMethod` enum in `libs/shared/types/`
3. Add UI component in `apps/web/src/components/payments/`
4. Write integration test in `apps/api/tests/integration/`

#### Database Migrations
```bash
npm run migrate:create <name>
npm run migrate:up
```

### Testing Strategy
- **Unit tests:** Pure functions, utilities
- **Integration tests:** API endpoints, database operations
- **E2E tests:** Critical user flows (checkout, refunds)

Target coverage: 80% (enforced by CI)

### Common Pitfalls
1. **Don't:** Mutate event objects after creation
2. **Don't:** Make synchronous calls between microservices
3. **Do:** Use message queue for inter-service communication
4. **Do:** Implement circuit breakers for external APIs

### Environment Variables
```bash
DATABASE_URL=postgresql://...
REDIS_URL=redis://...
STRIPE_SECRET_KEY=sk_test_...
JWT_SECRET=<from-1password>
```

## Code Generation Preferences
- Use functional components (not class components)
- Prefer `async/await` over Promise chains
- Use Zod for runtime validation
- Implement retry logic with exponential backoff

## Team Conventions
- PR title format: `[JIRA-123] Brief description`
- Commit messages follow Conventional Commits
- Feature flags for all new features (LaunchDarkly)
```

---

## 2. Team Notepads

**Team Notepads** provide shared context that persists across sessions and team members.

### Use Cases

#### 1. Architecture Decisions Records (ADR)

**File: `team-notes/ADR-001-event-sourcing.md`**
```markdown
# ADR-001: Adopt Event Sourcing for Payment Domain

**Status:** Accepted  
**Date:** 2026-01-10  
**Authors:** @tech-lead, @senior-dev-1

## Context
Payment operations need full audit trail and ability to reconstruct state at any point in time.

## Decision
Implement event sourcing pattern:
- All state changes captured as immutable events
- EventStore as source of truth
- Read models derived from events

## Consequences
**Positive:**
- Complete audit trail
- Time-travel debugging
- Eventual consistency support

**Negative:**
- Increased complexity
- Learning curve for team
- Eventual consistency challenges

## Implementation Notes
- Use EventStoreDB for persistence
- Snapshot every 100 events for performance
- Use Zod schemas for event validation

## Related Files
- `libs/shared/events/payment-events.ts`
- `apps/api/src/event-store/`
```

**Cursor Prompt:**
```
@team-notes/ADR-001-event-sourcing.md
Implement the RefundInitiated event following our event sourcing pattern.
Include event schema, handler, and unit tests.
```

#### 2. Incident Postmortems

**File: `team-notes/postmortem-2026-01-09.md`**
```markdown
# Postmortem: Payment Processing Outage (2026-01-09)

**Duration:** 14:23 - 15:47 UTC (84 minutes)  
**Impact:** 2,453 failed transactions ($487k total value)  
**Root Cause:** Redis connection pool exhaustion

## Timeline
- **14:23** - Payment success rate drops from 99.8% to 12%
- **14:26** - Alerts fire: "High error rate on /api/payments"
- **14:30** - On-call engineer investigates
- **14:45** - Root cause identified: Redis maxclients=100 (default)
- **15:15** - Redis config updated to maxclients=10000
- **15:30** - Service restarted, payments recovering
- **15:47** - Full recovery confirmed

## Root Cause
Redis connection pool configured with default maxclients=100. Under high load, connection pool exhausted.

## Resolution
1. Increased Redis maxclients to 10,000
2. Added connection pool monitoring
3. Implemented circuit breaker for Redis failures

## Action Items
- [ ] @devops: Add Redis connection pool alerts (P0)
- [ ] @backend: Implement graceful degradation when Redis unavailable (P1)
- [ ] @backend: Add connection pool size to Grafana dashboard (P2)
- [ ] @team-lead: Document Redis connection best practices (P2)

## Lessons Learned
1. Always override default connection limits for production
2. Monitor connection pool usage, not just request rates
3. Implement circuit breakers for all external dependencies

## Related Changes
- PR #456: Redis connection pool monitoring
- PR #457: Circuit breaker implementation
```

**Cursor Usage:**
```
@team-notes/postmortem-2026-01-09.md
Review our payment service and identify other services that might have
similar connection pool issues. Create tickets for each.
```

#### 3. Onboarding Checklists

**File: `team-notes/onboarding-checklist.md`**
```markdown
# New Developer Onboarding

## Week 1: Setup & Foundation

### Day 1: Environment Setup
- [ ] Install Cursor IDE from cursor.com
- [ ] Clone repository: `git clone git@github.com:company/payment-platform`
- [ ] Run `npm install` (takes ~5 minutes)
- [ ] Copy `.env.example` to `.env.local`
- [ ] Get secrets from 1Password (team vault: "Payment Platform Dev")
- [ ] Start services: `npm run dev:all`
- [ ] Verify: http://localhost:5173 shows login page

### Day 2: First Commit
- [ ] Read `instructions.md` (single source of truth)
- [ ] Read `.cursor/rules/00-project-context.mdrule`
- [ ] Review `team-notes/architecture-overview.md`
- [ ] Pick "good-first-issue" from Jira
- [ ] Use Cursor to implement fix
- [ ] Submit PR following team conventions

### Day 3: Testing & CI/CD
- [ ] Run full test suite: `npm test`
- [ ] Understand test patterns in `apps/api/tests/`
- [ ] Set up pre-commit hooks: `npm run setup-hooks`
- [ ] Review CI/CD pipeline in `.github/workflows/`

### Day 4-5: Domain Knowledge
- [ ] Pair with senior dev on payment flow
- [ ] Read Event Sourcing ADR: `team-notes/ADR-001-event-sourcing.md`
- [ ] Review recent postmortem: `team-notes/postmortem-*.md`
- [ ] Shadow on-call rotation

## Week 2: Feature Development
- [ ] Pick medium complexity ticket
- [ ] Implement using TDD pattern
- [ ] Use Cursor Composer for multi-file changes
- [ ] Request code review from team

## Resources
- **Slack channels:** #payment-platform-dev, #payment-alerts
- **Docs:** Confluence space "Payment Platform"
- **Monitoring:** Grafana dashboard "Payment Services"
- **Errors:** Sentry project "payment-api"
```

---

## 3. Pair Programming with AI Agent

### Pattern 1: Driver-Navigator (Human + AI)

**Scenario:** Implementing a new feature

**Workflow:**
1. **Human as Navigator** (strategic thinking)
   ```
   Cmd+I (Composer mode)
   "We need to implement a refund flow. Let's start by:
   1. Defining the RefundRequested event schema
   2. Creating the event handler
   3. Updating the read model
   4. Adding API endpoint
   
   @libs/shared/events/payment-events.ts
   @apps/api/src/handlers/
   
   Show me the event schema first, then we'll iterate."
   ```

2. **AI as Driver** (implementation)
   - Agent proposes event schema
   - Human reviews and provides feedback
   - Iterate until correct

3. **Switch Roles**
   - AI navigates: "Next we need to handle duplicate refund requests"
   - Human implements the idempotency check
   - AI reviews for edge cases

### Pattern 2: Parallel Exploration

Use **Parallel Agents** to explore multiple approaches simultaneously.

**Scenario:** Deciding between two architecture approaches

```
Cmd+I (open 2 parallel agents)

Agent 1 prompt:
"Implement refund flow using synchronous REST API call to Stripe.
@apps/api/src/services/stripe/
Show me the implementation with error handling."

Agent 2 prompt:
"Implement refund flow using asynchronous job queue pattern.
@apps/api/src/jobs/
Show me the implementation with retry logic."
```

**Review both solutions:**
- Compare latency implications
- Evaluate error handling
- Assess complexity
- Choose best approach for your use case

---

## 4. PR Review Workflows

### Using Cursor for Code Reviews

#### Reviewer Workflow

**Step 1: Get PR Context**
```bash
git fetch origin
git checkout pr-branch
```

**Step 2: Review with Cursor Chat**
```
Cmd+L (Chat mode)

"Review this PR for:
1. Test coverage
2. Error handling
3. Performance implications
4. Security vulnerabilities

@src/services/payment/
@src/services/payment/payment.test.ts

Provide specific line numbers and suggestions."
```

**Step 3: Deep Dive on Suspicious Code**
```
Select suspicious function
Cmd+K
"Analyze this function for potential race conditions and propose fixes."
```

**Step 4: Suggest Improvements**
```
Cmd+K on selected code
"Refactor this to follow our error handling pattern in
@.cursor/rules/02-code-standards.mdrule"
```

#### Author Workflow (Self-Review)

**Before submitting PR:**

```
Cmd+I
"Review my changes in this branch:
1. Do they follow our coding standards in @.cursor/rules/
2. Is test coverage adequate?
3. Are there any security concerns?
4. Does error handling follow our patterns?

@git-diff

Provide a checklist of issues to fix before PR submission."
```

---

### PR Review Checklist (Team Standard)

**File: `.cursor/rules/03-pr-review-checklist.mdrule`**
```markdown
# PR Review Checklist

## Automated Checks (CI)
- [ ] All tests pass
- [ ] Linter passes (ESLint)
- [ ] Type checker passes (TypeScript strict mode)
- [ ] Code coverage ≥ 80%
- [ ] No security vulnerabilities (npm audit)

## Code Quality
- [ ] Follows naming conventions
- [ ] Functions are single-purpose (max 50 lines)
- [ ] No code duplication (DRY principle)
- [ ] Appropriate abstraction level

## Error Handling
- [ ] All async operations have error handling
- [ ] Errors use structured error types
- [ ] User-facing errors have clear messages
- [ ] Errors are logged with context

## Testing
- [ ] Unit tests for business logic
- [ ] Integration tests for API endpoints
- [ ] Edge cases covered
- [ ] Negative test cases included

## Documentation
- [ ] JSDoc comments on public APIs
- [ ] README updated if public API changed
- [ ] CHANGELOG.md entry added

## Security
- [ ] No secrets in code
- [ ] Input validation on all user inputs
- [ ] SQL injection prevention (use parameterized queries)
- [ ] XSS prevention (sanitize outputs)

## Performance
- [ ] No N+1 queries
- [ ] Database indexes added if needed
- [ ] Large operations paginated
- [ ] Caching considered where appropriate
```

**Cursor Prompt for PR Author:**
```
Cmd+I
"Review my PR against our checklist:
@.cursor/rules/03-pr-review-checklist.mdrule
@git-diff

Create a table showing which items pass/fail and specific actions needed."
```

---

## 5. Onboarding Automation

### Day 1: Instant Productivity

**Step 1: Clone Repository**
```bash
git clone git@github.com:company/payment-platform
cd payment-platform
```

**Step 2: Open in Cursor**
```bash
cursor .
```

**Step 3: AI-Powered Setup**
```
Cmd+I
"I'm a new developer. Help me set up this project:
1. Check @instructions.md for setup steps
2. Identify missing environment variables
3. Guide me through first-time setup

@instructions.md
@.env.example"
```

**AI Response:**
```
Based on your project, here's your setup:

1. Install dependencies:
   npm install

2. Copy environment template:
   cp .env.example .env.local

3. You need these secrets (get from 1Password):
   - DATABASE_URL (PostgreSQL connection)
   - REDIS_URL (Redis connection)
   - STRIPE_SECRET_KEY (from Stripe dashboard)
   - JWT_SECRET (generate with: openssl rand -hex 32)

4. Start services:
   npm run dev:api
   npm run dev:web

5. Verify setup:
   curl http://localhost:3000/health
   
Would you like me to help with any of these steps?
```

### Day 1 First Task

**Prompt:**
```
Cmd+I
"I'm new to this codebase. Give me a tour:
1. Show me the main entry points
2. Explain the architecture
3. Identify a good first issue I could tackle

@instructions.md
@codebase"
```

**AI provides:**
- Architecture overview
- Key files to understand
- Suggested "good first issue" based on codebase analysis

---

## 6. Team Commands (Reusable Prompts)

Create shared commands for common tasks.

### Setup: Create Commands

**File: `.cursor/commands/refactor-to-typescript.md`**
```markdown
# Refactor to TypeScript

Convert this JavaScript code to TypeScript following our standards:

1. Add strict types (no `any`)
2. Use interfaces from `@libs/shared/types/`
3. Add JSDoc comments
4. Follow naming conventions in @.cursor/rules/02-code-standards.mdrule

Selected code: {SELECTION}
```

**File: `.cursor/commands/add-tests.md`**
```markdown
# Add Tests

Generate comprehensive tests for the selected code:

1. Unit tests for all public functions
2. Edge case coverage
3. Error handling tests
4. Mock external dependencies

Follow our testing patterns in @apps/api/tests/

Selected code: {SELECTION}
```

**Usage:**
```
Select code
Cmd+K
Type: /refactor-to-typescript
or
Type: /add-tests
```

---

## 7. Team Metrics & Continuous Improvement

### Track Velocity Improvements

**Metrics to measure:**
- PRs merged per week
- Time from PR open to merge
- Bug escape rate
- Code review turnaround time

**Example: Team Dashboard**
```markdown
# Team Velocity Metrics (Week of 2026-01-06)

## Before Cursor (baseline: Dec 2025)
- PRs merged: 23/week
- Avg PR size: 487 lines
- Time to merge: 3.2 days
- Bug escape rate: 8.4%

## After Cursor (Jan 2026, Week 1)
- PRs merged: 32/week (+39%)
- Avg PR size: 341 lines (-30%, better focused PRs)
- Time to merge: 2.1 days (-34%)
- Bug escape rate: 6.1% (-27%)

## Team Feedback
- "Cursor catches bugs before CI" - @dev-1
- "Refactoring is 3x faster" - @dev-2
- "Onboarding new dev took 2 days instead of 2 weeks" - @team-lead
```

---

## 8. Best Practices Summary

### Do's ✅

1. **Commit all Cursor configs to git**
   - `.cursor/rules/`
   - `instructions.md`
   - `.cursorignore`

2. **Use `instructions.md` as single source of truth**
   - Architecture overview
   - Common tasks
   - Environment setup

3. **Create team commands for repeated tasks**
   - Refactoring patterns
   - Test generation
   - Code review templates

4. **Document decisions in team notepads**
   - ADRs (Architecture Decision Records)
   - Postmortems
   - Onboarding checklists

5. **Review AI suggestions as a team**
   - Share interesting prompts in Slack
   - Evolve `.cursor/rules/` based on patterns

### Don'ts ❌

1. **Don't let configs diverge**
   - Regularly sync `.cursor/rules/` across team
   
2. **Don't commit secrets**
   - Use `.env.example` for templates
   - Store secrets in 1Password/Vault

3. **Don't blindly accept AI suggestions**
   - Always review generated code
   - Run tests before committing

4. **Don't skip code reviews**
   - AI assists but doesn't replace human review
   - Use AI to enhance review quality

---

## 9. Real-World Example: Team Rollout

### Company: PaymentCorp (12 engineers)

**Week 1: Setup**
- Tech lead creates `instructions.md`
- Creates `.cursor/rules/` with 5 rule files
- Commits to git
- Announces in team Slack

**Week 2: Training**
- 1-hour lunch & learn session
- Each dev installs Cursor
- Clones repo (configs auto-inherited)
- Completes "first task" exercise

**Week 3: Adoption**
- 4 devs actively using Cursor
- Shared feedback in #cursor-tips channel
- Updated `.cursor/rules/` based on learnings

**Week 4: Full Team**
- All 12 devs using Cursor
- Velocity metrics show +25% PRs merged
- Time-to-PR reduced by 30%

**Month 2: Optimization**
- Created 15 team commands
- 10 ADRs documented in team notepads
- New dev onboarded in 2 days (previously 2 weeks)

**ROI:**
- Cost: $240/month (12 devs × $20)
- Value: 30% velocity increase = ~4 "free" engineer-days/month
- Break-even: Week 3
- 12-month ROI: 450%

---

## 10. Next Steps

**Immediate actions:**
1. Create `instructions.md` for your team's main repository
2. Set up `.cursor/rules/` with 3-5 rule files
3. Commit configs to git
4. Schedule team training session

**Week 1 goals:**
- 50% of team actively using Cursor
- Collect feedback and iterate on rules

**Month 1 goals:**
- 100% team adoption
- Measure velocity improvements
- Document team patterns in notepads

---

**Next Section:** [Domain-Specific Patterns →](02-domain-patterns.md)

**Related Sections:**
- [Part 1: Environment & Project Setup](../../01-fundamentals-core-concepts/02-environment-project-setup.md)
- [Part 4: Context Management](../../04-context-management-prompting/01-context-management.md)
