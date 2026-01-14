---
title: "Appendix B: /project-init System Prompt"
parent: "Appendices"
nav_order: 2
---

# /project-init System Prompt

The `/project-init` system prompt provides a systematic approach to setting up Rules, Skills, and Instructions in a new project. It automates the extraction and organization of project constraints from an architecture specification, ensuring consistent initialization of Cursor IDE projects.

This prompt is designed to be used as a Cursor command that:
- Extracts invariants from architecture specs → Rules
- Identifies repeatable workflows → Skills  
- Captures intent and guidance → Instructions
- Leverages Context7 MCP for library-specific validation

---

## 💡 Usage Example

Below is a concrete example of how you would use the prompt with `/project-init` to scaffold a monorepo, using architecture and tech stack specifications.

### 1. What you provide to Cursor

#### A. Architecture Specification Document

A comprehensive system design document that serves as the source of truth for design decisions.

Example structure (`architecture-spec.md`):

```markdown
# E-commerce Platform Architecture

## Executive Summary
- Problem: Scale multi-tenant e-commerce platform to 10k merchants
- Solution: Event-driven microservices with isolated tenant data
- Success Metrics: 99.9% uptime, <200ms p95 response time

## Business Domain Model
- Order Processing Workflow: Cart → Checkout → Payment → Fulfillment
- Inventory Management: Real-time sync across warehouses
- Merchant Isolation: Tenant-specific schemas, strict data boundaries

## Architecture Overview
- Event-driven microservices architecture
- API Gateway for routing and auth
- Message broker for async processing
- Separate read/write databases (CQRS pattern)

## Core Components
- Order Service: Manages order lifecycle, publishes order events
- Payment Service: Handles payment processing, idempotent operations
- Inventory Service: Tracks stock levels, handles reservations
- Notification Service: Email/SMS alerts via queue consumers

## Security & Compliance
- PCI DSS compliant payment handling
- Multi-tenant data isolation at database level
- API rate limiting per merchant tier
- Audit logging for all financial transactions

## Migration Strategy
- Phased rollout: pilot merchants → tier-based migration
- Parallel run existing system for 30 days
- Automated data validation and reconciliation
```

This document defines **what to build** — the business logic, system design, and architectural principles.

#### B. Tech Stack Specification

A separate document defining **how to implement** the architecture.

Example (`tech-stack.md`):

```markdown
# Tech Stack

- Monorepo using pnpm workspaces
- Packages:
  - apps/web (Next.js)
  - apps/api (Node + Fastify)
  - packages/shared (types, utils)
- TypeScript strict everywhere
- Single root lint + test
- CI requires lint + test to pass
- No cross-app imports except via packages/shared
```

This defines the frameworks, tooling, and implementation constraints.

#### C. Slash command invocation

In Cursor chat, with both specifications available in context:

```
/project-init
```

No extra explanation needed. The agent will process both `architecture-spec.md` and `tech-stack.md`.

### 2. What the agent does internally

#### Step 1 — Extract invariants → Rules

The agent processes **both** `architecture-spec.md` and `tech-stack.md` to derive must-hold constraints.

**From architecture-spec.md** (design decisions):
- API boundaries between services
- Data isolation requirements (multi-tenant)
- Security policies (PCI DSS compliance)
- Idempotency requirements for payment operations

**From tech-stack.md** (implementation constraints):
- Monorepo structure and workspace rules
- TypeScript strict mode enforcement
- Cross-package import boundaries

Creates:

```
.cursor/rules/
├── build.md              # CI/lint/test requirements
├── typescript.md         # Language constraints
├── monorepo.md          # Workspace boundaries
├── boundaries.md        # Service isolation rules
├── security.md          # PCI compliance rules
└── workflow.md          # Required checks
```

Example (`monorepo.md` from tech stack):

```markdown
- Use pnpm workspaces at the repository root
- apps may not import from other apps
- Shared code must live in packages/shared
```

Example (`boundaries.md` from architecture spec):

```markdown
- Order Service communicates via events only
- Payment Service must be idempotent for all operations
- Tenant data must never cross database boundaries
- All financial transactions require audit logging
```

#### Step 2 — Extract workflows → Skills

From repeated or implied procedures.

Creates:

```
.cursor/skills/
├── scaffold-package/
│   └── SKILL.md
├── repo-verify/
│   └── SKILL.md
```

Example (`scaffold-package/SKILL.md`):

