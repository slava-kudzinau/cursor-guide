---
title: "Part 5: Team Collaboration & Domain Patterns"
nav_order: 6
---

# Cursor IDE: Team Collaboration & Domain Patterns
## Part 5 of 6: Teamwork and Domain-Specific Workflows

**Maintainer**: Viachaslau Kudzinau (viachaslau_kudzinau@epam.com)  
**Version**: 2.0  
**Last Updated**: December 2025

> **📚 Navigation**: [Index](../) | [← Part 4](04-context-prompting-strategies) | **Part 5** | [Part 6 →](06-troubleshooting-reference)

---

## Table of Contents

10. [Team Collaboration](#10-team-collaboration)
11. [Domain-Specific Patterns](#11-domain-specific-patterns)
12. [Runbooks & Guardrails](#12-runbooks--guardrails)

---

## 10. Team Collaboration (EXPANDED)

### 10.1 Shared Rules Repository

**Pattern:** Centralized rules for team consistency

**Option 1: Team Rules (Recommended for Team/Enterprise plans)**
- Managed from [Cursor dashboard](https://cursor.com/dashboard?tab=team-content)
- Centrally controlled by admins
- Can be enforced for all team members
- No git setup required
- Auto-synced to all team members

**Option 2: Git Submodule (Self-managed)**
```bash
# Git repo: company-cursor-rules
rules/
├── typescript/
│   └── RULE.md             # TS standards
├── react-patterns/
│   └── RULE.md             # React conventions
├── api-conventions/
│   └── RULE.md             # REST/GraphQL patterns
├── testing/
│   └── RULE.md             # Testing standards
├── security/
│   └── RULE.md             # Security requirements
├── performance/
│   └── RULE.md             # Performance guidelines
└── git-workflow/
    └── RULE.md             # Git commit conventions

# Each project:
git submodule add git@github.com:company/cursor-rules .cursor/rules

# Auto-sync on checkout:
git config submodule.recurse true
```

**Option 3: Remote Rules (GitHub Import)**
- Import rules directly from any GitHub repo
- `Cursor Settings > Rules > + Add Rule > Remote Rule (Github)`
- Stays synced with source repository

**Benefit:** Everyone follows same patterns, onboarding faster, code reviews easier.

### 10.2 Team Notepads (NEW)

**Pattern:** Shared knowledge base for common issues

**Architecture Notepad:** `.cursor/notepads/architecture.md`
```markdown
# System Architecture

## Services
- **user-service**: Authentication, profiles (port 3001)
- **order-service**: Orders, cart, checkout (port 3002)
- **payment-service**: Stripe integration (port 3003)
- **notification-service**: Email, SMS, push (port 3004)

## Data Flow
User places order → order-service creates order → payment-service charges → notification-service sends confirmation

## Tech Stack
- Backend: Node.js 20 + Fastify
- Database: PostgreSQL 15 + Prisma
- Cache: Redis 7
- Queue: RabbitMQ
- Frontend: Next.js 14 + React 18

## Known Patterns
- All services use JWT auth (shared secret in env)
- All services log to ELK stack
- All services expose /health and /metrics endpoints
```

**Usage:**
```
@notepads/architecture

I need to add a new review-service. What patterns should I follow?
```

**Agent:** Reads architecture notepad, suggests structure matching existing services.

**Troubleshooting Notepad:** `.cursor/notepads/troubleshooting.md`
```markdown
# Common Issues & Solutions

## Database Connection Fails in CI
**Symptom:** Tests fail with "connection refused"
**Cause:** DATABASE_URL not set in CI environment
**Solution:** 
1. Add DATABASE_URL to GitHub Secrets
2. Reference in .github/workflows/ci.yml:
```yaml
env:
  DATABASE_URL: ${{ secrets.DATABASE_URL }}
```
3. Update ConfigMap: `kubectl edit configmap app-config -n production`
4. Restart pods: `kubectl rollout restart deployment/app -n production`

## Search Returns Empty Results
**Symptom:** Elasticsearch returns `[]` for valid queries
**Cause:** Index not created or out of sync
**Solution:** 
```bash
npm run search:reindex
# Or manually:
curl -X POST localhost:9200/products/_refresh
```

## Tests Pass Locally, Fail in CI
**Symptom:** Same tests different results
**Cause:** Timezone differences (CI uses UTC)
**Fix:** Always use UTC in tests:
```javascript
// In test setup
process.env.TZ = 'UTC';
```
```

**Usage:**
```
@notepads/troubleshooting

Getting "relation users does not exist" error in CI.
What's the fix?
```

**Agent:** Reads troubleshooting notepad, provides exact solution + commands.

**Maintenance:** 
- Update after solving new issues
- Quarterly review to remove outdated entries
- Assign owner (tech lead) to maintain

### 10.3 Pair Programming with Agent

**Pattern:** You architect, agent implements

```
You: @instructions.md

We need to add rate limiting to the API. I'm thinking Redis-based, 
with separate limits per user and per endpoint. What do you suggest?

Agent: [Analyzes codebase]

I see you're using Fastify. I recommend @fastify/rate-limit with 
Redis store. Your existing auth middleware extracts userId from JWT, 
we can use that as the rate limit key.

Current architecture:
- Auth middleware in src/middleware/auth.ts
- Redis client in src/lib/redis.ts  
- Environment variables for limits would fit in .env

Proposed limits:
- 100 requests/hour per authenticated user
- 1000 requests/hour per endpoint (global)
- 20 requests/hour for auth endpoints (prevent brute force)

Should I proceed with this approach?

You: Yes, but let's start with just the /api/auth/* endpoints. 
Add logging for rate limit hits too.

Agent: [Generates middleware, updates auth routes, adds Winston logging]

You: [Reviews diff] 

Looks good, but the rate limit error message should include reset time.
Also add X-RateLimit-* headers to all responses.

Agent: [Updates code with retry-after header and X-RateLimit headers]

You: Perfect. Ship it.
```

**Why this works:**
- You stay in strategic mode
- Agent handles tactical execution
- Fast iteration
- Learn patterns for future tasks

### 10.4 Automated PR Reviews

**Command:** `.cursor/commands/pr-review.md`
```markdown
# PR Review Automation

Review this pull request against team standards:

## Architecture Review
@notepads/architecture
- Does this fit our service patterns?
- Any new dependencies that conflict with existing stack?
- Database changes follow migration patterns?

## Code Quality
@rules/typescript @rules/react-patterns
- Follows TypeScript standards?
- Follows React patterns?
- Proper error handling?
- No hardcoded secrets?

## Testing
@rules/testing
- Unit tests for new functions?
- Integration tests for API changes?
- E2E tests for user flows?
- Coverage >80%?

## Security
@rules/security
- Input validation on new endpoints?
- SQL injection prevention?
- XSS prevention?
- CSRF protection?
- Rate limiting on new endpoints?

## Performance
- No N+1 queries?
- Proper indexes on new DB columns?
- Large lists paginated?
- API response times acceptable?

## Documentation
- README updated?
- API documentation updated?
- Breaking changes documented?
- Migration guide if needed?

Run tests: npm test
Check coverage: npm run test:coverage
Verify types: tsc --noEmit

Generate comprehensive review report.
```

**Usage:**
```
/pr-review
```

**Result:** Agent produces detailed review with specific line-by-line feedback.

---

## 11. Domain-Specific Patterns

### 11.1 Kubernetes Manifest Generation

```
@instructions.md

Create Kubernetes manifests for microservice deployment:

Service: order-processor
Image: myregistry/order-processor:v1.2.3
Replicas: 3 (autoscale to 10)
Port: 8080

Requirements:
- Deployment with rolling update strategy (maxSurge: 1, maxUnavailable: 0)
- Service (ClusterIP for internal, LoadBalancer for external)
- Ingress (path: /api/orders, TLS enabled)
- ConfigMap for environment variables (non-sensitive)
- Secret for database credentials (from external secrets operator)
- HorizontalPodAutoscaler (CPU > 70% → scale, max 10 pods)
- Resource requests/limits (500m/1000m CPU, 512Mi/1Gi memory)
- Health checks:
  - Liveness: /health (fail after 3 checks, 10s interval)
  - Readiness: /ready (fail after 3 checks, 5s interval)
  - Startup: /health (fail after 30 checks, 10s interval)
- Pod anti-affinity (spread across nodes, preferredDuringScheduling)
- Pod disruption budget (min 2 available)
- Init container (wait for PostgreSQL, timeout 60s)

Labels:
- app: order-processor
- version: v1.2.3
- env: production
- team: payments

Annotations:
- prometheus.io/scrape: "true"
- prometheus.io/port: "9090"
- prometheus.io/path: "/metrics"

Security:
- Run as non-root (runAsUser: 1000)
- Read-only root filesystem
- Drop all capabilities
- No privilege escalation

Generate:
- deployment.yaml
- service.yaml
- ingress.yaml
- configmap.yaml
- secret.yaml (template only, actual values from external)
- hpa.yaml
- pdb.yaml (pod disruption budget)
- README.md (deployment instructions)

Validate with: kubectl apply --dry-run=client -f ./k8s/
```

### 11.2 Data Pipeline (dbt + Airflow)

```
@instructions.md

Create data pipeline for customer analytics:

Pipeline stages:
1. Extract: PostgreSQL (customers, orders tables)
2. Transform: dbt (aggregations, enrichment)
3. Load: BigQuery (data warehouse)

Schedule: Daily at 2 AM UTC
Orchestration: Airflow

Transformations (dbt):
1. Staging models:
   - stg_customers (clean, standardize)
   - stg_orders (clean, add derived fields)
   
2. Intermediate models:
   - int_customer_orders (join customers + orders)
   - int_customer_metrics (aggregate order stats)
   
3. Mart models:
   - fct_customer_analytics (fact table)
   - dim_customer (dimension table)

Derived metrics:
- total_orders (count)
- total_spent (sum)
- avg_order_value (average)
- customer_lifetime_value (calculation)
- churn_risk_score (ML model output)
- days_since_last_order
- order_frequency (orders per month)

Data quality checks (dbt tests):
- Not null on primary keys
- Unique on customer_id
- Referential integrity (customer_id exists in customers)
- Accepted values for status (active, churned)
- Custom: lifetime_value > 0

Airflow DAG:
- Task 1: Extract (Python operator, pg_dump or Pandas)
- Task 2: dbt run (dbt models)
- Task 3: dbt test (data quality)
- Task 4: Load to BigQuery (if tests pass)
- Task 5: Send Slack notification (success/failure)

Error handling:
- Retry: 3 times, exponential backoff
- On failure: Send alert to #data-team Slack channel
- Partial failure: Skip failed tables, continue with rest
- Idempotency: Use MERGE/UPSERT, safe to re-run

Generate:
- dags/customer_analytics.py (Airflow DAG)
- dbt/models/staging/ (staging models)
- dbt/models/intermediate/ (intermediate models)
- dbt/models/marts/ (mart models)
- dbt/tests/ (custom data quality tests)
- src/extract.py (PostgreSQL extraction)
- src/load.py (BigQuery writer)
- tests/ (unit tests for transformations)
- README.md (pipeline documentation)
```

### 11.3 ML Model Training Pipeline

```
@instructions.md

Create ML training pipeline for churn prediction:

Model: XGBoost binary classifier
Task: Predict customer churn (next 30 days)
Data: src/data/customers.csv (100k rows, 50 features)

Pipeline stages:
1. EDA (Exploratory Data Analysis)
   - Load data, check shape, dtypes
   - Analyze target distribution (class balance)
   - Identify missing values
   - Visualize feature distributions
   - Correlation analysis

2. Feature Engineering
   - Handle missing values (impute or drop)
   - One-hot encode categoricals (country, subscription_type)
   - Normalize numerics (StandardScaler)
   - Create interaction features (tenure * monthly_spend)
   - Create time-based features (days_since_signup, season)
   - Feature selection (remove low-variance, highly correlated)

3. Train/Validation Split
   - 80/20 split, stratified by target
   - Set random seed: 42 (reproducibility)
   - Validate time-based split (no data leakage)

4. Model Training
   - Baseline: Logistic Regression
   - Main model: XGBoost
   - Hyperparameter tuning: GridSearchCV
     - max_depth: [3, 5, 7]
     - learning_rate: [0.01, 0.1, 0.3]
     - n_estimators: [100, 200, 300]
     - subsample: [0.8, 1.0]
   - 5-fold cross-validation
   - Track experiments with MLflow

5. Evaluation
   - Metrics: accuracy, precision, recall, F1, ROC-AUC
   - Confusion matrix
   - Feature importance (top 20 features)
   - Prediction threshold optimization (maximize F1)
   - Error analysis (false positives/negatives)

6. Save Artifacts
   - model.pkl (trained XGBoost model)
   - preprocessor.pkl (feature engineering pipeline)
   - feature_names.json (for inference)
   - metrics.json (evaluation results)
   - feature_importance.png (visualization)

7. Model Card
   - Model description
   - Training data characteristics
   - Performance metrics
   - Limitations and biases
   - Intended use

Generate:
- notebooks/01_eda.ipynb (exploratory analysis)
- notebooks/02_train.ipynb (full training pipeline)
- src/features.py (feature engineering functions)
- src/model.py (model training class)
- src/evaluate.py (evaluation functions)
- src/predict.py (inference script)
- tests/ (unit tests for features, model)
- requirements.txt
- README.md (how to train, how to use)
- model_card.md
```

---

## 12. Runbooks & Guardrails

### 12.1 Project Rules Template (TypeScript)

```markdown
---
description: TypeScript coding standards and best practices
globs: ["**/*.ts", "**/*.tsx"]
alwaysApply: false
---

# TypeScript Standards

## Type Safety
- Enable strict mode in tsconfig.json
- No `any` types (use `unknown` + type guards or proper types)
- Prefer `interface` over `type` for objects (better error messages)
- Use discriminated unions for variant types
- Explicit return types on exported functions

Example:
```typescript
// BAD
function getUser(id: any): any {
  return db.users.find(id);
}

// GOOD
interface User {
  id: string;
  email: string;
  name: string;
}

function getUser(id: string): Promise<User | null> {
  return db.users.findUnique({ where: { id } });
}
```

## Imports
- Use absolute imports: `@/components/Button` not `../../components/Button`
- Group imports: React, external libs, internal, types, styles
- No default exports (except Next.js pages, React components where required)
- Explicit file extensions in imports for clarity

## Error Handling
- Use Result<T, E> type for fallible operations
- Never throw in async code (return Error wrapped in Result)
- Log errors with context (traceId, userId, operation)
- Return typed errors with error codes

Example:
```typescript
type Result<T, E = Error> = 
  | { ok: true; value: T }
  | { ok: false; error: E };

async function fetchUser(id: string): Promise<Result<User>> {
  try {
    const user = await api.get(`/users/${id}`);
    return { ok: true, value: user };
  } catch (error) {
    return { ok: false, error: new Error(`Failed to fetch user ${id}`) };
  }
}
```

## Testing
- Co-locate tests: `Button.tsx` → `Button.test.tsx`
- AAA pattern: Arrange, Act, Assert
- Mock external deps (API, DB, file system)
- Use factories for test data
- Test edge cases (null, undefined, empty arrays)

## Performance
- Lazy load routes with React.lazy + Suspense
- Memoize expensive computations with useMemo
- Debounce user input (300ms standard)
- Use React.memo for expensive components
- Profile before optimizing (React DevTools Profiler)

## Code Organization
- Max 200 lines per file (split if larger)
- Single responsibility per function
- Extract magic numbers to constants
- Use meaningful variable names (no single letters except loops)
```

### 12.2 Security Rules

```markdown
---
description: Security guidelines and requirements
alwaysApply: true
---

# Security Guidelines

## Secrets Management
- NEVER hardcode API keys, passwords, tokens
- Use .env for local dev (add to .gitignore)
- Use AWS Secrets Manager / HashiCorp Vault for production
- Rotate secrets regularly (90 days)
- Audit secret access logs monthly

## Input Validation
- Sanitize ALL user input (prevent XSS)
- Use parameterized queries (prevent SQL injection)
- Validate file uploads:
  - Check file type (whitelist, not blacklist)
  - Check file size (<10MB standard)
  - Scan for malware
  - Store in sandboxed location
- Validate content-type headers

## Authentication & Authorization
- Hash passwords with bcrypt (cost factor 12)
- Implement rate limiting:
  - 5 login attempts per 15 minutes
  - 100 API requests per hour per user
- Use HTTPS only (enforce with HSTS header)
- JWT tokens:
  - Short expiry (15 minutes access, 7 days refresh)
  - Store refresh tokens in httpOnly cookies
  - Rotate on use
- Implement CSRF protection (SameSite=Strict cookies)
- Principle of least privilege (minimal permissions)

## Dependencies
- Run `npm audit` weekly (or in CI)
- Auto-update patch versions
- Manually review major/minor updates
- Check for known vulnerabilities before adding new deps
- Use exact versions in package.json (no ^ or ~)

## Error Messages
- Don't leak implementation details
- Log full errors server-side (with context)
- Return generic messages to client: "An error occurred"
- Don't expose stack traces in production
- Use correlation IDs for error tracking

## Data Protection
- Encrypt PII at rest (AES-256)
- Encrypt in transit (TLS 1.3)
- Mask sensitive data in logs (credit cards, SSN, passwords)
- Implement data retention policies
- Support GDPR: right to deletion, data export

If agent suggests code that violates these rules, REJECT and request secure alternative.
```

### 12.3 YOLO Mode Guardrails

**Safe Commands (Allow):**
```
# Testing
npm test, yarn test, pnpm test
vitest, jest, playwright test
npm run test:*

# Building
npm run build, tsc, webpack, vite build
cargo build, go build, make

# Linting
eslint, prettier, cargo clippy
npm run lint, npm run format

# File operations (within project)
mkdir, touch, mv, cp
echo > file (create/overwrite files)

# Git (read-only)
git status, git diff, git log
git show, git branch

# Package management
npm install <package>, npm ci
```

**Dangerous Commands (DENY):**
```
# Destructive
rm -rf, rm -r, rmdir
drop database, truncate table, delete from
kill, killall

# Data exfiltration
curl, wget (unless specific URLs whitelisted)
scp, rsync, ftp

# Remote access
ssh, telnet

# Privilege escalation
sudo, su
chmod +x (requires review)

# Production operations
kubectl delete, kubectl apply (require review)
terraform destroy, terraform apply
docker rm -f, docker system prune
```

**Configuration:**
```
Settings > Agent > YOLO Mode

Prompt: "Allow tests, builds, linting, and file operations. Deny destructive commands, remote access, and production operations."

Allow patterns:
- ^npm (test|run|build|lint)
- ^(tsc|eslint|prettier)
- ^(mkdir|touch|echo)
- ^git (status|diff|log|show)

Deny patterns:
- ^rm -r
- ^(drop|truncate)
- ^(curl|wget|ssh|sudo)
- ^(kubectl|terraform) (apply|destroy|delete)
```

---

## Best Practices Summary

### Team Collaboration Checklist
- [ ] Use shared rules repository (git submodule)
- [ ] Maintain team notepads (architecture, troubleshooting)
- [ ] Pair program with agent (you architect, agent implements)
- [ ] Automate PR reviews with custom commands
- [ ] Weekly sync meetings to share prompts and patterns
- [ ] Track metrics (velocity improvements)

### Domain-Specific Patterns Checklist
- [ ] Use comprehensive templates for Kubernetes manifests
- [ ] Include health checks, security contexts, and resource limits
- [ ] Validate all generated configurations before applying
- [ ] Document deployment procedures
- [ ] Test data pipelines for idempotency
- [ ] Track ML experiments with MLflow or similar

### Runbooks & Guardrails Checklist
- [ ] Create project-specific rules templates
- [ ] Enforce security rules (`alwaysApply: true`)
- [ ] Configure YOLO mode with safe allow list
- [ ] Review and update rules quarterly
- [ ] Document dangerous operations requiring human approval

---

## Next Steps

Continue to [Part 6: Troubleshooting & Reference →](06-troubleshooting-reference.md)

Or return to the [Index](../) for the complete guide navigation.

---

**Part 5 of 6** | [← Part 4](04-context-prompting-strategies) | [Index](../) | [Part 6 →](06-troubleshooting-reference)

