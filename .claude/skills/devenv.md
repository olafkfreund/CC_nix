# Devenv Skill

A specialized skill for creating fast, declarative, reproducible development environments using devenv.sh and Nix.

## Skill Overview

**Purpose**: Provide comprehensive support for devenv configuration, environment setup, language configuration, service management, and development workflow automation.

**Invoke When**:
- Creating new development environments
- Configuring language tooling and dependencies
- Setting up development services (databases, caches, queues)
- Managing development processes and scripts
- Configuring pre-commit hooks
- Setting up containerized development
- Troubleshooting devenv issues
- Migrating from docker-compose or other dev tools

## Core Capabilities

### 1. Installation & Setup

**Prerequisites**
```bash
# Install Nix first (if not already installed)
curl -L https://nixos.org/nix/install | sh

# Or on NixOS - already installed
```

**Install devenv**
```bash
# Using nix-env
nix-env --install --attr devenv -f https://github.com/NixOS/nixpkgs/tarball/nixpkgs-unstable

# Using nix profile (recommended)
nix profile install nixpkgs#devenv

# Verify installation
devenv version
```

**On NixOS/home-manager**
```nix
# NixOS configuration.nix
environment.systemPackages = with pkgs; [
  devenv
];

# Or home-manager
home.packages = with pkgs; [
  devenv
];
```

**Optional: Cachix for faster builds**
```bash
# Install cachix
nix-env -iA cachix -f https://cachix.org/api/v1/install

# Use devenv cache
cachix use devenv
```

### 2. Project Initialization

**Initialize New Project**
```bash
# Create project directory
mkdir my-project && cd my-project

# Initialize devenv
devenv init

# This creates:
# - devenv.nix (main configuration)
# - devenv.yaml (inputs/composition)
# - .envrc (direnv integration)
# - .gitignore (devenv-specific ignores)
```

**Generated Files**

**devenv.nix** - Main configuration:
```nix
{ pkgs, lib, config, ... }:

{
  # Package list
  packages = [ pkgs.git ];

  # Language configuration
  languages.python = {
    enable = true;
    version = "3.11";
  };

  # Services
  services.postgres.enable = true;

  # Processes
  processes.app.exec = "python manage.py runserver";

  # Environment variables
  env.DATABASE_URL = "postgresql://localhost/mydb";

  # Scripts
  scripts.hello.exec = "echo Hello from devenv!";

  # Pre-commit hooks
  pre-commit.hooks = {
    nixfmt.enable = true;
  };
}
```

**devenv.yaml** - Inputs configuration:
```yaml
inputs:
  nixpkgs:
    url: github:NixOS/nixpkgs/nixpkgs-unstable

# Optional: Import other devenv configs
imports:
  - path: ../shared-config
    inputs: {}
```

**.envrc** - Direnv integration:
```bash
if ! has nix_direnv_version || ! nix_direnv_version 3.0.4; then
  source_url "https://raw.githubusercontent.com/nix-community/nix-direnv/3.0.4/direnvrc" "sha256-DzlYZ33mWF/Gs8DDeyjr8mnVmQGx7ASYqA5WlxwvBG4="
fi

use devenv
```

### 3. Core Commands

**Essential Commands**
```bash
# Enter development shell
devenv shell

# Run tests
devenv test

# Start all processes/services
devenv up

# Start in background
devenv up -d

# Search for packages
devenv search <package-name>

# Update inputs (like nix flake update)
devenv update

# Garbage collect old environments
devenv gc

# Print environment info
devenv info

# Show container configuration
devenv container
```

**Using with direnv**
```bash
# Allow direnv (first time)
direnv allow

# Auto-enter shell when cd'ing to directory
cd my-project  # Automatically activates devenv
```

### 4. Language Configuration

**55+ Languages Supported**

**Python**
```nix
languages.python = {
  enable = true;
  version = "3.11.3";  # Specific version

  # Virtual environment
  venv.enable = true;
  venv.requirements = ./requirements.txt;

  # Poetry support
  poetry.enable = true;
  poetry.activate.enable = true;
  poetry.install.enable = true;
};

# Add Python packages
packages = with pkgs; [
  python311Packages.black
  python311Packages.pytest
  python311Packages.mypy
];
```

**JavaScript/Node.js**
```nix
languages.javascript = {
  enable = true;
  package = pkgs.nodejs_20;

  # NPM support
  npm.enable = true;
  npm.install.enable = true;

  # Yarn support
  yarn.enable = true;
  yarn.install.enable = true;

  # PNPM support
  pnpm.enable = true;
  pnpm.install.enable = true;
};

# Or use bun
languages.javascript = {
  enable = true;
  bun.enable = true;
};
```

**TypeScript**
```nix
languages.typescript = {
  enable = true;
};

packages = with pkgs; [
  nodePackages.typescript
  nodePackages.typescript-language-server
];
```

**Rust**
```nix
languages.rust = {
  enable = true;
  channel = "stable";  # or "nightly", "beta"

  # Components
  components = [ "rustc" "cargo" "clippy" "rustfmt" ];

  # Targets
  targets = [ "wasm32-unknown-unknown" ];
};
```

**Go**
```nix
languages.go = {
  enable = true;
  package = pkgs.go_1_21;
};

env.GOPATH = "${config.env.DEVENV_ROOT}/.go";
env.GOBIN = "${config.env.DEVENV_ROOT}/.go/bin";
```

**Ruby**
```nix
languages.ruby = {
  enable = true;
  version = "3.2.2";

  # Bundler support
  bundler.enable = true;
};
```

**PHP**
```nix
languages.php = {
  enable = true;
  version = "8.2";

  # Extensions
  extensions = [ "mysqli" "pdo" "pdo_mysql" ];

  # Composer
  fpm.pools.web = {
    settings = {
      "pm" = "dynamic";
      "pm.max_children" = 5;
    };
  };
};

# Composer packages
packages = [ pkgs.php82Packages.composer ];
```

