# claude-box

A Claude Code plugin marketplace containing a complete plan workflow system for creating, validating, and executing implementation plans.

## Installation

Inside Claude Code, run:

```bash
# Add the marketplace
/plugin marketplace add jmagar/claude-box

# Install the plugin
/plugin install claude-box@claude-box
```

Or browse interactively:
```bash
/plugin
```

---

## Architecture

```
/superpowers:write-plan (command)
  ↓
writing-plans (skill)
  ↓
plan-writer (agent) ──────┐
                          │
                          ↓
                    [CREATES PLAN]
                          │
                          ↓
/validate-plan (command)  │
  ↓                       │
validating-plans (skill)  │
  ↓                       │
plan-validator (agent) ←──┘
  ↓
  ├──→ static-analyzer (agent)
  ├──→ environment-verifier (agent)
  └──→ architecture-reviewer (agent)
                          │
                          ↓
                   [VALIDATED PLAN]
                          │
                          ↓
/superpowers:execute-plan │
  ↓                       │
executing-plans (skill)   │
  ↓                       │
plan-implementor (agent) ←┘
  ↓
  ├──→ parallel-task-executor (parallel tasks)
  ├──→ debugger (when debugging needed)
  ├──→ code-reviewer (batch reviews)
  └──→ [other specialized agents]
```

---

## What's Included

### Commands

| Command | Description |
|---------|-------------|
| `/validate-plan` | Validate implementation plans before execution |
| `/save-to-md` | Save session documentation with Neo4j memory integration |
| `/quick-push` | Git add all, commit with Claude, and push to feature branch |
| `/proxy` | Create reverse proxy config for deployed services |
| `/screenshot` | Analyze the latest screenshot from screens directory |

### Skills

| Skill | Description |
|-------|-------------|
| `validating-plans` | Systematically validate implementation plans |

---

## Agents

### 1. plan-writer

**Purpose:** Creates comprehensive implementation plans following TDD and bite-sized task principles

**Used by:** writing-plans skill

**Responsibilities:**
- Create detailed, executable implementation plans
- Enforce DRY, YAGNI, KISS principles
- Ensure strict TDD compliance (red-green-refactor)
- Provide exact file paths, code snippets, and commands
- Break tasks into 2-5 minute steps
- Save plans to `docs/plans/YYYY-MM-DD-<feature>.md`

**Key Features:**
- Assumes engineer has zero codebase context
- Every feature follows red-green-refactor cycle
- Complete code snippets (not "add validation")
- Exact commands with expected output

### 2. plan-validator

**Purpose:** Orchestrates comprehensive validation of implementation plans

**Used by:** validating-plans skill

**Responsibilities:**
- Coordinate three specialized validators (static-analyzer, environment-verifier, architecture-reviewer)
- Launch validators in parallel (single message, 3 tool calls)
- Aggregate findings by severity
- Create TodoWrite tasks for plan fixes (NOT code fixes)
- Provide clear verdict: PASS, PASS WITH NOTES, or NEEDS REVISION
- Offer GitHub issue creation post-validation

**Tools:** Read, Grep, Glob, Bash, Task, TodoWrite, Edit

### 3. static-analyzer

**Purpose:** Validates plan structure, TDD compliance, and coding principles

**Used by:** plan-validator

**Validates:**
- ✅ TDD cycles: test before implementation, RED verification, GREEN verification
- ✅ Coding principles: DRY (no duplication), YAGNI (no over-engineering), KISS (simplicity)
- ✅ Task structure: bite-sized steps, exact file paths
- ✅ Internal consistency: dependency ordering, no circular refs
- ✅ Test quality: descriptive names, specific assertions

**Severity Levels:**
- 🔴 BLOCKER: Implementation before test, no RED/GREEN verification
- 🟠 CRITICAL: Vague expectations, batched tests, DRY/YAGNI violations
- 🟡 WARNING: Missing edge cases, KISS violations, suboptimal names

### 4. environment-verifier

**Purpose:** Checks plan assumptions against actual environment and security

**Used by:** plan-validator

**Validates:**
- ✅ Files: existence, line numbers, parent directories
- ✅ Packages: registry availability, version validity, exports
- ✅ APIs: signatures match usage, no deprecated APIs
- ✅ Security: no secrets, no SQL injection, no XSS, port compliance (53000+)
- ✅ Drift: temporal drift, uncommitted changes, dependency updates
- ✅ Commands: binary existence, flag validity

**Severity Levels:**
- 🔴 BLOCKER: Missing files/packages, security vulnerabilities, API mismatch
- 🟠 CRITICAL: Stale line numbers, port conflicts, deprecated packages
- 🟡 WARNING: File overwrites, drift detected, uncommitted changes

### 5. architecture-reviewer

**Purpose:** Reviews architectural soundness and design quality

**Used by:** plan-validator

**Validates:**
- ✅ SOLID principles: SRP, OCP, LSP, ISP, DIP
- ✅ Design patterns: appropriate usage, no anti-patterns
- ✅ Separation of concerns: proper layering
- ✅ DRY at architecture: no duplicate logic across services
- ✅ YAGNI at design: no unnecessary features/complexity
- ✅ Scalability: indexing, caching (when justified), pagination
- ✅ Security architecture: auth/authz, secure defaults
- ✅ Infrastructure: appropriate database, stack compatibility

**Severity Levels:**
- 🔴 BLOCKER: Missing auth, incompatible tech, circular deps, security flaws
- 🟠 CRITICAL: SOLID violations, wrong patterns, YAGNI violations, scalability issues
- 🟡 WARNING: Missing optimizations, minor improvements, documentation gaps

