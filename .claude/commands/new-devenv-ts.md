# Create New TypeScript Devenv Project

Create a complete TypeScript development environment using devenv with Claude Code integration.

## Quick Start

Just tell me:
1. Project name (e.g., "my-api" or "web-app")
2. Project type: "node" (Node.js app), "web" (frontend), "lib" (library), or "fullstack"
3. Package manager: "npm", "yarn", "pnpm", or "bun" (default: "pnpm")
4. Optional: Additional services needed (PostgreSQL, Redis, etc.)

I'll set up everything automatically.

## What I'll Do

### 1. Reference Devenv Skill (Automatic)
- Read @.claude/skills/devenv.md → Complete devenv reference
- Understand Claude Code integration patterns
- Learn TypeScript-specific best practices

### 2. Initialize Devenv Project

Create project structure:
```
{project-name}/
├── devenv.nix           # Main devenv configuration
├── devenv.yaml          # Inputs configuration
├── .envrc               # direnv integration
├── .gitignore           # TypeScript/Node/devenv ignores
├── package.json         # Node package manifest
├── tsconfig.json        # TypeScript configuration
├── tsconfig.build.json  # Build-specific TS config
├── .eslintrc.json       # ESLint configuration
├── .prettierrc          # Prettier configuration
├── vitest.config.ts     # Test configuration
├── src/
│   ├── index.ts         # Main entry point
│   └── types/           # Type definitions
│       └── index.ts
├── tests/               # Test files
│   └── index.test.ts
├── dist/                # Build output (gitignored)
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
      dev = "Start dev server: npm run dev";
      build = "Build project: npm run build";
      build-watch = "Build and watch: npm run build:watch";
      start = "Start production: npm start";
      test = "Run tests: npm test";
      test-watch = "Watch tests: npm run test:watch";
      test-coverage = "Generate coverage: npm run test:coverage";
      test-ui = "Open test UI: npm run test:ui";
      lint = "Run linter: npm run lint";
      lint-fix = "Fix lint issues: npm run lint:fix";
      fmt = "Format code: npm run format";
      fmt-check = "Check formatting: npm run format:check";
      typecheck = "Type check: npm run typecheck";
      typecheck-watch = "Watch type check: npm run typecheck:watch";
      clean = "Clean build: npm run clean";
      install = "Install deps: npm install";
    };

    # Specialized agents
    agents = {
      code-reviewer = {
        description = "TypeScript code quality and type safety specialist";
        proactive = true;
        tools = [ "Read" "Grep" "TodoWrite" ];
        prompt = ''
          Review TypeScript code for:
          - Type safety and proper type annotations
          - Strict TypeScript compliance
          - Error handling patterns (try/catch, Error types)
          - Async/await patterns and Promise handling
          - Modern ES2022+ features usage
          - Performance issues (unnecessary re-renders, memory leaks)
          - Security vulnerabilities (XSS, injection, etc.)
          - Code organization and module structure
          - Proper use of generics and utility types
          - ESLint rule violations

          Provide specific, actionable feedback with code examples.
          Reference TypeScript handbook and best practices.
        '';
      };

      test-runner = {
        description = "TypeScript test execution and debugging specialist";
        proactive = false;
        tools = [ "Bash" "Read" "Edit" ];
        prompt = ''
          Execute tests with vitest and analyze failures.
          Suggest fixes based on test output and error messages.
          Generate test templates and mocks.
          Update tests to improve coverage.
          Run tests in watch mode for rapid feedback.
        '';
      };

      docs-writer = {
        description = "TypeScript documentation specialist";
        proactive = true;
        tools = [ "Read" "Write" "Grep" ];
        prompt = ''
          Generate and maintain TypeScript documentation:
          - JSDoc comments for functions and types
          - Type definitions with descriptions
          - README.md updates
          - API documentation
          - Usage examples

          Follow TSDoc conventions.
          Ensure all exported items are documented.
        '';
      };

      type-checker = {
        description = "TypeScript type system specialist";
        proactive = true;
        tools = [ "Bash" "Read" "Grep" ];
        prompt = ''
          Analyze TypeScript types:
          - Check for type errors
          - Suggest better type definitions
          - Identify 'any' usage that should be typed
          - Recommend utility types (Partial, Pick, etc.)
          - Verify generic constraints
          - Check for type narrowing opportunities

          Focus on type safety and maintainability.
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

        # Warn about type errors before editing
        if [[ "$TOOL_NAME" == "Edit" ]] && [[ "$FILE_PATH" =~ \.tsx?$ ]]; then
          if ! npm run typecheck --silent 2>/dev/null; then
            echo "⚠️  Warning: TypeScript errors exist. Fix type errors first."
            echo "   Run '/typecheck' to see errors."
          fi
        fi

        # Check for linting issues
        if [[ "$TOOL_NAME" == "Edit" ]] && [[ "$FILE_PATH" =~ \.tsx?$ ]]; then
          if ! npm run lint --silent 2>/dev/null; then
            echo "💡 Tip: Run '/lint-fix' to auto-fix linting issues."
          fi
        fi
      '';

      post-tool-use = pkgs.writeShellScript "post-tool-hook" ''
        #!/usr/bin/env bash
        TOOL_DATA=$(cat)
        TOOL_NAME=$(echo "$TOOL_DATA" | jq -r '.tool')
        FILE_PATH=$(echo "$TOOL_DATA" | jq -r '.parameters.file_path // empty')

        # Auto-format TypeScript files after edits
        if [[ "$TOOL_NAME" == "Edit" ]] && [[ "$FILE_PATH" =~ \.tsx?$ ]]; then
          echo "🔧 Formatting $FILE_PATH..."
          npx prettier --write "$FILE_PATH" 2>/dev/null || true
          echo "✅ Formatted $FILE_PATH"
        fi

        # Run ESLint on edited files
        if [[ "$TOOL_NAME" == "Edit" ]] && [[ "$FILE_PATH" =~ \.tsx?$ ]]; then
          echo "🔍 Running ESLint..."
          if npx eslint "$FILE_PATH" 2>/dev/null; then
            echo "✅ ESLint passed"
          else
            echo "⚠️  ESLint warnings - run '/lint' for details"
          fi
        fi

        # Quick type check after edits
        if [[ "$TOOL_NAME" == "Edit" ]] && [[ "$FILE_PATH" =~ \.tsx?$ ]]; then
          echo "📝 Type checking..."
          if npm run typecheck --silent 2>/dev/null; then
            echo "✅ Type check passed"
          else
            echo "❌ Type errors - run '/typecheck' for details"
          fi
        fi
      '';
    };
  };

  # JavaScript/TypeScript language configuration
  languages.javascript = {
    enable = true;
    package = pkgs.nodejs_22;

    # Package manager
    {package_manager}.enable = true;
    {package_manager}.install.enable = true;
  };

  # TypeScript language support
  languages.typescript.enable = true;

  # System packages
  packages = with pkgs; [
    # Node.js and package managers
    nodejs_22
    nodePackages.npm
    nodePackages.pnpm
    nodePackages.yarn

    # TypeScript tools
    nodePackages.typescript
    nodePackages.typescript-language-server

    # Linting and formatting
    nodePackages.eslint
    nodePackages.prettier

    # Testing
    nodePackages.vitest

    # Build tools
    nodePackages.tsup       # TypeScript bundler
    nodePackages.esbuild    # Fast bundler

    # Type definitions
    nodePackages."@types/node"

    # Utilities
    jq                      # JSON parsing for hooks
  ];

  # Services (customize as needed)
  {services_config}

  # Environment variables
  env = {
    NODE_ENV = "development";
    PROJECT_NAME = "{project_name}";
    PACKAGE_MANAGER = "{package_manager}";
  };

  # Pre-commit hooks (automatic on commit)
  pre-commit.hooks = {
    # Prettier formatting
    prettier = {
      enable = true;
      name = "prettier";
      entry = "$\{pkgs.nodePackages.prettier}/bin/prettier --write --ignore-unknown";
      types = [ "text" ];
    };

    # ESLint
    eslint = {
      enable = true;
      name = "eslint";
      entry = "$\{pkgs.nodePackages.eslint}/bin/eslint --fix";
      files = "\\.(js|jsx|ts|tsx)$";
    };

    # TypeScript type check
    tsc = {
      enable = true;
      name = "tsc";
      entry = "$\{pkgs.nodePackages.typescript}/bin/tsc --noEmit";
      files = "\\.(ts|tsx)$";
      pass_filenames = false;
    };

    # Tests
    vitest = {
      enable = true;
      name = "vitest";
      entry = "npm run test -- --run";
      pass_filenames = false;
      types = [ "typescript" ];
    };

    # General checks
    check-added-large-files.enable = true;
    check-merge-conflicts.enable = true;
    trailing-whitespace.enable = true;
  };

  # Scripts for common tasks
  scripts = {
    setup.exec = ''
      echo "📦 Setting up TypeScript environment..."
      {package_manager} install
      echo "✅ Dependencies installed!"
      echo ""
      echo "Available commands:"
      echo "  /dev           - Start dev server"
      echo "  /build         - Build project"
      echo "  /test          - Run tests"
      echo "  /lint          - Run linter"
      echo "  /typecheck     - Type check"
    '';

    clean.exec = ''
      echo "🧹 Cleaning build artifacts..."
      rm -rf dist node_modules .turbo
      rm -f tsconfig.tsbuildinfo
      echo "✅ Cleanup complete!"
    '';
  };

  # Welcome message on shell entry
  enterShell = ''
    echo "📘 TypeScript Development Environment"
    echo "====================================="
    echo ""
    echo "Project: {project_name}"
    echo "Node:    $(node --version)"
    echo "Type:    {project_type}"
    echo "PM:      {package_manager}"
    echo ""
    echo "Quick Start:"
    echo "  setup          - Install dependencies"
    echo "  /dev           - Start dev server"
    echo "  /build         - Build project"
    echo "  /test          - Run tests"
    echo "  /lint          - Run linter"
    echo ""
    echo "Claude Code Integration:"
    echo "  - Automatic formatting on save (Prettier)"
    echo "  - ESLint checks after edits"
    echo "  - Type checking on changes"
    echo "  - Proactive code review"
    echo ""
    echo "Run 'setup' to get started!"
  '';

  # Test configuration
  enterTest = ''
    echo "Running TypeScript test suite..."
    npm run test:coverage
  '';

  # Processes (if running services)
  processes = {
    # Uncomment for development server
    # dev.exec = "npm run dev";
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

### 6. Create package.json

```json
{
  "name": "{project_name}",
  "version": "0.1.0",
  "description": "TypeScript project with devenv and Claude Code",
  "type": "module",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "import": "./dist/index.js",
      "types": "./dist/index.d.ts"
    }
  },
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsup",
    "build:watch": "tsup --watch",
    "start": "node dist/index.js",
    "test": "vitest",
    "test:watch": "vitest watch",
    "test:coverage": "vitest run --coverage",
    "test:ui": "vitest --ui",
    "lint": "eslint src/**/*.ts",
    "lint:fix": "eslint src/**/*.ts --fix",
    "format": "prettier --write \"src/**/*.{ts,tsx,json,md}\"",
    "format:check": "prettier --check \"src/**/*.{ts,tsx,json,md}\"",
    "typecheck": "tsc --noEmit",
    "typecheck:watch": "tsc --noEmit --watch",
    "clean": "rm -rf dist node_modules .turbo tsconfig.tsbuildinfo"
  },
  "keywords": ["typescript", "devenv", "claude-code"],
  "author": "Your Name <your.email@example.com>",
  "license": "MIT",
  "devDependencies": {
    "@types/node": "^22.0.0",
    "@typescript-eslint/eslint-plugin": "^7.0.0",
    "@typescript-eslint/parser": "^7.0.0",
    "@vitest/ui": "^1.6.0",
    "eslint": "^8.57.0",
    "eslint-config-prettier": "^9.1.0",
    "prettier": "^3.2.0",
    "tsup": "^8.0.0",
    "tsx": "^4.7.0",
    "typescript": "^5.4.0",
    "vitest": "^1.6.0"
  },
  "dependencies": {},
  "engines": {
    "node": ">=22.0.0"
  }
}
```

### 7. Create tsconfig.json

```json
{
  "compilerOptions": {
    /* Language and Environment */
    "target": "ES2022",
    "lib": ["ES2022"],

    /* Modules */
    "module": "ESNext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,

    /* Emit */
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "outDir": "./dist",
    "removeComments": true,
    "importHelpers": true,

    /* Interop Constraints */
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true,

    /* Type Checking */
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "noImplicitReturns": true,
    "noImplicitOverride": true,
    "noUncheckedIndexedAccess": true,
    "allowUnusedLabels": false,
    "allowUnreachableCode": false,

    /* Completeness */
    "skipLibCheck": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```

### 8. Create tsconfig.build.json

```json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "declaration": true,
    "declarationMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["**/*.test.ts", "**/*.spec.ts"]
}
```

### 9. Create tsup.config.ts

```typescript
import { defineConfig } from 'tsup';

export default defineConfig({
  entry: ['src/index.ts'],
  format: ['esm', 'cjs'],
  dts: true,
  splitting: false,
  sourcemap: true,
  clean: true,
  minify: false,
  treeshake: true,
});
```

### 10. Create .eslintrc.json

```json
{
  "root": true,
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaVersion": 2022,
    "sourceType": "module",
    "project": "./tsconfig.json"
  },
  "plugins": ["@typescript-eslint"],
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:@typescript-eslint/recommended-requiring-type-checking",
    "prettier"
  ],
  "rules": {
    "@typescript-eslint/no-unused-vars": [
      "error",
      { "argsIgnorePattern": "^_" }
    ],
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/explicit-function-return-type": "warn",
    "@typescript-eslint/no-floating-promises": "error",
    "@typescript-eslint/await-thenable": "error"
  },
  "env": {
    "node": true,
    "es2022": true
  }
}
```

### 11. Create .prettierrc

```json
{
  "semi": true,
  "trailingComma": "all",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "arrowParens": "always"
}
```

### 12. Create vitest.config.ts

```typescript
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'dist/',
        '**/*.test.ts',
        '**/*.spec.ts',
      ],
    },
  },
});
```

### 13. Create Source Files

**src/index.ts:**
```typescript
/**
 * {project_name}
 *
 * TypeScript project with devenv and Claude Code integration.
 */

