# claude-box

A Claude Code plugin marketplace containing custom tools for plan-driven development workflows, Git helpers, and productivity commands.

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

## What's Included

### Commands

| Command | Description |
|---------|-------------|
| `/save-to-md` | Save session documentation with Neo4j memory integration |
| `/validate-plan` | Validate implementation plans before execution |
| `/quick-push` | Git add all, commit with Claude, and push to feature branch |
| `/proxy` | Create reverse proxy config for deployed services |
| `/screenshot` | Analyze the latest screenshot from screens directory |

### Agents

| Agent | Description |
|-------|-------------|
| `plan-writer` | Creates comprehensive implementation plans following TDD principles |
| `plan-validator` | Orchestrates validation of implementation plans |
| `plan-implementor` | Executes validated plans task-by-task |
| `architecture-reviewer` | Reviews architectural soundness and design quality |
| `static-analyzer` | Validates plan structure, TDD compliance, and coding principles |
| `environment-verifier` | Verifies plan assumptions against actual environment |

### Skills

| Skill | Description |
|-------|-------------|
| `validating-plans` | Systematically validate implementation plans before execution |

## Workflow

The plan workflow follows this sequence:

1. **Write Plan** - Use `plan-writer` agent to create detailed implementation plan
2. **Validate Plan** - Run `/validate-plan` to verify references, TDD compliance, and catch issues
3. **Execute Plan** - Use `plan-implementor` agent to execute task-by-task

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

## Requirements

- Claude Code CLI
- GitHub CLI (`gh`) for repository operations
- Git for version control

## License

MIT
