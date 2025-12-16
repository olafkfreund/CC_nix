# Create New Go Devenv Project

Create a complete Go development environment using devenv with Claude Code integration.

## Quick Start

Just tell me:
1. Project name (e.g., "my-api" or "cli-tool")
2. Module path (e.g., "github.com/username/my-api")
3. Go version (e.g., "1.21" or "1.22", default: "1.21")
4. Optional: Additional services needed (PostgreSQL, Redis, etc.)

I'll set up everything automatically.

## What I'll Do

### 1. Reference Devenv Skill (Automatic)
- Read @.claude/skills/devenv.md → Complete devenv reference
- Understand Claude Code integration patterns
- Learn Go-specific best practices

### 2. Initialize Devenv Project

Create project structure:
```
{project-name}/
├── devenv.nix           # Main devenv configuration
├── devenv.yaml          # Inputs configuration
├── .envrc               # direnv integration
├── .gitignore           # Go/devenv ignores
├── go.mod               # Go module definition
├── go.sum               # Go dependency checksums
├── main.go              # Main application entry
├── cmd/
│   └── {project-name}/
│       └── main.go      # CLI entry point
├── internal/            # Private application code
│   └── .gitkeep
├── pkg/                 # Public library code
│   └── .gitkeep
├── api/                 # API definitions (OpenAPI, protobuf)
│   └── .gitkeep
├── web/                 # Web assets (if needed)
│   └── .gitkeep
├── scripts/             # Build and development scripts
│   └── .gitkeep
├── docs/                # Documentation
│   └── .gitkeep
├── Makefile             # Build targets
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
      build = "Build application: go build -v ./...";
      run = "Run application: go run ./cmd/{project_name}";
      test = "Run tests: go test -v ./...";
      test-coverage = "Generate coverage: go test -v -coverprofile=coverage.out ./... && go tool cover -html=coverage.out -o coverage.html";
      test-race = "Run race detector: go test -race ./...";
      bench = "Run benchmarks: go test -bench=. -benchmem ./...";
      lint = "Run linter: golangci-lint run ./...";
      fmt = "Format code: gofmt -s -w . && goimports -w .";
      vet = "Run go vet: go vet ./...";
      tidy = "Tidy dependencies: go mod tidy";
      vendor = "Vendor dependencies: go mod vendor";
      install = "Install dependencies: go mod download";
      generate = "Run go generate: go generate ./...";
    };

    # Specialized agents
    agents = {
      code-reviewer = {
        description = "Go code quality and performance specialist";
        proactive = true;
        tools = [ "Read" "Grep" "TodoWrite" ];
        prompt = ''
          Review Go code for:
          - Idiomatic Go patterns and best practices
          - Error handling (proper error wrapping, checking)
          - Goroutine and channel usage (leaks, deadlocks)
          - Interface design and composition
          - Performance issues (allocations, string concatenation)
          - Security vulnerabilities (SQL injection, command injection)
          - Race conditions and concurrency bugs
          - Resource management (defer, Close())
          - Testing coverage and table-driven tests

          Provide specific, actionable feedback with code examples.
          Reference Effective Go and Go Code Review Comments.
        '';
      };

      test-runner = {
        description = "Go test execution and debugging specialist";
        proactive = false;
        tools = [ "Bash" "Read" "Edit" ];
        prompt = ''
          Execute Go tests and analyze failures.
          Run race detector for concurrency issues.
          Suggest fixes based on test output and stack traces.
          Generate table-driven test templates.
          Update tests to improve coverage.
        '';
      };

      docs-writer = {
        description = "Go documentation specialist";
        proactive = true;
        tools = [ "Read" "Write" "Grep" ];
        prompt = ''
          Generate and maintain Go documentation:
          - Package documentation comments
          - Exported function/type documentation
          - Code examples in comments (testable examples)
          - README.md updates
          - API documentation

          Follow Go documentation conventions.
          Ensure examples compile and run correctly.
        '';
      };

      performance-optimizer = {
        description = "Go performance optimization specialist";
        proactive = false;
        tools = [ "Bash" "Read" "Grep" ];
        prompt = ''
          Analyze Go code for performance:
          - Run benchmarks and identify bottlenecks
          - Profile CPU and memory usage
          - Suggest optimization strategies
          - Identify unnecessary allocations
          - Recommend better data structures

          Use pprof for profiling analysis.
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

        # Warn about build failures before editing
        if [[ "$TOOL_NAME" == "Edit" ]] && [[ "$FILE_PATH" =~ \.go$ ]]; then
          if ! go build ./... 2>/dev/null; then
            echo "⚠️  Warning: Build is failing. Consider fixing build errors first."
            echo "   Run '/build' to see errors."
          fi
        fi

        # Check for race conditions before major changes
        if [[ "$TOOL_NAME" == "Edit" ]] && [[ "$FILE_PATH" =~ (internal|pkg)/.*\.go$ ]]; then
          echo "💡 Tip: Run '/test-race' after changes to check for race conditions."
        fi
      '';

      post-tool-use = pkgs.writeShellScript "post-tool-hook" ''
        #!/usr/bin/env bash
        TOOL_DATA=$(cat)
        TOOL_NAME=$(echo "$TOOL_DATA" | jq -r '.tool')
        FILE_PATH=$(echo "$TOOL_DATA" | jq -r '.parameters.file_path // empty')

        # Auto-format Go files after edits
        if [[ "$TOOL_NAME" == "Edit" ]] && [[ "$FILE_PATH" =~ \.go$ ]]; then
          echo "🔧 Formatting $FILE_PATH..."
          gofmt -s -w "$FILE_PATH" 2>/dev/null || true
          goimports -w "$FILE_PATH" 2>/dev/null || true
          echo "✅ Formatted $FILE_PATH"
        fi

        # Run quick build check after Go file edits
        if [[ "$TOOL_NAME" == "Edit" ]] && [[ "$FILE_PATH" =~ \.go$ ]]; then
          echo "🔨 Quick build check..."
          if go build ./... 2>/dev/null; then
            echo "✅ Build successful"
          else
            echo "❌ Build failed - run '/build' for details"
          fi
        fi

        # Run tests for the affected package
        if [[ "$TOOL_NAME" == "Edit" ]] && [[ "$FILE_PATH" =~ \.go$ ]]; then
          PACKAGE_DIR=$(dirname "$FILE_PATH")
          if [[ -n "$PACKAGE_DIR" ]]; then
            echo "🧪 Running tests for $PACKAGE_DIR..."
            if go test "./$PACKAGE_DIR" -short 2>/dev/null; then
              echo "✅ Tests passed"
            else
              echo "⚠️  Tests failed - run '/test' for details"
            fi
          fi
        fi
      '';
    };
  };

  # Go language configuration
  languages.go = {
    enable = true;
    package = pkgs.go_{go_version};
  };

  # System packages
  packages = with pkgs; [
    # Go tools
    go_{go_version}
    gopls              # Go language server
    gotools            # godoc, goimports, etc.
    go-tools           # staticcheck, etc.

    # Linting and formatting
    golangci-lint      # Comprehensive linter
    gofumpt            # Stricter gofmt
    golines            # Line length formatter

    # Testing
    gotestsum          # Better test output
    gotest             # Colorized test output

    # Code analysis
    staticcheck        # Static analysis
    go-critic          # Code critic
    errcheck           # Check error returns
    ineffassign        # Detect ineffectual assignments

    # Debugging
    delve              # Go debugger

    # Documentation
    godoc              # Documentation server

    # Build tools
    goreleaser         # Release automation
    ko                 # Container builder

    # Utilities
    jq                 # JSON parsing for hooks

    # Protobuf (if using gRPC)
    protobuf
    protoc-gen-go
    protoc-gen-go-grpc
  ];

  # Services (customize as needed)
  {services_config}

  # Environment variables
  env = {
    GOPATH = "$\{config.env.DEVENV_ROOT}/.go";
    GOBIN = "$\{config.env.DEVENV_ROOT}/.go/bin";
    GOCACHE = "$\{config.env.DEVENV_ROOT}/.cache/go-build";
    GOMODCACHE = "$\{config.env.DEVENV_ROOT}/.cache/go/mod";
    GO_VERSION = "{go_version}";
    PROJECT_NAME = "{project_name}";
    CGO_ENABLED = "1";  # Enable CGO by default
  };

  # Pre-commit hooks (automatic on commit)
  pre-commit.hooks = {
    # Go formatting
    gofmt = {
      enable = true;
      name = "gofmt";
      entry = "$\{pkgs.go_{go_version}}/bin/gofmt -s -w";
      types = [ "go" ];
    };

    goimports = {
      enable = true;
      name = "goimports";
      entry = "$\{pkgs.gotools}/bin/goimports -w";
      types = [ "go" ];
    };

    # Go vetting
    govet = {
      enable = true;
      name = "go vet";
      entry = "$\{pkgs.go_{go_version}}/bin/go vet";
      types = [ "go" ];
      pass_filenames = false;
    };

    # Linting
    golangci-lint = {
      enable = true;
      name = "golangci-lint";
      entry = "$\{pkgs.golangci-lint}/bin/golangci-lint run --fix";
      types = [ "go" ];
      pass_filenames = false;
    };

    # Tests
    go-test = {
      enable = true;
      name = "go test";
      entry = "$\{pkgs.go_{go_version}}/bin/go test -short";
      types = [ "go" ];
      pass_filenames = false;
    };

    # Module tidying
    go-mod-tidy = {
      enable = true;
      name = "go mod tidy";
      entry = "$\{pkgs.go_{go_version}}/bin/go mod tidy";
      files = "go\\.mod$";
      pass_filenames = false;
    };

    # Security checks
    check-added-large-files.enable = true;
    check-merge-conflicts.enable = true;
    trailing-whitespace.enable = true;
  };

  # Scripts for common tasks
  scripts = {
    setup.exec = ''
      echo "🔧 Setting up Go environment..."
      go mod download
      go mod verify
      echo "✅ Dependencies downloaded and verified!"
      echo ""
      echo "Available commands:"
      echo "  /build         - Build application"
      echo "  /run           - Run application"
      echo "  /test          - Run tests"
      echo "  /test-coverage - Generate coverage report"
      echo "  /test-race     - Run race detector"
      echo "  /bench         - Run benchmarks"
      echo "  /lint          - Run linter"
      echo "  /fmt           - Format code"
    '';

    clean.exec = ''
      echo "🧹 Cleaning build artifacts..."
      go clean -cache -testcache -modcache
      rm -rf .go .cache
      rm -f coverage.out coverage.html
      rm -f cpu.prof mem.prof
      echo "✅ Cleanup complete!"
    '';

    profile-cpu.exec = ''
      echo "📊 Profiling CPU usage..."
      go test -cpuprofile=cpu.prof -bench=. ./...
      go tool pprof -http=:8080 cpu.prof
    '';

    profile-mem.exec = ''
      echo "📊 Profiling memory usage..."
      go test -memprofile=mem.prof -bench=. ./...
      go tool pprof -http=:8080 mem.prof
    '';
  };

  # Welcome message on shell entry
  enterShell = ''
    echo "🐹 Go Development Environment"
    echo "=============================="
    echo ""
    echo "Project: {project_name}"
    echo "Go:      {go_version}"
    echo "Module:  {module_path}"
    echo ""
    echo "Quick Start:"
    echo "  setup          - Download dependencies"
    echo "  /build         - Build application"
    echo "  /run           - Run application"
    echo "  /test          - Run tests"
    echo "  /lint          - Run linter"
    echo ""
    echo "Claude Code Integration:"
    echo "  - Automatic code formatting on save"
    echo "  - Proactive code review for Go idioms"
    echo "  - Race detector reminders"
    echo "  - Performance optimization suggestions"
    echo ""
    echo "GOPATH: $GOPATH"
    echo "GOBIN:  $GOBIN"
    echo ""
    echo "Run 'setup' to get started!"
  '';

  # Test configuration
  enterTest = ''
    echo "Running Go test suite..."
    go test -v -race -coverprofile=coverage.out ./...
    go tool cover -func=coverage.out
  '';

  # Processes (if running services)
  processes = {
    # Uncomment if you have a server
    # server.exec = "go run ./cmd/{project_name}";
  };
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

### 6. Initialize Go Module

```bash
go mod init {module_path}
```

**go.mod:**
```go
module {module_path}

