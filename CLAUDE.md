# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

**CC_nix** is a community repository providing reusable Claude Code resources for NixOS and nixpkgs development. This repo contains generic slash commands, subagents, skills, and best practices that can be used in any NixOS project.

## What This Repository Provides

### For NixOS Developers
- **Slash commands**: Ready-to-use commands for common NixOS tasks
- **Subagent definitions**: Specialized AI agents for Nix workflows
- **Skills**: Composable capabilities for NixOS development
- **Best practices**: Patterns that work well with AI-assisted development
- **Documentation**: Anti-patterns, prompting strategies, and workflows

### For Claude Code
- **Generic templates**: Host-agnostic configurations and commands
- **Validation workflows**: Syntax checking, anti-pattern detection, security audits
- **Safety patterns**: Pre-deployment validation and rollback mechanisms
- **Context documents**: Essential files to read for NixOS work

## Core Philosophy

### AI-Friendly NixOS Development

This repository demonstrates how to:

1. **Structure projects** for optimal AI understanding
2. **Document patterns** that Claude Code can follow
3. **Create workflows** that combine AI assistance with safety
4. **Avoid pitfalls** specific to AI-assisted Nix development

### Key Principles

- **Generic by default**: No hardcoded hosts, users, or infrastructure
- **Explicit over implicit**: Clear references and well-documented patterns
- **Safety first**: Validation before deployment, rollback on failure
- **Community-driven**: Contributions welcome, patterns evolve

## Essential Files for Claude Code

### Critical Documentation

When working on any NixOS project, Claude Code should reference:

1. **`.claude/NIX_ANTIPATTERNS.md`** - Critical patterns to avoid
2. **`PATTERNS.md`** - Proven NixOS patterns (if exists in project)
3. **Module documentation** - Option descriptions and examples
4. **Flake structure** - Inputs, outputs, dependencies

### How to Use This Repository

**Option 1: Copy to your project**
```bash
# Copy .claude/ directory to your NixOS project
cp -r /path/to/CC_nix/.claude /your/nixos/project/

# Customize as needed for your specific infrastructure
```

**Option 2: Reference globally**
```bash
# Link commands to your global Claude Code config
ln -s /path/to/CC_nix/.claude/commands ~/.claude/commands/nix/
```

## Available Resources

### Slash Commands (`.claude/commands/`)

Generic, reusable commands for NixOS development:

- **`/nix-module`** - Create NixOS modules following best practices
- **`/nix-fix`** - Detect and fix common anti-patterns
- **`/review`** - Comprehensive code review against patterns
- **`/nix-security`** - Security audit with recommendations
- **`/nix-deploy`** - Smart deployment with validation
- **`/nix-optimize`** - Performance analysis
- **`/new_task`** - Create well-structured GitHub issues
- **`/check_tasks`** - Track project progress

All commands are **host-agnostic** and can be adapted to any NixOS project.

### Subagents (`.claude/agents/`)

Specialized agents for automated NixOS workflows:

- **`nix-check`** - Comprehensive NixOS configuration linting and fixing
  - Uses: deadnix (dead code), statix (anti-patterns), nixpkgs-fmt (formatting)
  - Auto-runs: On file changes, pre-commit hooks, code reviews
  - Features: Syntax validation, auto-fix, detailed reports, CI integration
  - Enforces: Best practices, security hardening, code quality

Future specialized agents:
- **nixos-module-builder** - Expert in creating NixOS modules
- **nix-security-auditor** - Security-focused code review
- **nix-optimizer** - Performance and efficiency analysis

### Skills (`.claude/skills/`)

Specialized capabilities for specific NixOS workflows:

- **`home-manager`** - Home Manager configuration and user environment management
  - Use when: Configuring user packages, dotfiles, programs, and services
  - Covers: Installation, program configuration, service management, file management, desktop integration

- **`devenv`** - Fast, declarative, reproducible development environments
  - Use when: Setting up project development environments, configuring languages, managing services
  - Covers: 55+ languages, 30+ services, process management, containers, pre-commit hooks, CI integration

- **`node2nix`** - Convert NPM packages to Nix expressions
  - Use when: Packaging Node.js applications, managing npm dependencies with Nix, building reproducible Node.js apps
  - Covers: Package conversion, lock file support, private registries, native modules, NixOS integration, troubleshooting

- **`uv2nix`** - Convert Python/uv projects to Nix expressions
  - Use when: Packaging Python applications with uv, managing Python dependencies with Nix, building reproducible Python apps
  - Covers: uv workspace integration, pyproject.toml support, dependency management, NixOS services, testing, monorepo support

Future composable capabilities:
- **nix-syntax-validation** - Fast syntax checking
- **nix-anti-pattern-detection** - Pattern recognition
- **nix-security-hardening** - Systemd and service hardening

## Prompting Best Practices

### Effective Prompts for NixOS

**✅ Specific with Context**
```
"Create a NixOS module for Redis with:
- DynamicUser for security isolation
- Systemd hardening (ProtectSystem=strict, NoNewPrivileges)
- Configuration via module options
- Follow patterns from .claude/NIX_ANTIPATTERNS.md"
```

**❌ Vague**
```
"Add Redis to my system"
```

### Multi-Step Workflows

```
# 1. Understand existing patterns
"Analyze modules/ directory and document the common module structure"

# 2. Apply patterns
"Create a new monitoring module following the same structure"

# 3. Validate
"/review Check against NIX_ANTIPATTERNS.md"
```

## Project Structure for AI Assistance

### Recommended Organization

