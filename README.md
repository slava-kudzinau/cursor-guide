# Cursor IDE: Complete Technical Guide
## Master Index & Navigation

**Maintainer**: Viachaslau Kudzinau (viachaslau_kudzinau@epam.com)  
**Version**: 2.0  
**Last Updated**: December 2025

---

## 📖 About This Guide

This comprehensive guide covers everything you need to master Cursor IDE 2.0, from fundamentals to advanced enterprise patterns. The guide is split into 6 logically organized parts for easy navigation and reference.

**What's New in Version 2.0:**
- Cursor 2.0 features (Parallel agents, Composer model, Plan Mode)
- Debug Mode & Visual Editor
- Model Context Protocol (MCP) integration
- Latest Context7 MCP documentation
- Screenshot-driven development patterns
- DevOps & Infrastructure workflows
- Real-world productivity benchmarks (2025 data)
- Team collaboration patterns
- Comprehensive troubleshooting guide

---

## 🗂️ Guide Structure

### [Part 1: Fundamentals & Setup](01-cursor-fundamentals.md)
**Essential foundations for all users**

**Sections Covered:**
1. **Concepts & Mental Models** - Understanding Cursor's architecture
   - Agent system prompt structure
   - Tool selection matrix (Tab, Cmd+K, Composer, Debug Mode, etc.)
   - Model selection guide (Cursor Model, Claude, GPT-4o, Gemini)
   - Context windows & indexing pipeline
   - Rules architecture (`.cursorrules` vs `.cursor/rules/*.mdc`)
   - Model Context Protocol (MCP) basics
   - Context7 MCP - live documentation access

2. **Environment & Repo Setup** - Setting up for success
   - Codebase indexing strategies (small repos vs monorepos)
   - Project structure best practices
   - `instructions.md` pattern (universal)
   - Commands feature (reusable team prompts)
   - Monorepo best practices

3. **Core Workflows** - Daily development patterns
   - TDD with Agent (test-first pattern)
   - Refactoring pattern (multi-step, incremental)
   - Debug Mode workflow (hypothesis-driven)
   - Parallel agents (exploring multiple approaches)
   - Infrastructure-as-Code (Terraform, CDK)
   - Lambda/Serverless recipes
   - Progressive enhancement pattern

**Key Takeaways:**
- Tool selection based on task complexity
- Clean context = 30% faster responses
- `@instructions.md` is your single source of truth
- YOLO mode for automated testing loops

---

### [Part 2: Advanced Recipes & Visual Development](02-advanced-recipes-visual-dev.md)
**Complex workflows and visual patterns**

**Sections Covered:**
4. **Advanced Recipes** - Complex multi-phase tasks
   - API migration (REST → GraphQL)
   - Database schema migrations with Prisma
   - Large-scale refactoring (Class → Hooks)
   - Dependency upgrades (major versions)
   - Micro-frontend integration (Module Federation)

5. **Visual Development Patterns** - Screenshot-driven workflows
   - Screenshot-to-code development
   - Comparative visual debugging
   - Visual Editor workflow (NEW in Dec 2025)
   - Reverse engineering UIs

**Key Takeaways:**
- Break migrations into analysis → plan → execute phases
- Use screenshots for exact visual reference
- Visual Editor for real-time styling
- Always validate + review database migrations

---

### [Part 3: DevOps & Backend Architecture](03-devops-backend-patterns.md)
**Infrastructure and backend patterns**

**Sections Covered:**
6. **DevOps & Infrastructure** - Cloud and container workflows
   - Kubernetes with MCP integration (live cluster access)
   - Terraform patterns (multi-stage infrastructure)
   - Docker multi-stage builds (optimized images)
   - CI/CD pipeline templates (GitHub Actions, matrix strategy)
   - Helm chart generation

7. **Backend Architecture Patterns** - Scalable system design
   - Microservices consistency pattern
   - Event sourcing implementation
   - CQRS (Command Query Responsibility Segregation)
   - API gateway pattern

**Key Takeaways:**
- MCP enables live infrastructure debugging
- Follow existing patterns for consistency
- Use event sourcing for audit trails
- Separate read/write with CQRS for scalability

---

### [Part 4: Context Management & Prompting](04-context-prompting-strategies.md)
**Mastering context and advanced prompts**

