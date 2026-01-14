---
title: "Runbooks & Guardrails"
nav_order: 3
parent: "Part 5: Team Collaboration & Domain Patterns"
permalink: /docs/05-team-collaboration-domain/runbooks-guardrails/
---

# Section 3: Runbooks & Guardrails

**Target Audience:** Team leads, DevOps engineers, SREs  
**Time to Complete:** 30-45 minutes  
**Prerequisites:** [Part 1: Fundamentals](../../01-fundamentals-core-concepts/)

---

## 📋 Overview

Learn how to create safety guardrails, project rules templates, and operational runbooks that keep teams productive while preventing dangerous operations.

**What you'll learn:**
- YOLO mode guardrails
- Project rules templates for different tech stacks
- Incident response runbooks
- Safe automation patterns
- Preventing common mistakes

---

## 1. Understanding YOLO Mode + Hooks

### What is YOLO Mode?

**YOLO mode** (You Only Live Once) in Cursor enables the AI agent to execute commands and make changes automatically without asking for confirmation at each step.

**Traditional YOLO:** Agent executes one command at a time, stops, waits for next instruction.

**YOLO + Hooks:** Agent executes AND loops automatically based on verification conditions.

**Benefits:**
- ✅ Extremely fast iterations
- ✅ Multi-step workflows complete autonomously
- ✅ Great for well-defined tasks
- ✅ With hooks: True autonomous iteration until goal met

**Risks:**
- ⚠️ Can execute destructive commands
- ⚠️ May make unwanted changes
- ⚠️ Harder to control once started
- ⚠️ With hooks: Can loop indefinitely if conditions never met

### How Hooks Control YOLO Mode

**Hooks** are lifecycle scripts that control when the agent should stop iterating:

**Configuration:** `.cursor/hooks.json`

```json
{
  "hooks": {
    "stop": {
      "script": "./scripts/should-stop.sh",
      "description": "Determines if agent should continue or stop"
    }
  }
}
```

**Stop hook script:**
```bash
#!/bin/bash
# Exit 0 = continue iterating
# Exit 1 = stop iteration

# Check condition (e.g., tests passing)
npm test --silent

if [ $? -eq 0 ]; then
  echo "✅ Tests pass - stopping"
  exit 1  # Stop
else
  echo "❌ Tests fail - continuing"
  exit 0  # Continue
fi
```

**Workflow with YOLO + Hooks:**
1. Agent makes changes
2. Runs tests (via YOLO mode)
3. Hook checks result
4. If fail → Hook returns "continue" → Agent fixes and loops back to step 2
5. If pass → Hook returns "stop" → Agent stops and reports completion

---

## 2. Hook-Based Safety Guardrails

### Strategy 1: Stop Hooks for Safety

**Stop hooks** can prevent dangerous operations by immediately halting the agent.

**Example: Production Safety Hook**

**`.cursor/hooks.json`:**
```json
{
  "hooks": {
    "stop": {
      "script": "./scripts/production-safety.sh",
      "description": "Stop if production environment detected"
    }
  }
}
```

**`./scripts/production-safety.sh`:**
```bash
#!/bin/bash
# Exit 0 = continue, Exit 1 = stop immediately

# Check for production indicators
if [ "$ENVIRONMENT" = "production" ]; then
  echo "🚨 Production environment detected - STOPPING"
  exit 1  # Stop immediately
fi

if [ -f ".production" ]; then
  echo "🚨 Production marker file found - STOPPING"
  exit 1  # Stop immediately
fi

# Check git branch
BRANCH=$(git branch --show-current)
if [ "$BRANCH" = "main" ] || [ "$BRANCH" = "master" ]; then
  echo "⚠️  On protected branch ${BRANCH} - verifying safety"
  # Additional checks...
fi

echo "✅ Safe environment - continuing"
exit 0  # Continue
```

**Usage:**
```
@instructions.md

Deploy new feature to staging environment.
YOLO mode enabled with production safety hook.
```

Agent will automatically stop if it detects production context.

### Strategy 2: Quality Gate Hooks

**Example: Code Coverage Hook**

