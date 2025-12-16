# Create New Rust Devenv Project

Create a complete Rust development environment using devenv with Claude Code integration.

## Quick Start

Just tell me:
1. Project name (e.g., "my-cli" or "web-server")
2. Project type: "bin" (binary/application) or "lib" (library)
3. Rust channel (e.g., "stable", "nightly", "beta", default: "stable")
4. Optional: Additional services needed (PostgreSQL, Redis, etc.)

I'll set up everything automatically.

## What I'll Do

### 1. Reference Devenv Skill (Automatic)
- Read @.claude/skills/devenv.md → Complete devenv reference
- Understand Claude Code integration patterns
- Learn Rust-specific best practices

### 2. Initialize Devenv Project

Create project structure:
```
{project-name}/
├── devenv.nix           # Main devenv configuration
├── devenv.yaml          # Inputs configuration
├── .envrc               # direnv integration
├── .gitignore           # Rust/devenv ignores
├── Cargo.toml           # Rust project manifest
├── Cargo.lock           # Dependency lock file
├── rust-toolchain.toml  # Rust toolchain specification
├── src/
│   ├── lib.rs           # Library root (if lib)
│   └── main.rs          # Binary root (if bin)
├── tests/               # Integration tests
│   └── integration_test.rs
├── benches/             # Benchmarks
│   └── benchmark.rs
├── examples/            # Example programs
│   └── example.rs
├── docs/                # Documentation
│   └── .gitkeep
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
      build = "Build project: cargo build";
      build-release = "Build release: cargo build --release";
      run = "Run project: cargo run";
      run-release = "Run release: cargo run --release";
      test = "Run tests: cargo test";
      test-all = "Run all tests: cargo test --all-features";
      test-doc = "Test documentation: cargo test --doc";
      bench = "Run benchmarks: cargo bench";
      check = "Quick check: cargo check";
      check-all = "Check all targets: cargo check --all-targets --all-features";
      clippy = "Run clippy: cargo clippy -- -D warnings";
      clippy-fix = "Auto-fix clippy: cargo clippy --fix --allow-dirty";
      fmt = "Format code: cargo fmt";
      fmt-check = "Check formatting: cargo fmt -- --check";
      doc = "Generate docs: cargo doc --no-deps --open";
      doc-private = "Generate all docs: cargo doc --document-private-items --no-deps --open";
      clean = "Clean build artifacts: cargo clean";
      update = "Update dependencies: cargo update";
      outdated = "Check outdated deps: cargo outdated";
      audit = "Security audit: cargo audit";
      expand = "Expand macros: cargo expand";
      watch = "Watch and rebuild: cargo watch -x check -x test";
    };

    # Specialized agents
    agents = {
      code-reviewer = {
        description = "Rust code quality and safety specialist";
        proactive = true;
        tools = [ "Read" "Grep" "TodoWrite" ];
        prompt = ''
          Review Rust code for:
          - Ownership, borrowing, and lifetime issues
          - Unsafe code usage and safety invariants
          - Error handling patterns (Result, Option, ?)
          - Memory safety and resource management
          - Idiomatic Rust patterns (iterators, traits, etc.)
          - Performance issues (unnecessary clones, allocations)
          - Clippy lint violations
          - Type design and API ergonomics
          - Concurrency patterns (Send, Sync, Arc, Mutex)
          - Zero-cost abstractions usage

          Provide specific, actionable feedback with code examples.
          Reference The Rust Programming Language book and Rust API Guidelines.
        '';
      };

      test-runner = {
        description = "Rust test execution and debugging specialist";
        proactive = false;
        tools = [ "Bash" "Read" "Edit" ];
        prompt = ''
          Execute cargo tests and analyze failures.
          Run with --nocapture for debugging output.
          Suggest fixes based on test failures and panics.
          Generate unit test templates.
          Update tests to improve coverage.
          Run doc tests to verify examples.
        '';
      };

      docs-writer = {
        description = "Rust documentation specialist";
        proactive = true;
        tools = [ "Read" "Write" "Grep" ];
        prompt = ''
          Generate and maintain Rust documentation:
          - Module-level documentation (//!)
          - Function/struct documentation (///)
          - Doctests with examples
          - README.md updates
          - CHANGELOG.md maintenance
          - API documentation

          Follow Rust documentation conventions.
          Ensure all public items are documented.
          Include safety documentation for unsafe code.
        '';
      };

      performance-optimizer = {
        description = "Rust performance optimization specialist";
        proactive = false;
        tools = [ "Bash" "Read" "Grep" ];
        prompt = ''
          Analyze Rust code for performance:
          - Run cargo bench for benchmarks
          - Identify unnecessary clones and allocations
          - Suggest zero-cost abstractions
          - Recommend efficient data structures
          - Profile with flamegraph
          - Check for unnecessary heap usage
          - Optimize hot paths

          Focus on maintainability first, then performance.
        '';
      };

      unsafe-auditor = {
        description = "Unsafe code auditing specialist";
        proactive = true;
        tools = [ "Read" "Grep" "TodoWrite" ];
        prompt = ''
          Audit unsafe code blocks:
          - Verify safety invariants are documented
          - Check for undefined behavior risks
          - Suggest safe alternatives when possible
          - Ensure unsafe is minimally scoped
          - Review FFI boundary safety
          - Check raw pointer usage

          Create TodoWrite items for unsafe code that needs review.
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
        if [[ "$TOOL_NAME" == "Edit" ]] && [[ "$FILE_PATH" =~ \.rs$ ]]; then
          if ! cargo check --quiet 2>/dev/null; then
            echo "⚠️  Warning: Project doesn't compile. Fix errors first."
            echo "   Run '/check' to see compilation errors."
          fi
        fi

        # Check for unsafe code edits
        if [[ "$TOOL_NAME" == "Edit" ]] && [[ "$FILE_PATH" =~ \.rs$ ]]; then
          if grep -q "unsafe" "$FILE_PATH" 2>/dev/null; then
            echo "⚠️  This file contains unsafe code."
            echo "   Ensure safety invariants are documented."
          fi
        fi
      '';

      post-tool-use = pkgs.writeShellScript "post-tool-hook" ''
        #!/usr/bin/env bash
        TOOL_DATA=$(cat)
        TOOL_NAME=$(echo "$TOOL_DATA" | jq -r '.tool')
        FILE_PATH=$(echo "$TOOL_DATA" | jq -r '.parameters.file_path // empty')

        # Auto-format Rust files after edits
        if [[ "$TOOL_NAME" == "Edit" ]] && [[ "$FILE_PATH" =~ \.rs$ ]]; then
          echo "🔧 Formatting $FILE_PATH..."
          cargo fmt -- "$FILE_PATH" 2>/dev/null || true
          echo "✅ Formatted $FILE_PATH"
        fi

        # Run clippy on edited files
        if [[ "$TOOL_NAME" == "Edit" ]] && [[ "$FILE_PATH" =~ \.rs$ ]]; then
          echo "🔍 Running clippy..."
          if cargo clippy --quiet 2>/dev/null; then
            echo "✅ Clippy passed"
          else
            echo "⚠️  Clippy warnings - run '/clippy' for details"
          fi
        fi

        # Quick check after edits
        if [[ "$TOOL_NAME" == "Edit" ]] && [[ "$FILE_PATH" =~ \.rs$ ]]; then
          echo "🔨 Quick compile check..."
          if cargo check --quiet 2>/dev/null; then
            echo "✅ Compiles successfully"
          else
            echo "❌ Compilation failed - run '/check' for details"
          fi
        fi
      '';
    };
  };

  # Rust language configuration
  languages.rust = {
    enable = true;
    channel = "{rust_channel}";  # stable, nightly, or beta

    # Components
    components = [ "rustc" "cargo" "clippy" "rustfmt" "rust-analyzer" ];

    # Additional targets (for cross-compilation)
    # targets = [ "wasm32-unknown-unknown" ];
  };

  # System packages
  packages = with pkgs; [
    # Rust core tools (provided by languages.rust)

    # Additional Rust tools
    cargo-watch        # Auto-rebuild on file changes
    cargo-edit         # cargo add/rm/upgrade commands
    cargo-outdated     # Check for outdated dependencies
    cargo-audit        # Security vulnerability scanner
    cargo-expand       # Expand macros
    cargo-flamegraph   # Flamegraph profiler
    cargo-bloat        # Find what takes space in binary
    cargo-udeps        # Find unused dependencies
    cargo-deny         # Dependency linting
    cargo-nextest      # Next-generation test runner
    cargo-llvm-cov     # Code coverage with LLVM
    cargo-criterion    # Benchmarking
    bacon              # Background rust code checker

    # Cross-compilation (if needed)
    # cargo-cross

    # Documentation
    mdbook             # Build books from markdown

    # Utilities
    jq                 # JSON parsing for hooks

    # Protobuf (if using gRPC/protobuf)
    # protobuf
    # protoc-gen-rust
  ];

  # Services (customize as needed)
  {services_config}

  # Environment variables
  env = {
    RUST_BACKTRACE = "1";
    RUST_LOG = "info";
    CARGO_HOME = "$\{config.env.DEVENV_ROOT}/.cargo";
    CARGO_TARGET_DIR = "$\{config.env.DEVENV_ROOT}/target";
    PROJECT_NAME = "{project_name}";
    RUST_CHANNEL = "{rust_channel}";
  };

  # Pre-commit hooks (automatic on commit)
  pre-commit.hooks = {
    # Rust formatting
    rustfmt = {
      enable = true;
      name = "rustfmt";
      entry = "cargo fmt --";
      types = [ "rust" ];
    };

    # Clippy linting
    clippy = {
      enable = true;
      name = "clippy";
      entry = "cargo clippy --all-targets --all-features -- -D warnings";
      pass_filenames = false;
      types = [ "rust" ];
    };

    # Quick check
    cargo-check = {
      enable = true;
      name = "cargo check";
      entry = "cargo check --all-targets --all-features";
      pass_filenames = false;
      types = [ "rust" ];
    };

    # Tests
    cargo-test = {
      enable = true;
      name = "cargo test";
      entry = "cargo test --all-features";
      pass_filenames = false;
      types = [ "rust" ];
    };

    # Security audit
    cargo-audit = {
      enable = true;
      name = "cargo audit";
      entry = "$\{pkgs.cargo-audit}/bin/cargo audit";
      pass_filenames = false;
      files = "Cargo\\.(toml|lock)$";
    };

    # General checks
    check-added-large-files.enable = true;
    check-merge-conflicts.enable = true;
    trailing-whitespace.enable = true;
  };

  # Scripts for common tasks
  scripts = {
    setup.exec = ''
      echo "🦀 Setting up Rust environment..."
      cargo fetch
      cargo build
      echo "✅ Setup complete!"
      echo ""
      echo "Available commands:"
      echo "  /build         - Build project"
      echo "  /run           - Run project"
      echo "  /test          - Run tests"
      echo "  /bench         - Run benchmarks"
      echo "  /clippy        - Run clippy"
      echo "  /doc           - Generate documentation"
      echo "  /watch         - Auto-rebuild on changes"
    '';

    clean.exec = ''
      echo "🧹 Cleaning build artifacts..."
      cargo clean
      rm -rf target/
      echo "✅ Cleanup complete!"
    '';

    coverage.exec = ''
      echo "📊 Generating code coverage..."
      cargo llvm-cov --html --open
    '';

    profile-release.exec = ''
      echo "🔥 Profiling release build..."
      cargo flamegraph --release
    '';

    bloat.exec = ''
      echo "📦 Analyzing binary size..."
      cargo bloat --release
    '';

    unused-deps.exec = ''
      echo "🔍 Checking for unused dependencies..."
      cargo udeps
    '';
  };

  # Welcome message on shell entry
  enterShell = ''
    echo "🦀 Rust Development Environment"
    echo "================================"
    echo ""
    echo "Project: {project_name}"
    echo "Rust:    {rust_channel}"
    echo "Type:    {project_type}"
    echo ""
    echo "Quick Start:"
    echo "  setup          - Fetch dependencies and build"
    echo "  /build         - Build project"
    echo "  /run           - Run project"
    echo "  /test          - Run tests"
    echo "  /clippy        - Run linter"
    echo "  /watch         - Auto-rebuild on changes"
    echo ""
    echo "Claude Code Integration:"
    echo "  - Automatic formatting on save (rustfmt)"
    echo "  - Clippy linting after edits"
    echo "  - Quick compile check on changes"
    echo "  - Proactive code review for safety"
    echo "  - Unsafe code auditing"
    echo ""
    echo "CARGO_HOME: $CARGO_HOME"
    echo ""
    echo "Run 'setup' to get started!"
  '';

  # Test configuration
  enterTest = ''
    echo "Running Rust test suite..."
    cargo test --all-features -- --nocapture
    echo ""
    echo "Running clippy..."
    cargo clippy --all-targets --all-features -- -D warnings
    echo ""
    echo "Checking formatting..."
    cargo fmt -- --check
  '';
}
```