**Java**
```nix
languages.java = {
  enable = true;
  jdk.package = pkgs.jdk17;

  # Gradle
  gradle.enable = true;

  # Maven
  maven.enable = true;
};
```

**Elixir**
```nix
languages.elixir = {
  enable = true;
  package = pkgs.elixir_1_15;
};

# With Erlang version
languages.erlang = {
  enable = true;
  package = pkgs.erlang_26;
};
```

**Haskell**
```nix
languages.haskell = {
  enable = true;
  package = pkgs.ghc94;

  # Stack support
  stack.enable = true;
};
```

**Terraform/OpenTofu**
```nix
languages.terraform = {
  enable = true;
};

# Or OpenTofu (Terraform fork)
languages.opentofu = {
  enable = true;
};
```

**Nix**
```nix
languages.nix = {
  enable = true;
};

packages = with pkgs; [
  nixd  # Nix LSP
  nixfmt  # Formatter
  statix  # Linter
];
```

### 5. Services Configuration

**30+ Pre-configured Services**

**PostgreSQL**
```nix
services.postgres = {
  enable = true;
  package = pkgs.postgresql_15;

  # Initial databases
  initialDatabases = [
    { name = "myapp_dev"; }
    { name = "myapp_test"; }
  ];

  # Initial SQL script
  initialScript = ''
    CREATE USER myapp WITH PASSWORD 'dev';
    GRANT ALL PRIVILEGES ON DATABASE myapp_dev TO myapp;
  '';

  # Listen address
  listen_addresses = "127.0.0.1";

  # Port
  port = 5432;

  # Extensions
  extensions = extensions: [
    extensions.postgis
    extensions.pg_cron
  ];

  # Settings
  settings = {
    max_connections = 100;
    shared_buffers = "128MB";
  };
};

# Access via:
env.DATABASE_URL = "postgresql://localhost:5432/myapp_dev";
```

**MySQL**
```nix
services.mysql = {
  enable = true;
  package = pkgs.mysql80;

  # Initial databases
  initialDatabases = [
    { name = "myapp"; }
  ];

  # Settings
  settings = {
    mysqld = {
      port = 3306;
      bind_address = "127.0.0.1";
    };
  };
};

env.DATABASE_URL = "mysql://root@localhost:3306/myapp";
```

**Redis**
```nix
services.redis = {
  enable = true;
  port = 6379;

  # Bind address
  bind = "127.0.0.1";
};

env.REDIS_URL = "redis://localhost:6379";
```

**MongoDB**
```nix
services.mongodb = {
  enable = true;

  # Additional config
  additionalConfig = ''
    net:
      port: 27017
      bindIp: 127.0.0.1
  '';
};
```

**Elasticsearch**
```nix
services.elasticsearch = {
  enable = true;
  package = pkgs.elasticsearch7;

  # Cluster name
  cluster_name = "devenv";

  # Port
  port = 9200;
};
```

**RabbitMQ**
```nix
services.rabbitmq = {
  enable = true;

  # Management plugin
  managementPlugin.enable = true;
  managementPlugin.port = 15672;
};

env.RABBITMQ_URL = "amqp://localhost:5672";
```

**Kafka**
```nix
services.kafka = {
  enable = true;
  port = 9092;

  # Zookeeper (required)
  zookeeper.enable = true;
};
```

**Nginx**
```nix
services.nginx = {
  enable = true;

  httpConfig = ''
    server {
      listen 8080;
      server_name localhost;

      location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
      }
    }
  '';
};
```

**Caddy**
```nix
services.caddy = {
  enable = true;

  config = ''
    :8080 {
      reverse_proxy localhost:3000
    }
  '';
};
```

**MinIO (S3-compatible storage)**
```nix
services.minio = {
  enable = true;

  # Credentials
  accessKey = "minioadmin";
  secretKey = "minioadmin";

  # Buckets
  buckets = [ "my-bucket" ];
};

env.AWS_ENDPOINT = "http://localhost:9000";
env.AWS_ACCESS_KEY_ID = "minioadmin";
env.AWS_SECRET_ACCESS_KEY = "minioadmin";
```

**Mailhog (Email testing)**
```nix
services.mailhog = {
  enable = true;

  # SMTP port
  smtpPort = 1025;

  # Web UI port
  uiPort = 8025;
};

env.SMTP_HOST = "localhost";
env.SMTP_PORT = "1025";
```

**Vault (Secrets management)**
```nix
services.vault = {
  enable = true;

  # Address
  address = "127.0.0.1:8200";
};
```

### 6. Process Management

**Define Processes**
```nix
# Simple processes
processes = {
  # Web server
  web.exec = "python manage.py runserver 0.0.0.0:8000";

  # Worker
  worker.exec = "celery -A myapp worker -l info";

  # Frontend dev server
  frontend.exec = "cd frontend && npm run dev";

  # Background task
  scheduler.exec = "python manage.py scheduler";
};
```

**Advanced Process Configuration**
```nix
processes = {
  api = {
    exec = "python -m uvicorn main:app --reload";

    # Process-specific environment
    process-compose = {
      environment = {
        PORT = "8000";
        LOG_LEVEL = "debug";
      };

      # Working directory
      working_dir = "${config.env.DEVENV_ROOT}/backend";

      # Availability check
      availability = {
        restart = "on_failure";
        max_restarts = 3;
      };

      # Readiness probe
      readiness_probe = {
        http_get = {
          host = "localhost";
          port = 8000;
          path = "/health";
        };
        initial_delay_seconds = 2;
        period_seconds = 10;
      };

      # Dependencies
      depends_on.db.condition = "process_healthy";
    };
  };

  db = {
    exec = "postgres";
    process-compose = {
      readiness_probe = {
        exec.command = "pg_isready -h localhost";
      };
    };
  };
};
```