```bash
#!/bin/bash
# scripts/coverage-gate.sh

coverage=$(npm test --coverage --silent | grep "Statements" | awk '{print $3}' | tr -d '%')

if [ -z "$coverage" ]; then
  echo "⚠️  Cannot determine coverage - continuing"
  exit 0  # Continue (safe default)
fi

if [ "$coverage" -lt 80 ]; then
  echo "❌ Coverage ${coverage}% < 80% - continuing to improve"
  exit 0  # Continue
else
  echo "✅ Coverage ${coverage}% ≥ 80% - stopping"
  exit 1  # Stop
fi
```

**Usage:**
```
@instructions.md @src/auth/

Implement authentication module with comprehensive tests.
Continue until code coverage reaches 80%.
```

### Strategy 3: Build Verification Hooks

```bash
#!/bin/bash
# scripts/build-gate.sh

echo "Running build verification..."
npm run build 2>&1 | tee /tmp/build-output.log

BUILD_EXIT_CODE=${PIPESTATUS[0]}

if [ $BUILD_EXIT_CODE -eq 0 ]; then
  echo "✅ Build successful - stopping"
  exit 1  # Stop
else
  echo "❌ Build failed - continuing to fix"
  # Extract error for agent to see
  tail -20 /tmp/build-output.log
  exit 0  # Continue
fi
```

### Strategy 4: Combining Hooks and Rules

**Rules** (`.cursor/rules/*.md`) define **what is allowed**.  
**Hooks** (`.cursor/hooks.json`) control **when to stop**.

**Example: Safe database migration**

**Rule** (`.cursor/rules/database.md`):
```markdown
---
description: "Database migration safety rules"
alwaysApply: true
---

# Database Migration Safety

## Required Checks
- All migrations must have rollback scripts
- Test migrations in development first
- Never DROP tables in production
- Always use transactions
```

**Hook** (`.cursor/hooks.json` + script):
```bash
#!/bin/bash
# scripts/migration-gate.sh

# Run migrations in test environment
NODE_ENV=test npm run migrate:up

if [ $? -eq 0 ]; then
  # Try rollback
  NODE_ENV=test npm run migrate:down
  
  if [ $? -eq 0 ]; then
    echo "✅ Migration safe (up and down work) - stopping"
    exit 1  # Stop
  else
    echo "❌ Rollback failed - fix migration"
    exit 0  # Continue
  fi
else
  echo "❌ Migration failed - fix before stopping"
  exit 0  # Continue
fi
```

**Usage:**
```
@instructions.md @migrations/

Create database migration to add user_preferences table.
Follow our migration safety rules.
```

Agent will:
1. Follow rules (must have rollback, use transactions)
2. Create migration
3. Hook tests migration up/down
4. If both work → Hook stops
5. If either fails → Hook continues, agent fixes

---

## 3. Rules-Based Safety (Traditional Approach)

### Strategy: Rules-Based Safety

**Problem:** YOLO mode might execute dangerous operations like:
- `rm -rf /` 
- `git push --force origin main`
- `kubectl delete namespace production`
- `DROP DATABASE production;`

**Solution:** Define explicit guardrails in project rules.

---

### Example: Production Safety Guardrails

**File: `.cursor/rules/00-safety-guardrails.md`**
```markdown
# Safety Guardrails

## CRITICAL: Production Protection Rules

These rules MUST be followed at all times, especially in YOLO mode.

### Database Operations

**NEVER execute these commands:**
- `DROP DATABASE` on any production database
- `TRUNCATE TABLE` on production tables
- `DELETE FROM` without `WHERE` clause on production

**Required confirmations for:**
- Any `ALTER TABLE` on production (require explicit human approval)
- Any `UPDATE` affecting > 1000 rows
- Any data migration on tables with > 1M rows

**Safe pattern:**
```sql
-- Always use transactions for data changes
BEGIN;
  UPDATE payments SET status = 'refunded' WHERE id = 12345;
  -- Verify affected rows
  SELECT COUNT(*) FROM payments WHERE status = 'refunded' AND id = 12345;
  -- Human must review before COMMIT
COMMIT; -- or ROLLBACK;
```

### Git Operations

**NEVER execute:**
- `git push --force` to main/master branch
- `git push --force` to any protected branch
- `git reset --hard` on main/master
- `git clean -fd` without explicit confirmation

**Required pattern:**
```bash
# Instead of force push, create new branch
git checkout -b fix/issue-123-retry
git push origin fix/issue-123-retry
# Open PR for review
```

### Kubernetes Operations

**NEVER execute:**
- `kubectl delete namespace production`
- `kubectl delete namespace staging`
- `kubectl delete deployment` in production without approval
- `kubectl scale --replicas=0` in production without approval

**Required pattern:**
```bash
# Always specify namespace explicitly
kubectl get pods -n production