go {go_version}

require (
    // Add your dependencies here
)
```

### 7. Create Main Application Files

**main.go:**
```go
package main

import (
    "fmt"
    "log"

    "{module_path}/internal"
)

func main() {
    fmt.Println("Hello from {project_name}!")
    log.Println("Go development environment is ready.")
}
```

**cmd/{project_name}/main.go:**
```go
package main

import (
    "context"
    "flag"
    "fmt"
    "log"
    "os"
    "os/signal"
    "syscall"
)

var (
    version = "dev"
    commit  = "none"
    date    = "unknown"
)

func main() {
    // Parse flags
    var (
        showVersion = flag.Bool("version", false, "show version information")
    )
    flag.Parse()

    if *showVersion {
        fmt.Printf("%s version %s (commit: %s, built: %s)\n",
            "{project_name}", version, commit, date)
        os.Exit(0)
    }

    // Setup context with signal handling
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()

    // Handle shutdown signals
    sigCh := make(chan os.Signal, 1)
    signal.Notify(sigCh, os.Interrupt, syscall.SIGTERM)

    go func() {
        <-sigCh
        log.Println("Shutting down gracefully...")
        cancel()
    }()

    // Run application
    if err := run(ctx); err != nil {
        log.Fatalf("Application error: %v", err)
    }
}