**Start Processes**
```bash
# Start all processes with TUI
devenv up

# Start in background
devenv up -d

# Watch logs
tail -f .devenv/state/process-compose/*.log

# Stop processes
# Ctrl+C (if in foreground)
# or kill the background process
```

### 7. Scripts & Tasks

**Define Scripts**
```nix
scripts = {
  # Simple script
  hello.exec = ''
    echo "Hello from devenv!"
  '';

  # Database setup
  db-setup.exec = ''
    echo "Setting up database..."
    psql -h localhost -U postgres -d myapp_dev -f schema.sql
    echo "Database ready!"
  '';

  # Run tests
  test.exec = ''
    pytest tests/ -v
  '';

  # Build project
  build.exec = ''
    echo "Building project..."
    npm run build
    echo "Build complete!"
  '';

  # Migration
  migrate.exec = ''
    python manage.py migrate
  '';

  # Seed data
  seed.exec = ''
    python manage.py loaddata fixtures/*.json
  '';
};
```

**Use Scripts**
```bash
# Inside devenv shell
hello
db-setup
test
build
migrate
seed
```

**Tasks (Entertest/Entersh hooks)**
```nix
# Run on shell entry
enterShell = ''
  echo "Welcome to the development environment!"
  echo "Database: $DATABASE_URL"
  echo "Redis: $REDIS_URL"
  echo ""
  echo "Available commands:"
  echo "  - db-setup: Initialize database"
  echo "  - test: Run test suite"
  echo "  - migrate: Run migrations"
'';

# Run on test
enterTest = ''
  echo "Running tests..."
  pytest tests/
  echo "Running linters..."
  flake8 src/
'';
```

### 8. Environment Variables

**Configuration**
```nix
# Simple variables
env = {
  DATABASE_URL = "postgresql://localhost:5432/myapp";
  REDIS_URL = "redis://localhost:6379";

  # API keys (for development only!)
  API_KEY = "dev-key-12345";

  # Application config
  DEBUG = "true";
  LOG_LEVEL = "debug";

  # Paths
  DATA_DIR = "${config.env.DEVENV_ROOT}/data";
  UPLOAD_DIR = "${config.env.DEVENV_ROOT}/uploads";
};

# Dynamic variables
env.GIT_ROOT = config.git.root;
env.DEVENV_STATE = config.devenv.state;
```

**Dotenv Support**
```nix
# Load from .env file
dotenv.enable = true;
dotenv.filename = ".env.local";

# Or disallow .env files (force explicit config)
dotenv.disableHint = true;
```

**Secret Management**
```nix
# Use sops-nix or similar for real secrets
# Never commit secrets to devenv.nix

# Example with sops
imports = [ inputs.sops-nix.nixosModules.sops ];

sops = {
  defaultSopsFile = ./secrets.yaml;
  secrets.api_key = {
    path = "${config.env.DEVENV_ROOT}/.secrets/api_key";
  };
};

# Reference in environment
env.API_KEY = "$(cat ${config.env.DEVENV_ROOT}/.secrets/api_key)";
```

### 9. Pre-commit Hooks

**Common Hooks**
```nix
pre-commit.hooks = {
  # Nix formatting
  nixfmt.enable = true;

  # Nix linting
  statix.enable = true;

  # Python
  black.enable = true;
  isort.enable = true;
  flake8.enable = true;
  mypy.enable = true;
  pylint.enable = true;

  # JavaScript/TypeScript
  prettier.enable = true;
  eslint.enable = true;

  # Shell
  shellcheck.enable = true;
  shfmt.enable = true;

  # YAML
  yamllint.enable = true;

  # Markdown
  markdownlint.enable = true;

  # Trailing whitespace
  trailing-whitespace.enable = true;

  # File size check
  check-added-large-files.enable = true;

  # Merge conflict check
  check-merge-conflicts.enable = true;
};
```

**Custom Hooks**
```nix
pre-commit.hooks = {
  custom-check = {
    enable = true;
    name = "Custom Check";
    entry = "${pkgs.python3}/bin/python scripts/check.py";
    files = "\\.py$";
    pass_filenames = true;
  };

  license-header = {
    enable = true;
    name = "License Header Check";
    entry = "${pkgs.bash}/bin/bash scripts/check-license.sh";
    files = "\\.(py|js|ts)$";
  };
};
```

**Install Hooks**
```bash
# Inside devenv shell
git init  # if not already initialized
devenv shell  # Hooks auto-install on shell entry
```

### 10. Containers & Deployment

**Generate Container Image**
```nix
# Enable container support
containers.myapp.name = "myapp";
containers.myapp.registry = "docker.io/myuser";

# Startup command
containers.myapp.startupCommand = pkgs.writeScript "start" ''
  #!${pkgs.bash}/bin/bash
  python manage.py runserver 0.0.0.0:8000
'';

# Copy files
containers.myapp.copyToRoot = [
  (pkgs.buildEnv {
    name = "myapp-root";
    paths = [ ./app ./requirements.txt ];
  })
];
```

**Build Container**
```bash
# Generate container
devenv container myapp

# Outputs container tarball
# Load into Docker:
docker load < myapp.tar.gz
docker run -p 8000:8000 myapp
```

### 11. Packages & Dependencies

**System Packages**
```nix
packages = with pkgs; [
  # Development tools
  git
  gh  # GitHub CLI
  jq
  yq

  # Database clients
  postgresql
  redis

  # HTTP tools
  curl
  httpie

  # Monitoring
  htop
  bottom

  # Editors (if not using system)
  vim
  neovim

  # Debugging
  gdb
  lldb

  # Build tools
  cmake
  gnumake

  # Documentation
  pandoc
];
```

