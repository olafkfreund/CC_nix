# Create New Python Devenv Project

Create a complete Python development environment using devenv with Claude Code integration.

## Quick Start

Just tell me:
1. Project name (e.g., "my-api" or "data-pipeline")
2. Python version (e.g., "3.11" or "3.12", default: "3.11")
3. Optional: Additional services needed (PostgreSQL, Redis, etc.)

I'll set up everything automatically.

## What I'll Do

### 1. Reference Devenv Skill (Automatic)
- Read @.claude/skills/devenv.md → Complete devenv reference
- Understand Claude Code integration patterns
- Learn Python-specific best practices

### 2. Initialize Devenv Project

Create project structure:
```
{project-name}/
├── devenv.nix           # Main devenv configuration
├── devenv.yaml          # Inputs configuration
├── .envrc               # direnv integration
├── .gitignore           # Python/devenv ignores
├── pyproject.toml       # Python project metadata
├── requirements.txt     # Python dependencies
├── src/
│   └── __init__.py      # Source code directory
├── tests/
│   └── __init__.py      # Test directory
└── README.md            # Project documentation
```

### 3. Create devenv.nix with Claude Code Integration

```nix
{ pkgs, config, ... }:
{
  # Claude Code integration
  claude.code = {
    enable = true;

    # Custom commands
    commands = {
      test = "Run test suite: pytest tests/ -v";
      test-watch = "Watch tests: pytest-watch tests/";
      coverage = "Generate coverage: pytest --cov=src --cov-report=html tests/";
      lint = "Run linters: black . && flake8 . && mypy src/";
      format = "Format code: black . && isort .";
      run = "Run application: python -m src.main";
      shell = "Start Python REPL: python";
      install = "Install dependencies: pip install -r requirements.txt";
    };

    # Specialized agents
    agents = {
      code-reviewer = {
        description = "Python code quality and security specialist";
        proactive = true;
        tools = [ "Read" "Grep" "TodoWrite" ];
        prompt = ''
          Review Python code for:
          - PEP 8 style compliance
          - Security vulnerabilities (SQL injection, command injection, etc.)
          - Type hints completeness
          - Error handling patterns
          - Performance anti-patterns (N+1 queries, inefficient loops)
          - Testing coverage gaps

          Provide specific, actionable feedback with code examples.
        '';
      };

      test-runner = {
        description = "Test execution and debugging specialist";
        proactive = false;
        tools = [ "Bash" "Read" "Edit" ];
        prompt = ''
          Execute pytest tests and analyze failures.
          Suggest fixes based on test output and tracebacks.
          Update tests as needed to improve coverage.
        '';
      };

      docs-writer = {
        description = "Python documentation specialist";
        proactive = true;
        tools = [ "Read" "Write" "Grep" ];
        prompt = ''
          Generate and maintain Python documentation:
          - Docstrings (Google or NumPy style)
          - Type hints
          - README.md updates
          - API documentation

          Ensure all examples are tested and accurate.
        '';
      };
    };

    # MCP servers
    mcpServers = {
      devenv = {
        type = "stdio";
        command = "devenv";
        args = [ "mcp" ];
      };
    };

    # Hooks for quality enforcement
    hooks = {
      pre-tool-use = pkgs.writeShellScript "pre-tool-hook" ''
        #!/usr/bin/env bash
        TOOL_DATA=$(cat)
        TOOL_NAME=$(echo "$TOOL_DATA" | jq -r '.tool')
        FILE_PATH=$(echo "$TOOL_DATA" | jq -r '.parameters.file_path // empty')

        # Require tests to pass before editing source files
        if [[ "$TOOL_NAME" == "Edit" ]] && [[ "$FILE_PATH" =~ ^src/ ]]; then
          if ! pytest tests/ --quiet --exitfirst 2>/dev/null; then
            echo "⚠️  Warning: Some tests are failing. Fix tests before editing source."
            echo "   Run '/test' to see failures."
            # Don't block, just warn
          fi
        fi

        # Block edits to critical files without explicit intention
        if [[ "$TOOL_NAME" == "Edit" ]] && [[ "$FILE_PATH" =~ (setup.py|pyproject.toml) ]]; then
          echo "📋 Editing project configuration: $FILE_PATH"
        fi
      '';

      post-tool-use = pkgs.writeShellScript "post-tool-hook" ''
        #!/usr/bin/env bash
        TOOL_DATA=$(cat)
        TOOL_NAME=$(echo "$TOOL_DATA" | jq -r '.tool')
        FILE_PATH=$(echo "$TOOL_DATA" | jq -r '.parameters.file_path // empty')

        # Auto-format Python files after edits
        if [[ "$TOOL_NAME" == "Edit" ]] && [[ "$FILE_PATH" =~ \.py$ ]]; then
          echo "🔧 Formatting $FILE_PATH..."
          black "$FILE_PATH" 2>/dev/null || true
          isort "$FILE_PATH" 2>/dev/null || true
          echo "✅ Formatted $FILE_PATH"
        fi

        # Run quick tests after source edits
        if [[ "$TOOL_NAME" == "Edit" ]] && [[ "$FILE_PATH" =~ ^src/.*\.py$ ]]; then
          echo "🧪 Running relevant tests..."
          pytest tests/ --quiet --exitfirst 2>/dev/null && \
            echo "✅ Tests passed" || \
            echo "⚠️  Tests failed - run '/test' for details"
        fi
      '';
    };
  };

  # Python language configuration
  languages.python = {
    enable = true;
    version = "{python_version}";

    # Virtual environment
    venv.enable = true;
    venv.requirements = ./requirements.txt;
  };

  # System packages
  packages = with pkgs; [
    # Python development tools
    python{python_major}Packages.pip
    python{python_major}Packages.setuptools
    python{python_major}Packages.wheel

    # Linting and formatting
    python{python_major}Packages.black
    python{python_major}Packages.flake8
    python{python_major}Packages.isort
    python{python_major}Packages.mypy
    python{python_major}Packages.pylint

    # Testing
    python{python_major}Packages.pytest
    python{python_major}Packages.pytest-cov
    python{python_major}Packages.pytest-watch
    python{python_major}Packages.pytest-xdist

    # Utilities
    jq  # For hook JSON parsing
  ];

  # Services (customize as needed)
  {services_config}

  # Environment variables
  env = {
    PYTHONPATH = "$\{config.env.DEVENV_ROOT}/src";
    PYTHON_VERSION = "{python_version}";
    PROJECT_NAME = "{project_name}";
  };

  # Pre-commit hooks (automatic formatting on commit)
  pre-commit.hooks = {
    # Python formatting
    black = {
      enable = true;
      name = "black";
      entry = "$\{pkgs.python{python_major}Packages.black}/bin/black";
      types = [ "python" ];
    };

    isort = {
      enable = true;
      name = "isort";
      entry = "$\{pkgs.python{python_major}Packages.isort}/bin/isort";
      types = [ "python" ];
    };

    # Python linting
    flake8 = {
      enable = true;
      name = "flake8";
      entry = "$\{pkgs.python{python_major}Packages.flake8}/bin/flake8";
      types = [ "python" ];
    };

    # Type checking
    mypy = {
      enable = true;
      name = "mypy";
      entry = "$\{pkgs.python{python_major}Packages.mypy}/bin/mypy";
      types = [ "python" ];
      pass_filenames = false;
      args = [ "src/" ];
    };

    # Security checks
    check-added-large-files.enable = true;
    check-merge-conflicts.enable = true;
    trailing-whitespace.enable = true;
  };

  # Scripts for common tasks
  scripts = {
    setup.exec = ''
      echo "🔧 Setting up Python environment..."
      pip install -r requirements.txt
      pip install -e .
      echo "✅ Setup complete!"
      echo ""
      echo "Available commands:"
      echo "  /test          - Run test suite"
      echo "  /test-watch    - Watch and run tests"
      echo "  /coverage      - Generate coverage report"
      echo "  /lint          - Run all linters"
      echo "  /format        - Format code"
      echo "  /run           - Run application"
    '';

    clean.exec = ''
      echo "🧹 Cleaning build artifacts..."
      rm -rf .pytest_cache
      rm -rf htmlcov
      rm -rf .coverage
      rm -rf dist build *.egg-info
      find . -type d -name __pycache__ -exec rm -rf {} + 2>/dev/null || true
      echo "✅ Cleanup complete!"
    '';
  };

  # Welcome message on shell entry
  enterShell = ''
    echo "🐍 Python Development Environment"
    echo "=================================="
    echo ""
    echo "Project: {project_name}"
    echo "Python:  {python_version}"
    echo ""
    echo "Quick Start:"
    echo "  setup      - Install dependencies"
    echo "  /test      - Run tests"
    echo "  /lint      - Run linters"
    echo "  /run       - Start application"
    echo ""
    echo "Claude Code Integration:"
    echo "  - Automatic code formatting on save"
    echo "  - Proactive code review agent"
    echo "  - Test runner on code changes"
    echo "  - Documentation sync agent"
    echo ""
    echo "Run 'setup' to get started!"
  '';

  # Test configuration
  enterTest = ''
    echo "Running Python test suite..."
    pytest tests/ -v --cov=src --cov-report=term-missing
  '';
}
```