func run(ctx context.Context) error {
    log.Printf("Starting %s...", "{project_name}")

    // Your application logic here
    <-ctx.Done()

    log.Println("Application stopped")
    return nil
}
```

**internal/app.go:**
```go
package internal

// Package internal contains private application code
// that should not be imported by other projects.
```

**pkg/example/example.go:**
```go
package example

// Add returns the sum of a and b.
func Add(a, b int) int {
    return a + b
}
```

**pkg/example/example_test.go:**
```go
package example

import "testing"

func TestAdd(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positive numbers", 2, 3, 5},
        {"negative numbers", -2, -3, -5},
        {"mixed signs", -2, 3, 1},
        {"zeros", 0, 0, 0},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := Add(tt.a, tt.b)
            if result != tt.expected {
                t.Errorf("Add(%d, %d) = %d; want %d",
                    tt.a, tt.b, result, tt.expected)
            }
        })
    }
}

// Benchmark for Add function
func BenchmarkAdd(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Add(1, 2)
    }
}

// Example function (appears in godoc)
func ExampleAdd() {
    result := Add(2, 3)
    println(result)
    // Output: 5
}
```

### 8. Create Makefile

```makefile
.PHONY: help build run test test-coverage test-race bench lint fmt clean install dev

# Default target
help:
	@echo "Available targets:"
	@echo "  build         - Build the application"
	@echo "  run           - Run the application"
	@echo "  test          - Run tests"
	@echo "  test-coverage - Generate test coverage report"
	@echo "  test-race     - Run tests with race detector"
	@echo "  bench         - Run benchmarks"
	@echo "  lint          - Run linters"
	@echo "  fmt           - Format code"
	@echo "  clean         - Clean build artifacts"
	@echo "  install       - Install dependencies"
	@echo "  dev           - Run in development mode with auto-reload"

