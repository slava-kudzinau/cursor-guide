---
title: "Section 1: Advanced Recipes"
parent: "Part 2: Advanced Features & Visual Development"
nav_order: 1
---

# Part 2, Section 1: Advanced Recipes

**Part of**: [Cursor IDE: Complete Technical Guide](../../README.md)  
**Estimated reading time**: 90 minutes  
**Prerequisites**: [Part 1](../01-fundamentals-core-concepts/)

---

## 📋 Overview

This section covers complex multi-phase workflows for advanced development tasks.

**What you'll learn:**
- API migration strategies (REST → GraphQL)
- Database schema migrations with Prisma
- Large-scale refactoring patterns
- Major dependency upgrades
- Micro-frontend integration

---

## 🔄 API Migration (REST → GraphQL)

### Multi-Stage Prompt

```
@instructions.md @api/routes/ @models/

Phase 1: Schema Design
Analyze existing REST endpoints. Design GraphQL schema:
- Types (User, Post, Comment)
- Queries (getUser, listPosts)
- Mutations (createPost, updateUser)
- Relationships (User.posts, Post.comments)

Output schema.graphql
Wait for review.

Phase 2: Resolvers
Generate resolvers for each type. Map to existing service layer:
- @services/userService.ts → User resolvers
- @services/postService.ts → Post resolvers

Output resolvers/index.ts
Wait for review.

Phase 3: Integration
Update @server.ts:
- Add apollo-server-express
- Mount /graphql endpoint
- Keep REST endpoints (backward compat)

Wait for review.

Phase 4: Tests
Migrate @tests/api/ REST tests to GraphQL queries.
Use supertest + graphql-request.

Run all tests.

Execute phase-by-phase. After each, run tests.
```

---

## 🗄️ Database Schema Migration

### Pattern: Generate + Review

```
@instructions.md @database/schema.prisma @migrations/

Task: Add soft deletes to User, Post, Comment tables

Steps:
1. Update Prisma schema:
   - Add deletedAt DateTime? to models
   - Add @@index on deletedAt

2. Generate migration:
   - Run `npx prisma migrate dev --name add-soft-deletes`
   - Show SQL diff

3. Update service layer:
   - Modify queries to filter WHERE deletedAt IS NULL
   - Add softDelete() methods
   - Update existing delete calls

4. Update tests:
   - Verify soft-deleted records not returned
   - Test restore functionality
   - Test cascade soft deletes

5. Generate SQL rollback script

Execute step-by-step. Show diffs before applying.
DO NOT run migrations without approval.
```

**Why manual review:** Schema migrations = data risk. Always verify SQL before running.

---

## ⚛️ Large-Scale Refactor (Class → Hooks)

### Problem

50 React class components → functional + hooks

### Strategy: Batch + Template

```
@instructions.md @components/ @hooks/

Task: Convert class components to functional + hooks

Template:
- State → useState
- Lifecycle → useEffect
- this.props → destructured props
- Class methods → useCallback
- Refs → useRef

Process:
1. Identify all class components (find `extends React.Component`)
2. Generate conversion plan (list files, complexity estimate)
3. Convert one component per iteration (start with leaf components)
4. After each conversion:
   - Update imports in parent components
   - Run Storybook tests
   - Run Jest snapshots
   - Update snapshots if needed
5. Commit after each successful conversion

Start with leaf components (no children).
Work up the tree to avoid breaking child components.

Generate automation script to track progress.
```

---

## 📦 Dependency Upgrade (Major Version)

### Problem: React 17 → 18

Breaking changes: concurrent rendering, new APIs

### Prompt

```
@instructions.md @package.json @components/ @docs/react-18-migration

Task: Upgrade React 17 → 18

Pre-flight:
1. Review breaking changes (@docs)
2. Identify unsafe patterns:
   - findDOMNode usage
   - Legacy context API
   - Deprecated lifecycle methods
3. Estimate effort per component

Migration:
1. Update package.json (react, react-dom, @types/react)
2. Update root render (ReactDOM.render → createRoot)
3. Fix unsafe patterns (one component at a time)
4. Test after each fix
5. Update documentation

Rollback plan: Git revert + npm install
```

---

## 🧩 Micro-Frontend Integration

### Pattern: Module Federation

```
@instructions.md @webpack.config.js @remotes/

Task: Integrate micro-frontend using Module Federation

Setup:
1. Configure host app:
   - Add ModuleFederationPlugin
   - Define remotes
   - Share dependencies (react, react-dom)

2. Configure remote app:
   - Expose components
   - Define shared deps
   - Build as standalone

3. Integration:
   - Lazy load remote components
   - Error boundaries
   - Loading states

4. Testing:
   - Local development (both apps running)
   - Production build
   - Fallback behavior

Follow Module Federation best practices.
```

---

## 🎯 Key Takeaways

### Migration Workflow
1. **Analysis** - Understand current state
2. **Planning** - Break into phases
3. **Execute** - One phase at a time
4. **Validate** - Test after each phase
5. **Rollback** - Have a plan

### Best Practices
- Always analyze before executing
- Break large tasks into 3-5 phases
- Validate after each step
- Commit after successful steps
- Have rollback plans ready

---

## 📖 Next Steps

Continue to [Section 2: Visual Development Patterns →](./02-visual-development-patterns.md)

---

**Part 2, Section 1** | [Back to Part 2 Index](./README.md) | [Next: Section 2 →](./02-visual-development-patterns.md)