**Language-specific packages**
```nix
# Python
packages = with pkgs.python311Packages; [
  requests
  flask
  django
  pytest
  black
];

# Node.js
packages = with pkgs.nodePackages; [
  typescript
  typescript-language-server
  prettier
  eslint
];
```

### 12. Composition & Imports

**Shared Configuration**
```nix
# shared/common.nix
{ pkgs, ... }:
{
  packages = with pkgs; [
    git
    jq
    curl
  ];

  languages.nix.enable = true;

  pre-commit.hooks.nixfmt.enable = true;
}
```

**Import in Project**
```nix
# devenv.nix
{ pkgs, inputs, ... }:
{
  imports = [
    ./shared/common.nix
  ];

  # Project-specific config
  languages.python.enable = true;
  services.postgres.enable = true;
}
```

**Using devenv.yaml**
```yaml
# devenv.yaml
inputs:
  nixpkgs:
    url: github:NixOS/nixpkgs/nixpkgs-unstable

  # Import shared devenv config
  shared:
    url: path:../shared-devenv

# Use shared modules
imports:
  - path: shared
    inputs: {}
```

### 13. Testing Integration

**devenv test**
```nix
# Define tests
enterTest = ''
  echo "Running test suite..."

  # Unit tests
  pytest tests/unit -v

  # Integration tests
  pytest tests/integration -v

  # Linting
  flake8 src/
  black --check src/

  # Type checking
  mypy src/

  echo "All tests passed!"
'';
```

**Run Tests**
```bash
# Run all tests
devenv test

# In CI
nix develop --command devenv test
```

**CI Integration**
```yaml
# .github/workflows/test.yml
name: Test
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: cachix/install-nix-action@v22
      - uses: cachix/cachix-action@v12
        with:
          name: devenv
      - run: nix profile install nixpkgs#devenv
      - run: devenv test
```

## Common Patterns

### Pattern 1: Full-Stack Web Application

```nix
{ pkgs, ... }:
{
  # Frontend
  languages.javascript = {
    enable = true;
    npm.enable = true;
  };

  # Backend
  languages.python = {
    enable = true;
    venv.enable = true;
  };

  # Database
  services.postgres.enable = true;
  services.redis.enable = true;

  # Processes
  processes = {
    frontend.exec = "cd frontend && npm run dev";
    backend.exec = "python manage.py runserver";
    worker.exec = "celery -A app worker";
  };

  # Environment
  env = {
    DATABASE_URL = "postgresql://localhost/myapp";
    REDIS_URL = "redis://localhost:6379";
  };

  # Scripts
  scripts = {
    setup.exec = ''
      npm install --prefix frontend
      pip install -r requirements.txt
      python manage.py migrate
    '';

    test.exec = ''
      pytest backend/tests
      npm run test --prefix frontend
    '';
  };
}
```

### Pattern 2: Microservices Development

```nix
{ pkgs, ... }:
{
  # Multiple languages
  languages.go.enable = true;
  languages.rust.enable = true;
  languages.javascript.enable = true;

  # Shared services
  services.postgres.enable = true;
  services.redis.enable = true;
  services.kafka.enable = true;

  # All services as processes
  processes = {
    user-service.exec = "cd services/user && go run main.go";
    auth-service.exec = "cd services/auth && cargo run";
    api-gateway.exec = "cd gateway && npm start";
  };

  # Shared tools
  packages = with pkgs; [
    protobuf
    grpcurl
    kubectl
  ];
}
```

### Pattern 3: Data Science Environment

```nix
{ pkgs, ... }:
{
  languages.python = {
    enable = true;
    version = "3.11";
  };

  # Data science packages
  packages = with pkgs.python311Packages; [
    numpy
    pandas
    matplotlib
    scikit-learn
    jupyter
    ipython
  ];

  # Services
  services.postgres.enable = true;
  services.mongodb.enable = true;

  # Jupyter notebook
  processes.jupyter.exec = "jupyter notebook --ip=0.0.0.0";

  # Scripts
  scripts = {
    notebook.exec = "jupyter notebook";
    lab.exec = "jupyter lab";
  };
}
```

### Pattern 4: Mobile Development (React Native)

```nix
{ pkgs, ... }:
{
  languages.javascript = {
    enable = true;
    npm.enable = true;
  };

  packages = with pkgs; [
    nodejs
    watchman

    # Android
    android-tools
    androidStudioPackages.stable

    # iOS (macOS only)
    # cocoapods
  ];

  # Metro bundler
  processes.metro.exec = "npx react-native start";

  # Environment
  env = {
    ANDROID_HOME = "${pkgs.android-tools}/sdk";
  };

  scripts = {
    android.exec = "npx react-native run-android";
    ios.exec = "npx react-native run-ios";
  };
}
```

### Pattern 5: DevOps/Infrastructure

```nix
{ pkgs, ... }:
{
  languages.terraform.enable = true;

  packages = with pkgs; [
    kubectl
    kubernetes-helm
    terraform
    ansible
    docker
    docker-compose

    # Cloud CLIs
    awscli2
    google-cloud-sdk
    azure-cli
  ];

  scripts = {
    k8s-dev.exec = "kubectl config use-context dev";
    k8s-prod.exec = "kubectl config use-context prod";

    deploy-dev.exec = ''
      terraform workspace select dev
      terraform apply -auto-approve
    '';
  };
}
```

## Best Practices

### DO ✅

1. **Use devenv.yaml for input management**
   ```yaml
   inputs:
     nixpkgs:
       url: github:NixOS/nixpkgs/nixpkgs-unstable
   ```

2. **Pin dependencies with devenv.lock**
   ```bash
   # Commit devenv.lock to version control
   git add devenv.lock
   ```

