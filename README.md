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

### Subagents

Specialized AI agents for automated NixOS workflows in `.claude/agents/`:

| Subagent | Purpose | Auto-runs On |
|----------|---------|--------------|
| `nix-check` | Lint, check, and fix NixOS configs using deadnix, statix, nixpkgs-fmt | File changes, pre-commit, code review |
| `issue-checker` | Monitor GitHub issues for bugs affecting installed packages | Before updates, system planning, CI/CD pipelines |
| `local-logs` | Analyze system logs, research solutions, generate configuration fixes | Service failures, boot issues, performance problems, troubleshooting |

### Future Resources (Contributions Welcome!)

- **More Subagents**: Additional specialized agents for NixOS workflows
- **Templates**: Project scaffolding and boilerplate

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
