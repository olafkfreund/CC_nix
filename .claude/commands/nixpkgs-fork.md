# Fork and Setup Nixpkgs for Contributions

Automatically fork nixpkgs and setup local environment for contributions.

## Quick Start

Just tell me:
1. Your GitHub username (if not already configured)
2. Fork location (default: ~/nixpkgs)

I'll handle everything automatically.

## What I'll Do

### 1. Check Prerequisites (Automatic)
- Verify GitHub CLI (gh) is installed
- Check git configuration
- Verify you're authenticated with GitHub
- Check for existing nixpkgs directory

### 2. Fork Nixpkgs on GitHub

If you don't have a fork:
```bash
gh repo fork NixOS/nixpkgs --clone=false
```

If you already have a fork:
- Skip forking
- Use existing fork

### 3. Clone Your Fork

```bash
cd ~/
gh repo clone YOUR_USERNAME/nixpkgs
cd nixpkgs
```

**Options:**
- Shallow clone (faster, less disk): `--depth=1`
- Full clone (required for commits): full history

### 4. Setup Git Remotes

```bash
# Your fork is already 'origin'
git remote -v

# Add upstream (NixOS/nixpkgs)
git remote add upstream https://github.com/NixOS/nixpkgs.git

# Verify remotes
git remote -v
# origin    https://github.com/YOUR_USERNAME/nixpkgs (fetch)
# origin    https://github.com/YOUR_USERNAME/nixpkgs (push)
# upstream  https://github.com/NixOS/nixpkgs (fetch)
# upstream  https://github.com/NixOS/nixpkgs (push)
```

### 5. Configure Git for Nixpkgs

```bash
# Set up git config specific to nixpkgs
cd ~/nixpkgs

# Use your GitHub email
git config user.name "Your Name"
git config user.email "your.email@example.com"

# Nixpkgs-specific settings
git config pull.rebase true
git config fetch.prune true
git config diff.algorithm histogram
```

### 6. Setup Cachix (Optional but Recommended)

```bash
# Install cachix if not already installed
nix-env -iA cachix -f https://cachix.org/api/v1/install

# Use nixpkgs cache for faster builds
cachix use nixpkgs

# Use your own cache (optional)
# cachix use YOUR_CACHE_NAME
```

### 7. Create Development Branch

```bash
# Update master from upstream
git fetch upstream
git checkout master
git merge --ff-only upstream/master

# Push to your fork
git push origin master

# Create development branch structure
# (branches created as needed per package)
```

### 8. Setup direnv (Optional)

Create `.envrc` in nixpkgs directory:
```bash
#!/usr/bin/env bash
# Load nixpkgs development environment

use flake

# Add nixpkgs-review to PATH
export PATH="$PWD/result/bin:$PATH"
```

Then:
```bash
direnv allow
```

### 9. Install Development Tools

```bash
# Install nixpkgs-review (essential for testing PRs)
nix-env -f '<nixpkgs>' -iA nixpkgs-review

# Install nix-update (for updating packages)
nix-env -f '<nixpkgs>' -iA nix-update

# Install nixfmt (for formatting)
nix-env -f '<nixpkgs>' -iA nixfmt

# Install nix-init (for creating new packages)
nix-env -f '<nixpkgs>' -iA nix-init
```

Or use nix-shell:
```bash
# Create shell.nix in ~/nixpkgs
cat > ~/nixpkgs/shell.nix <<'EOF'
{ pkgs ? import <nixpkgs> {} }:

pkgs.mkShell {
  buildInputs = with pkgs; [
    nixpkgs-review
    nix-update
    nixfmt
    nix-init
    git
    gh
  ];

  shellHook = ''
    echo "Nixpkgs development environment loaded"
    echo "Available tools: nixpkgs-review, nix-update, nixfmt, nix-init"
  '';
}
EOF
```

### 10. Verify Setup

```bash
# Check git remotes
git remote -v

# Check current branch
git branch

# Test build (quick check)
nix-build -A hello

# Check nixpkgs-review
nixpkgs-review --help

# Check gh CLI
gh auth status
```

### 11. Create Helper Scripts