### 6. plan-implementor

**Purpose:** Elite 10x master full-stack engineer executing validated plans with zero hallucination

**Used by:** executing-plans skill

**Key Features:**
- **Zero Assumptions**: Verifies file existence, package availability, command validity before acting
- **Research-Driven**: Reads docs, inspects source, never guesses API signatures
- **Parallel Orchestration**: Dispatches subagents for independent tasks to keep context fresh
- **TDD Strict**: Red (write failing test) → Green (minimal implementation) → Refactor (optional) → Commit
- **Quality Gates**: All steps executed, tests passing, verifications successful before marking complete
- **Clear Communication**: Structured batch reports with exact commands, output, and git status

**Execution Process:**
1. Load and review plan (raise concerns before starting)
2. Create TodoWrite with all tasks
3. Execute in batches (default: 3 tasks)
4. Provide batch completion report (what implemented, verification output, git status)
5. Wait for feedback before next batch
6. Complete development → invoke finishing-a-development-branch skill

---

## Severity Levels

### 🔴 BLOCKER
**Execution will fail. Must fix before proceeding.**

Examples:
- File doesn't exist for "Modify" operation
- Package doesn't exist in registry
- Implementation before test (TDD violation)
- Missing authentication/authorization
- Security vulnerability (SQL injection, XSS, secrets)
- Circular dependencies

### 🟠 CRITICAL
**High risk of rework. Should fix before proceeding.**

Examples:
- Stale line numbers
- Vague test failure expectations
- DRY/YAGNI violations
- SOLID principle violations
- Port conflicts
- Wrong design patterns

### 🟡 WARNING
**Suboptimal but not blocking. Consider fixing.**

Examples:
- Missing edge case tests
- Drift detected (files modified after plan)
- Missing optimizations (indexing, caching)
- KISS violations (unnecessary complexity)
- Documentation gaps

---

## TodoWrite Strategy

**CRITICAL DISTINCTION:**

✅ **CORRECT (fixes PLAN):**
- "Fix plan blocker: Update Task 3 to import from starlette.responses instead of hallucinated fastapi_utils"
- "Fix plan critical: Reorder Task 2 steps to test-first (TDD violation)"
- "Fix plan blocker: Change Task 5 target from non-existent src/services/legacy_auth.py to correct path"

❌ **WRONG (fixes CODE):**
- "Fix: Install fastapi_utils package" (no, fix the plan to not use it)
- "Fix: Create missing src/services/legacy_auth.py file" (no, fix the plan path)
- "Add error handling to Task 3" (this changes implementation, not plan)

---

## Integration with Superpowers

**Complete Workflow:**

```
1. Brainstorm (creates worktree)
2. Write-plan (plan-writer creates plan)
3. Validate-plan (plan-validator + 3 validators check plan)
4. Execute-plan (batch execution with checkpoints)
5. Finish-branch (completion workflow)
```

**Plan Header Requirement:**

Every plan must include:

```markdown
> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.
```

**Plan Location Standard:**

Plans saved to: `docs/plans/YYYY-MM-DD-<feature-name>.md`

**TDD 5-Step Pattern:**

1. Write failing test
2. Run to verify fail
3. Write minimal code
4. Run to verify pass
5. Commit

---

## Usage Examples

### Example 1: Create Plan

```
User: Create a plan for implementing user authentication

→ /superpowers:write-plan activates
→ plan-writer agent creates comprehensive plan
→ Plan saved to docs/plans/2025-12-08-user-authentication.md
→ Offers validation or execution
```

### Example 2: Validate Plan

```
User: Validate the authentication plan

→ /validate-plan docs/plans/2025-12-08-user-authentication.md
→ plan-validator orchestrates 3 parallel validators
→ static-analyzer: ✅ PASS (proper TDD cycles)
→ environment-verifier: 🔴 FAIL (package fastapi_utils doesn't export CacheControl)
→ architecture-reviewer: ✅ PASS (SOLID principles followed)
→ Verdict: 🔴 NEEDS REVISION
→ TodoWrite: "Fix plan blocker: Use starlette.responses.CacheControl"
```

### Example 3: Clean Validation

```
User: Validate the updated plan

→ /validate-plan docs/plans/2025-12-08-user-authentication.md
→ All 3 validators: ✅ PASS
→ Verdict: ✅ PASS
→ Offers GitHub issue creation
→ User confirms
→ GitHub issue created: #123 "Implementation: User Authentication"
```

---

## Best Practices

1. **Always validate plans** before execution to catch issues early
2. **Fix plan issues** in the plan file, not in code
3. **Use TodoWrite** to track plan fixes systematically
4. **Run validators in parallel** for efficiency (single message, 3 tool calls)
5. **Review all severity levels** - even warnings can indicate design issues
6. **Create GitHub issues** for validated plans to track implementation
7. **Keep plans in version control** for team collaboration
8. **Move completed plans** to `docs/plans/complete/` after verification

---

## Troubleshooting

### Validators Don't Find Issues

- Check if plan follows writing-plans format
- Verify file paths are absolute, not relative
- Ensure code snippets are complete, not placeholders

### Too Many Warnings

- Warnings are informative, not blocking
- Focus on blockers and criticals first
- Consider warnings as improvement opportunities

### Validation Takes Too Long

- Ensure validators run in parallel (single message, 3 tool calls)
- Check network connectivity for package verification
- Consider caching package lookups (future enhancement)

---

## Requirements

- Claude Code CLI
- GitHub CLI (`gh`) for repository operations
- Git for version control
- [superpowers plugin](https://github.com/obra/superpowers) (for write-plan and execute-plan commands)

## License

MIT
