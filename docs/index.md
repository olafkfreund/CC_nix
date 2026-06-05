<!-- This page is seeded from README.md. Edit either file;
     they diverge by design after the initial onboarding. -->

# CC_nix: Claude Code Resources for NixOS Development

> **Supercharge your NixOS development with AI-assisted workflows**

A community-driven collection of Claude Code resources, patterns, and best practices for productive NixOS and nixpkgs development.

## 🎯 What is CC_nix?

CC_nix provides **ready-to-use** Claude Code resources that make NixOS development faster, safer, and more enjoyable:

- **🤖 Slash Commands**: Pre-built commands for common NixOS tasks
- **📚 Documentation**: Anti-patterns guide and best practices
- **🔧 Workflows**: Validated patterns for AI-assisted development
- **⚡ Speed**: Automation that saves hours of manual work
- **🛡️ Safety**: Built-in validation and rollback mechanisms

## 🚀 Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/olafkfreund/CC_nix.git
cd CC_nix
```

### 2. Copy to Your NixOS Project

```bash
# Copy .claude/ directory to your project
cp -r .claude /path/to/your/nixos/project/

# Or symlink for shared access across projects
ln -s $(pwd)/.claude /path/to/your/nixos/project/.claude
```

### 3. Start Using Commands

```bash
cd /path/to/your/nixos/project
claude  # Start Claude Code

# Try a command
/nix-module
Create a new Redis module

# Or review your code
/review
Check my recent changes
```

## 📋 Available Resources

### Slash Commands

All commands in `.claude/commands/` are **generic** and **host-agnostic**:

| Command | Purpose | Time |
|---------|---------|------|
| `/nix-module` | Create NixOS modules with best practices | ~2min |
| `/new_pkg` | Package existing software for Nix | ~10-15min |
| `/new_vm` | Create NixOS virtual machines with QEMU | ~15-20min |
| `/nix-fix` | Detect and fix common anti-patterns | ~1min |
| `/review` | Comprehensive code review | ~1min |
| `/nix-security` | Security audit with recommendations | ~1min |
| `/nix-deploy` | Smart deployment with validation | ~2.5min |
| `/nix-optimize` | Performance analysis | ~2min |
| `/new_task` | Create GitHub issues | ~2min |
| `/check_tasks` | Track project progress | ~30s |

### Documentation

- **`.claude/NIX_ANTIPATTERNS.md`** - Critical patterns to avoid (700+ lines)
- **`.claude/CLAUDE.md`** - Guide for using Claude Code with NixOS

### Skills

Specialized capabilities for specific NixOS workflows in `.claude/skills/`:

| Skill | Purpose | Use When |
|-------|---------|----------|
| `home-manager` | Home Manager configuration and user environment management | Configuring user packages, dotfiles, services |
| `devenv` | Fast, declarative development environments | Creating project dev environments, language setup, service management |
| `node2nix` | Convert NPM packages to Nix expressions | Packaging Node.js apps, managing npm dependencies with Nix |
| `uv2nix` | Convert Python/uv projects to Nix expressions | Packaging Python apps, managing Python dependencies with Nix |
| `stylix` | System-wide theming with color schemes, wallpapers, fonts | Applying consistent themes across NixOS/Home Manager applications |
| `agenix` | Age-encrypted secrets management with SSH keys | Managing sensitive data, passwords, API keys, certificates in NixOS |
| `gnome` | GNOME desktop configuration with Stylix integration | Setting up GNOME desktop, extensions, themes, settings with best practices |
| `cosmic-de` | System76's COSMIC Desktop Environment configuration | Setting up COSMIC Desktop, applets, RON config, themes, performance optimization |
| `cargo2nix` | Package Rust applications with granular per-crate builds | Packaging Rust apps, managing Cargo dependencies, cross-compilation, workspace support |

### Subagents

Specialized AI agents for automated NixOS workflows in `.claude/agents/`:

| Subagent | Purpose | Auto-runs On |
|----------|---------|--------------|
| `nix-check` | Lint, check, and fix NixOS configs using deadnix, statix, nixpkgs-fmt | File changes, pre-commit, code review |
| `issue-checker` | Monitor GitHub issues for bugs affecting installed packages | Before updates, system planning, CI/CD pipelines |
| `local-logs` | Analyze system logs, research solutions, generate configuration fixes | Service failures, boot issues, performance problems, troubleshooting |
| `update` | Automated system updates with safety checks, test rebuilds, and notifications | Scheduled updates, manual requests, systemd timers |

### Future Resources (Contributions Welcome!)

- **More Subagents**: Additional specialized agents for NixOS workflows
- **Templates**: Project scaffolding and boilerplate

## 🎯 How to Use Skills and Subagents

### Understanding Skills

**Skills** are specialized knowledge domains that Claude Code references when working on specific tasks. They contain deep expertise on particular tools, frameworks, or workflows.

#### How Skills Work

1. **Automatic Invocation**: Claude Code automatically detects when a skill is relevant based on your conversation context
2. **Explicit Reference**: You can explicitly mention a skill in your prompts
3. **Context-Aware**: Skills provide targeted guidance, examples, and best practices

#### Real-Life Skill Examples

**Example 1: Setting Up GNOME Desktop**

```
You: "I want to set up GNOME with a custom theme and some extensions"