### 4. Create devenv.yaml

```yaml
inputs:
  nixpkgs:
    url: github:NixOS/nixpkgs/nixpkgs-unstable

  # Optional: Use rust-overlay for more Rust versions
  # rust-overlay:
  #   url: github:oxalica/rust-overlay
```

### 5. Create .envrc for direnv Integration

```bash
if ! has nix_direnv_version || ! nix_direnv_version 3.0.4; then
  source_url "https://raw.githubusercontent.com/nix-community/nix-direnv/3.0.4/direnvrc" "sha256-DzlYZ33mWF/Gs8DDeyjr8mnVmQGx7ASYqA5WlxwvBG4="
fi

use devenv
```

### 6. Create Cargo.toml

**For binary project:**
```toml
[package]
name = "{project_name}"
version = "0.1.0"
edition = "2021"
rust-version = "1.70"
authors = ["Your Name <your.email@example.com>"]
description = "A Rust project with devenv and Claude Code"
readme = "README.md"
repository = "https://github.com/username/{project_name}"
license = "MIT OR Apache-2.0"
keywords = []
categories = []

[dependencies]
# Add your dependencies here

[dev-dependencies]
criterion = "0.5"
proptest = "1.0"

[profile.release]
opt-level = 3
lto = true
codegen-units = 1
strip = true

[profile.dev]
opt-level = 0

[profile.bench]
inherits = "release"
```