Create `~/nixpkgs/scripts/update-from-upstream.sh`:
```bash
#!/usr/bin/env bash
# Update local master from upstream

set -e

echo "Fetching upstream changes..."
git fetch upstream

echo "Switching to master..."
git checkout master

echo "Merging upstream/master..."
git merge --ff-only upstream/master

echo "Pushing to origin..."
git push origin master

echo "✓ Updated successfully!"
```

Create `~/nixpkgs/scripts/new-package-branch.sh`:
```bash
#!/usr/bin/env bash
# Create new branch for package work

if [ -z "$1" ]; then
  echo "Usage: $0 <package-name>"
  exit 1
fi

PACKAGE="$1"
BRANCH="update-${PACKAGE}"

set -e

echo "Updating master..."
git checkout master
git fetch upstream
git merge --ff-only upstream/master

echo "Creating branch: $BRANCH"
git checkout -b "$BRANCH"

echo "✓ Branch $BRANCH created!"
echo "  Now you can work on $PACKAGE"
```

Make scripts executable:
```bash
chmod +x ~/nixpkgs/scripts/*.sh
```

### 12. Create README in Your Fork

Create `~/nixpkgs/CONTRIBUTING_LOCAL.md`:
```markdown
# Local Nixpkgs Development Setup

This is my local fork of nixpkgs for contributions.

## Quick Commands

### Update from upstream
```bash
./scripts/update-from-upstream.sh
```

### Create new package branch
```bash
./scripts/new-package-branch.sh PACKAGE_NAME
```

### Test package
```bash
nix-build -A PACKAGE_NAME
```

### Review PR
```bash
nixpkgs-review pr PR_NUMBER
```

### Create PR
```bash
# After committing changes
gh pr create --web
```

## Workflow

1. Update master: `./scripts/update-from-upstream.sh`
2. Create branch: `./scripts/new-package-branch.sh firefox`
3. Make changes to package
4. Test: `nix-build -A firefox`
5. Commit: `git commit -m "firefox: 120.0 -> 121.0"`
6. Push: `git push origin update-firefox`
7. Create PR: `gh pr create --web`

## Resources

- [Nixpkgs Manual](https://nixos.org/manual/nixpkgs/stable/)
- [Contributing Guide](https://github.com/NixOS/nixpkgs/blob/master/CONTRIBUTING.md)
- [Package Guidelines](https://nixos.org/manual/nixpkgs/stable/#chap-package-rules)
```

## What You Get

After running this command, you'll have:

1. **Forked Repository**
   - Your fork on GitHub
   - Local clone in ~/nixpkgs
   - Proper git remotes configured

2. **Development Tools**
   - nixpkgs-review (PR testing)
   - nix-update (package updates)
   - nixfmt (code formatting)
   - nix-init (new packages)

3. **Helper Scripts**
   - Update from upstream
   - Create package branches
   - Common workflows automated

4. **Optimized Configuration**
   - Git settings for nixpkgs
   - Cachix for faster builds
   - direnv for auto-loading

5. **Documentation**
   - Local contributing guide
   - Quick reference commands
   - Workflow examples

## Common Workflows After Setup

### Workflow 1: Update Existing Package

```bash
# 1. Update master
cd ~/nixpkgs
./scripts/update-from-upstream.sh

# 2. Create branch
./scripts/new-package-branch.sh firefox

# 3. Update package
nix-update firefox

# 4. Test build
nix-build -A firefox

# 5. Test with nixpkgs-review
nixpkgs-review rev HEAD

# 6. Commit
git commit -m "firefox: 120.0 -> 121.0"

# 7. Push and create PR
git push origin update-firefox
gh pr create --title "firefox: 120.0 -> 121.0" --body "Automated update"
```

### Workflow 2: Create New Package

```bash
# 1. Update master
cd ~/nixpkgs
./scripts/update-from-upstream.sh

# 2. Create branch
git checkout -b add-mypackage

# 3. Generate package template
nix-init

# 4. Edit package file
# pkgs/applications/misc/mypackage/default.nix

# 5. Add to all-packages.nix
# pkgs/top-level/all-packages.nix

# 6. Test build
nix-build -A mypackage

# 7. Test with nixpkgs-review
nixpkgs-review rev HEAD

# 8. Commit and push
git add .
git commit -m "mypackage: init at 1.0.0"
git push origin add-mypackage
gh pr create --web
```