# Always dry-run first
kubectl apply -f deployment.yaml --dry-run=client

# Use --confirm flag for destructive operations (custom)
kubectl delete pod suspicious-pod -n production --confirm
```

### File Operations

**NEVER execute:**
- `rm -rf /`
- `rm -rf ~/*`
- `rm -rf .git`
- Deletion of files matching `*.env`, `*.pem`, `*.key` without confirmation

**Required pattern:**
```bash
# Always use trash/recycling bin instead of rm
trash node_modules  # instead of rm -rf node_modules

# If rm is necessary, confirm files first
ls -la temp-dir/
# Review output
rm -rf temp-dir/
```

### Environment Variables

**NEVER:**
- Log or print environment variables containing:
  - `*_SECRET`, `*_KEY`, `*_PASSWORD`, `*_TOKEN`
- Commit files containing secrets
- Send secrets to external APIs

**Required pattern:**
```bash
# Use secret management
kubectl create secret generic api-key --from-literal=key=$API_KEY

# Never echo secrets
# BAD: echo $DATABASE_PASSWORD
# GOOD: echo "Database password configured: ${DATABASE_PASSWORD:0:3}***"
```

### AWS Operations

**NEVER execute:**
- `aws s3 rb s3://production-* --force`
- `aws rds delete-db-instance` on production
- `aws ec2 terminate-instances` without specific instance ID
- Any AWS operation without `--dry-run` flag (when available)

**Required pattern:**
```bash
# Always use tags to identify resources
aws ec2 describe-instances --filters "Name=tag:Environment,Values=production"

# Use --dry-run when available
aws ec2 run-instances --dry-run --image-id ami-12345 ...
```

---

## Confirmation Prompts

Before executing any HIGH-RISK operation, AI MUST:
1. Show exactly what will be executed
2. Explain the impact
3. Wait for explicit human approval
4. Offer a safer alternative if available

Example:
```
⚠️ HIGH-RISK OPERATION DETECTED ⚠️

You've requested: kubectl delete deployment payment-api -n production

Impact:
- Will terminate all payment-api pods immediately
- Payment processing will stop
- Estimated downtime: 2-5 minutes until redeployment

Safer alternatives:
1. Scale down gradually: kubectl scale deployment payment-api --replicas=1 -n production
2. Use canary deployment to replace
3. Schedule maintenance window

Do you want to proceed? (yes/no)
```
```

---

### Implementing Guardrails in Code

**Pattern:** Pre-execution validation

```typescript
// File: .cursor/scripts/validate-command.ts

interface CommandRisk {
  level: 'safe' | 'caution' | 'danger' | 'critical';
  command: string;
  reason: string;
  alternative?: string;
}

function analyzeCommand(cmd: string, context: { environment: string }): CommandRisk {
  const { environment } = context;
  
  // Critical: Production database drops
  if (cmd.includes('DROP DATABASE') && environment === 'production') {
    return {
      level: 'critical',
      command: cmd,
      reason: 'DROP DATABASE on production is forbidden',
      alternative: 'Create backup first, then use staging for testing'
    };
  }
  
  // Critical: Force push to protected branches
  if (cmd.match(/git push.*--force.*(main|master|production)/)) {
    return {
      level: 'critical',
      command: cmd,
      reason: 'Force push to protected branch is forbidden',
      alternative: 'Create new branch and open PR'
    };
  }
  
  // Danger: Kubernetes namespace deletion
  if (cmd.match(/kubectl delete namespace (production|staging)/)) {
    return {
      level: 'critical',
      command: cmd,
      reason: 'Cannot delete production/staging namespace',
      alternative: 'Delete individual resources instead'
    };
  }
  
  // Danger: Mass file deletion
  if (cmd.includes('rm -rf') && !cmd.includes('node_modules')) {
    return {
      level: 'danger',
      command: cmd,
      reason: 'Recursive deletion requires confirmation',
      alternative: 'Review files with ls first, use trash command'
    };
  }
  
  // Caution: Production deployments
  if (cmd.includes('kubectl apply') && environment === 'production') {
    return {
      level: 'caution',
      command: cmd,
      reason: 'Production deployment - verify changes first',
      alternative: 'Run --dry-run=client first'
    };
  }
  
  return { level: 'safe', command: cmd, reason: 'Command appears safe' };
}

function requireConfirmation(risk: CommandRisk): boolean {
  if (risk.level === 'critical') {
    console.error(`🚨 BLOCKED: ${risk.reason}`);
    if (risk.alternative) {
      console.log(`💡 Alternative: ${risk.alternative}`);
    }
    return false; // Block command entirely
  }
  
  if (risk.level === 'danger') {
    console.warn(`⚠️  HIGH RISK: ${risk.reason}`);
    console.warn(`Command: ${risk.command}`);
    if (risk.alternative) {
      console.log(`💡 Safer alternative: ${risk.alternative}`);
    }
    return confirm('Type "yes" to proceed: ') === 'yes';
  }
  
  if (risk.level === 'caution') {
    console.log(`⚡ CAUTION: ${risk.reason}`);
    return confirm('Continue? (yes/no): ') === 'yes';
  }
  
  return true; // Safe commands proceed
}

// Example usage
const command = process.argv[2];
const environment = process.env.ENVIRONMENT || 'development';

const risk = analyzeCommand(command, { environment });
if (requireConfirmation(risk)) {
  console.log('✅ Executing:', command);
  // Execute command
} else {
  console.log('❌ Command blocked for safety');
  process.exit(1);
}
```

**Integration with Cursor:**
```markdown
# File: .cursor/rules/01-command-validation.md

Before executing ANY command:
1. Run it through validation: `ts-node .cursor/scripts/validate-command.ts "<command>"`
2. If validation fails, DO NOT execute
3. Propose safer alternative to user
4. Wait for explicit approval for caution/danger level commands
```

---

## 3. Project Rules Templates

### Template 1: React + TypeScript + Node.js

**File: `.cursor/rules/template-react-typescript-node.md`**
```markdown
# React + TypeScript + Node.js Project Rules

## Tech Stack
- **Frontend:** React 19, TypeScript 5.8, Vite
- **Backend:** Node.js 22, Express, TypeScript
- **Database:** PostgreSQL 16
- **Testing:** Vitest, React Testing Library, Supertest
- **Styling:** TailwindCSS 4
- **State:** Zustand
- **Forms:** React Hook Form + Zod validation

## Project Structure
```
/
├── apps/
│   ├── web/          # React frontend
│   │   ├── src/
│   │   │   ├── components/
│   │   │   ├── pages/
│   │   │   ├── hooks/
│   │   │   ├── stores/
│   │   │   └── utils/
│   │   └── tests/
│   └── api/          # Node.js backend
│       ├── src/
│       │   ├── routes/
│       │   ├── controllers/
│       │   ├── services/
│       │   ├── models/
│       │   └── middleware/
│       └── tests/
└── packages/
    └── shared/       # Shared types
        └── src/
            └── types/
```

## Code Standards

### Components
- Use functional components only
- One component per file
- Props interface named `<Component>Props`
- Export component as named export

```typescript
// Good
interface ButtonProps {
  label: string;
  onClick: () => void;
  variant?: 'primary' | 'secondary';
}

export function Button({ label, onClick, variant = 'primary' }: ButtonProps) {
  return (
    <button 
      className={`btn btn-${variant}`}
      onClick={onClick}
    >
      {label}
    </button>
  );
}
```

### API Routes
- Use express.Router() for route grouping
- Async route handlers with try/catch
- Zod validation on all inputs
- Structured error responses

```typescript
// Good
import { Router } from 'express';
import { z } from 'zod';

const router = Router();

const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(2).max(100),
});

router.post('/users', async (req, res) => {
  try {
    const data = CreateUserSchema.parse(req.body);
    const user = await userService.create(data);
    res.status(201).json({ user });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return res.status(400).json({ errors: error.errors });
    }
    res.status(500).json({ error: 'Internal server error' });
  }
});

export default router;
```

### Testing
- Test files co-located: `Button.tsx` → `Button.test.tsx`
- Use React Testing Library (not Enzyme)
- Mock external dependencies
- Test user interactions, not implementation

```typescript
// Good
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from './Button';

describe('Button', () => {
  it('calls onClick when clicked', () => {
    const handleClick = vi.fn();
    render(<Button label="Click me" onClick={handleClick} />);
    
    fireEvent.click(screen.getByText('Click me'));
    
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
});
```

## Error Handling
Always use structured errors:

```typescript
class AppError extends Error {
  constructor(
    message: string,
    public statusCode: number = 500,
    public code: string = 'INTERNAL_ERROR'
  ) {
    super(message);
    this.name = 'AppError';
  }
}

// Usage
throw new AppError('User not found', 404, 'USER_NOT_FOUND');
```

## Environment Variables
- Use `.env.example` for template
- Never commit `.env` files
- Validate env vars on startup

```typescript
// env.ts
import { z } from 'zod';

const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  NODE_ENV: z.enum(['development', 'production', 'test']),
});

export const env = envSchema.parse(process.env);
```

## Git Conventions
- Branch naming: `feature/description`, `fix/description`, `chore/description`
- Commit messages: Conventional Commits format
- PR title format: `[TYPE] Brief description`

## Common Commands
```bash
# Development
npm run dev:web    # Start frontend
npm run dev:api    # Start backend

# Testing
npm test           # Run all tests
npm run test:watch # Watch mode

# Build
npm run build      # Build for production

# Database
npm run migrate:up    # Run migrations
npm run migrate:down  # Rollback migration
```
```

---

### Template 2: Python + FastAPI + PostgreSQL

**File: `.cursor/rules/template-python-fastapi.md`**
```markdown
# Python + FastAPI + PostgreSQL Project Rules

## Tech Stack
- **Backend:** Python 3.12, FastAPI 0.110
- **Database:** PostgreSQL 16, SQLAlchemy 2.0, Alembic
- **Testing:** pytest, httpx
- **Validation:** Pydantic v2
- **Auth:** JWT (python-jose)

## Project Structure
```
/
├── app/
│   ├── api/
│   │   └── v1/
│   │       ├── endpoints/
│   │       └── deps.py
│   ├── core/
│   │   ├── config.py
│   │   ├── security.py
│   │   └── database.py
│   ├── models/
│   ├── schemas/
│   ├── services/
│   └── main.py
├── tests/
├── alembic/
│   └── versions/
└── requirements.txt
```

## Code Standards

### API Endpoints
- Use APIRouter for grouping
- Pydantic models for request/response
- Dependency injection for database
- Async endpoints

```python
# Good
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from app.api.deps import get_db
from app.schemas.user import UserCreate, UserResponse
from app.services.user import create_user

router = APIRouter()

@router.post("/users", response_model=UserResponse, status_code=201)
async def create_user_endpoint(
    user_in: UserCreate,
    db: Session = Depends(get_db)
):
    """Create new user"""
    try:
        user = await create_user(db, user_in)
        return user
    except ValueError as e:
        raise HTTPException(status_code=400, detail=str(e))
```

### Database Models
- Use SQLAlchemy 2.0 style
- Type hints on all columns
- Explicit table names

```python
# Good
from sqlalchemy import String, Integer, DateTime
from sqlalchemy.orm import Mapped, mapped_column
from datetime import datetime
from app.core.database import Base

class User(Base):
    __tablename__ = "users"
    
    id: Mapped[int] = mapped_column(Integer, primary_key=True)
    email: Mapped[str] = mapped_column(String(255), unique=True, nullable=False)
    hashed_password: Mapped[str] = mapped_column(String(255), nullable=False)
    created_at: Mapped[datetime] = mapped_column(DateTime, default=datetime.utcnow)
```

### Pydantic Schemas
- Separate Create/Update/Response schemas
- Use ConfigDict for ORM mode

```python
# Good
from pydantic import BaseModel, EmailStr, ConfigDict
from datetime import datetime

class UserBase(BaseModel):
    email: EmailStr

class UserCreate(UserBase):
    password: str

class UserResponse(UserBase):
    id: int
    created_at: datetime
    
    model_config = ConfigDict(from_attributes=True)
```

### Testing
- Use pytest fixtures
- httpx.AsyncClient for API tests
- Separate test database

```python
# Good
import pytest
from httpx import AsyncClient
from app.main import app

@pytest.mark.asyncio
async def test_create_user(client: AsyncClient):
    response = await client.post(
        "/api/v1/users",
        json={"email": "test@example.com", "password": "password123"}
    )
    assert response.status_code == 201
    data = response.json()
    assert data["email"] == "test@example.com"
    assert "id" in data
```

## Error Handling
Custom exception handlers:

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()

class AppException(Exception):
    def __init__(self, status_code: int, detail: str, code: str):
        self.status_code = status_code
        self.detail = detail
        self.code = code

@app.exception_handler(AppException)
async def app_exception_handler(request: Request, exc: AppException):
    return JSONResponse(
        status_code=exc.status_code,
        content={"detail": exc.detail, "code": exc.code}
    )
```

## Environment Variables
Use Pydantic Settings:

```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    database_url: str
    jwt_secret: str
    jwt_algorithm: str = "HS256"
    
    model_config = SettingsConfigDict(env_file=".env")

settings = Settings()
```

## Database Migrations
```bash
# Create migration
alembic revision --autogenerate -m "Add users table"

# Apply migrations
alembic upgrade head

# Rollback
alembic downgrade -1
```
```

---

## 4. Incident Response Runbooks

### Runbook Template

**File: `runbooks/high-memory-usage.md`**
```markdown
# Runbook: High Memory Usage Alert

## Alert Trigger
- **Metric:** Container memory > 85% of limit
- **Duration:** > 5 minutes
- **Severity:** P2 (can become P1 if OOMKill occurs)

## Immediate Actions (5 minutes)

### Step 1: Verify the alert
```bash
# Check current memory usage
kubectl top pods -n production | grep payment-api

# Get pod details
kubectl describe pod <pod-name> -n production
```

**Expected output:**
```
NAME                          CPU    MEMORY
payment-api-7d4f9c8b-xj4k2    450m   3200Mi/4000Mi (80%)
```

### Step 2: Check for OOMKills
```bash
# Look for recent restarts
kubectl get pods -n production | grep payment-api

# Check events for OOM
kubectl get events -n production --sort-by='.lastTimestamp' | grep OOM
```

**If OOMKills detected:** Escalate to P1, proceed to mitigation.

### Step 3: Quick mitigation
```bash
# Increase memory limit temporarily (requires approval)
kubectl set resources deployment payment-api -n production \
  --limits=memory=6Gi \
  --requests=memory=4Gi

# Monitor rollout
kubectl rollout status deployment payment-api -n production
```

## Investigation (15 minutes)

### Cursor AI-Assisted Investigation

**Prompt:**
```
Cmd+I
"@kubernetes
Investigate high memory usage in payment-api pods:

1. Get memory metrics for last 2 hours
2. Analyze heap dumps if available
3. Check for memory leaks in recent deployments
4. Review code changes in last 24 hours

@apps/payment-api/
@runbooks/high-memory-usage.md
```

### Manual Investigation Steps

**Check application logs:**
```bash
# Get recent logs
kubectl logs -n production deployment/payment-api --since=1h | grep -i "memory\|heap\|oom"

# Check for memory warnings
kubectl logs -n production deployment/payment-api --since=1h | grep "WARN"
```

**Check metrics in Grafana:**
1. Open dashboard: "Payment API Memory"
2. Compare current vs baseline (last 7 days)
3. Look for correlation with traffic spikes

**Analyze heap dump (if available):**
```bash
# Get heap dump from pod
kubectl exec -n production <pod-name> -- jmap -dump:format=b,file=/tmp/heap.bin 1

# Copy to local
kubectl cp production/<pod-name>:/tmp/heap.bin ./heap.bin

# Analyze with Cursor
```

**Prompt:**
```
Cmd+I
"Analyze this heap dump and identify memory leak sources:
@heap.bin

Compare with our application code:
@apps/payment-api/src/
```

## Root Cause Analysis

### Common Causes

1. **Memory Leak in Code**
   - Symptom: Memory grows linearly over time
   - Fix: Identify and fix leak, redeploy

2. **Inadequate Memory Limits**
   - Symptom: Memory usage stable but close to limit
   - Fix: Increase memory limits

3. **Traffic Spike**
   - Symptom: Memory correlates with request rate
   - Fix: Scale horizontally (more replicas)

4. **Large Object Processing**
   - Symptom: Memory spikes during specific operations
   - Fix: Implement streaming/pagination

## Long-Term Fix

### Option 1: Code Fix (Memory Leak)
```bash
# Create fix branch
git checkout -b fix/memory-leak

# Use Cursor to fix leak
# Cmd+I: "Fix memory leak in event handler: @src/handlers/event.ts"

# Test fix
npm test
npm run test:load  # Load test to verify

# Deploy
git push origin fix/memory-leak
# Open PR, get review, merge, deploy
```

### Option 2: Configuration Update
```yaml
# Update deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-api
spec:
  template:
    spec:
      containers:
      - name: payment-api
        resources:
          requests:
            memory: "2Gi"
            cpu: "500m"
          limits:
            memory: "4Gi"
            cpu: "1000m"
```

### Option 3: Architectural Change
- Implement caching layer (Redis)
- Move heavy processing to background jobs
- Implement connection pooling

## Prevention

### Add Monitoring
```yaml
# Add memory alerts
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: payment-api-memory-alerts
spec:
  groups:
  - name: memory
    rules:
    - alert: HighMemoryUsage
      expr: container_memory_usage_bytes{pod=~"payment-api-.*"} / container_spec_memory_limit_bytes > 0.85
      for: 5m
      annotations:
        summary: "Pod {{ $labels.pod }} high memory usage"
```

### Update Runbook
Document the incident in `runbooks/incidents/`:
```markdown
# Incident: 2026-01-12 High Memory Usage

**Root Cause:** Connection pool leak in EventStore client

**Fix:** Updated event-handler.ts to properly close connections

**Prevention:** 
- Added unit test for connection cleanup
- Added memory usage monitoring in CI
- Updated runbook with new debugging steps
```

## Checklist

- [ ] Alert verified
- [ ] Immediate mitigation applied (if needed)
- [ ] Root cause identified
- [ ] Long-term fix implemented
- [ ] Incident documented
- [ ] Runbook updated
- [ ] Post-mortem scheduled (for P1 incidents)
```

---

## 5. Safe Automation Patterns

### Pattern 1: Dry-Run First

**Always test commands with dry-run before execution:**

```markdown
# File: .cursor/rules/02-dry-run-pattern.md

For any deployment or infrastructure change:
1. Run with --dry-run first
2. Show user the planned changes
3. Wait for approval
4. Execute actual command

Example:
```bash
# Step 1: Dry run
kubectl apply -f deployment.yaml --dry-run=client

# Step 2: Show diff
kubectl diff -f deployment.yaml

# Step 3: Get approval
# (User reviews and approves)

# Step 4: Execute
kubectl apply -f deployment.yaml
```
```

---

### Pattern 2: Gradual Rollout

**For deployments, always use progressive rollout:**

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-api
spec:
  replicas: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2        # Add 2 new pods at a time
      maxUnavailable: 1  # Allow 1 old pod down at a time
  template:
    spec:
      containers:
      - name: payment-api
        image: payment-api:v2.0.0
        readinessProbe:  # Wait for pod to be ready
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:   # Kill pod if unhealthy
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
```

**Cursor prompt for safe deployment:**
```
Cmd+I
"Deploy new version of payment-api with progressive rollout:

1. Update image tag to v2.0.0
2. Apply with kubectl
3. Watch rollout progress
4. If error rate > 1%, automatic rollback
5. If successful, continue to 100%

@infra/k8s/payment-api-deployment.yaml
@runbooks/safe-deployment.md
```

---

### Pattern 3: Feature Flags

**Use feature flags for risky changes:**

```typescript
// feature-flags.ts
import { LaunchDarkly } from '@launchdarkly/node-server-sdk';

const ldClient = LaunchDarkly.init(process.env.LAUNCHDARKLY_SDK_KEY);

export async function isFeatureEnabled(
  flagKey: string,
  userId: string,
  defaultValue: boolean = false
): Promise<boolean> {
  await ldClient.waitForInitialization();
  return ldClient.variation(flagKey, { key: userId }, defaultValue);
}

// Usage in code
if (await isFeatureEnabled('new-payment-flow', user.id)) {
  return processPaymentV2(request);
} else {
  return processPaymentV1(request);
}
```

**Benefits:**
- Can enable for 5% of users first
- Instant rollback without code deployment
- A/B testing built-in

---

## 6. Preventing Common Mistakes

### Mistake 1: Committing Secrets

**Prevention: Pre-commit hooks**

**File: `.husky/pre-commit`**
```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

# Check for secrets
npx secretlint "**/*"