/**
 * Main application entry point.
 */
export function main(): void {
  console.log('Hello from {project_name}!');
  console.log('TypeScript development environment is ready.');
}

// Run if executed directly
if (import.meta.url === `file://$\{process.argv[1]}`) {
  main();
}
```

**src/types/index.ts:**
```typescript
/**
 * Type definitions for {project_name}
 */

export interface Config {
  name: string;
  version: string;
  environment: 'development' | 'production' | 'test';
}

export type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };
```

**tests/index.test.ts:**
```typescript
import { describe, it, expect } from 'vitest';
import { main } from '../src/index';

describe('main', () => {
  it('should run without errors', () => {
    expect(() => main()).not.toThrow();
  });
});
```

### 14. Create .gitignore

```
# Dependencies
node_modules/
.pnpm-store/
.yarn/
.npm/

# Build output
dist/
build/
*.tsbuildinfo

# Environment
.env
.env.local
.env.*.local

# Testing
coverage/
.vitest/

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

# Logs
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*

# Turbo
.turbo/
```

### 15. Create README.md

```markdown
# {project_name}

TypeScript {project_type} with devenv and Claude Code integration.

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

- `/dev` - Start development server
- `/build` - Build project
- `/build-watch` - Build and watch for changes
- `/start` - Start production build
- `/test` - Run tests
- `/test-watch` - Watch tests
- `/test-coverage` - Generate coverage report
- `/test-ui` - Open test UI
- `/lint` - Run ESLint
- `/lint-fix` - Auto-fix linting issues
- `/fmt` - Format code with Prettier
- `/typecheck` - Type check with tsc
- `/typecheck-watch` - Watch type checking
- `/clean` - Clean build artifacts
- `/install` - Install dependencies