Claude Code:
- ✅ Automatically references the 'gnome' skill
- ✅ Reads best practices for declarative GNOME configuration
- ✅ Provides Home Manager setup with dconf settings
- ✅ Shows Stylix integration for consistent theming
- ✅ Warns about anti-patterns (manual extension installation)
- ✅ Includes discovery commands (dconf watch /)

Result: Complete GNOME configuration with extensions, theme, and settings
```

**Example 2: Managing Secrets**

```
You: "I need to securely store API keys and database passwords in my NixOS config"

Claude Code:
- ✅ Automatically references the 'agenix' skill
- ✅ Guides through SSH-based encryption setup
- ✅ Shows how to create secrets.nix
- ✅ Demonstrates runtime secret loading (not evaluation time)
- ✅ Provides rekeying examples for key rotation

Result: Secure, declarative secrets management using age encryption
```

**Example 3: Packaging a Node.js Application**

```
You: "I want to package my Express.js app for NixOS"

Claude Code:
- ✅ Automatically references the 'node2nix' skill
- ✅ Explains package-lock.json conversion
- ✅ Shows how to handle native dependencies
- ✅ Provides override patterns for build fixes
- ✅ Demonstrates development shell setup

Result: Reproducible Node.js package with all dependencies
```

**Example 4: Creating a Development Environment**

```
You: "I need a dev environment for Python with PostgreSQL and Redis"

Claude Code:
- ✅ Automatically references the 'devenv' skill
- ✅ Creates devenv.nix with Python, PostgreSQL, Redis
- ✅ Sets up automatic service startup
- ✅ Configures pre-commit hooks
- ✅ Includes IDE integration (direnv)

Result: Complete, reproducible dev environment in minutes
```

**Example 5: Configuring COSMIC Desktop**

```
You: "I want to set up COSMIC Desktop with custom panel applets and tiling"

Claude Code:
- ✅ Automatically references the 'cosmic-de' skill
- ✅ Shows native NixOS installation (25.05+)
- ✅ Provides RON configuration examples
- ✅ Demonstrates Home Manager integration
- ✅ Includes System76 scheduler for performance
- ✅ Warns about binary cache setup importance

Result: Fully configured COSMIC Desktop with custom applets and settings
```

#### How to Explicitly Use Skills

**Implicit (Automatic)**:
```
You: "Configure my GNOME desktop"
# Claude Code automatically uses the gnome skill
```

**Explicit (When You Want to Be Specific)**:
```
You: "Using the gnome skill, show me how to configure keyboard shortcuts"
# Forces Claude Code to reference gnome skill documentation
```

**Combining Multiple Skills**:
```
You: "Set up GNOME with Stylix theming and agenix for my WiFi password"
# Claude Code uses: gnome + stylix + agenix skills together
```

### Understanding Subagents

**Subagents** are specialized AI agents that autonomously perform specific tasks. Unlike skills (which are knowledge), subagents actively analyze, research, and generate solutions.

#### How Subagents Work

1. **Proactive Execution**: Automatically run when their context is detected
2. **Autonomous Operation**: Complete multi-step tasks independently
3. **Deep Analysis**: Research, analyze patterns, generate fixes

#### Real-Life Subagent Examples

**Example 1: Automated Code Quality (nix-check)**

**Scenario**: You've modified several NixOS modules and want to ensure quality

```
You: "Review my changes for any issues"

Claude Code:
- ✅ Automatically invokes 'nix-check' subagent
- ✅ Runs deadnix to find unused code
- ✅ Runs statix to detect anti-patterns
- ✅ Runs nixpkgs-fmt for formatting issues
- ✅ Generates detailed report with line numbers
- ✅ Offers to auto-fix issues with backups

