---
title: "Part 6: Troubleshooting & Reference"
nav_order: 7
---

# Cursor IDE: Troubleshooting & Reference
## Part 6 of 6: Complete Reference Guide

**Maintainer**: Viachaslau Kudzinau (viachaslau_kudzinau@epam.com)  
**Version**: 2.0  
**Last Updated**: December 2025

> **📚 Navigation**: [Index](../) | [← Part 5](05-team-domain-patterns) | **Part 6 (Final)**

---

## Table of Contents

13. [Failure Modes & Anti-patterns](#13-failure-modes--anti-patterns)
14. [Reference Prompts & Templates](#14-reference-prompts--templates)
15. [Productivity Benchmarks](#15-productivity-benchmarks)
16. [Troubleshooting Guide](#16-troubleshooting-guide)
17. [Future-Proofing](#17-future-proofing)

---

## 13. Failure Modes & Anti-patterns (EXPANDED)

### 13.1 Common Failure Modes

**1. Context Overflow → Hallucination**
- **Symptom:** Agent invents APIs, functions, libraries that don't exist
- **Cause:** Too much noise in context (large files, irrelevant imports, cluttered conversation)
- **Fix:** 
  - Narrow scope with @file mentions
  - Update `.cursorignore` to exclude non-code assets
  - Close irrelevant tabs
  - Start fresh conversation if context cluttered
- **Prevention:** Clean context hygiene (see Part 4)

**2. Infinite Loop (Edit Cycles)**
- **Symptom:** Agent proposes fix → breaks test → proposes fix → breaks test (repeats)
- **Cause:** Ambiguous requirements, flaky tests, or LLM confusion
- **Fix:**
  - STOP the agent immediately
  - Clarify requirements in new conversation
  - Fix flaky tests manually first
  - Provide explicit constraints
- **Prevention:** Clear requirements + stable tests before starting

**3. Code Duplication**
- **Symptom:** Agent re-generates existing code (forgets prior suggestions)
- **Cause:** Context window limit exceeded, agent loses track of changes
- **Fix:**
  - Use `@codebase` to pull existing code into context
  - Split task into smaller chunks
  - Commit after each successful change
- **Prevention:** Incremental changes with git commits

**4. Unintended Changes**
- **Symptom:** Agent modifies files outside requested scope
- **Cause:** Overly broad prompt, agent "helpfully" fixes perceived issues
- **Fix:** Use explicit constraints in prompt
  ```
  ONLY modify @users.ts
  Do NOT touch @schema.prisma
  Do NOT modify any test files
  ```
- **Prevention:** Specific file mentions + constraints

**5. Terminal Commands Fail / Hang**
- **Symptom:** Agent runs command → infinite loading or skipped
- **Cause:** Command requires user input, or runs in watch mode
- **Fix:** Specify non-interactive mode
  ```
  Run `npm test --run` (single run, no watch mode)
  Run `npm test --watchAll=false`
  ```
- **Prevention:** Always specify non-interactive flags

**6. Outdated Dependencies / APIs**
- **Symptom:** Agent suggests deprecated APIs (React 17 patterns, old Next.js)
- **Cause:** Training data cutoff (models trained on data up to early 2025)
- **Fix:**
  - Add current docs: `@next.js.org/docs/15`
  - Use `@web` for latest info
  - Update rules to specify current versions
- **Prevention:** Keep @docs up to date, specify versions in prompts

**7. Parallel Agents Same Output**
- **Symptom:** All parallel agents generate identical code
- **Cause:** Same prompt, same context
- **Fix:** Vary the prompt slightly for each agent
  ```
  Agent 1: "Implement using approach A (JWT auth)"
  Agent 2: "Implement using approach B (session auth)"
  Agent 3: "Implement using approach C (OAuth)"
  ```

### 13.2 Anti-Patterns Deep Dive

**❌ Vague Prompts**
```
"Make this better"
"Fix this"
"Refactor the code"
"Add features"
```
**Result:** Agent guesses intent, makes arbitrary changes, wastes time.

**✅ Specific Prompts**
```
"Refactor calculateTotal() to handle null prices (return 0) and use reduce() instead of for-loop"

"Fix TypeScript error on line 42: Property 'userId' does not exist on type 'Request'. 
Add userId to custom Request type in src/types/express.d.ts"

"Add loading state to Button component: show spinner icon, disable clicks, use 'cursor-wait'"
```

---

**❌ Accepting All Changes Blindly**
```
Agent proposes 500-line diff → "Accept All" without review
```
**Result:** Broken code, regressions, security issues, wasted debugging time.

**✅ Review Incrementally**
```
Agent proposes diff → Review line-by-line (or file-by-file for large changes)
Accept valid changes → Reject invalid → Ask agent to fix → Repeat
Use git diff for final review before committing
```

---

**❌ Using Agent for Everything**
```
Need to add import → Ask agent
Rename variable → Ask agent
Fix typo → Ask agent
```
**Result:** Slow workflow, context pollution, agent fatigue (and yours).

**✅ Use Right Tool for Job**
```
Tab → Autocomplete boilerplate, imports
Cmd+K → Rename, small edits, single-line fixes
Agent → Multi-file refactors, new features, complex changes
IDE features → Refactoring, find/replace, code navigation
```

---

**❌ No Version Control**
```
Agent refactors 10 files → Something breaks → No way to rollback
Iterate many times → Lose track of what changed → Can't revert
```
**Result:** Hours of manual recovery, lost work, frustration.

**✅ Commit Before AI Edits**
```bash
# Before agent makes changes
git add .
git commit -m "pre-refactor checkpoint"

# Run agent
[agent makes changes]

# Review changes
git diff

# If good:
git add .
git commit -m "refactor: agent-assisted migration to Fastify"

# If bad:
git reset --hard HEAD  # instant rollback
```

---

**❌ No Validation**
```
Agent generates code → Assume it works → Deploy → Breaks in production
```

**✅ Always Validate**
```
Agent generates code
→ Review code
→ Run tests
→ Run locally
→ Check for regressions
→ THEN commit
→ Deploy to staging first
→ Verify in staging
→ THEN production
```

---

## 14. Reference Prompts & Templates

### 14.1 TDD Template
```
@instructions.md

Write tests first, then implementation, then run tests until passing.

Context:
- Framework: [Vitest/Jest/Playwright]
- Coverage target: >80%
- Edge cases: [null, undefined, empty arrays, large inputs, special characters]

Task: [Feature description]

Process:
1. Generate comprehensive test suite
   - Happy path
   - Edge cases
   - Error conditions
   - Boundary values
2. Wait for test review and approval
3. Implement to pass tests
4. Run tests (YOLO mode enabled)
5. Fix failures iteratively
6. Report when all pass + coverage met

Expected outputs:
- tests/ directory with test files
- src/ directory with implementation
- Test run output (all passing)
- Coverage report (>80%)
```

### 14.2 Refactoring Template
```
@instructions.md @target-files

Refactoring task: [Description]

Step 1: Analyze
Review code and identify:
- Code smells (long functions, duplication, tight coupling)
- Performance issues (N+1 queries, unnecessary re-renders, memory leaks)
- Tech debt (deprecated APIs, unsafe patterns)
- Test coverage gaps

Generate analysis report.
Wait for review.

Step 2: Plan
Propose refactoring strategy:
- High-level approach (extract service, add abstraction, etc.)
- Break into 3-5 atomic steps
- Dependencies between steps
- Risk assessment (breaking changes, data migration)
- Estimated effort per step

Wait for approval.

Step 3: Execute
Implement step 1 only.
- Make changes
- Show diff
- Update tests if needed
- Run tests
Wait for approval before step 2.

Step 4: Validate
- Run full test suite
- Check for regressions
- Verify performance not degraded
- Update documentation

Step 5: Repeat
Continue with remaining steps one at a time.

Constraints:
- [Maintain backward compatibility / No breaking changes]
- [Keep existing tests passing]
- [No changes to public API]
```

### 14.3 Infrastructure Template
```
@instructions.md @terraform-docs

Create [Terraform/CDK/Pulumi] for:

Resources:
- [Resource 1]
- [Resource 2]
- [Resource 3]

Requirements:
- Security: least privilege, encryption at rest/transit
- Cost: use smallest viable instance types, enable autoscaling
- Tagging: Environment, Project, Owner, CostCenter
- Networking: VPC, subnets, security groups
- Monitoring: CloudWatch alarms, logging enabled
- Backup: automated backups, retention policy

Generate:
- main.tf / main.ts (primary resource definitions)
- variables.tf / config.ts (all parameters)
- outputs.tf / exports.ts (important resource IDs/ARNs)
- examples/ (basic usage example)
- README.md (usage instructions, architecture diagram)

Follow best practices:
- [AWS Well-Architected / GCP Best Practices / Azure WAF]
- Remote state (S3 + DynamoDB / Terraform Cloud)
- Modules for reusability
- Consistent naming convention
```

### 14.4 Migration Template
```
@instructions.md @codebase

Migration: [Old Tech] → [New Tech]

Pre-flight:
1. Review breaking changes (@docs or @web)
2. Identify incompatible patterns in @codebase
3. Estimate scope:
   - Number of files affected
   - Complexity rating (Low/Medium/High)
   - Estimated time
4. List dependencies that need updates
5. Identify risks and mitigation strategies

Generate pre-flight report.
Wait for approval.

Migration Plan:
1. Update dependencies (package.json)
2. Run automated codemods (if available)
3. Manually fix remaining issues
   - List specific files and changes needed
4. Update tests
5. Update documentation
6. Deploy to staging
7. Full QA testing
8. Deploy to production

Rollback Plan:
- Git commit after each major step
- Document manual changes
- Prepare rollback script
- Database migrations reversible

Execute step-by-step.
Run tests after each step.
Stop if any step fails.
```

### 14.5 Anti-Hallucination Template
```
@instructions.md

You are a senior [Stack] engineer. 
Your goal: Generate accurate, production-ready code WITHOUT hallucinations.

RULES (NON-NEGOTIABLE):
1. ONLY use provided context (@files, @docs)
2. If information is unclear or missing, ASK - do not guess
3. Cite sources when making technical decisions: "Based on @file.ts line 42"
4. No speculation. If task is impossible with given context, explain why and suggest alternatives
5. Do not invent APIs, functions, or libraries that aren't in the codebase or docs

PROCESS (MANDATORY):
1. ANALYZE
   - Summarize current codebase state
   - List relevant files, dependencies, constraints
   - Identify risks and unknowns
   - Ask clarifying questions if needed

2. PLAN
   - Outline detailed approach
   - Break into 3-5 atomic steps
   - Define success criteria for each step
   - List edge cases to handle

3. VERIFY
   - Cross-check plan against @docs and existing code
   - List all assumptions explicitly
   - Confirm patterns match existing codebase
   - Identify any potential conflicts

4. IMPLEMENT
   - Generate code with inline comments explaining non-obvious logic
   - Follow existing code style and patterns exactly
   - Include error handling for all edge cases
   - No placeholder code - everything must be functional

5. TEST
   - Write comprehensive tests (unit + integration)
   - Cover happy path and edge cases
   - Run tests and show results

6. REVIEW
   - Self-check for hallucinations (invented APIs, functions, patterns)
   - If any found, STOP and clarify with user
   - Verify all imports exist
   - Confirm all dependencies available

Task: [Specific feature description]

Context: [Relevant files, docs, constraints]

Begin with ANALYZE phase.
```

### 14.6 Code Review Template
```
@instructions.md @rules/pr-review @diff

Review this PR/branch:

1. Correctness
   - Does code do what it claims?
   - Are edge cases handled?
   - Any logical errors?
   - Proper error handling?

2. Tests
   - Are new features tested?
   - Edge cases covered? (null, empty, large inputs, concurrent access)
   - Integration tests for APIs?
   - E2E tests for critical flows?
   - Test coverage >80%?

3. Performance
   - Any N+1 queries? (check database calls)
   - Unnecessary loops or computations?
   - Memory leaks? (event listeners, intervals)
   - Bundle size impact? (for frontend)

4. Security
   - Input validation?
   - SQL injection prevention?
   - XSS prevention?
   - CSRF protection?
   - Secrets exposed?
   - Auth on protected routes?

5. Maintainability
   - Clear naming?
   - Comments for complex logic?
   - Follows project conventions?
   - Reasonable file sizes?
   - Proper error messages?

6. Style
   - Follows style guide? (@rules/typescript-standards)
   - Consistent formatting?
   - Proper imports organization?

Output format:
## Critical Issues (must fix)
- [issue description + suggested fix]

## Major Issues (should fix)
- [issue description + suggested fix]

## Minor Issues (nice to have)
- [issue description + suggested fix]

## Approval Status
- [ ] Approve
- [ ] Request Changes
- [ ] Comment only

Run tests: [test command]
Check coverage: [coverage command]
```

---

## 15. Productivity Benchmarks (2025 Data)

### 15.1 Cursor vs Competitors

**Speed (SWE-Bench Verified, 500 tasks):**
- **Cursor**: 62.95s avg per task
- GitHub Copilot (GPT-4): 89.91s avg per task
- **Winner:** Cursor (30% faster)

**Accuracy (resolution rate):**
- Cursor: 51.7% (258/500 tasks)
- **GitHub Copilot**: 56.5% (283/500 tasks)
- **Winner:** Copilot (slightly more correct)

**Interpretation:** Cursor trades slight accuracy for significant speed gains.

---

**Autocomplete Latency:**
- **Cursor Tab**: ~320ms (custom 8B model)
- GitHub Copilot: ~890ms
- **Winner:** Cursor (feels more responsive)

---

**Context Window:**
- Cursor: ~8,000 lines (standard)
- GitHub Copilot: ~8,000 lines
- **Gemini in Cursor**: 100,000+ lines (massive context)
- **Winner:** Cursor (with Gemini option for large codebases)

---

**Multi-File Edits:**
- **Cursor Composer**: Native support, visual diff, parallel agents
- GitHub Copilot Edits: Beta, often slower, limited to sequential
- **Winner:** Cursor

---

**Project-Wide Understanding:**
- **Cursor**: Indexes entire repo, semantic search, MCP integration
- GitHub Copilot: Focused on open files + imports
- **Winner:** Cursor (better for monorepos, cross-cutting changes)

---

**Verdict:**
- **Cursor**: Best for large codebases, multi-file refactors, speed, visual development
- **Copilot**: Best for small tasks, inline suggestions, tight GitHub integration
- **Real-world**: Use Cursor for complex projects, Copilot as fallback for simple edits

### 15.2 Productivity Metrics (Real-World Data)

**University of Chicago Study (2025):**
- **39% more PRs merged** after adopting Cursor Agent mode
- **Median time-to-PR reduced by 26%**
- **Onboarding time reduced by 40%** for new engineers

**Developer Velocity (Before vs After):**
- Boilerplate code (CRUD APIs, tests): **5-10x faster**
- Refactoring (large-scale changes): **3-5x faster**
- Debugging (with Debug Mode): **2-3x faster**
- Documentation (README, API docs): **10x faster**
- Learning new codebases: **4-6x faster**

**Team Adoption (Fortune 500 companies):**
- **50%+ of Fortune 500** use Cursor (mid-2025)
- **$1B+ ARR, $10B valuation** (December 2025)
- Average team sees ROI within 3 months

**Time Investment:**
- Learning curve: **2-4 weeks to proficiency**
- Most effective: pair programming (senior + junior + agent)
- ROI positive after ~20 hours of use

**Cost Analysis:**
- Pro subscription: $20/month
- Pro+: $60/month (3x usage, background agents)
- Ultra: $200/month (20x usage, priority features)
- Typical productivity gain: **30-40% faster development**
- Break-even: ~4 hours saved per month

**Context Window Impact (Measured):**
- Clean context (good .cursorignore): **30% faster responses**
- Focused @mentions vs @codebase: **2x faster** for specific tasks
- Fresh conversation vs cluttered: **40% fewer errors**

---

## 16. Troubleshooting Guide

### 16.1 Cursor Won't Index Codebase

**Symptoms:** @codebase returns "No results found" or search is empty

**Solutions:**
1. Check `.cursorignore` → ensure not ignoring all files
   ```bash
   # Bad .cursorignore (ignores everything)
   *
   
   # Good .cursorignore
   node_modules/
   dist/
   .git/
   ```

2. Restart indexing:
   - `Cmd+Shift+P` > "Reindex Codebase"
   - Or: Settings > Indexing & Docs > Clear Cache > Reindex

3. Check storage space:
   - Cursor uses ~100MB per 10k files
   - Clear old indexes: Settings > Indexing > Clear Cache

4. Disable Privacy Mode if enabled:
   - Settings > Privacy > Privacy Mode (blocks cloud indexing)
   - With Privacy Mode: only local embeddings (slower search)

5. Check file types:
   - Settings > Indexing > View included files
   - Ensure your file extensions are included

### 16.2 Agent Keeps Failing to Run Commands

**Symptoms:** Terminal commands never execute, timeout, or skip

**Solutions:**
1. Enable YOLO Mode:
   - Settings > Agent > YOLO Mode
   - Add commands to allow list

2. Check command in deny list:
   - Settings > Agent > YOLO Mode > Deny List
   - Remove if safe to execute

3. Use non-interactive commands:
   ```bash
   # BAD (requires interaction)
   npm test  # might run in watch mode
   
   # GOOD (non-interactive)
   npm test --run
   npm test --watchAll=false
   ```

4. Verify working directory:
   - Agent runs from project root
   - Check with: `pwd` command first

5. Check command syntax:
   - Must be valid shell commands
   - Test manually in terminal first

### 16.3 Tab Completion Slow or Missing

**Symptoms:** Autocomplete takes >2s, doesn't appear, or shows wrong suggestions

**Solutions:**
1. Check network connection:
   - Tab uses cloud model (requires internet)
   - Test with: Settings > Account > Check Connection

2. Verify API key/subscription:
   - Settings > Account > Subscription Status
   - Ensure active subscription

3. Disable on slow connections:
   - Settings > Features > Tab > Disable
   - Use manual Cmd+K instead

4. Clear cache:
   - `Cmd+Shift+P` > "Clear Cache and Reload"

5. Check file type:
   - Tab works best with: TS, JS, Python, Go, Rust
   - Limited support for other languages

6. Context issue:
   - Close irrelevant files (Tab uses open files for context)
   - Ensure imports are correct (Tab uses import context)

### 16.4 Rules Not Applied

**Symptoms:** Agent ignores .cursorrules or .cursor/rules/

**Solutions:**
1. Verify syntax:
   - Rules must be valid Markdown
   - Check for YAML frontmatter errors in .mdc files
   ```markdown
   ---
   description: TypeScript standards
   alwaysApply: false
   ---
   # Rule content here
   ```

2. Check location:
   - `.cursorrules` → project root
   - `.cursor/rules/*.mdc` → in .cursor/rules/ directory

3. Restart Cursor:
   - Rules loaded on startup
   - Cmd+Q (quit) then reopen

4. Be explicit in prompts:
   - Rules are tools, not system prompts
   - Agent decides whether to fetch
   - Force with: "Follow @typescript-standards rule"

5. Check alwaysApply flag:
   ```yaml
   # In .mdc file
   alwaysApply: true  # Agent always sees this rule
   alwaysApply: false # Agent only fetches when relevant
   ```

6. Simplify rules:
   - Too many rules = agent ignores some
   - Keep rules focused and concise
   - Max 5-10 rules per project

### 16.5 Out of Memory Errors

**Symptoms:** Cursor crashes, freezes on large repos, or shows memory warnings

**Solutions:**
1. Aggressive `.cursorignore`:
   ```bash
   # Exclude large directories
   node_modules/
   .next/
   dist/
   build/
   coverage/
   
   # Exclude media
   *.jpg
   *.png
   *.mp4
   *.pdf
   
   # Exclude generated
   **/*.generated.ts
   **/dist/
   **/build/
   ```

2. Close unused tabs:
   - Each open file = context in memory
   - Close all: Cmd+K W (close all editors)

3. Disable auto-indexing:
   - Settings > Indexing > Disable Automatic Indexing
   - Index manually when needed

4. Split monorepo:
   - Open subdirectories as separate workspaces
   - `cursor frontend/` instead of `cursor .`

5. Increase memory limit:
   - Cursor > Settings > Advanced > Max Memory (MB)
   - Increase to 8192 or 16384
   - Restart Cursor

6. Use Privacy Mode (local embeddings):
   - Settings > Privacy > Privacy Mode
   - Uses less memory (no cloud sync)
   - Trade-off: slower search

### 16.6 MCP Servers Not Connecting

**Symptoms:** MCP tools not available, connection errors

**Solutions:**
1. Check MCP configuration:
   ```json
   // .cursor/mcp.json
   {
     "mcpServers": {
       "gdrive": {
         "command": "npx",
         "args": ["-y", "@modelcontextprotocol/server-gdrive"],
         "env": {
           "GDRIVE_CLIENT_ID": "${GDRIVE_CLIENT_ID}"
         }
       }
     }
   }
   ```

2. Verify environment variables:
   - Check env vars are set
   - Use `echo $GDRIVE_CLIENT_ID` to verify

3. OAuth setup:
   - Some MCP servers require OAuth flow
   - Check Settings > MCP > Authorize [Server]

4. Check server compatibility:
   - Cursor MCP version must match server version
   - Update both to latest versions

5. SSH issues (known bug):
   - MCP over SSH has connectivity issues
   - Use local MCP servers when possible
   - Or: Use port forwarding

6. Check logs:
   - Help > Toggle Developer Tools > Console
   - Look for MCP connection errors

### 16.7 Visual Editor Not Working

**Symptoms:** Visual Editor shows blank, can't inspect elements

**Solutions:**
1. Check file type:
   - Visual Editor works with: React, HTML, JSX, TSX
   - Not supported: Vue, Svelte (yet)

2. Verify dev server running:
   - Visual Editor needs live preview
   - Start dev server: `npm run dev`

3. Check port configuration:
   - Settings > Visual Editor > Preview Port
   - Must match dev server port (usually 3000)

4. Browser compatibility:
   - Visual Editor uses Chromium
   - Ensure Chrome DevTools protocol supported

5. Reload preview:
   - Visual Editor > Refresh button
   - Or: Close and reopen Visual Editor

---

## 17. Future-Proofing

### 17.1 Upcoming Features (Confirmed Roadmap)

**Near-term (Q1 2026):**
- **Cursor for Teams**: Centralized rules, usage analytics, admin controls
- **Enhanced MCP support**: More servers, better reliability
- **Model customization**: Fine-tune Cursor Model on your codebase
- **Advanced memory**: Persistent context across sessions
- **Mobile app**: Code review, agent triggers on mobile

**Medium-term (Q2-Q3 2026):**
- **VS Code extension**: Cursor features in VS Code (confirmed)
- **Multi-repo support**: Work across multiple repos simultaneously
- **Advanced debugging**: Deeper integration with debuggers
- **Code ownership**: Track agent contributions vs human
- **Custom workflows**: Build reusable agent workflows

**Experimental (Beta access):**
- **Research Mode**: Multi-agent research for complex problems
- **Automated testing**: Agent generates tests for untested code
- **Security scanning**: Built-in vulnerability detection
- **Performance profiling**: Agent identifies bottlenecks

### 17.2 Migration Strategies

**If Cursor pricing changes or company pivots:**

**Option 1: GitHub Copilot (fallback)**
- Export `.cursorrules` → Convert to Copilot instructions
- Keep: Autocomplete, inline edits, GitHub integration
- Lose: Multi-file agent, Composer, codebase indexing, MCP, visual editor

Migration effort: Low (1-2 days)

**Option 2: Continue (open-source alternative)**
- Fork of VS Code with Cursor-like features
- Self-hosted, local models
- Export rules → Import to Continue (similar format)
- Keep: Most agent features, customization
- Lose: Cursor-specific features, speed optimizations

Migration effort: Medium (1 week)

**Option 3: Build custom with APIs**
- Use Anthropic/OpenAI APIs directly
- Replicate Cursor's tool-calling pattern
- Example tools: Aider (CLI), Mentat, GPT Pilot
- Full control, portable rules
- Lose: Polish, UI, integrations

Migration effort: High (2-4 weeks)

**Option 4: Sourcegraph Cody**
- Enterprise-grade code AI
- Massive context windows (100k+ lines)
- Strong codebase understanding
- Migration: Export rules, reindex codebase
- Keep: Similar features, better for large orgs
- Lose: Some Cursor-specific optimizations

Migration effort: Medium (1-2 weeks)

**Best practice:**
- Keep rules in git (version controlled, portable)
- Document agent patterns (not tool-specific)
- Avoid tool-specific features in critical workflows
- Regular exports of project context

### 17.3 Preparing for AI Evolution

**Skills that remain valuable:**
1. **Prompt engineering**: Core skill across all AI tools
2. **Code review**: AI generates, humans validate
3. **Architecture**: Strategic decisions, not tactical
4. **Debugging**: Understanding why, not just what
5. **Domain knowledge**: Business logic AI can't invent

**Team practices:**
- Document agent successes/failures (build institutional knowledge)
- Share effective prompts (team prompt library)
- Regular retrospectives on AI usage (what works, what doesn't)
- Invest in code quality (AI amplifies existing patterns)
- Maintain human code review standards (don't over-rely on agents)

**Long-term bets:**
- **Context will grow**: 10M+ token context windows coming
- **Agents will improve**: More reliable, fewer hallucinations
- **Multi-agent workflows**: Coordinated agents for complex tasks
- **Domain-specific models**: Models trained on your industry
- **Real-time collaboration**: Human + AI pair programming live

---

## Summary: Best Practices Checklist

### ✅ Daily Workflow
- [ ] Start with clean context (close irrelevant tabs)
- [ ] Reference @instructions.md in most prompts
- [ ] Use specific @mentions (not vague @codebase)
- [ ] Commit before major agent changes
- [ ] Review all diffs before accepting
- [ ] Run tests after agent changes
- [ ] Use YOLO mode for test/build automation

### ✅ Agent Prompts
- [ ] Be specific (not vague)
- [ ] Provide context (@files, @docs)
- [ ] Include constraints (what NOT to change)
- [ ] Specify expected output (files, format)
- [ ] Include validation steps (tests, commands)
- [ ] Use incremental approach (step-by-step)

### ✅ Context Management
- [ ] Maintain instructions.md (single source of truth)
- [ ] Keep .cursorignore updated (exclude noise)
- [ ] Update notepads regularly (architecture, troubleshooting)
- [ ] Reset conversations when cluttered (>20 messages)
- [ ] Use @Recommended for quick context

### ✅ Code Quality
- [ ] Always include tests in prompts
- [ ] Follow existing patterns (@existing-files as reference)
- [ ] Run linters after agent changes
- [ ] Check for security issues (no hardcoded secrets)
- [ ] Verify performance (no N+1 queries)

### ✅ Team Collaboration
- [ ] Share effective prompts (team prompt library)
- [ ] Use shared rules (.cursor/rules/)
- [ ] Update troubleshooting notepad (common issues)
- [ ] Document agent failures (learn from mistakes)
- [ ] Code review agent-generated code (don't blindly accept)

### ✅ Troubleshooting
- [ ] Clean context = faster responses
- [ ] Specific prompts = better results
- [ ] Fresh conversation = fewer errors
- [ ] Git commits = easy rollback
- [ ] Test validation = catch regressions early

---

## Final Recommendations

**For Maximum Productivity:**

1. **Master the basics first**: Tab → Cmd+K → Chat → Composer (in that order)
2. **Maintain instructions.md**: Single source of truth, reference in every prompt
3. **Write project-specific rules early**: Encode conventions once, benefit forever
4. **Use YOLO mode**: Automate testing loops, save time
5. **Always commit before AI edits**: Git is your safety net
6. **Review all diffs**: Never accept blindly, verify changes
7. **Start with analysis**: "Analyze first, then implement" prevents hallucination
8. **Use incremental approach**: Small steps > big refactors
9. **Keep context clean**: Close tabs, update .cursorignore, reset cluttered conversations
10. **Validate everything**: Tests + manual checks before committing

**For Team Adoption:**

1. **Shared rules repository**: Consistency across engineers (git submodule)
2. **Weekly sync meetings**: Share prompts, patterns, failures
3. **Pair programming**: Senior + junior + agent (most effective learning)
4. **Automated PR reviews**: First-pass with agent, human final review
5. **Track metrics**: Measure velocity improvements (PRs/week, time-to-merge)
6. **Invest in training**: 2-4 weeks to proficiency, provide resources
7. **Build prompt library**: Document effective prompts for common tasks
8. **Celebrate wins**: Share successful agent-assisted features
9. **Learn from failures**: Post-mortems on agent mistakes
10. **Iterate on process**: Continuous improvement of AI workflows

**Red Flags (Stop and Reassess):**

⚠️ Agent generates >500 line diff without tests  
⚠️ Agent invents APIs or libraries (hallucination)  
⚠️ Commands hang or fail repeatedly (bad prompt or setup)  
⚠️ Same error loops 3+ times (agent confused, needs new approach)  
⚠️ You can't understand generated code (prompt too vague or code too complex)  
⚠️ Tests fail after agent changes (regressions)  
⚠️ Agent makes unintended changes (prompt too broad)

---

## Conclusion

Cursor 2.0 represents a significant evolution in AI-assisted development. The combination of:
- **Fast autocomplete** (Tab)
- **Surgical edits** (Cmd+K)
- **Multi-file orchestration** (Composer)
- **Visual development** (Visual Editor)
- **Live infrastructure access** (MCP)
- **Parallel exploration** (Multiple agents)
- **Hypothesis-driven debugging** (Debug Mode)

...creates a powerful development environment that can accelerate productivity by 3-10x when used effectively.

**Key insight:** Cursor is a force multiplier, not a replacement for engineering judgment. The most successful teams use AI for velocity and humans for strategy. Master the tool, encode your team's patterns in rules, maintain clean context, and always validate AI output.

**The future is collaborative:** Human architects and strategists working with AI implementers and code generators. Those who master this collaboration will build faster, with higher quality, than those who don't.

**Remember:** Good prompts + good context + good validation = great results.

---

**Version History:**
- v2.0 (Dec 2025): Major update for Cursor 2.0, added visual development, MCP, parallel agents, Debug Mode
- v1.0 (Jan 2025): Initial release

**Contributors:**
- Viachaslau Kudzinau (primary author)
- Community feedback and real-world usage

**Feedback:** Open issues/PRs or contact maintainer via email.

---

**Part 6 of 6 (Complete)** | [← Part 5](05-team-domain-patterns) | [Return to Index](../)

*This guide is a living document. As Cursor evolves and new patterns emerge, we'll update it. Bookmark, share, and contribute!*

