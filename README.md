# claude-box

A Claude Code plugin marketplace containing a complete plan workflow system, Git helpers, and productivity commands.

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

## Plan Workflow System

Complete agent-based system for creating, validating, and executing implementation plans.

### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                  PLAN WORKFLOW SYSTEM                        │
└─────────────────────────────────────────────────────────────┘

1. CREATION PHASE
   /superpowers:write-plan
     ↓
   writing-plans skill
     ↓
   plan-writer agent → docs/plans/YYYY-MM-DD-feature.md

2. VALIDATION PHASE
   /validate-plan
     ↓
   validating-plans skill
     ↓
   plan-validator agent
     ↓
     ├─→ static-analyzer (structure, TDD, DRY/YAGNI/KISS)
     ├─→ environment-verifier (files, packages, security, drift)
     └─→ architecture-reviewer (SOLID, patterns, scalability)
     ↓
   Validation Report + TodoWrite (if issues) OR GitHub Issue (if passed)

3. EXECUTION PHASE
   /superpowers:execute-plan
     ↓
   executing-plans skill
     ↓
   plan-implementor agent
     ↓
     ├─→ parallel-task-executor (independent tasks)
     ├─→ debugger (error handling)
     └─→ code-reviewer (batch reviews)
     ↓
   Batch execution → Review → Continue → Complete
```

### Workflow Example

```bash
# Step 1: Create Plan
User: "Create a plan for implementing user authentication"
→ /superpowers:write-plan activates
→ plan-writer agent creates comprehensive plan
→ Plan saved: docs/plans/2025-12-08-user-authentication.md

# Step 2: Validate Plan
→ /validate-plan docs/plans/2025-12-08-user-authentication.md
→ plan-validator orchestrates 3 parallel validators:
  - static-analyzer: ✅ PASS (proper TDD cycles)
  - environment-verifier: ✅ PASS (all files/packages exist)
  - architecture-reviewer: ✅ PASS (SOLID principles)
→ Verdict: ✅ PASS
→ "Create GitHub issue? (y/n):" → y
→ GitHub issue created: #123

# Step 3: Execute Plan
→ /superpowers:execute-plan
→ plan-implementor executes tasks in batches
→ All tasks complete → PR created
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

### Agents

| Agent | Purpose |
|-------|---------|
| `plan-writer` | Create comprehensive TDD-based implementation plans |
| `plan-validator` | Orchestrate validation with 3 parallel validators |
| `plan-implementor` | Execute validated plans task-by-task |
| `static-analyzer` | Validate structure, TDD compliance, DRY/YAGNI/KISS |
| `environment-verifier` | Verify files, packages, security, drift detection |
| `architecture-reviewer` | Review SOLID principles, patterns, scalability |

### Skills

| Skill | Description |
|-------|-------------|
| `validating-plans` | Systematically validate implementation plans |

---

## Validation Checks

### Static Analyzer
- TDD compliance (red-green-refactor cycles)
- DRY, YAGNI, KISS principles
- Task granularity (2-5 minute steps)
- Dependency ordering

### Environment Verifier
- File existence and line numbers
- Package availability in registries
- API signatures and exports
- Security vulnerabilities (SQL injection, XSS, secrets)
- Port compliance (53000+ requirement)
- Drift detection

### Architecture Reviewer
- SOLID principles
- Design pattern appropriateness
- Separation of concerns
- Scalability considerations

### Severity Levels

| Level | Symbol | Meaning |
|-------|--------|---------|
| BLOCKER | 🔴 | Execution will fail, must fix |
| CRITICAL | 🟠 | High risk of rework, should fix |
| WARNING | 🟡 | Suboptimal, consider fixing |
| NIT | 🔵 | Style/preference, optional |

---

## Usage Examples

### Validate a Plan

```bash
/validate-plan docs/plans/my-feature.md
```

### Quick Git Commit

```bash
/quick-push
```

### Create Proxy Config

```bash
/proxy myservice myapp 8080
```

---

## Requirements

- Claude Code CLI
- GitHub CLI (`gh`) for repository operations
- Git for version control
- [superpowers plugin](https://github.com/obra/superpowers) (for write-plan and execute-plan commands)

## License

MIT