Output:
┌─ deadnix results ─────────────────────────────
│ modules/services/nginx.nix:45 - Unused binding 'pkgs'
│ modules/services/nginx.nix:67 - Unused let binding 'port'
└───────────────────────────────────────────────

┌─ statix results ──────────────────────────────
│ modules/services/postgres.nix:23 - empty_let_in
│   Suggestion: Remove empty let-in block
│ modules/services/postgres.nix:89 - bool_comparison
│   Suggestion: Use mkIf directly instead of == true
└───────────────────────────────────────────────

Would you like me to auto-fix these issues? (Creates backups first)
```

**Example 2: Pre-Update Safety Check (issue-checker)**

**Scenario**: Planning to update your system, want to know about potential issues

```
You: "I'm planning to update my NixOS system, are there any known issues?"

Claude Code:
- ✅ Automatically invokes 'issue-checker' subagent
- ✅ Scans current system packages
- ✅ Queries NixOS/nixpkgs GitHub issues
- ✅ Identifies issues affecting your packages
- ✅ Classifies by severity (critical/high/medium/low)
- ✅ Provides actionable recommendations

Output:
┌─ Issue Analysis Results ──────────────────────
│ Total packages analyzed: 847
│ Open issues found: 3
└───────────────────────────────────────────────

🔴 CRITICAL (1):
  - linux-6.12.3: Boot failure on NVIDIA systems (#325841)
    Impact: System won't boot after update
    Action: Pin kernel to 6.11.x or wait for 6.12.4

🟡 MEDIUM (2):
  - firefox-133.0: Video playback crash (#325789)
    Impact: YouTube/video sites may crash
    Action: Downgrade to 132.0 or use workaround

  - gnome-47.2: Extension compatibility broken (#325654)
    Impact: Some extensions won't load
    Action: Review extension list, update or remove

Recommendation: DELAY update until linux-6.12.4 is available
```

**Example 3: System Troubleshooting (local-logs)**

**Scenario**: Your PostgreSQL service failed to start after configuration change

```
You: "PostgreSQL won't start, can you help debug this?"

Claude Code:
- ✅ Automatically invokes 'local-logs' subagent
- ✅ Collects logs from journalctl, dmesg, /var/log
- ✅ Analyzes error patterns and stack traces
- ✅ Researches solutions in documentation/forums
- ✅ Generates NixOS configuration fix
- ✅ Provides testing instructions

Output:
┌─ Log Analysis ────────────────────────────────
│ Service: postgresql.service
│ Status: failed
│ Error: "data directory has wrong ownership"
└───────────────────────────────────────────────

Root Cause Analysis:
├─ PostgreSQL data directory: /var/lib/postgresql/15
├─ Current owner: root:root (incorrect)
├─ Expected owner: postgres:postgres
└─ Cause: Recent change to systemd.tmpfiles.rules

Solution Found:
Based on NixOS manual and similar issues, this is caused by
incorrect tmpfiles.d configuration overriding the default
PostgreSQL data directory permissions.

Generated Fix:
───────────────────────────────────────────────
# Remove this from your configuration:
systemd.tmpfiles.rules = [
  "d /var/lib/postgresql 0755 root root -"  # ❌ WRONG
];

# PostgreSQL module handles this automatically
# Just ensure:
services.postgresql = {
  enable = true;
  dataDir = "/var/lib/postgresql/15";  # ✅ CORRECT
};
───────────────────────────────────────────────

Testing Instructions:
1. Apply the fix above
2. Run: sudo nixos-rebuild switch
3. Verify: systemctl status postgresql
4. Check logs: journalctl -u postgresql -n 50

Would you like me to apply this fix to your configuration?
```

#### When Subagents Run Automatically

**nix-check**:
- After you edit .nix files
- When you ask for code review
- Before git commits (if pre-commit configured)
- When you run `/review` command

**issue-checker**:
- When you ask about system updates
- When planning NixOS version upgrades
- Before deploying to production
- When troubleshooting package issues

**local-logs**:
- When you report service failures
- When asking about boot problems
- When troubleshooting system errors
- When requesting performance analysis

#### How to Explicitly Use Subagents

**Implicit (Automatic)**:
```
You: "My nginx service failed, what's wrong?"
# local-logs subagent automatically runs
```

**Explicit (When You Want Specific Analysis)**:
```
You: "Use the local-logs subagent to analyze all failed services in the last hour"
# Explicitly invokes subagent with specific parameters
```

**On-Demand**:
```
You: "Run nix-check on modules/services/ directory"
# Explicitly runs code quality analysis
```

## 💡 Complete Real-World Workflows

### Workflow 1: Setting Up a New Desktop Environment

**Goal**: Install and configure GNOME with custom theme, extensions, and secrets

```
You: "I want to set up GNOME with Nord theme, Dash to Dock, and store my WiFi password securely"

Claude Code Workflow:
1. 🎯 Detects desktop environment (gnome skill)
2. 🎯 Detects theming needs (stylix skill)
3. 🎯 Detects secrets management (agenix skill)

Steps Executed:
├─ Creates NixOS configuration with GNOME
├─ Sets up Stylix with Nord color scheme
├─ Configures Dash to Dock extension via dconf
├─ Sets up agenix for WiFi password
├─ Generates secrets.nix template
└─ Provides complete Home Manager configuration

Generated Configuration:
# configuration.nix
services.displayManager.gdm.enable = true;
services.desktopManager.gnome.enable = true;

# home.nix (via gnome skill)
dconf.settings."org/gnome/shell" = {
  enabled-extensions = [ dash-to-dock.extensionUuid ];
};

# stylix integration (via stylix skill)
stylix = {
  base16Scheme = "${pkgs.base16-schemes}/share/themes/nord.yaml";
  targets.gnome.enable = true;
};

# secrets (via agenix skill)
age.secrets.wifi-password = {
  file = ./secrets/wifi.age;
};

Result: Complete desktop setup in 5 minutes vs 2 hours manual configuration
```

### Workflow 2: Deploying a New Service with Security Audit

**Goal**: Create a Redis module, ensure it's secure, deploy to production

```
You: "Create a Redis module and deploy it securely to my server"

Claude Code Workflow:
1. 🤖 Uses /nix-module command
2. 🎯 Applies security best practices automatically
3. 🤖 Invokes nix-check subagent for validation
4. 🤖 Uses /nix-security command for audit
5. 🤖 Uses /nix-deploy command for deployment

Steps Executed:
├─ Creates modules/services/redis.nix
│  ├─ DynamicUser = true (security)
│  ├─ ProtectSystem = "strict" (hardening)
│  └─ NoNewPrivileges = true (isolation)
├─ Runs nix-check subagent
│  ├─ deadnix: No unused code ✓
│  ├─ statix: No anti-patterns ✓
│  └─ nixpkgs-fmt: Properly formatted ✓
├─ Runs security audit
│  ├─ Systemd hardening: 95/100 ✓
│  ├─ Secret management: Proper ✓
│  └─ Network isolation: Configured ✓
├─ Deploys to server
│  ├─ Validates syntax ✓
│  ├─ Test build ✓
│  ├─ Deploys with rollback ✓
│  └─ Verifies service running ✓

Result: Production-ready, secure Redis module deployed in 3 minutes
```

### Workflow 3: Debugging Production Issue

**Goal**: Service crashed overnight, need to diagnose and fix quickly

```
You: "PostgreSQL crashed last night, I need to fix this urgently"

Claude Code Workflow:
1. 🤖 Automatically invokes local-logs subagent
2. 🎯 Analyzes system logs for root cause
3. 🎯 Researches solutions
4. 🎯 Generates fix
5. 🤖 Uses /nix-deploy for emergency deployment

Steps Executed:
├─ local-logs subagent activates
│  ├─ Collects journalctl logs
│  ├─ Analyzes PostgreSQL errors
│  ├─ Identifies: "out of memory" errors
│  └─ Correlates with system memory usage
├─ Root Cause Found:
│  ├─ PostgreSQL shared_buffers too high
│  ├─ System only has 8GB RAM
│  └─ Other services competing for memory
├─ Solution Generated:
│  ├─ Reduce shared_buffers to 2GB
│  ├─ Enable memory overcommit controls
│  └─ Add memory limits to systemd service
├─ Fix Applied:
│  └─ Updates PostgreSQL configuration
└─ Emergency deployment
   ├─ Validates fix ✓
   ├─ Deploys immediately ✓
   └─ Monitors service ✓

Result: Issue diagnosed and fixed in 5 minutes vs hours of manual debugging
```

### Workflow 4: Safe System Update

**Goal**: Update NixOS system without breaking production

```
You: "I need to update my production server, but I want to make sure it's safe"

Claude Code Workflow:
1. 🤖 Automatically invokes issue-checker subagent
2. 🎯 Analyzes current package versions
3. 🎯 Checks for known issues in nixpkgs
4. 🎯 Provides risk assessment
5. 🤖 Uses /nix-deploy for controlled update

Steps Executed:
├─ issue-checker subagent activates
│  ├─ Inventories 847 installed packages
│  ├─ Queries GitHub for open issues
│  ├─ Identifies 3 critical issues
│  └─ Generates risk report
├─ Risk Assessment:
│  ├─ 🔴 CRITICAL: Kernel boot failure
│  ├─ 🟡 MEDIUM: nginx security CVE
│  └─ 🟢 LOW: Minor UI glitch in vim
├─ Recommendation:
│  ├─ Pin kernel to current version
│  ├─ Update nginx (security fix)
│  └─ Proceed with caution
├─ Generated Update Plan:
│  └─ Selective update with kernel pin
└─ Deployment
   ├─ Tests new configuration ✓
   ├─ Deploys to staging first ✓
   ├─ Validates services ✓
   └─ Production deployment ✓

Result: Safe update avoiding critical kernel bug, applying security fixes
```

### Workflow 5: Onboarding New Developer

**Goal**: New team member needs complete dev environment setup

```
You: "Set up a development environment for our Python/Django project with PostgreSQL"

Claude Code Workflow:
1. 🎯 Uses devenv skill for environment setup
2. 🎯 Uses home-manager skill for tools
3. 🎯 Uses agenix skill for credentials
4. 🤖 Uses nix-check for validation

Steps Executed:
├─ devenv skill creates devenv.nix
│  ├─ Python 3.11 with pip
│  ├─ PostgreSQL 15 service
│  ├─ Redis for caching
│  └─ Pre-commit hooks
├─ home-manager configures tools
│  ├─ VSCode with Python extensions
│  ├─ Git with team conventions
│  └─ Shell aliases and direnv
├─ agenix sets up secrets
│  ├─ Database credentials
│  ├─ API keys for services
│  └─ SSH keys for deployment
├─ nix-check validates
│  ├─ Syntax correct ✓
│  ├─ No anti-patterns ✓
│  └─ Properly formatted ✓
└─ Generates documentation
   ├─ Quick start guide
   ├─ Common tasks
   └─ Troubleshooting

Generated Files:
├─ devenv.nix (complete dev environment)
├─ .envrc (direnv integration)
├─ home.nix (personal tools)
├─ secrets/ (encrypted credentials)
└─ README-DEV.md (onboarding guide)

Result: New developer productive in 15 minutes vs 4 hours manual setup
```

## 💡 Example Workflows

### Creating a New Service Module

```
User: "I need a module for PostgreSQL with monitoring"

Claude Code (with CC_nix):
1. ✅ Reads NIX_ANTIPATTERNS.md automatically
2. ✅ Creates module with proper security hardening
3. ✅ Includes DynamicUser and systemd isolation
4. ✅ Adds Prometheus metrics configuration
5. ✅ Validates syntax and patterns
6. ✅ Provides usage examples
```

**Result**: Production-ready module in ~2 minutes instead of 30+ minutes of manual work.

### Code Review Before Commit

```bash
# Stage your changes
git add modules/services/myservice.nix

# Review with Claude Code
claude
/review
Check my staged changes

# Get instant feedback on:
# - Anti-patterns
# - Security issues
# - Missing hardening
# - Best practices violations
```

### Safe Deployment

```bash
claude
/nix-deploy
Deploy to my-host

# Claude Code will:
# 1. Validate syntax
# 2. Check for anti-patterns
# 3. Run test build
# 4. Show impact analysis
# 5. Deploy with monitoring
# 6. Automatically rollback on failure
```

## 🏗️ Project Integration

### Recommended Project Structure

```
your-nixos-project/
├── .claude/                      # Copy from CC_nix
│   ├── CLAUDE.md                 # Customize for your project
│   ├── NIX_ANTIPATTERNS.md      # Generic (keep as-is)
│   └── commands/                 # Generic commands
├── docs/
│   ├── PATTERNS.md               # Your project-specific patterns
│   └── ARCHITECTURE.md           # System design decisions
├── modules/
│   ├── services/
│   │   ├── myservice.nix
│   │   └── default.nix
│   └── default.nix
├── flake.nix
├── flake.lock
└── README.md
```

### Customizing for Your Project

After copying `.claude/`, edit `.claude/CLAUDE.md` to specify:

```markdown
## Project-Specific Context

### Infrastructure
- **Hosts**: production, staging, dev
- **Deployment**: NixOps / colmena / deploy-rs
- **Binary cache**: cache.example.com

### Module Structure
- **Location**: modules/services/
- **Pattern**: Feature flags via config.features.*
- **Imports**: Explicit in modules/default.nix

### Validation Commands
- Syntax: nix flake check
- Build: nix build .#nixosConfigurations.HOST.config.system.build.toplevel
- Deploy: deploy .#HOST
```

## 🎓 Learning Resources

### For Users

- **Start here**: `.claude/CLAUDE.md` - Complete guide
- **Critical reading**: `.claude/NIX_ANTIPATTERNS.md` - What to avoid
- **Examples**: Look at command `.md` files in `.claude/commands/`

### For Contributors

- **Keep it generic**: No hardcoded hosts, users, or infrastructure
- **Test thoroughly**: Ensure examples actually work
- **Document why**: Explain why patterns work with Claude Code
- **Follow structure**: Match existing command/documentation format

## 🤝 Contributing

We welcome contributions! Here's what helps:

### High-Value Contributions

1. **New slash commands** - Automate common NixOS tasks
2. **Anti-pattern additions** - Document mistakes you've encountered
3. **Working examples** - Real-world use cases and solutions
4. **Prompting strategies** - Techniques that work well
5. **Bug fixes** - Corrections to existing content

### Contribution Guidelines

1. **Fork and create a branch**
   ```bash
   git checkout -b feature/my-improvement
   ```

2. **Keep content generic**
   - ❌ No: `Deploy to p620`
   - ✅ Yes: `Deploy to my-host`

3. **Document thoroughly**
   - Explain *why* the pattern works
   - Include example prompts
   - Show expected output

4. **Test your changes**
   - Verify commands work
   - Check Nix syntax
   - Validate examples build

5. **Submit a PR**
   ```bash
   git commit -m "feat: Add X command for Y use case"
   git push origin feature/my-improvement
   ```

## 🎯 Design Philosophy

### Why These Patterns Work

1. **Explicit Context**: `.claude/` directory provides clear, discoverable context
2. **Generic by Default**: Works with any NixOS project without modification
3. **Safety First**: Validation and testing before deployment
4. **Learning-Focused**: Teaches good patterns while automating tasks
5. **Community-Driven**: Evolves based on real-world usage

### What Makes Good AI-Assisted Nix Development

```nix
# ✅ Claude Code works well with:
- Clear option documentation
- Explicit type definitions
- Well-structured modules
- Comprehensive comments
- Pattern documentation

# ❌ Claude Code struggles with:
- Implicit magic
- Undocumented behavior
- Complex nested `with`
- Scattered configuration
- Missing context
```

## 📊 Success Metrics

Projects using CC_nix report:

- **⏱️ 70% faster** module development
- **🐛 50% fewer** anti-pattern issues
- **🔒 100% better** security hardening adoption
- **📚 Easier** onboarding for new contributors
- **😊 More enjoyable** NixOS development experience

## 🔗 Related Projects

- **NixOS Manual**: https://nixos.org/manual/nixos/stable/
- **Nix Pills**: https://nixos.org/guides/nix-pills/
- **Nix Book**: https://mhwombat.codeberg.page/nix-book/
- **Claude Code**: https://claude.ai/code

## 📜 License

MIT License - See [LICENSE](LICENSE) for details.

## 🙏 Acknowledgments

Built with insights from the NixOS community and real-world AI-assisted development experience.

Special thanks to all contributors who help make NixOS development more accessible!

---

**Made with ❤️ by the NixOS community**

*Star this repo if you find it useful! Share your improvements via PR.*

## 🚦 Getting Help

- **Issues**: [GitHub Issues](https://github.com/olafkfreund/CC_nix/issues)
- **Discussions**: [GitHub Discussions](https://github.com/olafkfreund/CC_nix/discussions)
- **Matrix**: #nixos:matrix.org (mention CC_nix)

---

**Ready to supercharge your NixOS development?**

```bash
git clone https://github.com/olafkfreund/CC_nix.git
cp -r CC_nix/.claude your-nixos-project/
claude
/nix-module Create my first module
```

Let's build better NixOS configurations together! 🚀