3. **Use services instead of manual processes**
   ```nix
   # ✅ Good - use service
   services.postgres.enable = true;

   # ❌ Bad - manual process
   processes.postgres.exec = "postgres -D ./data";
   ```

4. **Leverage pre-commit hooks**
   ```nix
   pre-commit.hooks.nixfmt.enable = true;
   pre-commit.hooks.black.enable = true;
   ```

5. **Use enterShell for welcome messages**
   ```nix
   enterShell = ''
     echo "Development environment ready!"
     echo "Run 'devenv up' to start services"
   '';
   ```

6. **Define scripts for common tasks**
   ```nix
   scripts.setup.exec = ''
     # First-time setup
   '';
   ```

7. **Use composition for shared config**
   ```nix
   imports = [ ./shared/common.nix ];
   ```

8. **Set up CI with devenv test**
   ```nix
   enterTest = ''
     pytest tests/
   '';
   ```

9. **Use environment-specific configs**
   ```nix
   env.DEBUG = "true";  # Development
   # Override in production
   ```

10. **Document your environment**
    ```nix
    # Add comments explaining decisions
    services.postgres = {
      enable = true;
      # Using PostgreSQL 15 for JSON features
      package = pkgs.postgresql_15;
    };
    ```

### DON'T ❌

1. **Don't commit secrets to devenv.nix**
   ```nix
   # ❌ Never do this
   env.API_KEY = "secret-key-12345";

   # ✅ Use dotenv or sops
   dotenv.enable = true;
   ```

2. **Don't use global nix-env installations**
   ```bash
   # ❌ Don't install globally
   nix-env -iA nixpkgs.nodejs

   # ✅ Use devenv packages
   packages = [ pkgs.nodejs ];
   ```

3. **Don't ignore devenv.lock**
   ```bash
   # ✅ Commit lock file
   git add devenv.lock
   ```

4. **Don't hardcode paths**
   ```nix
   # ❌ Bad
   env.DATA_DIR = "/Users/me/projects/app/data";

   # ✅ Good
   env.DATA_DIR = "${config.env.DEVENV_ROOT}/data";
   ```

5. **Don't skip direnv integration**
   ```bash
   # ✅ Use direnv for auto-activation
   direnv allow
   ```

6. **Don't mix package managers**
   ```bash
   # ❌ Don't use npm install globally
   npm install -g typescript

   # ✅ Use devenv packages
   packages = [ pkgs.nodePackages.typescript ];
   ```

7. **Don't ignore enterTest**
   ```nix
   # ✅ Always define tests
   enterTest = "pytest tests/";
   ```

## Troubleshooting

### Issue 1: "command not found" in devenv shell

**Solution:**
```bash
# Ensure you're in the shell
devenv shell

# Or use direnv
direnv allow
cd .  # Re-enter directory
```

### Issue 2: Service won't start

**Solution:**
```bash
# Check service state directory
ls -la .devenv/state/

# Remove service data and restart
rm -rf .devenv/state/postgres
devenv up
```

### Issue 3: Slow shell activation

**Solution:**
```bash
# Use cachix for faster builds
cachix use devenv

# Or build in advance
devenv shell --build-first
```

### Issue 4: Processes not starting

**Solution:**
```nix
# Check process health probes
processes.api = {
  exec = "python main.py";
  process-compose.readiness_probe = {
    http_get = {
      host = "localhost";
      port = 8000;
      path = "/health";
    };
  };
};
```

### Issue 5: Python venv issues

**Solution:**
```bash
# Remove and recreate venv
rm -rf .venv
devenv shell

# Or disable venv
languages.python.venv.enable = false;
```

### Issue 6: Node modules not installing

**Solution:**
```nix
# Ensure npm install is enabled
languages.javascript = {
  enable = true;
  npm.install.enable = true;
};

# Or run manually
scripts.install.exec = "npm install";
```

## Advanced Features

### Custom Modules

**Create Module**
```nix
# modules/my-service.nix
{ config, lib, pkgs, ... }:
with lib;
let
  cfg = config.services.my-service;
in {
  options.services.my-service = {
    enable = mkEnableOption "My Service";

    port = mkOption {
      type = types.int;
      default = 3000;
    };
  };

  config = mkIf cfg.enable {
    processes.my-service = {
      exec = "${pkgs.my-service}/bin/my-service";
      process-compose.environment.PORT = toString cfg.port;
    };
  };
}
```

**Use Module**
```nix
{
  imports = [ ./modules/my-service.nix ];

  services.my-service = {
    enable = true;
    port = 8080;
  };
}
```

### Overlays

```nix
{ pkgs, ... }:
{
  nixpkgs.overlays = [
    (final: prev: {
      my-custom-pkg = prev.callPackage ./pkgs/my-pkg {};
    })
  ];

  packages = [ pkgs.my-custom-pkg ];
}
```

### Conditional Configuration

```nix
{ pkgs, lib, ... }:
let
  isDarwin = pkgs.stdenv.isDarwin;
  isLinux = pkgs.stdenv.isLinux;
in {
  packages = with pkgs;
    [ git curl ]
    ++ lib.optionals isDarwin [ darwin.apple_sdk.frameworks.Security ]
    ++ lib.optionals isLinux [ systemd ];
}
```

## Integration Examples

### VSCode Integration

**.vscode/settings.json**
```json
{
  "nix.enableLanguageServer": true,
  "nix.serverPath": "nixd",
  "nixEnvSelector.nixFile": "${workspaceRoot}/devenv.nix"
}
```

### JetBrains Integration

Use the Nix IDE plugin and point it to your devenv.nix file.

### CI/CD Integration

**GitHub Actions**
```yaml
name: CI
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: cachix/install-nix-action@v22
      - uses: cachix/cachix-action@v12
        with:
          name: devenv
      - run: nix profile install nixpkgs#devenv
      - run: devenv test
```