```
your-nixos-project/
├── .claude/                      # Copy from CC_nix
│   ├── CLAUDE.md                 # Project-specific context
│   ├── NIX_ANTIPATTERNS.md      # Generic anti-patterns
│   └── commands/                 # NixOS slash commands
├── docs/
│   ├── PATTERNS.md               # Your project's patterns
│   └── ARCHITECTURE.md           # System design
├── modules/
│   ├── services/
│   └── default.nix
├── flake.nix
├── flake.lock                    # Always commit!
└── README.md
```

### Why This Works

1. **`.claude/`**: Centralized AI context - Claude Code checks here first
2. **Clear documentation**: PATTERNS.md, ANTIPATTERNS.md are self-explanatory
3. **Module organization**: Logical hierarchy aids navigation
4. **Explicit imports**: No magic, everything declared

## NixOS-Specific AI Challenges

### Common Issues When Using AI for Nix

**1. Type System Confusion**
- **Problem**: Nix's lazy evaluation confuses LLMs
- **Solution**: Always specify expected types in prompts
- **Example**: "Create a string option for hostname" not "add hostname option"

**2. Path vs String Ambiguity**
- **Problem**: `./path` vs `"path"` vs path concatenation
- **Solution**: Reference NIX_ANTIPATTERNS.md, be explicit in prompts
- **Example**: "Use a string path" or "Use a Nix path literal"

**3. Evaluation vs Runtime**
- **Problem**: When secrets/files are read (evaluation vs runtime)
- **Solution**: Always use `*File` options for secrets
- **Example**: `passwordFile = ...` not `password = builtins.readFile ...`

**4. Module Option Complexity**
- **Problem**: Nested options and conditionals are complex
- **Solution**: Request step-by-step, use `let` bindings
- **Example**: "Break this into let bindings for clarity"

## Validation Workflows

### Pre-Deployment Checklist

```bash
# 1. Syntax validation (fast, ~5s)
nix flake check

# 2. Build test (thorough, ~1min)
nix build .#nixosConfigurations.YOUR_HOST.config.system.build.toplevel

# 3. Anti-pattern check
# Use /nix-fix slash command

# 4. Security audit
# Use /nix-security slash command
```

### Safety Mechanisms

Generic patterns demonstrated in this repo:

1. **Pre-deployment validation**: Syntax, patterns, security
2. **Test builds**: Full configuration validation
3. **Change detection**: Deploy only when necessary
4. **Rollback on failure**: Automatic recovery
5. **Post-deployment checks**: Service verification

## Contributing to CC_nix

### What to Contribute

**Slash Commands**
- Generic, reusable commands for common NixOS tasks
- Include clear documentation and examples
- No hardcoded hosts, paths, or infrastructure

**Documentation**
- Patterns that work well with Claude Code
- Anti-patterns to avoid
- Prompting strategies
- Real-world examples

**Subagents/Skills**
- Specialized agents for NixOS workflows
- Composable capabilities
- Clear scope and purpose

### Contribution Guidelines

1. **Keep it generic**: No specific hosts, users, or infrastructure
2. **Document thoroughly**: Explain why patterns work
3. **Include examples**: Show prompts that generate good results
4. **Test extensively**: Ensure code actually builds
5. **Follow conventions**: Use existing command/skill structure

## Example Workflows

### Creating a Service Module

```
User: "I need a module for nginx with security hardening"

Claude Code (using CC_nix resources):
1. Reads .claude/NIX_ANTIPATTERNS.md
2. Uses /nix-module command template
3. Creates module with:
   - Proper option structure
   - DynamicUser security
   - Systemd hardening
   - Clear documentation
4. Validates syntax
5. Suggests testing approach
```

### Code Review

```
User: "/review modules/services/custom-service.nix"

Claude Code:
1. Reads NIX_ANTIPATTERNS.md
2. Checks against documented patterns
3. Analyzes for:
   - Anti-patterns
   - Security issues
   - Type safety
   - Best practices
4. Provides specific fixes with code
```

### Safe Deployment

```
User: "/nix-deploy Deploy my changes"

Claude Code:
1. Detects changed files
2. Runs syntax validation
3. Checks anti-patterns
4. Performs test build
5. Shows impact analysis
6. Deploys with monitoring
7. Verifies services
8. Reports status (generic template)
```

## Integration with Your Projects

### Step 1: Copy Resources

```bash
# Copy .claude/ to your project
cp -r CC_nix/.claude your-nixos-project/

# Customize CLAUDE.md for your infrastructure
vim your-nixos-project/.claude/CLAUDE.md
```

### Step 2: Adapt Commands

```bash
# In your project's .claude/CLAUDE.md, specify:
- Your hosts (if using /nix-deploy)
- Your module structure
- Your deployment workflow
- Your validation requirements
```

### Step 3: Use Commands

```bash
# Commands automatically adapt to your project structure
cd your-nixos-project
claude  # Start Claude Code

# Use commands
/nix-module     # Creates module in your modules/ directory
/review         # Reviews against YOUR patterns
/nix-deploy     # Deploys to YOUR infrastructure
```

## References

### Essential Reading

- **Nix Book**: https://mhwombat.codeberg.page/nix-book/
- **NixOS Manual**: https://nixos.org/manual/nixos/stable/
- **Nix Pills**: https://nixos.org/guides/nix-pills/

### This Repository

- `.claude/NIX_ANTIPATTERNS.md` - Critical patterns to avoid
- `.claude/commands/` - Reusable slash commands
- `docs/` - Additional documentation (as repository grows)

## Meta: About CC_nix

This repository is:
- **Community-driven**: Contributions welcome
- **Generic by design**: Works with any NixOS project
- **Best practices**: Proven patterns for AI-assisted development
- **Living documentation**: Evolves with the NixOS ecosystem

**Not a complete NixOS configuration** - it's a toolkit for building them with Claude Code.

---

💡 **Tip**: Star this repo and adapt these resources for your NixOS projects. Share your improvements back to the community!