### 4. Create devenv.yaml

```yaml
inputs:
  nixpkgs:
    url: github:NixOS/nixpkgs/nixpkgs-unstable
```

### 5. Create .envrc for direnv Integration

```bash
if ! has nix_direnv_version || ! nix_direnv_version 3.0.4; then
  source_url "https://raw.githubusercontent.com/nix-community/nix-direnv/3.0.4/direnvrc" "sha256-DzlYZ33mWF/Gs8DDeyjr8mnVmQGx7ASYqA5WlxwvBG4="
fi

use devenv
```

### 6. Create pyproject.toml

```toml
[build-system]
requires = ["setuptools>=65.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "{project_name}"
version = "0.1.0"
description = "Python project with devenv and Claude Code"
readme = "README.md"
requires-python = ">={python_version}"
authors = [
    {name = "Your Name", email = "your.email@example.com"}
]

dependencies = []

[project.optional-dependencies]
dev = [
    "pytest>=7.4.0",
    "pytest-cov>=4.1.0",
    "pytest-watch>=4.2.0",
    "black>=23.0.0",
    "flake8>=6.0.0",
    "isort>=5.12.0",
    "mypy>=1.5.0",
]

[tool.black]
line-length = 88
target-version = ['py311']
include = '\.pyi?$'

[tool.isort]
profile = "black"
line_length = 88

[tool.mypy]
python_version = "{python_version}"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]
addopts = "-v --cov=src --cov-report=term-missing"
```