**For library project:**
```toml
[package]
name = "{project_name}"
version = "0.1.0"
edition = "2021"
rust-version = "1.70"
authors = ["Your Name <your.email@example.com>"]
description = "A Rust library with devenv and Claude Code"
readme = "README.md"
repository = "https://github.com/username/{project_name}"
license = "MIT OR Apache-2.0"
keywords = []
categories = []

[lib]
name = "{project_name_snake}"
path = "src/lib.rs"

[dependencies]
# Add your dependencies here

[dev-dependencies]
criterion = "0.5"
proptest = "1.0"

[[bench]]
name = "benchmark"
harness = false

[profile.release]
opt-level = 3
lto = true
codegen-units = 1

[profile.dev]
opt-level = 0

[profile.bench]
inherits = "release"
```

### 7. Create rust-toolchain.toml

```toml
[toolchain]
channel = "{rust_channel}"
components = [ "rustfmt", "clippy", "rust-analyzer" ]
profile = "default"
```

### 8. Create Source Files

**For binary (src/main.rs):**
```rust
//! # {project_name}
//!
//! A Rust application with devenv and Claude Code integration.

use std::error::Error;

fn main() -> Result<(), Box<dyn Error>> {
    println!("Hello from {project_name}!");
    println!("Rust development environment is ready.");

    Ok(())
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_main() {
        // Test that main runs without errors
        assert!(main().is_ok());
    }
}
```