**GitLab CI**
```yaml
test:
  image: nixos/nix
  script:
    - nix profile install nixpkgs#devenv
    - devenv test
```

## Command Reference

```bash
# Environment
devenv init              # Initialize new project
devenv shell             # Enter development shell
devenv info              # Show environment info

# Packages
devenv search <name>     # Search for packages
devenv search --detailed # Detailed search results

# Updates
devenv update            # Update inputs
devenv update nixpkgs    # Update specific input

# Processes
devenv up                # Start all processes (foreground)
devenv up -d             # Start in background
devenv up <process>      # Start specific process

# Testing
devenv test              # Run tests

# Containers
devenv container <name>  # Build container image

# Maintenance
devenv gc                # Garbage collect old environments
devenv repl              # Open Nix REPL with config

# Debugging
devenv print-dev-env     # Print shell environment
```

## Claude Code Integration

### Overview

Devenv provides native integration with Claude Code through hooks, custom commands, specialized agents, and MCP server configuration. This enables AI-assisted development workflows with automatic tooling and context-aware assistance.

### Global Configuration

**Set up Claude Code to use devenv globally:**

```markdown
# ~/.claude/CLAUDE.md
Use devenv for running commands, ensuring all tools and dependencies are available.
```

This instructs Claude Code to always use the devenv environment when executing commands in your projects.

### Basic Setup

**Enable Claude Code integration in your project:**

```nix
{
  claude.code.enable = true;
}
```

This enables:
- Automatic code formatting after edits (via git-hooks)
- Pre-commit hook integration
- Tool availability verification

### Custom Slash Commands

**Create project-specific commands that Claude Code can execute:**

```nix
{
  claude.code.commands = {
    test = "Run the test suite: cargo test";
    build = "Build in release mode: cargo build --release";
    deploy = "Deploy to staging: ./scripts/deploy.sh staging";
    lint = "Run all linters: npm run lint && cargo clippy";
    db-migrate = "Run database migrations: python manage.py migrate";
  };
}
```

**Usage in Claude Code:**
```bash
# Claude Code can now execute these as:
/test
/build
/deploy
# etc.
```

### Specialized Agents

**Configure AI assistants with restricted tool access for specific tasks:**

```nix
{
  claude.code.agents = {
    code-reviewer = {
      description = "Code quality and security specialist";
      proactive = true;
      tools = [ "Read" "Grep" "TodoWrite" ];
      prompt = ''
        Review code for:
        - Security vulnerabilities
        - Best practices adherence
        - Performance issues
        - Code style consistency
        Provide specific, actionable feedback with examples.
      '';
    };

    test-runner = {
      description = "Test execution and debugging specialist";
      proactive = false;
      tools = [ "Bash" "Read" "Edit" ];
      prompt = ''
        Execute tests and analyze failures.
        Suggest fixes based on test output.
        Update tests as needed.
      '';
    };

    docs-writer = {
      description = "Documentation specialist";
      proactive = true;
      tools = [ "Read" "Write" "Grep" ];
      prompt = ''
        Generate and maintain documentation.
        Keep docs in sync with code changes.
        Ensure examples are accurate and tested.
      '';
    };
  };
}
```

**Agent Properties:**
- **description**: Human-readable purpose
- **proactive**: Auto-invoke when context matches (true/false)
- **tools**: Restricted set of tools agent can use
- **prompt**: Behavior definition and instructions

### MCP Servers

**Configure Model Context Protocol servers for extended capabilities:**

```nix
{
  claude.code.mcpServers = {
    # Devenv MCP server (provides devenv context)
    devenv = {
      type = "stdio";
      command = "devenv";
      args = [ "mcp" ];
    };

    # HTTP-based server with authentication
    custom-api = {
      type = "http";
      url = "http://localhost:8080/mcp";
      headers = {
        "Authorization" = "Bearer ${config.env.API_TOKEN}";
      };
    };

    # Database query server
    postgres = {
      type = "stdio";
      command = "psql-mcp-server";
      args = [ "--database" "myapp" ];
    };
  };
}
```

**MCP Server Types:**
- **stdio**: Standard input/output communication
- **http**: HTTP-based with optional authentication

### Custom Hooks

**Five hook types for workflow automation:**

#### 1. PreToolUse (Blocking)
Runs before tool execution, can block actions:

```nix
{
  claude.code.hooks = {
    pre-tool-use = pkgs.writeShellScript "pre-tool-hook" ''
      #!/usr/bin/env bash
      # Receives JSON via stdin
      TOOL_DATA=$(cat)

      # Extract tool name and parameters
      TOOL_NAME=$(echo "$TOOL_DATA" | jq -r '.tool')
      FILE_PATH=$(echo "$TOOL_DATA" | jq -r '.parameters.file_path // empty')

      # Block dangerous operations
      if [[ "$TOOL_NAME" == "Edit" ]] && [[ "$FILE_PATH" == "config/production.yml" ]]; then
        echo "❌ Cannot edit production config without approval"
        exit 1  # Non-zero exit blocks the action
      fi

      # Allow action
      exit 0
    '';
  };
}
```

#### 2. PostToolUse (Non-blocking)
Runs after tool execution:

```nix
{
  claude.code.hooks = {
    post-tool-use = pkgs.writeShellScript "post-tool-hook" ''
      #!/usr/bin/env bash
      TOOL_DATA=$(cat)

      TOOL_NAME=$(echo "$TOOL_DATA" | jq -r '.tool')
      FILE_PATH=$(echo "$TOOL_DATA" | jq -r '.parameters.file_path // empty')

      # Auto-format after edits
      if [[ "$TOOL_NAME" == "Edit" ]] && [[ "$FILE_PATH" =~ \.nix$ ]]; then
        nixpkgs-fmt "$FILE_PATH"
        echo "✅ Formatted $FILE_PATH"
      fi

      # Run tests after code changes
      if [[ "$TOOL_NAME" == "Edit" ]] && [[ "$FILE_PATH" =~ \.rs$ ]]; then
        cargo test --quiet
        echo "✅ Tests passed"
      fi
    '';
  };
}
```

