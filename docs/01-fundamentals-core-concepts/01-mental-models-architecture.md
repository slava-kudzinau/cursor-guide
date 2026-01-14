---
title: "Section 1: Mental Models & Architecture"
parent: "Part 1: Fundamentals & Core Concepts"
nav_order: 1
---

# Part 1, Section 1: Mental Models & Architecture

**Part of**: [Cursor IDE: Complete Technical Guide](../../README.md)  
**Estimated reading time**: 45 minutes  
**Prerequisites**: None - start here!

---

## 📋 Overview

Before diving into Cursor IDE, you need to understand **how it thinks** and **how you should think about it**. This section builds the mental models that will make everything else click.

**What you'll learn:**
- The Cursor IDE ecosystem and its components
- How different AI models work and when to use each
- The architecture of Tab, Inline Edit, Agent, and Composer
- How codebase indexing and context management work
- Rules, Skills, and Instructions hierarchy
- Skills architecture for reusable workflows
- Hooks for lifecycle control
- Model Context Protocol (MCP)

**Why this matters:** Understanding these concepts prevents frustration and unlocks Cursor's full potential. You'll know *why* something works, not just *how* to use it.

---

## 🧠 The Cursor IDE Ecosystem

### Core Architecture

**Agent System Prompt Structure**

```mermaid
graph TD
    A[System Prompt Claude 3.5 / Cursor Model] -.-> B[Context Assembly]
    B -.-> C[Tool Execution]
    
    A1[Persona: powerful agentic AI] -.-> A
    A2[Communication guidelines] -.-> A
    A3[Tool usage guidelines] -.-> A
    A4[Code change guidelines] -.-> A
    A5[Debugging guidelines] -.-> A
    
    B1[Open files] -.-> B
    B2[Cursor position] -.-> B
    B3[Recently viewed files] -.-> B
    B4[Edit history] -.-> B
    B5[Linter errors] -.-> B
    B6["@mentions codebase, files, docs"] -.-> B
    
    C1[read_file, write_file] -.-> C
    C2[edit_file find/replace] -.-> C
    C3[search_codebase] -.-> C
    C4[run_terminal_command] -.-> C
    C5[create_file, delete_file] -.-> C
    C6[MCP tools external systems] -.-> C
```

**Why this matters:** Cursor isn't a monolithic LLM—it's an orchestration layer that:
- Maintains conversation state but has **no memory** between completions
- Feeds full history + state in each request (context window management critical)
- Uses provider-native tool calling (Anthropic, OpenAI, Gemini formats)
- Runs tools **client-side** (Cursor orchestrates, not the LLM)
- Can connect to external systems via MCP (Model Context Protocol)

---

## 🔧 Tool Selection Matrix

Understanding when to use each Cursor feature is critical for productivity.

### Tab (Autocomplete)
**Fast, predictive code completion**