**For library (src/lib.rs):**
```rust
//! # {project_name}
//!
//! A Rust library with devenv and Claude Code integration.
//!
//! ## Example
//!
//! ```
//! use {project_name_snake}::add;
//!
//! let result = add(2, 3);
//! assert_eq!(result, 5);
//! ```

#![warn(missing_docs)]
#![warn(clippy::all)]

/// Adds two numbers together.
///
/// # Examples
///
/// ```
/// use {project_name_snake}::add;
///
/// let result = add(2, 3);
/// assert_eq!(result, 5);
/// ```
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_add() {
        let test_cases = vec![
            (2, 3, 5),
            (-2, -3, -5),
            (-2, 3, 1),
            (0, 0, 0),
        ];

        for (a, b, expected) in test_cases {
            assert_eq!(add(a, b), expected, "add({}, {}) should equal {}", a, b, expected);
        }
    }
}
```

### 9. Create Test Files

**tests/integration_test.rs:**
```rust
//! Integration tests for {project_name}

#[cfg(test)]
mod tests {
    #[test]
    fn integration_test() {
        // Add integration tests here
        assert!(true);
    }
}
```

**benches/benchmark.rs:**
```rust
//! Benchmarks for {project_name}

use criterion::{black_box, criterion_group, criterion_main, Criterion};
use {project_name_snake}::*;