# Check for common secret patterns
if git diff --cached | grep -i "password\s*=\|secret\s*=\|api[_-]key\s*="; then
  echo "❌ Potential secret detected in commit"
  echo "Use environment variables instead"
  exit 1
fi
```

---

### Mistake 2: Breaking Changes Without Tests

**Prevention: Required test coverage**

**File: `.cursor/rules/03-test-requirements.md`**
```markdown
# Test Requirements

ALL code changes must include tests:

1. Unit tests for business logic (required)
2. Integration tests for API endpoints (required)
3. E2E tests for critical user flows (required for user-facing changes)

Minimum coverage: 80%

**CI will fail if:**
- Coverage drops below 80%
- No tests added for new code
- Existing tests break
```

---

### Mistake 3: Deploying Untested Code

**Prevention: Staging environment requirement**

**File: `runbooks/deployment-checklist.md`**
```markdown
# Deployment Checklist

## Before Production Deployment

- [ ] All tests pass in CI
- [ ] Code reviewed and approved
- [ ] Deployed to staging environment
- [ ] Smoke tests pass on staging
- [ ] Load tests pass (for performance-critical changes)
- [ ] Database migrations tested on staging
- [ ] Rollback plan documented
- [ ] Monitoring dashboards ready
- [ ] Team notified in #deployments channel