### Workflow 3: Review Someone's PR

```bash
cd ~/nixpkgs

# Review PR #12345
nixpkgs-review pr 12345

# Check build results
ls result/

# Test the package
result/bin/program-name --version

# Add review comment
gh pr review 12345 --comment -b "Builds successfully, tested on NixOS 23.11"
```

## Troubleshooting

### Issue 1: "gh: command not found"

**Solution:**
```bash
# Install GitHub CLI
nix-env -iA nixpkgs.gh

# Or on NixOS
nix-shell -p gh

# Authenticate
gh auth login
```

### Issue 2: "Permission denied (publickey)"

**Solution:**
```bash
# Setup SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"

# Add to GitHub
gh ssh-key add ~/.ssh/id_ed25519.pub

# Test connection
ssh -T git@github.com
```

### Issue 3: "Already exists"

**Solution:**
```bash
# If nixpkgs directory already exists
cd ~/nixpkgs
git remote -v

# If it's a proper fork, just verify remotes
# If not, move it and start fresh
mv ~/nixpkgs ~/nixpkgs.backup
# Then re-run /nixpkgs-fork
```

### Issue 4: "Disk space" Issues

**Solution:**
```bash
# Use shallow clone
gh repo clone YOUR_USERNAME/nixpkgs -- --depth=1

# Or cleanup old builds
nix-collect-garbage -d

# Check disk usage
du -sh ~/nixpkgs
```

### Issue 5: Slow Downloads

**Solution:**
```bash
# Setup Cachix
cachix use nixpkgs

# Use binary cache
nix-build -A package --option substituters "https://cache.nixos.org"
```

## Best Practices

### DO

1. **Always update master before starting work**
   ```bash
   ./scripts/update-from-upstream.sh
   ```

2. **Create separate branches for each package**
   ```bash
   git checkout -b update-firefox
   ```

3. **Test builds before creating PR**
   ```bash
   nix-build -A firefox
   nixpkgs-review rev HEAD
   ```

4. **Follow commit message format**
   ```
   package-name: 1.0.0 -> 1.1.0
   package-name: init at 1.0.0
   package-name: fix build on darwin
   ```

5. **Use nixpkgs-review for testing**
   ```bash
   nixpkgs-review rev HEAD
   ```

### DON'T

1. **Don't commit directly to master**
   - Always use feature branches

2. **Don't mix multiple package updates in one PR**
   - One package per PR

3. **Don't forget to test on your platform**
   - Test builds locally first

4. **Don't push without committing**
   - Always commit before push

5. **Don't ignore ofborg feedback**
   - Address build failures

## Integration with Other Commands

After setup, you can use:

- `/nixpkgs-test PACKAGE` - Test package builds
- `/nixpkgs-pr "message"` - Create PR with testing
- `/nixpkgs-review PR_NUMBER` - Review others' PRs
- `/nix-update PACKAGE` - Update package version

## Success Checklist

- [x] Nixpkgs forked on GitHub
- [x] Repository cloned locally
- [x] Git remotes configured (origin + upstream)
- [x] Git config set for nixpkgs
- [x] Development tools installed (nixpkgs-review, nix-update)
- [x] Cachix configured
- [x] Helper scripts created
- [x] Documentation created
- [x] Setup verified (test build)
- [x] GitHub CLI authenticated

## Speed Optimization

This command completes in **5-10 minutes**:
- 1-2min: Fork on GitHub
- 2-3min: Clone repository (depending on connection)
- 1min: Configure git and remotes
- 2-3min: Install development tools
- 1min: Create helper scripts and documentation

## Next Steps

After setup, you're ready to:

1. **Update a package**: `/nixpkgs-test firefox` → `/nixpkgs-pr`
2. **Create new package**: Use `nix-init` → test → PR
3. **Review PRs**: `/nixpkgs-review 12345`

Ready to fork nixpkgs? Just tell me your GitHub username and preferred location!