fn benchmark_add(c: &mut Criterion) {
    c.bench_function("add", |b| {
        b.iter(|| add(black_box(2), black_box(3)))
    });
}

criterion_group!(benches, benchmark_add);
criterion_main!(benches);
```

**examples/example.rs:**
```rust
//! Example usage of {project_name}

fn main() {
    println!("Example running...");
    // Add example code here
}
```

### 10. Create .gitignore

```
# Rust
/target/
Cargo.lock  # Remove this line for applications (keep for libraries)
**/*.rs.bk
*.pdb

# Flamegraph
flamegraph.svg
perf.data
perf.data.old

# Coverage
*.profraw
*.profdata
coverage/
lcov.info

# Devenv
.devenv/
.direnv/
devenv.lock
.cargo/

# IDE
.vscode/
.idea/
*.swp
*.swo
*~

# OS
.DS_Store
Thumbs.db

# Documentation
/target/doc/
```

### 11. Create rustfmt.toml

```toml
edition = "2021"
max_width = 100
hard_tabs = false
tab_spaces = 4
newline_style = "Unix"
use_small_heuristics = "Default"
reorder_imports = true
reorder_modules = true
remove_nested_parens = true
```

### 12. Create clippy.toml

```toml
# Clippy configuration
msrv = "1.70"

# Allow certain lints if needed
# allow = []

# Warn on these lints
warn = [
    "clippy::all",
    "clippy::pedantic",
    "clippy::cargo",
]

# Deny these lints
deny = [
    "clippy::unwrap_used",
    "clippy::expect_used",
]
```

### 13. Create README.md

```markdown
# {project_name}

Rust {project_type} with devenv and Claude Code integration.

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

2. Build the project:
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

- `/build` - Build project
- `/build-release` - Build optimized release
- `/run` - Run the application
- `/test` - Run all tests
- `/test-all` - Run tests with all features
- `/bench` - Run benchmarks
- `/check` - Quick compile check (no codegen)
- `/clippy` - Run clippy linter
- `/fmt` - Format code with rustfmt
- `/doc` - Generate and open documentation
- `/watch` - Auto-rebuild on file changes
- `/audit` - Security vulnerability audit
- `/expand` - Expand macros
- `/update` - Update dependencies

### Scripts

Shell scripts available in devenv:

- `setup` - Fetch dependencies and build
- `clean` - Remove build artifacts
- `coverage` - Generate code coverage report
- `profile-release` - Profile with flamegraph
- `bloat` - Analyze binary size
- `unused-deps` - Find unused dependencies

### Testing

```bash
# Run all tests
cargo test

# Run with output
cargo test -- --nocapture

# Run specific test
cargo test test_name

# Run benchmarks
cargo bench

# Test with all features
cargo test --all-features
```

### Building

```bash
# Debug build
cargo build

# Release build (optimized)
cargo build --release

# Check without building
cargo check

# Build documentation
cargo doc --open
```

### Code Quality

Pre-commit hooks automatically run:
- **rustfmt** - Code formatting
- **clippy** - Comprehensive linting
- **cargo check** - Compilation check
- **cargo test** - Test suite
- **cargo audit** - Security audit

Run manually:
```bash
cargo fmt
cargo clippy -- -D warnings
cargo audit
```

## Claude Code Integration

This project includes Claude Code integration with:

### Automatic Features

1. Code Formatting: Rust files auto-formatted after edits (rustfmt)
2. Linting: Clippy runs automatically after changes
3. Compile Check: Quick check after edits
4. Unsafe Auditing: Warns about unsafe code blocks

### Specialized Agents

- **code-reviewer**: Reviews for safety, idioms, performance, error handling
- **test-runner**: Executes tests, generates test templates
- **docs-writer**: Maintains documentation and doctests
- **performance-optimizer**: Analyzes benchmarks, suggests optimizations
- **unsafe-auditor**: Audits unsafe code for safety invariants

### Custom Hooks

- **Pre-tool-use**: Warns about compilation failures, unsafe code
- **Post-tool-use**: Auto-formats, runs clippy, quick compile check

## Project Structure

```
{project_name}/
├── src/                   # Source code
│   ├── lib.rs            # Library root (if lib)
│   └── main.rs           # Binary root (if bin)
├── tests/                # Integration tests
│   └── integration_test.rs
├── benches/              # Benchmarks
│   └── benchmark.rs
├── examples/             # Example programs
│   └── example.rs
├── docs/                 # Documentation
├── Cargo.toml            # Project manifest
├── Cargo.lock            # Dependency lock
├── rust-toolchain.toml   # Rust version
├── rustfmt.toml          # Formatting config
├── clippy.toml           # Linting config
└── README.md
```

## Rust Best Practices

This project follows:

- **The Rust Programming Language**: https://doc.rust-lang.org/book/
- **Rust API Guidelines**: https://rust-lang.github.io/api-guidelines/
- **Effective Rust**: https://www.lurklurk.org/effective-rust/

### Key Patterns

- Ownership and borrowing
- Error handling with `Result` and `?`
- Iterator chains over loops
- Type-driven design with traits
- Zero-cost abstractions

## Performance

### Profiling

```bash
# Flamegraph profiling
profile-release

# Benchmark with criterion
/bench

# Check binary size
bloat
```

### Optimization Tips

- Use `cargo build --release` for production
- Profile before optimizing
- Avoid unnecessary clones
- Use references where possible
- Consider using `Cow` for clone-on-write

## License

Licensed under either of:

- Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE))
- MIT license ([LICENSE-MIT](LICENSE-MIT))

at your option.

## Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be
dual licensed as above, without any additional terms or conditions.
```

### 14. Initialize and Validate

```bash
# Initialize direnv
direnv allow

# Enter devenv shell
devenv shell

# Run setup
setup

# Build and test
/build
/test
/clippy
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

## Claude Code Features

### 1. Automatic Code Review
After every code change, the code-reviewer agent checks for:
- Memory safety and ownership issues
- Proper error handling patterns
- Idiomatic Rust code
- Performance optimizations
- API design and ergonomics

### 2. Auto-formatting and Checking
Every Rust file edit triggers:
- `rustfmt` for formatting
- `clippy` for linting
- Quick compile check
- Safety warnings for unsafe blocks

### 3. Unsafe Code Auditing
The unsafe-auditor agent:
- Detects all unsafe blocks
- Verifies safety documentation
- Suggests safe alternatives
- Tracks unsafe usage

### 4. Performance Optimization
The performance-optimizer agent:
- Runs benchmarks with criterion
- Generates flamegraphs
- Identifies allocation hotspots
- Suggests zero-cost abstractions

### 5. Documentation Sync
The docs-writer agent ensures:
- All public items documented
- Doctests are valid
- Examples compile and run
- README is up-to-date

## Best Practices Enforced

**Safety**
- No unwrap/expect in production
- Documented unsafe invariants
- Proper error propagation

**Performance**
- Release profile optimized (LTO, strip)
- Benchmark suite included
- Profiling tools integrated

**Code Quality**
- Clippy on pedantic mode
- Comprehensive formatting
- Test coverage tracking

**Documentation**
- All public APIs documented
- Doctests for examples
- README with usage

## Speed Optimization

This command completes in **under 3 minutes**:
- 30s: Reference devenv skill
- 90s: Create project structure and files
- 30s: Initialize Cargo project and devenv
- 30s: Build and test

## Success Checklist

- [x] devenv.nix with Rust and Claude Code integration
- [x] Cargo.toml with proper configuration
- [x] rust-toolchain.toml for version pinning
- [x] Source files (main.rs or lib.rs)
- [x] Test suite with examples
- [x] Benchmark suite with criterion
- [x] rustfmt.toml and clippy.toml configuration
- [x] Pre-commit hooks configured
- [x] Claude Code agents (reviewer, tester, docs, optimizer, unsafe-auditor)
- [x] Custom hooks (pre/post tool use)
- [x] All slash commands configured
- [x] README with complete documentation
- [x] Optional services (if requested)
- [x] Validated and tested

Ready to create your Rust devenv project? Just tell me the project name, type (bin/lib), and Rust channel!