#### 3. Notification Hook
Triggered on Claude notifications:

```nix
{
  claude.code.hooks = {
    notification = pkgs.writeShellScript "notification-hook" ''
      #!/usr/bin/env bash
      # Send notifications to monitoring system
      notify-send "Claude Code" "$(cat)"
    '';
  };
}
```

#### 4. Stop Hook
Executes when Claude finishes responding:

```nix
{
  claude.code.hooks = {
    stop = pkgs.writeShellScript "stop-hook" ''
      #!/usr/bin/env bash
      # Log session completion
      echo "[$(date)] Session completed" >> .devenv/claude-sessions.log

      # Save conversation context
      cp .devenv/claude-context.json .devenv/sessions/$(date +%s).json
    '';
  };
}
```

#### 5. SubagentStop Hook
Runs when subagent tasks complete:

```nix
{
  claude.code.hooks = {
    subagent-stop = pkgs.writeShellScript "subagent-stop-hook" ''
      #!/usr/bin/env bash
      SUBAGENT_DATA=$(cat)

      AGENT_NAME=$(echo "$SUBAGENT_DATA" | jq -r '.agent_name')
      STATUS=$(echo "$SUBAGENT_DATA" | jq -r '.status')

      echo "[$(date)] Agent $AGENT_NAME completed with status: $STATUS" \
        >> .devenv/subagent-log.txt
    '';
  };
}
```

### Hook Input Format

**Hooks receive JSON via stdin:**

```json
{
  "tool": "Edit",
  "parameters": {
    "file_path": "/path/to/file.rs",
    "old_string": "...",
    "new_string": "..."
  },
  "timestamp": "2025-01-15T10:30:00Z"
}
```

**Parse using jq:**
```bash
TOOL_NAME=$(echo "$INPUT" | jq -r '.tool')
FILE_PATH=$(echo "$INPUT" | jq -r '.parameters.file_path // empty')
```

### Complete Integration Example

**Full devenv.nix with Claude Code integration:**

```nix
{ pkgs, config, ... }:
{
  # Enable Claude Code integration
  claude.code = {
    enable = true;

    # Custom commands
    commands = {
      test = "Run test suite: pytest tests/ -v";
      lint = "Run linters: black . && flake8 .";
      build = "Build project: python setup.py build";
      deploy-dev = "Deploy to dev: ./scripts/deploy.sh dev";
    };

    # Specialized agents
    agents = {
      reviewer = {
        description = "Code reviewer";
        proactive = true;
        tools = [ "Read" "Grep" "TodoWrite" ];
        prompt = "Review for quality and security";
      };

      tester = {
        description = "Test specialist";
        proactive = false;
        tools = [ "Bash" "Read" ];
        prompt = "Run and debug tests";
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

    # Hooks
    hooks = {
      pre-tool-use = pkgs.writeShellScript "pre-hook" ''
        # Validate before edits
        TOOL_DATA=$(cat)
        TOOL_NAME=$(echo "$TOOL_DATA" | jq -r '.tool')

        if [[ "$TOOL_NAME" == "Edit" ]]; then
          # Check if tests pass before allowing edits
          pytest tests/ --quiet || {
            echo "❌ Tests failing - fix tests first"
            exit 1
          }
        fi
      '';

      post-tool-use = pkgs.writeShellScript "post-hook" ''
        # Auto-format after edits
        TOOL_DATA=$(cat)
        FILE_PATH=$(echo "$TOOL_DATA" | jq -r '.parameters.file_path // empty')

        if [[ "$FILE_PATH" =~ \.py$ ]]; then
          black "$FILE_PATH"
          echo "✅ Formatted $FILE_PATH"
        fi
      '';
    };
  };

  # Language configuration
  languages.python = {
    enable = true;
    venv.enable = true;
  };

  # Services
  services.postgres.enable = true;

  # Pre-commit hooks (auto-triggered by Claude Code)
  pre-commit.hooks = {
    black.enable = true;
    flake8.enable = true;
    mypy.enable = true;
  };

  # Scripts
  scripts = {
    setup.exec = ''
      pip install -r requirements.txt
      python manage.py migrate
    '';
  };
}
```

### Automatic Code Formatting

**When git-hooks are enabled, Claude Code automatically formats files after editing:**

```nix
{
  # Enable pre-commit hooks
  pre-commit.hooks = {
    # Python
    black.enable = true;
    isort.enable = true;

    # Nix
    nixfmt.enable = true;
    statix.enable = true;

    # JavaScript
    prettier.enable = true;
    eslint.enable = true;

    # Rust
    rustfmt.enable = true;
    clippy.enable = true;
  };
}
```

**After Claude Code edits a file:**
1. Saves changes
2. Runs pre-commit hook
3. Applies formatting
4. Reports formatting result

### Best Practices

#### DO ✅

1. **Use meaningful command names**
   ```nix
   commands.test = "Run test suite: pytest";  # Clear and descriptive
   ```

2. **Configure proactive agents for repetitive tasks**
   ```nix
   agents.reviewer.proactive = true;  # Auto-review after changes
   ```

3. **Restrict agent tools appropriately**
   ```nix
   tools = [ "Read" "Grep" ];  # Read-only access for analysis
   ```

4. **Use pre-hooks to prevent mistakes**
   ```nix
   pre-tool-use = # Block dangerous operations
   ```

5. **Enable automatic formatting**
   ```nix
   pre-commit.hooks.nixfmt.enable = true;
   ```