build:
	go build -v -o bin/{project_name} ./cmd/{project_name}

run:
	go run ./cmd/{project_name}

test:
	go test -v ./...

test-coverage:
	go test -v -coverprofile=coverage.out ./...
	go tool cover -html=coverage.out -o coverage.html
	@echo "Coverage report: coverage.html"

test-race:
	go test -race ./...

bench:
	go test -bench=. -benchmem ./...

lint:
	golangci-lint run ./...

fmt:
	gofmt -s -w .
	goimports -w .

clean:
	go clean -cache -testcache -modcache
	rm -rf bin/ .go/ .cache/
	rm -f coverage.out coverage.html

install:
	go mod download
	go mod verify

dev:
	go run ./cmd/{project_name}
```

### 9. Create .gitignore

```
# Binaries
bin/
*.exe
*.exe~
*.dll
*.so
*.dylib

# Test binary
*.test

# Output of the go coverage tool
*.out
coverage.html

# Go workspace file
go.work

# Dependency directories
vendor/

# Go build cache
.go/
.cache/

# Environment
.env
.env.local

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

# Profiling
*.prof
cpu.prof
mem.prof

# Debugging
debug
__debug_bin
```

### 10. Create .golangci.yml (Linter Configuration)

```yaml
linters:
  enable:
    - errcheck       # Check error returns
    - gosimple       # Simplify code
    - govet          # Go vet
    - ineffassign    # Detect ineffectual assignments
    - staticcheck    # Static analysis
    - unused         # Detect unused code
    - gofmt          # Check formatting
    - goimports      # Check imports
    - misspell       # Fix spelling
    - revive         # Golint replacement
    - gosec          # Security issues
    - unconvert      # Unnecessary conversions
    - gocyclo        # Cyclomatic complexity
    - gocritic       # Code critic
    - godox          # Detect FIXME/TODO
    - bodyclose      # Check HTTP body close
    - noctx          # Detect missing context
    - rowserrcheck   # Check sql.Rows.Err
    - sqlclosecheck  # Check SQL close