## Production Deployment

- [ ] Deploy during low-traffic window (if possible)
- [ ] Monitor error rates (< 0.1% increase allowed)
- [ ] Monitor latency (P99 < 500ms)
- [ ] Monitor memory/CPU usage
- [ ] Verify critical user flows work
- [ ] Check logs for errors

## Post-Deployment

- [ ] Monitor for 30 minutes
- [ ] Update status page if needed
- [ ] Document any issues in #incidents
- [ ] Update changelog
```

---

## 7. Best Practices Summary

### Do's ✅

1. **Use hooks for autonomous safety**
   - Stop hooks prevent dangerous operations
   - Quality gate hooks ensure standards
   - Combine hooks + YOLO for safe automation

2. **Define explicit safety guardrails**
   - Rules for what's allowed (`.cursor/rules/*.md`)
   - Hooks for when to stop (`.cursor/hooks.json`)
   - Combine both for layered defense

3. **Use project rules templates**
   - Consistent patterns across team
   - Faster onboarding

4. **Create runbooks for common incidents**
   - Faster resolution
   - Knowledge sharing

5. **Implement dry-run pattern**
   - Preview changes before applying
   - Catch mistakes early

6. **Use gradual rollouts**
   - Canary deployments
   - Feature flags
   - Progressive delivery

7. **Test hooks independently**
   - Verify hook logic before using with agent
   - Ensure hooks are fast (< 5 seconds)
   - Handle all edge cases

### Don'ts ❌

1. **Don't use YOLO mode in production without hooks and rules**
   - Too risky without safety measures
   - Always have stop hooks for production

2. **Don't skip staging environment**
   - Test everything before production

3. **Don't deploy without monitoring**
   - Must have metrics to detect issues

4. **Don't ignore failed tests**
   - Tests exist for a reason

5. **Don't create slow hooks**
   - Hooks block agent execution
   - Keep hooks under 5 seconds

6. **Don't rely only on rules or only on hooks**
   - Use both for layered safety
   - Rules define constraints, hooks verify results

---

## 8. Next Steps

**Implement guardrails for your team:**

1. Create `.cursor/rules/00-safety-guardrails.md`
2. Add command validation script
3. Set up pre-commit hooks
4. Document runbooks for top 3 incidents
5. Create deployment checklist

**Template to start:**
```bash
# Initialize safety framework
mkdir -p .cursor/rules .cursor/scripts runbooks

# Create first guardrail
cat > .cursor/rules/00-safety-guardrails.md << 'EOF'
# Safety Guardrails

## Production Protection
- Never DROP DATABASE on production
- Never git push --force to main
- Never kubectl delete namespace production
- Always use --dry-run first
EOF

# Commit to git
git add .cursor/
git commit -m "Add Cursor safety guardrails"
```

---

**Series Complete!** 🎉

You've now mastered:
- ✅ Team collaboration patterns
- ✅ Domain-specific workflows
- ✅ Safety guardrails and runbooks

**Next Part:** [Part 6: Troubleshooting & Reference](../../06-troubleshooting-reference/)

**Related Sections:**
- [Part 1: Project Setup](../../01-fundamentals-core-concepts/02-environment-project-setup.md)
- [Part 3: DevOps Patterns](../../03-devops-backend-architecture/)