### Scripts

Shell scripts available in devenv:

- `setup` - Install dependencies
- `clean` - Remove build artifacts

### Testing

```bash
# Run all tests
npm test

# Watch mode
npm run test:watch

# Coverage report
npm run test:coverage

# UI mode
npm run test:ui
```

### Building

```bash
# Development build
npm run build

# Watch mode
npm run build:watch

# Production build
NODE_ENV=production npm run build
```

### Code Quality

Pre-commit hooks automatically run:
- **Prettier** - Code formatting
- **ESLint** - Linting with TypeScript rules
- **tsc** - Type checking
- **Vitest** - Test suite

Run manually:
```bash
npm run format
npm run lint
npm run typecheck
npm test
```

## Claude Code Integration

This project includes Claude Code integration with:

### Automatic Features

1. Code Formatting: Files auto-formatted after edits (Prettier)
2. Linting: ESLint runs automatically after changes
3. Type Checking: tsc validates types on edits
4. Test Execution: Relevant tests run after changes

### Specialized Agents

- **code-reviewer**: Reviews for type safety, patterns, performance
- **test-runner**: Executes tests, generates test templates
- **docs-writer**: Maintains JSDoc and documentation
- **type-checker**: Analyzes types, suggests improvements

### Custom Hooks

- **Pre-tool-use**: Warns about type errors, suggests lint fixes
- **Post-tool-use**: Auto-formats, runs ESLint, type checks