linters-settings:
  gocyclo:
    min-complexity: 15
  govet:
    check-shadowing: true
  misspell:
    locale: US
  revive:
    confidence: 0.8

issues:
  exclude-use-default: false
  max-issues-per-linter: 0
  max-same-issues: 0

run:
  timeout: 5m
  tests: true
```

### 11. Create README.md

```markdown
# {project_name}

Go application with devenv and Claude Code integration.

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

3. Build the application:
   ```bash
   /build
   ```

4. Run tests:
   ```bash
   /test
   ```

## Development

### Available Commands

Claude Code provides these slash commands:

- `/build` - Build the application
- `/run` - Run the application
- `/test` - Run tests
- `/test-coverage` - Generate HTML coverage report
- `/test-race` - Run tests with race detector
- `/bench` - Run benchmarks
- `/lint` - Run golangci-lint
- `/fmt` - Format code (gofmt + goimports)
- `/vet` - Run go vet
- `/tidy` - Tidy go.mod
- `/install` - Download dependencies
- `/generate` - Run go generate

### Scripts

Shell scripts available in devenv:

- `setup` - Download and verify dependencies
- `clean` - Remove build artifacts and caches
- `profile-cpu` - Profile CPU usage
- `profile-mem` - Profile memory usage

### Testing

```bash
# Run all tests
go test ./...

# Run with coverage
go test -cover ./...

# Run with race detector
go test -race ./...

# Run benchmarks
go test -bench=. ./...
```

### Code Quality

Pre-commit hooks automatically run:
- **gofmt** - Code formatting
- **goimports** - Import organization
- **go vet** - Static analysis
- **golangci-lint** - Comprehensive linting
- **go test** - Quick tests
- **go mod tidy** - Module cleanup

Format manually:
```bash
gofmt -s -w .
goimports -w .
golangci-lint run ./...
```

### Building

```bash
# Development build
go build -o bin/{project_name} ./cmd/{project_name}

# Production build with optimizations
go build -ldflags="-s -w" -o bin/{project_name} ./cmd/{project_name}

# Cross-compilation
GOOS=linux GOARCH=amd64 go build -o bin/{project_name}-linux-amd64 ./cmd/{project_name}
```

## Claude Code Integration

This project includes Claude Code integration with:

### Automatic Features

1. Code Formatting: Go files auto-formatted after edits (gofmt + goimports)
2. Build Verification: Quick build check after changes
3. Test Execution: Package tests run automatically after edits
4. Race Detection: Reminders to check for race conditions

### Specialized Agents

- **code-reviewer**: Reviews for Go idioms, errors, concurrency, performance
- **test-runner**: Executes and debugs tests, generates test templates
- **docs-writer**: Maintains package documentation and examples
- **performance-optimizer**: Analyzes benchmarks and suggests optimizations

### Custom Hooks

- **Pre-tool-use**: Warns about build failures, suggests race detector
- **Post-tool-use**: Auto-formats, builds, and tests affected packages

## Project Structure