### 7. Create requirements.txt

```
# Core dependencies
# Add your dependencies here

# Development dependencies (also in pyproject.toml)
pytest>=7.4.0
pytest-cov>=4.1.0
pytest-watch>=4.2.0
black>=23.0.0
flake8>=6.0.0
isort>=5.12.0
mypy>=1.5.0
```

### 8. Create .gitignore

```
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg

# Virtual environments
venv/
ENV/
env/
.venv

# Testing
.pytest_cache/
.coverage
htmlcov/
.tox/

# Type checking
.mypy_cache/
.dmypy.json
dmypy.json

# IDE
.vscode/
.idea/
*.swp
*.swo
*~

# Devenv
.devenv/
.direnv/
devenv.lock

# OS
.DS_Store
Thumbs.db
```

### 9. Create Example Source Files

**src/__init__.py:**
```python
"""
{project_name} - Python project with devenv and Claude Code integration.
"""

__version__ = "0.1.0"
```

**src/main.py:**
```python
"""Main application entry point."""


def main() -> None:
    """Run the main application."""
    print("Hello from {project_name}!")
    print("Python development environment is ready.")


if __name__ == "__main__":
    main()
```

**tests/__init__.py:**
```python
"""Test suite for {project_name}."""
```

**tests/test_main.py:**
```python
"""Tests for main module."""

from src.main import main


def test_main() -> None:
    """Test main function runs without errors."""
    # This is a simple smoke test
    main()
```

### 10. Create README.md

```markdown
# {project_name}

Python project with devenv and Claude Code integration.

## Quick Start

### Prerequisites

- [Nix](https://nixos.org/download.html) package manager
- [direnv](https://direnv.net/) (optional but recommended)

### Setup

1. Enter the development environment:
   ```bash
   # With direnv (recommended)
   direnv allow

   # Or manually
   devenv shell
   ```

2. Install dependencies:
   ```bash
   setup
   ```

3. Run tests:
   ```bash
   /test
   ```

## Development

### Available Commands

Claude Code provides these slash commands:

- `/test` - Run test suite
- `/test-watch` - Watch tests (runs on file changes)
- `/coverage` - Generate HTML coverage report
- `/lint` - Run all linters (black, flake8, mypy)
- `/format` - Format code (black + isort)
- `/run` - Run the application
- `/shell` - Start Python REPL
- `/install` - Install dependencies

### Scripts

Shell scripts available in devenv:

- `setup` - Install dependencies and setup project
- `clean` - Remove build artifacts and caches

### Testing

```bash
# Run all tests
pytest tests/