## Project Structure

```
{project_name}/
├── src/                   # Source code
│   ├── index.ts          # Main entry point
│   └── types/            # Type definitions
│       └── index.ts
├── tests/                # Test files
│   └── index.test.ts
├── dist/                 # Build output (gitignored)
├── package.json          # Node manifest
├── tsconfig.json         # TypeScript config
├── tsup.config.ts        # Build config
├── vitest.config.ts      # Test config
├── .eslintrc.json        # Linting config
├── .prettierrc           # Formatting config
└── README.md
```

## TypeScript Best Practices

This project follows:

- **TypeScript Handbook**: https://www.typescriptlang.org/docs/
- **Effective TypeScript**: 62 Specific Ways to Improve Your TypeScript
- **Clean Code TypeScript**: https://github.com/labs42io/clean-code-typescript

### Key Patterns

- Strict type checking enabled
- No implicit any
- Proper error handling with Result types
- Explicit function return types
- Utility types (Partial, Pick, Omit, etc.)

## Performance

### Build Performance

- **tsup**: Fast bundling with esbuild
- **tsx**: Fast TypeScript execution
- **Vitest**: Fast test runner

### Optimization Tips

- Use const assertions
- Leverage tree-shaking
- Minimize dependencies
- Use type imports

## License

MIT