```
{project_name}/
├── cmd/                    # Command-line applications
│   └── {project_name}/
│       └── main.go        # Main entry point
├── internal/              # Private application code
├── pkg/                   # Public library code
│   └── example/
│       ├── example.go
│       └── example_test.go
├── api/                   # API definitions (OpenAPI, protobuf)
├── web/                   # Web assets
├── scripts/               # Build and development scripts
├── docs/                  # Documentation
├── devenv.nix            # Development environment
├── go.mod                # Go module definition
├── Makefile              # Build targets
└── README.md
```

## Go Best Practices

This project follows:

- **Effective Go**: https://go.dev/doc/effective_go
- **Go Code Review Comments**: https://go.dev/wiki/CodeReviewComments
- **Standard Project Layout**: https://github.com/golang-standards/project-layout

### Key Patterns

- Error wrapping with `fmt.Errorf("%w", err)`
- Table-driven tests
- Context-aware functions
- Graceful shutdown handling
- Interface-based design

## Performance

### Profiling

```bash
# CPU profiling
profile-cpu

# Memory profiling
profile-mem

# Custom profiling
go test -cpuprofile=cpu.prof -memprofile=mem.prof -bench=. ./...
go tool pprof cpu.prof
```

### Benchmarking

```bash
# Run all benchmarks
/bench

# Benchmark specific package
go test -bench=. -benchmem ./pkg/example

# Compare benchmarks
go test -bench=. ./... > old.txt
# Make changes
go test -bench=. ./... > new.txt
benchcmp old.txt new.txt
```

## License

[Your License Here]
```

### 12. Initialize and Validate

```bash
# Initialize Go module
go mod init {module_path}

# Initialize direnv
direnv allow

# Enter devenv shell
devenv shell

# Run setup
setup

# Build and test
/build
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

### NATS (Message Broker)
```nix
services.nats = {
  enable = true;
  port = 4222;
};
env.NATS_URL = "nats://localhost:4222";
```

### Kafka
```nix
services.kafka = {
  enable = true;
  port = 9092;
};
env.KAFKA_BROKERS = "localhost:9092";
```

## Claude Code Features

### 1. Automatic Code Review
After every code change, the code-reviewer agent checks for:
- Idiomatic Go patterns
- Proper error handling and wrapping
- Goroutine leaks and race conditions
- Interface design and composition
- Performance anti-patterns
- Security vulnerabilities

### 2. Auto-formatting and Building
Every Go file edit triggers:
- `gofmt` for formatting
- `goimports` for import organization
- Quick build verification
- Package test execution

### 3. Performance Optimization
The performance-optimizer agent:
- Runs benchmarks
- Analyzes profiles (CPU, memory)
- Suggests optimization strategies
- Identifies unnecessary allocations

### 4. Documentation Sync
The docs-writer agent ensures:
- Package documentation is complete
- Exported functions are documented
- Examples are testable and accurate
- README reflects current features

## Best Practices Enforced

**Idiomatic Go**
- Proper error handling
- Context usage
- Interface-based design
- Channel and goroutine patterns

**Testing**
- Table-driven tests
- Testable examples
- Benchmark tests
- Race detector

**Performance**
- Profiling integration
- Benchmark tracking
- Allocation awareness

**Code Quality**
- Comprehensive linting
- Static analysis
- Security scanning

## Speed Optimization

This command completes in **under 3 minutes**:
- 30s: Reference devenv skill
- 90s: Create project structure and files
- 30s: Initialize Go module and devenv
- 30s: Build and test

## Success Checklist

- [x] devenv.nix with Go and Claude Code integration
- [x] Go module initialized (go.mod)
- [x] Project structure (cmd/, internal/, pkg/)
- [x] Example code with tests and benchmarks
- [x] Makefile with build targets
- [x] golangci-lint configuration
- [x] Pre-commit hooks configured
- [x] Claude Code agents (reviewer, tester, docs, optimizer)
- [x] Custom hooks (pre/post tool use)
- [x] All slash commands configured
- [x] README with complete documentation
- [x] Optional services (if requested)
- [x] Validated and tested

Ready to create your Go devenv project? Just tell me the project name, module path, and Go version!