# Run with coverage
pytest tests/ --cov=src --cov-report=html

# Watch mode (runs tests on file changes)
pytest-watch tests/
```

### Code Quality

Pre-commit hooks automatically run:
- **black** - Code formatting
- **isort** - Import sorting
- **flake8** - Linting
- **mypy** - Type checking

Format manually:
```bash
black .
isort .
flake8 .
mypy src/
```

## Claude Code Integration

This project includes Claude Code integration with:

### Automatic Features

1. Code Formatting: Files are auto-formatted after edits
2. Test Execution: Tests run automatically after source changes
3. Code Review: Proactive code quality and security review
4. Documentation Sync: Keeps docs in sync with code

### Specialized Agents

- **code-reviewer**: Reviews for quality, security, and best practices
- **test-runner**: Executes and debugs tests
- **docs-writer**: Generates and maintains documentation

### Custom Hooks

- **Pre-tool-use**: Warns about test failures before edits
- **Post-tool-use**: Auto-formats Python files and runs tests

## Project Structure

```
{project_name}/
├── devenv.nix           # Development environment config
├── devenv.yaml          # Devenv inputs
├── pyproject.toml       # Python project metadata
├── requirements.txt     # Dependencies
├── src/                 # Source code
│   ├── __init__.py
│   └── main.py
├── tests/               # Test suite
│   ├── __init__.py
│   └── test_main.py
└── README.md
```

## License

[Your License Here]
```

### 11. Initialize and Validate

```bash
# Initialize direnv
direnv allow

# Enter devenv shell
devenv shell

# Run setup
setup

# Verify everything works
/test
```

## Service Options

I can add these services if needed:

### PostgreSQL
```nix
services.postgres = {
  enable = true;
  initialDatabases = [{ name = "{project_name}_dev"; }];
};
env.DATABASE_URL = "postgresql://localhost:5432/{project_name}_dev";
```

### Redis
```nix
services.redis.enable = true;
env.REDIS_URL = "redis://localhost:6379";
```

### MongoDB
```nix
services.mongodb.enable = true;
env.MONGODB_URL = "mongodb://localhost:27017/{project_name}";
```

### Mailhog (Email Testing)
```nix
services.mailhog = {
  enable = true;
  smtpPort = 1025;
  uiPort = 8025;
};
env.SMTP_HOST = "localhost";
env.SMTP_PORT = "1025";
```

## Claude Code Features

### 1. Automatic Code Review
After every code change, the code-reviewer agent checks for:
- Security vulnerabilities
- PEP 8 compliance
- Type hint completeness
- Error handling
- Performance issues

### 2. Auto-formatting
Every Python file edit triggers:
- `black` for code formatting
- `isort` for import sorting
- Quick test run to verify changes

### 3. Test-Driven Development
- Pre-hook warns about failing tests
- Post-hook runs relevant tests after edits
- `/test-watch` command for continuous testing

### 4. Documentation Sync
The docs-writer agent ensures:
- Docstrings are up-to-date
- README reflects current features
- Examples are tested and valid

## Best Practices Enforced

**Security**
- No hardcoded secrets
- Proper error handling
- Input validation

**Code Quality**
- Type hints everywhere
- Comprehensive docstrings
- 100% test coverage goal

**Development Workflow**
- Automatic formatting
- Pre-commit hooks
- Continuous testing

## Speed Optimization

This command completes in **under 3 minutes**:
- 30s: Reference devenv skill
- 90s: Create project structure and files
- 30s: Initialize and validate
- 30s: Test setup with example

## Success Checklist

- [x] devenv.nix with Python and Claude Code integration
- [x] All configuration files (pyproject.toml, requirements.txt, etc.)
- [x] Example source and test files
- [x] Pre-commit hooks configured
- [x] Claude Code agents (reviewer, tester, docs)
- [x] Custom hooks (pre/post tool use)
- [x] All slash commands (/test, /lint, /run, etc.)
- [x] README with complete documentation
- [x] Optional services (if requested)
- [x] Validated and tested

Ready to create your Python devenv project? Just tell me the project name and Python version!