- Uses custom 8B-param model trained on code patterns
- ~320ms latency (vs GitHub Copilot's 890ms)
- Predicts next edit location
- Auto-imports unresolved symbols (TypeScript/Python)
- **When to use:** Boilerplate, repetitive patterns, known APIs

**Example use cases:**
- Writing import statements
- Completing function signatures
- Generating test boilerplate
- Auto-completing API calls

---

### Cmd+K (Inline Edit)
**Surgical, focused changes**

- Single-file scope, focused edits
- Shows diff (red = removed, green = added)
- Fast response time
- **When to use:** Refactoring a function, renaming, small fixes

**Example use cases:**
- Refactor a function to use async/await
- Rename variables consistently
- Add error handling to existing code
- Convert class to functional component

**Keyboard shortcut:**
- Windows/Linux: `Ctrl+K`
- macOS: `Cmd+K`

---

### Cmd+I (Composer / Agent)
**Multi-file orchestration**

- **NEW in Cursor 2.0:** Cursor's own frontier model (4x faster)
- Multi-file edits with dependency tracking
- Can reference `@codebase`, `@folder`, `@docs`, `@web`
- Built-in Plan Mode for complex tasks
- **When to use:** Large refactors, migrations, architectural changes

**Example use cases:**
- Migrate REST API to GraphQL
- Refactor class components to hooks across multiple files
- Add authentication to entire application
- Generate tests for multiple modules

**Keyboard shortcut:**
- Windows/Linux: `Ctrl+I`
- macOS: `Cmd+I`

---

### Chat (Ask/Edit/Agent Mode)
**Conversational coding**

- **Ask Mode:** Q&A, explanations, planning (read-only)
- **Edit Mode:** Apply changes directly to files
- **Agent Mode:** Autonomous file creation, searches, terminal commands
- **When to use Agent:** Cross-cutting changes, scaffolding, TDD loops

**Example use cases:**
- Ask: "How does authentication work in this codebase?"
- Edit: "Add error handling to all API calls"
- Agent: "Set up Jest testing with coverage reports"

**Keyboard shortcut:**
- Windows/Linux: `Ctrl+L`
- macOS: `Cmd+L`

---

### Debug Mode (NEW)
**Hypothesis-driven debugging**

- Agent generates hypotheses → instruments code → collects logs → fixes
- Works across languages/stacks
- **When to use:** Elusive bugs, race conditions, runtime issues

**Example use cases:**
- Debugging intermittent test failures
- Finding race conditions in async code
- Tracking down memory leaks
- Understanding performance bottlenecks

---

### Parallel Agents (NEW in 2.0)
**Run multiple agents simultaneously**

- Uses git worktrees or remote machines
- Auto-evaluates and picks best solution
- Up to 8 agents in parallel
- **When to use:** Exploring multiple approaches, complex refactors

**Example use cases:**
- Compare different implementation approaches
- Test multiple refactoring strategies
- Explore various architectural patterns
- A/B test different solutions

---

### Visual Editor (NEW Dec 2025)
**Design + code simultaneously**

- Chrome DevTools-style inspection
- Click, drag, style components
- Real-time visual feedback
- **When to use:** UI development, visual debugging

**Example use cases:**
- Adjusting component layouts
- Fine-tuning CSS styles
- Visual debugging of responsive designs
- Rapid UI prototyping

---

## 🎯 Tool Selection Decision Tree

```mermaid
graph TD
    START{"What do you<br/>need to do?"}
    
    START -->|Autocomplete code| TAB["Tab<br/>320ms, predictive"]
    START -->|Single file edit| INLINE["Inline Edit<br/>Cmd+K"]
    START -->|Multi-file feature| COMPOSER["Composer<br/>Cmd+I"]
    START -->|Ask questions| CHAT["Chat<br/>Cmd+L"]
    START -->|Debug issues| DEBUG["Debug Mode"]
    START -->|Explore options| PARALLEL["Parallel Agents"]
    START -->|Visual UI work| VISUAL["Visual Editor"]
    
    style START fill:#e1f5ff,stroke:#0066cc,stroke-width:2px
    style TAB fill:#d4edda
    style INLINE fill:#fff4e1
    style COMPOSER fill:#fff3cd
    style CHAT fill:#cfe2ff
    style DEBUG fill:#ffe4e1
    style PARALLEL fill:#f0e1ff
    style VISUAL fill:#e1f5e1
```

**Trade-off:** Agent/Composer have higher latency but avoid "golf the prompt" cycles. Use Tab for speed, Agent for correctness.

---

## 🤖 Model Selection Guide

### Available Models (2026)

Based on official Cursor documentation, here are the current models:

#### Claude 4.5 Opus
- **Provider:** Anthropic
- **Context:** 200k tokens (default), 200k (max mode)
- **Capabilities:** Agent, Thinking, Image support
- **Best for:** Complex reasoning, deep analysis

#### Claude 4.5 Sonnet (Chat default)
- **Provider:** Anthropic
- **Context:** 200k tokens (default), 1M (max mode)
- **Capabilities:** Agent, Thinking, Image support
- **Best for:** Understanding legacy code, following complex instructions
- **Strong reasoning capabilities**

#### Cursor Composer 1
- **Provider:** Cursor (custom model)
- **Context:** 200k tokens
- **Capabilities:** Agent, Image support
- **Best for:** Multi-file refactors, fast execution
- **4x faster than comparable models**

#### Gemini 3 Flash
- **Provider:** Google
- **Context:** 200k tokens (default), 1M (max mode)
- **Capabilities:** Agent, Thinking, Image support
- **Best for:** Large monorepos, full-context analysis

#### Gemini 3 Pro
- **Provider:** Google
- **Context:** 200k tokens (default), 1M (max mode)
- **Capabilities:** Agent, Thinking, Image support
- **Best for:** Massive codebases, comprehensive reviews

#### GPT-5.1 Codex Max
- **Provider:** OpenAI
- **Context:** 272k tokens
- **Capabilities:** Agent, Thinking, Image support
- **Best for:** Code-specific tasks, fast responses

#### GPT-5.2
- **Provider:** OpenAI
- **Context:** 272k tokens
- **Capabilities:** Agent, Thinking, Image support
- **Best for:** General purpose, balanced performance

#### Grok Code
- **Provider:** xAI
- **Context:** 256k tokens
- **Capabilities:** Agent, Thinking
- **Best for:** Alternative perspective, code analysis

---

### Model Switching Strategy

```mermaid
graph LR
    A[Planning Phase] --> B[Claude 4.5 Sonnet]
    C[Execution Phase] --> D[Cursor Composer 1]
    E[Review Phase] --> F[Gemini 3 Pro]
    
    B --> G[Deep analysis<br/>Complex instructions]
    D --> H[4x faster<br/>Multi-file edits]
    F --> I[1M context<br/>Full codebase]
    
    style A fill:#e1f5ff
    style C fill:#fff4e1
    style E fill:#fff3cd
    style B fill:#d4edda
    style D fill:#d4edda
    style F fill:#d4edda
```

**Recommended workflow:**
1. **Planning phase** → Claude 4.5 Sonnet (deep analysis)
2. **Execution phase** → Cursor Composer 1 (speed)
3. **Review phase** → Gemini 3 Pro (full context)

---

## 📊 Context Windows & Indexing

### Codebase Indexing Pipeline

```mermaid
graph LR
    A[Local Files] --> B[Chunking]
    B --> C[Embedding<br/>OpenAI/custom]
    C --> D[Turbopuffer<br/>vector DB]
    B --> E[Merkle Tree<br/>detects changes every 10min]
    E --> F[Incremental Upload<br/>only changed files]
```

### Key Stats

- ~8,000 lines max context per request (Cursor)
- Cursor Composer: Optimized for coding context
- Gemini 3 Pro: ~100,000+ lines (massive context with 1M tokens)
- Embeddings stored with obfuscated paths + line ranges
- Code **never stored server-side** (only embeddings + metadata)

### Privacy Model

- Embeddings are reversible for short strings (academic attacks exist)
- Cursor uses encryption + obfuscation to mitigate
- Privacy Mode disables cloud indexing, keeps embeddings local

### When @Codebase triggers:

1. Query embedding computed locally
2. Vector search in Turbopuffer (nearest neighbors)
3. Client receives obfuscated paths + line ranges
4. Client reads local files → sends to LLM

**Why Cursor is fast:** No pre-processing wait. Indexing runs in background, queries hit cached embeddings.

---

## 📜 Rules, Skills, and Instructions: The Decision Hierarchy

### Understanding the Three-Tier System

Cursor uses a three-tier system for guiding AI behavior:

1. **Rules** - Always-on guardrails and constraints
2. **Skills** - Optional, reusable workflows
3. **Instructions** - Project context and documentation

```mermaid
graph TD
    A[User Request] --> B{Check Rules}
    B -->|Rules Apply| C[Enforce Constraints]
    C --> D{Relevant Skill?}
    B -->|No Rules| D
    D -->|Skill Found| E[Execute Skill Workflow]
    D -->|No Skill| F[Use Instructions]
    E --> G[Generate Response]
    F --> G
    
    style B fill:#ffe4e1
    style D fill:#fff4e1
    style F fill:#e1f5ff
    style G fill:#d4edda
```

### Decision Framework: Rules vs Skills vs Instructions

**Before creating any guidance file, ask:**

| Question | Answer → Type | Purpose |
|----------|---------------|---------|
| **Must this ALWAYS hold?** | → **Rule** | Non-negotiable constraints |
| **Is this a reusable workflow?** | → **Skill** | Optional productivity pattern |
| **Does this explain context/intent?** | → **Instructions** | Understanding and background |

**Examples:**

| Guidance | Type | Reasoning |
|----------|------|-----------|
| "Use TypeScript strict mode" | Rule | Must always apply |
| "Refactor API endpoints workflow" | Skill | Useful sometimes, optional |
| "Why we chose Fastify over Express" | Instructions | Context and rationale |
| "Never commit secrets" | Rule | Critical safety constraint |
| "Test-fix-iterate loop" | Skill | Productivity pattern |
| "Current sprint goals" | Instructions | Project context |

---

## 📜 Rules Architecture

### Current Format (2026)

Cursor rules have evolved to a more powerful structure:

- **Project Rules** (`.cursor/rules/*.md`) - **RECOMMENDED** ✅
  - Any `.md` file in `.cursor/rules/` directories
  - Frontmatter metadata controls when rules apply
  - Version-controlled and scoped to your codebase
  - Cursor recursively loads all `.md` files
  
- **AGENTS.md** - Simple alternative
  - Plain markdown file in project root
  - No metadata or complex configuration
  - Supports nested files in subdirectories
  
- **User Rules** - Global preferences in Cursor Settings
  - Apply across all projects
  - Perfect for personal coding style
  
- **Team Rules** - Dashboard-managed (Team/Enterprise)
  - Centrally managed by admins
  - Can be enforced for all team members
  - Plain text format
  
- **`.cursorrules`** - **LEGACY** ⚠️
  - Single file format (will be deprecated)
  - Migrate to Project Rules or AGENTS.md

### How Rules Work (Non-Obvious)

**Important:** Rules are included in model context based on their type:

| Rule Type | When Applied |
|-----------|--------------|
| **Always Apply** | Included in every chat session |
| **Apply Intelligently** | Agent decides based on `description` field |
| **Apply to Specific Files** | When file matches `globs` pattern |
| **Apply Manually** | When @-mentioned in chat (e.g., `@typescript`) |

### Rule Precedence

When multiple rules apply, they are merged in this order:
1. **Team Rules** (highest priority)
2. **Project Rules**
3. **User Rules** (lowest priority)

### Implications

**Write rules as encyclopedia articles, not commands:**

❌ **DON'T:**
```
You are a senior engineer using React 18.
Always use functional components.
```

✅ **DO:**
```markdown
---
description: "React 18 patterns and best practices"
alwaysApply: false
---

# React 18 Patterns

## Component Architecture
- Use functional components with hooks
- Avoid class components (legacy pattern)
- Use TypeScript for type safety

## State Management
- useState for local state
- useContext for shared state
- React Query for server state
```

### Team Rules (Team/Enterprise Plans)

- Managed from [Cursor dashboard](https://cursor.com/dashboard?tab=team-content)
- Can be enforced (required for all team members)
- Plain text format (no frontmatter)
- Apply across all repositories for the team

### Rule Management (Monorepos)

Project Rules and AGENTS.md can be nested:

```
project/
  .cursor/rules/        # Root-level rules
  AGENTS.md             # Global instructions
  frontend/
    .cursor/rules/      # Frontend-specific rules
    AGENTS.md           # Frontend instructions
  backend/
    .cursor/rules/      # Backend-specific rules
    AGENTS.md           # Backend instructions
```

**Benefits:**
- Granular control per area of codebase
- More specific instructions take precedence
- No need for separate workspaces

---

## 🎨 Skills Architecture

**Skills** are optional, reusable workflows that agents can invoke when relevant. Unlike rules (always on), skills activate conditionally based on task context.

### What Are Skills?

Skills are **micro-workflows** stored in `.cursor/skills/<skill-name>/SKILL.md` that:
- Are **optional** - Agent chooses when to use them
- Are **reusable** - Work across different contexts
- Are **composable** - Can be combined with other skills
- Have **clear activation conditions** - Agent knows when they're relevant

### Skills Structure

```
.cursor/skills/
  test-loop/
    SKILL.md              # Skill definition
  scaffold-package/
    SKILL.md
  api-migration/
    SKILL.md
  code-review/
    SKILL.md
```

### Skill File Format

**`.cursor/skills/test-loop/SKILL.md`:**
```markdown
---
name: "test-loop"
description: "Run tests, fix failures, iterate until all pass"
commands:
  - "test-loop"
  - "tdd"
---

# Test-Fix-Iterate Loop

## When to Use
- Implementing TDD workflow
- Fixing failing tests
- Ensuring test coverage

## Workflow
1. Run test suite
2. If failures exist:
   - Analyze failure messages
   - Identify root cause
   - Apply fix
   - Re-run tests
3. Repeat until all tests pass
4. Report final status

## Success Criteria
- All tests passing
- No warnings in output
- Code coverage maintained or improved

## Example Usage
```bash
npm test
# Analyze failures
# Fix code
npm test
# Repeat until green
```
```

### Skills vs Rules: Key Differences

| Dimension | Rules | Skills |
|-----------|-------|--------|
| **Scope** | Global, always active | Situational, conditional |
| **Activation** | Automatic | Agent decides or user invokes |
| **Optional** | ❌ No - always enforced | ✅ Yes - used when relevant |
| **Best for** | Guardrails, standards | Productivity workflows |
| **Examples** | "Use TypeScript strict", "Never commit secrets" | "TDD loop", "Refactor workflow" |
| **Override** | Cannot be ignored | Can be skipped if not relevant |

### Agent Skill Selection

**How agents choose skills:**

1. **Automatic selection** - Agent analyzes request and goal
   - Matches goal to skill descriptions
   - Considers current context
   - Invokes relevant skills automatically

2. **Manual invocation** - User explicitly requests
   - Via slash commands: `/test-loop`
   - In prompts: "Use the test-loop skill"
   - In follow-ups: "Apply code-review skill"

3. **Chaining** - Multiple skills in sequence
   - Agent can combine skills
   - Example: scaffold-package → test-loop → code-review

### Example Skills

**1. Test-Loop Skill**
```markdown
Purpose: TDD workflow automation
Workflow: test → fix → test → repeat
Success: All tests green
```

**2. Scaffold-Package Skill**
```markdown
Purpose: Create new workspace package
Workflow: directory → package.json → tsconfig → README → tests
Success: Package compiles and has initial tests
```

**3. API Migration Skill**
```markdown
Purpose: Migrate REST endpoints to GraphQL
Workflow: analyze → plan → schema → resolvers → tests → docs
Success: Feature parity with original REST API
```

**4. Code Review Skill**
```markdown
Purpose: Automated code review checklist
Workflow: security → performance → tests → style → docs
Success: All checks pass, issues documented
```

### When to Create a Skill

Create a skill when you have a:
- ✅ **Multi-step workflow** that's repeatable
- ✅ **Common pattern** used across projects
- ✅ **Optional process** that doesn't always apply
- ✅ **Clear success criteria** to know when it's done

Don't create a skill for:
- ❌ **Always-on constraints** → Use Rules instead
- ❌ **One-time tasks** → Use direct prompts
- ❌ **Project-specific context** → Use Instructions
- ❌ **Simple one-step actions** → Use direct commands

---

## 🪝 Hooks: Lifecycle Control

**Hooks** extend agent control flow by running scripts before/after agent actions. They enable verification loops, safety checks, and stop conditions.

### What Are Hooks?

Hooks are **lifecycle interceptors** that:
- Run at specific points in agent execution
- Can stop or continue agent loops
- Enable verification and validation
- Provide safety guardrails

### Hooks Configuration

**Configuration file:** `.cursor/hooks.json`

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

### Hook Types

#### 1. Stop Hook

Controls when the agent should stop iterating.

**`.cursor/hooks.json`:**
```json
{
  "hooks": {
    "stop": {
      "script": "./scripts/stop-hook.sh",
      "description": "Continue until all tests pass"
    }
  }
}
```

**`./scripts/stop-hook.sh`:**
```bash
#!/bin/bash
# Exit 0 = continue, Exit 1 = stop

# Run tests
npm test --silent

# If tests pass, stop iterating
if [ $? -eq 0 ]; then
  echo "✅ All tests passing - stopping"
  exit 1  # Stop
else
  echo "❌ Tests failing - continuing"
  exit 0  # Continue
fi
```

### Hook Use Cases

#### 1. Verification Loops

**Scenario:** Continue until tests pass

```bash
#!/bin/bash
# stop-hook.sh
npm test
[ $? -eq 0 ] && exit 1 || exit 0
```

**Behavior:** Agent keeps fixing code until tests pass, then stops automatically.

---

#### 2. Safety Checks

**Scenario:** Stop if production environment detected

```bash
#!/bin/bash
# stop-hook.sh
if [ "$ENVIRONMENT" = "production" ]; then
  echo "🚨 Production environment detected - stopping"
  exit 1  # Stop immediately
fi
exit 0  # Continue
```

**Behavior:** Agent immediately halts if it detects production context.

---

#### 3. Quality Gates

**Scenario:** Stop if code coverage drops below threshold

```bash
#!/bin/bash
# stop-hook.sh
coverage=$(npm test --coverage --silent | grep "Statements" | awk '{print $3}' | tr -d '%')

if [ "$coverage" -lt 80 ]; then
  echo "❌ Coverage ${coverage}% < 80% - continuing"
  exit 0  # Continue fixing
else
  echo "✅ Coverage ${coverage}% ≥ 80% - stopping"
  exit 1  # Stop
fi
```

**Behavior:** Agent continues improving code until coverage reaches 80%.

---

#### 4. Build Verification

**Scenario:** Continue until build succeeds

```bash
#!/bin/bash
# stop-hook.sh
npm run build 2>/dev/null
[ $? -eq 0 ] && exit 1 || exit 0
```

**Behavior:** Agent fixes build errors iteratively until successful.

### YOLO Mode + Hooks = Autonomous Iteration

**Traditional YOLO mode:** Agent executes commands automatically but stops after each step.

**YOLO + Hooks:** Agent executes AND loops automatically based on hook logic.

**Example: TDD Loop with Hooks**

```markdown
Prompt: "Implement user authentication with TDD"

Hook: Continue until tests pass

Agent behavior:
1. Write tests
2. Run tests (fail)
3. Implement code
4. Run tests (fail)
5. Fix bugs
6. Run tests (pass) ← Hook stops here
7. Report completion
```

**Without hooks:** Agent would stop at step 2 and wait for user input.  
**With hooks:** Agent loops through steps 2-6 automatically until tests pass.

### Hooks vs Skills vs Rules

| Feature | Rules | Skills | Hooks |
|---------|-------|--------|-------|
| **Purpose** | Constraints | Workflows | Control flow |
| **When applied** | Always | When relevant | At lifecycle points |
| **User control** | No override | Optional invocation | Automatic |
| **Example** | "Use TypeScript" | "TDD workflow" | "Stop when tests pass" |

### Hooks Best Practices

✅ **Do:**
- Keep hooks fast (< 5 seconds)
- Use clear exit codes (0 = continue, 1 = stop)
- Log hook decisions for debugging
- Test hooks independently

❌ **Don't:**
- Make hooks slow (blocks agent)
- Use complex logic (keep hooks simple)
- Forget to handle errors (use set -e)
- Hardcode values (use env vars)

---

## 🔌 Model Context Protocol (MCP)

**NEW in Cursor 2.0:** Standardized protocol for connecting external tools and data sources.

### What is MCP

- Protocol for LLMs to access external systems
- One-click OAuth setup for popular services
- Bidirectional communication (read/write)
- Client-side execution (secure)

### Available MCP Servers (2026)

- **Google Drive**: Search and read documents
- **Slack**: Read channels, send messages
- **GitHub**: Query repos, create issues/PRs
- **Linear**: Manage issues and projects
- **PostgreSQL**: Query databases
- **Kubernetes**: Manage clusters
- **Custom**: Build your own

### MCP vs @Docs

- **@Docs:** Static documentation (indexed once)
- **MCP:** Live data (queries on demand)

### Setup Example

```json
// .cursor/mcp.json
{
  "mcpServers": {
    "gdrive": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-gdrive"],
      "env": {
        "GDRIVE_CLIENT_ID": "${GDRIVE_CLIENT_ID}",
        "GDRIVE_CLIENT_SECRET": "${GDRIVE_CLIENT_SECRET}"
      }
    }
  }
}
```

### Current Limitations

- 40 tool limit per session
- SSH connectivity issues (known bug)
- Some servers require manual OAuth setup

---

## 📚 Context7 MCP - Live Documentation

**Pre-configured in Cursor 2.0** - No setup required.

### What it Provides

- Real-time, version-specific library documentation
- 10,000+ libraries with official docs and code examples
- Two modes: `code` (API reference) and `info` (guides)
- Auto-resolves library names to Context7-compatible IDs

### Usage Patterns

#### Pattern 1: Implicit (Recommended)
```
Implement file upload using AWS SDK v3 S3 client.
Follow latest best practices.
```
Agent automatically uses Context7 for latest docs. Watch for 🔧 tool indicators.

#### Pattern 2: Explicit Version
```
Use React Query v5.62.0 to implement data fetching.
Check Context7 for version-specific breaking changes.
```
Forces Context7 lookup for specific version (training data won't have recent versions).

#### Pattern 3: Migration
```
Migrate from Next.js 14 to 15.
Use Context7 to compare App Router changes between versions.
```
Context7 fetches both versions for comparison.

### Quick Verification

Ask agent to cite version number. If it shows exact recent version (e.g., "v5.62.0"), Context7 was used.

### When Context7 Beats @Docs

- Library updates frequently (React, Next.js, AWS SDK)
- Need specific version (not latest)
- Setup/configuration steps (build tools, CI/CD)
- Training data is >6 months old

### When to Use @Docs Instead

- Custom/internal documentation
- Static content (faster, cached locally)
- Privacy-sensitive projects

---

## 🎯 Key Takeaways

### Tool Selection
- **Tab** for fast autocomplete (320ms)
- **Cmd+K** for single-file edits
- **Cmd+I** for multi-file features
- **Agent** for autonomous work
- **Debug Mode** for elusive bugs
- **Parallel Agents** for exploration

### Model Selection
- **Claude 4.5 Sonnet** for planning and analysis
- **Cursor Composer 1** for fast execution
- **Gemini 3 Pro** for full codebase review

### Context Management
- Clean `.cursorignore` = faster indexing
- Close irrelevant tabs before agent tasks
- Use `@codebase` for semantic search
- Embeddings stored securely, code never leaves machine

### Rules, Skills, and Instructions
- **Rules** (`.cursor/rules/*.md`) - Always-on constraints and guardrails
- **Skills** (`.cursor/skills/*/SKILL.md`) - Optional reusable workflows
- **Instructions** (`INSTRUCTIONS.md`) - Project context and documentation
- Decision framework: "Must always hold?" → Rule, "Useful sometimes?" → Skill, "Explains intent?" → Instructions
- Team Rules managed from dashboard

### Skills Architecture
- Skills are optional workflows that activate conditionally
- Agent selects skills automatically or via slash commands
- Examples: test-loop, scaffold-package, api-migration
- Create skills for repeatable multi-step patterns

### Hooks for Control Flow
- Hooks control agent iteration lifecycle
- Stop hooks determine when agent should stop
- Enable verification loops (continue until tests pass)
- Combine with YOLO mode for autonomous iteration

### MCP Integration
- Connect to external systems (Slack, GitHub, databases)
- Context7 provides live documentation
- Watch for 🔧 tool indicators

---

## 📖 Next Steps

Now that you understand Cursor's architecture and mental models, proceed to:

**[Section 2: Environment & Project Setup →](./02-environment-project-setup.md)**

Learn how to configure Cursor IDE optimally for your projects, set up `instructions.md`, configure rules, and prepare your development environment.

---

**Part 1, Section 1** | [Back to Part 1 Index](./README.md) | [Next: Section 2 →](./02-environment-project-setup.md)