**Sections Covered:**
8. **Context Management** - Maximizing agent effectiveness
   - Context-first approach (anti-hallucination)
   - Focused context pattern (window hygiene)
   - Conversation reset pattern (when to restart)
   - @ Mention strategy reference (file/folder/codebase/docs/web)
   - Context prioritization (what agent sees first)

9. **Advanced Prompting Strategies** - Expert techniques
   - Chain-of-thought for complex features
   - Multi-tool orchestration (complex workflows)
   - The "over-specification" anti-pattern
   - Model ensemble strategy (different models for different phases)

**Key Takeaways:**
- Analyze before implementing (prevent hallucinations)
- Close irrelevant tabs before agent tasks
- Reset conversation after 3 failed attempts
- Use Claude for planning, Cursor Model for execution, Gemini for review

---

### [Part 5: Team Collaboration & Domain Patterns](05-team-domain-patterns.md)
**Teamwork and specialized workflows**

**Sections Covered:**
10. **Team Collaboration** - Scaling AI across teams
    - Shared rules repository (git submodules)
    - Team notepads (architecture, troubleshooting)
    - Pair programming with agent
    - Automated PR reviews

11. **Domain-Specific Patterns** - Industry workflows
    - Kubernetes manifest generation
    - Data pipelines (dbt + Airflow)
    - ML model training pipelines (XGBoost)

12. **Runbooks & Guardrails** - Safety and standards
    - Project rules templates (TypeScript, Security)
    - YOLO mode guardrails (safe vs dangerous commands)

**Key Takeaways:**
- Centralize rules for team consistency
- Document common issues in team notepads
- Automate PR reviews with custom commands
- Configure YOLO mode with explicit allow/deny lists

---

### [Part 6: Troubleshooting & Reference](06-troubleshooting-reference.md)
**Complete reference guide and solutions**

**Sections Covered:**
13. **Failure Modes & Anti-patterns** - What to avoid
    - Common failure modes (context overflow, infinite loops, code duplication)
    - Anti-patterns deep dive (vague prompts, accepting changes blindly, no version control)

14. **Reference Prompts & Templates** - Copy-paste templates
    - TDD template
    - Refactoring template
    - Infrastructure template
    - Migration template
    - Anti-hallucination template
    - Code review template

15. **Productivity Benchmarks** - Real-world data (2025)
    - Cursor vs Competitors (speed, accuracy, features)
    - Productivity metrics (39% more PRs, 26% faster time-to-PR)
    - Developer velocity improvements (5-10x for boilerplate)
    - Cost analysis and ROI calculations

16. **Troubleshooting Guide** - Solutions to common issues
    - Cursor won't index codebase
    - Agent keeps failing to run commands
    - Tab completion slow or missing
    - Rules not applied
    - Out of memory errors
    - MCP servers not connecting
    - Visual Editor not working

17. **Future-Proofing** - Preparing for the future
    - Upcoming features (confirmed roadmap for 2026)
    - Migration strategies (Copilot, Continue, custom APIs, Cody)
    - Preparing for AI evolution

**Key Takeaways:**
- Commit before AI edits (git is your safety net)
- Review all diffs incrementally
- Use specific prompts, not vague ones
- Cursor is 30% faster than Copilot
- Clean context = 40% fewer errors

---

## 🚀 Quick Start Paths

### For New Users (Start Here!)
1. [Part 1: Fundamentals](01-cursor-fundamentals.md) - Read sections 1-3
2. [Part 4: Context Management](04-context-prompting-strategies.md) - Section 8
3. [Part 6: Reference Templates](06-troubleshooting-reference.md) - Section 14

**Time investment:** 30-45 minutes  
**Outcome:** Understand basics, set up properly, have templates ready

### For Individual Contributors
1. [Part 1: Core Workflows](01-cursor-fundamentals.md) - Section 3
2. [Part 2: Advanced Recipes](02-advanced-recipes-visual-dev.md) - Sections 4-5
3. [Part 4: Prompting Strategies](04-context-prompting-strategies.md) - Section 9
4. [Part 6: Troubleshooting](06-troubleshooting-reference.md) - Section 16

**Time investment:** 2-3 hours  
**Outcome:** Master daily workflows, advanced patterns, troubleshooting