```markdown
name: scaffold-package
description: Scaffold a new workspace package following monorepo conventions.

workflow:
- create package directory
- add package.json with workspace settings
- configure tsconfig reference
- update workspace config
```

#### Step 3 — Extract intent → Instructions

Creates:

```
.cursor/instructions.md
```

Example:

```markdown
## Monorepo Intent

This repository is a pnpm-based monorepo.
Apps are isolated deployment units.
Shared code must live in packages/shared.

## Adding New Packages
Use the scaffold-package skill.
Do not bypass workspace boundaries.
```

### 3. Resulting repo (initial scaffold)

```
.
├── apps/
│   ├── web/
│   └── api/
├── packages/
│   └── shared/
├── .cursor/
│   ├── rules/
│   ├── skills/
│   └── instructions.md
├── pnpm-workspace.yaml
├── tsconfig.base.json
└── package.json
```

### 4. How you continue using it

#### Scaffolding a new package later

You can simply say:

```
"Add a new internal package for auth utilities"
```

Agent will:
- Obey Rules
- Auto-select scaffold-package skill
- Respect architecture spec

Or force it:

```
/scaffold-package
```

### 5. Why this works (pattern)

- **Architecture Spec** = design decisions (what to build)
- **Tech Stack** = implementation choices (how to build)
- **Rules** = invariants
- **Skills** = workflows
- **Instructions** = intent
- **Slash command** = explicit control only when needed

#### One-line takeaway

> You give architecture once, then `/project-init` turns it into enforceable structure + reusable workflows.

---

## 📋 System Prompt Specification

Below is the complete system prompt for `/project-init`:
```
### System Prompt — /project-init

**You are a Cursor Agent initializing a new project.**

You are given:

1. **Context7 (MCP)** → on-demand access to up-to-date, version-specific library documentation and official code examples
2. **A project architecture specification** provided in the conversation context

Context7 is not architectural memory.
Use it only to:

- Validate library usage
- Fetch correct APIs, configs, and setup patterns
- Avoid stale or deprecated practices

**The architecture spec remains the source of truth for design decisions.**

### 1. Operating Principles

- Architecture spec > defaults
- Context7 supplements specs with current library knowledge
- Prefer deterministic, explicit setups
- Encode decisions once, reuse everywhere
- Optimize for agent repeatability, not intelligence

**If Context7 docs conflict with the architecture spec, follow the spec.**

### 2. Rules (.cursor/rules/)

#### Purpose
Define non-negotiable invariants derived from the architecture spec.

#### Include
- Commands (build, test, lint, deploy)
- Language / framework constraints
- Architectural boundaries
- Safety rules
- Mandatory workflow checks

#### Exclude
- Procedures
- Conditional logic
- Multi-step flows

#### Structure
- Multiple small .md files
- Recursive loading
- Group by concern or domain

**Rules must be:**
- Declarative
- Enforceable
- Minimal

### 3. Skills (.cursor/skills/)

#### Purpose
Encode optional, reusable workflows implied or recommended by the architecture spec.

#### Include
- Verification loops
- Migrations
- Refactors
- Repetitive operational tasks

#### Design
- Outcome-oriented descriptions
- Deterministic steps
- Safe to rerun
- Auto-selectable by the agent

**Skills are capabilities, not guarantees.**

### 4. Instructions (INSTRUCTIONS.md or .cursor/instructions.md)

#### Purpose
Explain how to work with the project, for humans and agents.

#### Include
- Architectural intent
- Key tradeoffs and rationale
- How to extend rules and skills
- Contribution expectations

#### Exclude
- Enforced constraints
- Automated workflows

**Instructions are explanatory, not prescriptive.**

### 5. Decision Rules (internal)

- **Must always hold** → Rule
- **Useful sometimes** → Skill
- **Explains intent** → Instructions

**If uncertain, default to Instructions.**

### 6. Execution Mode

When initializing:

1. Extract invariants from the architecture spec → Rules
2. Extract repeatable workflows → Skills
3. Extract intent and guidance → Instructions
4. Use Context7 only to validate library usage and configs
5. Do not invent patterns not present in the spec
6. Keep output minimal and structured

#### Final Constraint

Produce a setup that results in:
**predictable agents, stable workflows, and low cognitive load.**
```
---

**[← Back to Appendices](./README.md)** | **[← Back to Environment Setup](../01-fundamentals-core-concepts/02-environment-project-setup.md)**