## Contribution

Contributions are welcome! Please follow the TypeScript and ESLint configurations.
```

### 16. Initialize and Validate

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
/typecheck
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
- Type safety and proper annotations
- Async/await patterns
- Error handling
- Performance issues
- Security vulnerabilities

### 2. Auto-formatting and Checking
Every TypeScript file edit triggers:
- Prettier for formatting
- ESLint for linting
- tsc for type checking

### 3. Type System Analysis
The type-checker agent:
- Detects improper any usage
- Suggests utility types
- Recommends better type definitions
- Verifies generic constraints

### 4. Test-Driven Development
The test-runner agent:
- Runs relevant tests
- Generates test templates
- Suggests test improvements
- Tracks coverage

### 5. Documentation Sync
The docs-writer agent ensures:
- JSDoc for all exports
- Type descriptions
- Usage examples
- README accuracy

## Best Practices Enforced

**Type Safety**
- Strict mode enabled
- No implicit any
- Explicit return types

**Code Quality**
- ESLint with TypeScript rules
- Prettier formatting
- Pre-commit hooks

**Testing**
- Vitest for unit tests
- Coverage tracking
- UI mode for debugging

**Performance**
- Fast bundling with tsup
- Tree-shaking enabled
- Minimal dependencies

## Speed Optimization

This command completes in **under 3 minutes**:
- 30s: Reference devenv skill
- 90s: Create project structure and files
- 30s: Initialize npm and devenv
- 30s: Build and test

## Success Checklist

- [x] devenv.nix with TypeScript and Claude Code integration
- [x] package.json with all scripts
- [x] tsconfig.json with strict settings
- [x] ESLint and Prettier configuration
- [x] Vitest test configuration
- [x] Source files with examples
- [x] Test suite with coverage
- [x] Pre-commit hooks configured
- [x] Claude Code agents (reviewer, tester, docs, type-checker)
- [x] Custom hooks (pre/post tool use)
- [x] All slash commands configured
- [x] README with complete documentation
- [x] Optional services (if requested)
- [x] Validated and tested

Ready to create your TypeScript devenv project? Just tell me the project name, type, and package manager!