### For DevOps/Platform Engineers
1. [Part 1: Fundamentals](01-cursor-fundamentals.md) - Sections 1-2
2. [Part 3: DevOps & Backend](03-devops-backend-patterns.md) - Sections 6-7
3. [Part 5: Domain Patterns](05-team-domain-patterns.md) - Section 11
4. [Part 6: Templates](06-troubleshooting-reference.md) - Section 14.3

**Time investment:** 2-3 hours  
**Outcome:** Infrastructure workflows, MCP integration, backend patterns

### For Team Leads
1. [Part 1: Setup](01-cursor-fundamentals.md) - Section 2
2. [Part 4: Context Management](04-context-prompting-strategies.md) - Sections 8-9
3. [Part 5: Team Collaboration](05-team-domain-patterns.md) - Sections 10, 12
4. [Part 6: Benchmarks](06-troubleshooting-reference.md) - Section 15

**Time investment:** 3-4 hours  
**Outcome:** Team setup, shared rules, productivity metrics for buy-in

### For Frontend Developers
1. [Part 1: Fundamentals](01-cursor-fundamentals.md) - Sections 1-3
2. [Part 2: Visual Development](02-advanced-recipes-visual-dev.md) - Section 5
3. [Part 4: Prompting](04-context-prompting-strategies.md) - Section 9
4. [Part 6: Templates](06-troubleshooting-reference.md) - Section 14

**Time investment:** 2-3 hours  
**Outcome:** Visual workflows, screenshot-driven development, component patterns

---

## 📋 Quick Reference Tables

### Tool Selection Matrix

| Task Type | Use This | Why |
|-----------|----------|-----|
| Boilerplate, imports | **Tab** (autocomplete) | Fastest (320ms), predictive |
| Single-line fix | **Cmd+K** | Surgical, shows diff |
| Refactor function | **Cmd+K** | Focused scope, fast |
| Multi-file feature | **Cmd+I** (Composer) | Multi-file orchestration |
| Visual UI work | **Visual Editor** | Real-time styling |
| Complex architecture | **Chat (Agent mode)** | Autonomous, cross-cutting |
| Elusive bugs | **Debug Mode** | Hypothesis-driven debugging |
| Exploring approaches | **Parallel Agents** | Best-of-N solutions |

### Model Selection Guide

| Phase | Model | Why |
|-------|-------|-----|
| **Planning/Analysis** | Claude 3.5 Sonnet | Deep reasoning, complex instructions |
| **Execution** | Cursor Model | 4x faster, optimized for coding |
| **Review** | Gemini 2.5 Flash | 2M token context, full codebase review |
| **Speed over accuracy** | GPT-4o | Fast boilerplate generation |

### Context @ Mention Strategy

| Scope | Syntax | Use Case |
|-------|--------|----------|
| **Single file** | `@src/auth.ts` | Focused edits |
| **Folder** | `@src/components/` | Cross-component refactors |
| **Codebase** | `@codebase` | Discovering patterns |
| **Docs** | `@react-docs.org` | Static documentation |
| **Web** | `@web` | Latest news/updates |
| **Context7** | Auto-invoked | Version-specific library docs |
| **Auto-context** | `@Recommended` | Quick starts |

### Productivity Benchmarks

| Metric | Improvement | Source |
|--------|-------------|---------|
| **PRs merged** | +39% | University of Chicago (2025) |
| **Time-to-PR** | -26% | University of Chicago (2025) |
| **Onboarding time** | -40% | Fortune 500 companies |
| **Boilerplate code** | 5-10x faster | Real-world measurements |
| **Refactoring** | 3-5x faster | Real-world measurements |
| **Debugging** | 2-3x faster | With Debug Mode |
| **Documentation** | 10x faster | README, API docs |

---

## 🎯 Learning Milestones

### Week 1: Basics
- [ ] Set up `.cursorrules` or `.cursor/rules/`
- [ ] Create `instructions.md` for your project
- [ ] Configure `.cursorignore` properly
- [ ] Master Tab, Cmd+K, Cmd+I basics
- [ ] Write first 5 prompts with good context

**Checkpoint:** Can complete simple tasks 2-3x faster

### Week 2-3: Intermediate
- [ ] Use Composer for multi-file refactors
- [ ] Implement TDD pattern with agent
- [ ] Set up YOLO mode for automation
- [ ] Create first custom command
- [ ] Use Debug Mode successfully