6. **Document hook behavior**
   ```bash
   # Block edits to production config without approval
   ```

#### DON'T ❌

1. **Don't grant excessive permissions**
   ```nix
   # ❌ Bad - reviewer can execute arbitrary commands
   tools = [ "Bash" "Edit" "Write" ];

   # ✅ Good - reviewer has read-only access
   tools = [ "Read" "Grep" ];
   ```

2. **Don't forget error handling in hooks**
   ```bash
   # ❌ Bad - silent failures
   command_that_might_fail

   # ✅ Good - explicit error handling
   command_that_might_fail || {
     echo "❌ Command failed"
     exit 1
   }
   ```

3. **Don't hardcode paths in hooks**
   ```bash
   # ❌ Bad
   FILE_PATH="/home/user/project/file.py"

   # ✅ Good - use parameters from stdin
   FILE_PATH=$(echo "$TOOL_DATA" | jq -r '.parameters.file_path')
   ```

4. **Don't make hooks too slow**
   ```bash
   # ❌ Bad - runs entire test suite on every edit
   pytest tests/

   # ✅ Good - quick validation only
   pytest tests/ --quiet --exitfirst
   ```

### Debugging Integration

**Test hooks manually:**

```bash
# Simulate hook input
echo '{"tool":"Edit","parameters":{"file_path":"test.py"}}' | \
  .devenv/state/.hooks/pre-tool-use

# Check exit code
echo $?  # 0 = success, non-zero = blocked
```

**View hook logs:**

```bash
# Hook output
cat .devenv/state/claude-hooks.log

# Subagent logs
cat .devenv/subagent-log.txt
```

**Debug MCP servers:**

```bash
# Test devenv MCP server
devenv mcp

# Check server connectivity
curl http://localhost:8080/mcp/health
```

### Example Workflows

#### Workflow 1: Automated Code Review

```nix
{
  claude.code.agents.reviewer = {
    description = "Automated code reviewer";
    proactive = true;
    tools = [ "Read" "Grep" "TodoWrite" ];
    prompt = ''
      Review all code changes for:
      - Security vulnerabilities (SQL injection, XSS, etc.)
      - Best practices adherence
      - Performance anti-patterns
      - Missing error handling

      Create TodoWrite items for issues found.
      Provide specific file:line references.
    '';
  };

  pre-commit.hooks = {
    security-check.enable = true;
    complexity-check.enable = true;
  };
}
```

#### Workflow 2: Test-Driven Development

```nix
{
  claude.code = {
    commands = {
      test = "Run tests: pytest tests/ -v";
      test-watch = "Watch tests: pytest-watch tests/";
      coverage = "Coverage report: pytest --cov=src tests/";
    };

    hooks.pre-tool-use = pkgs.writeShellScript "tdd-hook" ''
      TOOL_DATA=$(cat)
      TOOL_NAME=$(echo "$TOOL_DATA" | jq -r '.tool')
      FILE_PATH=$(echo "$TOOL_DATA" | jq -r '.parameters.file_path // empty')

      # Require tests to pass before edits
      if [[ "$TOOL_NAME" == "Edit" ]] && [[ "$FILE_PATH" =~ src/ ]]; then
        pytest tests/ --quiet || {
          echo "❌ Fix failing tests before editing code"
          exit 1
        }
      fi
    '';

    hooks.post-tool-use = pkgs.writeShellScript "test-after-edit" ''
      TOOL_DATA=$(cat)
      FILE_PATH=$(echo "$TOOL_DATA" | jq -r '.parameters.file_path // empty')

      if [[ "$FILE_PATH" =~ \.(py|rs|ts)$ ]]; then
        pytest tests/ --quiet
        echo "✅ Tests passed after changes"
      fi
    '';
  };
}
```

#### Workflow 3: Documentation Sync

```nix
{
  claude.code.agents.docs = {
    description = "Documentation specialist";
    proactive = true;
    tools = [ "Read" "Write" "Grep" ];
    prompt = ''
      Keep documentation in sync with code:
      - Update API docs when signatures change
      - Refresh examples when implementations change
      - Add missing documentation
      - Fix outdated information

      Ensure all examples are tested and valid.
    '';
  };

  claude.code.hooks.post-tool-use = pkgs.writeShellScript "doc-check" ''
    TOOL_DATA=$(cat)
    FILE_PATH=$(echo "$TOOL_DATA" | jq -r '.parameters.file_path // empty')

    # Update docs after source changes
    if [[ "$FILE_PATH" =~ ^src/ ]]; then
      echo "📝 Reminder: Update documentation in docs/"
    fi
  '';
}
```

### Integration with CI/CD

**GitHub Actions with Claude Code hooks:**

```yaml
# .github/workflows/claude-code.yml
name: Claude Code Integration
on: [push, pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: cachix/install-nix-action@v22
      - uses: cachix/cachix-action@v12
        with:
          name: devenv

      - name: Install devenv
        run: nix profile install nixpkgs#devenv

      - name: Test hooks
        run: |
          devenv shell --command bash <<EOF
          # Simulate hook execution
          echo '{"tool":"Edit","parameters":{"file_path":"test.py"}}' | \
            .devenv/state/.hooks/pre-tool-use
          EOF

      - name: Run tests
        run: devenv test
```

## Success Metrics

- **Fast Setup**: New developers productive in minutes
- **Reproducible**: Same environment across all machines
- **Declarative**: Everything in version control
- **Composable**: Share configurations across projects
- **Tested**: CI integration ensures quality
- **Documented**: Clear configuration and scripts
- **AI-Assisted**: Claude Code integration for automated workflows
- **Quality**: Automatic code review and formatting
- **Safe**: Pre-hooks prevent dangerous operations

Ready to create fast, reproducible development environments with devenv and Claude Code! 🚀