**Checkpoint:** Handle complex tasks independently

### Week 4+: Advanced
- [ ] Run parallel agents
- [ ] Use MCP for live infrastructure access
- [ ] Implement team notepads
- [ ] Create shared rules repository
- [ ] Train team members

**Checkpoint:** 30-40% velocity improvement, team adoption

---

## 🔧 Common Issues (Quick Fixes)

| Issue | Quick Fix | Full Details |
|-------|-----------|--------------|
| **Slow responses** | Close tabs, clean `.cursorignore` | [Part 4, Section 8.3](04-context-prompting-strategies.md#83-focused-context-pattern) |
| **Hallucinations** | Use context-first approach | [Part 4, Section 8.2](04-context-prompting-strategies.md#82-context-first-approach-anti-hallucination) |
| **Indexing fails** | Check `.cursorignore`, restart indexing | [Part 6, Section 16.1](06-troubleshooting-reference.md#161-cursor-wont-index-codebase) |
| **Commands fail** | Enable YOLO mode, use non-interactive flags | [Part 6, Section 16.2](06-troubleshooting-reference.md#162-agent-keeps-failing-to-run-commands) |
| **Rules ignored** | Check syntax, restart Cursor | [Part 6, Section 16.4](06-troubleshooting-reference.md#164-rules-not-applied) |
| **MCP not connecting** | Verify env vars, check OAuth | [Part 6, Section 16.6](06-troubleshooting-reference.md#166-mcp-servers-not-connecting) |

---

## 💡 Best Practices Summary

### Daily Workflow Checklist
- [ ] Close irrelevant tabs before starting
- [ ] Reference `@instructions.md` in prompts
- [ ] Commit before major agent changes
- [ ] Review diffs before accepting
- [ ] Run tests after agent changes

### Prompt Engineering Checklist
- [ ] Be specific, not vague
- [ ] Provide context (`@files`, `@docs`)
- [ ] Include constraints (what NOT to change)
- [ ] Specify expected output
- [ ] Use incremental approach

### Team Collaboration Checklist
- [ ] Maintain shared rules repository
- [ ] Update team notepads regularly
- [ ] Document effective prompts
- [ ] Code review agent-generated code
- [ ] Track velocity metrics

---

## 📚 Additional Resources

### Official Documentation
- **Cursor Website**: [cursor.com](https://cursor.com)
- **Cursor Docs**: [cursor.com/docs](https://cursor.com/docs)
- **Context7 MCP**: Latest documentation integrated throughout this guide

### Community
- **Discord**: [cursor.com/discord](https://cursor.com/discord)
- **Twitter**: [@cursor](https://twitter.com/cursor)
- **GitHub**: [github.com/getcursor](https://github.com/getcursor)

### This Guide
- **Version**: 2.0 (December 2025)
- **Maintainer**: Viachaslau Kudzinau
- **Email**: viachaslau_kudzinau@epam.com
- **Feedback**: Open issues or contact maintainer

---

## 🗺️ Navigation

**Start reading:** [Part 1: Fundamentals & Setup →](01-cursor-fundamentals.md)

**All parts:**
1. [Fundamentals & Setup](01-cursor-fundamentals.md)
2. [Advanced Recipes & Visual Development](02-advanced-recipes-visual-dev.md)
3. [DevOps & Backend Architecture](03-devops-backend-patterns.md)
4. [Context Management & Prompting](04-context-prompting-strategies.md)
5. [Team Collaboration & Domain Patterns](05-team-domain-patterns.md)
6. [Troubleshooting & Reference](06-troubleshooting-reference.md)

---

## 📝 Version History

**v2.0 (December 2025) - Major Update:**
- Complete restructure into 6 logical parts
- Added Cursor 2.0 features (Parallel agents, Composer model, Plan Mode)
- Added Debug Mode & Visual Editor documentation
- Integrated latest Context7 MCP documentation
- Added DevOps & Infrastructure patterns
- Updated with 2025 productivity benchmarks
- Comprehensive troubleshooting guide
- Future-proofing and migration strategies

**v1.0 (January 2025):**
- Initial release

---

*This is a living document that evolves with Cursor. Bookmark, share, and contribute to keep it up-to-date!*

**Master Index** | [Begin with Part 1 →](01-cursor-fundamentals.md)

