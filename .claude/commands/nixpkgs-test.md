# Test Nixpkgs Package Locally

Test package builds comprehensively before submitting PR to nixpkgs.

## Quick Start

Just tell me:
1. Package name (e.g., "firefox", "python311", "vscode")
2. Test level: "quick", "full", or "all" (default: "full")

I'll run all appropriate tests automatically.

## What I'll Do

### 1. Validate Environment (Automatic)
- Check if in nixpkgs directory
- Verify git status
- Check for uncommitted changes
- Identify changed files

### 2. Quick Build Test

**Default build:**
```bash
nix-build -A PACKAGE_NAME
```

**Example:**
```bash
nix-build -A firefox
# Builds firefox from your local nixpkgs
```

### 3. Full Build Test (Recommended)

Build all outputs and run package tests:

```bash
nix-build -A PACKAGE_NAME --keep-going
```

**With tests:**
```bash
nix-build -A PACKAGE_NAME.tests --keep-going
```

**Check all outputs:**
```bash
# List all outputs
nix-instantiate --eval -E 'with import ./. {}; PACKAGE_NAME.outputs'

# Build all outputs
nix-build -A PACKAGE_NAME.out
nix-build -A PACKAGE_NAME.dev  # if exists
nix-build -A PACKAGE_NAME.doc  # if exists
```

### 4. Comprehensive Test with nixpkgs-review

**Test your changes:**
```bash
nixpkgs-review rev HEAD
```

This:
- Builds all changed packages
- Builds reverse dependencies
- Creates isolated environment
- Reports build results

**Test specific commit:**
```bash
nixpkgs-review rev COMMIT_HASH
```

**Test against upstream:**
```bash
nixpkgs-review rev --against upstream/master HEAD
```

### 5. Cross-Platform Testing

**Test on different platforms:**

```bash
# Current platform (default)
nix-build -A PACKAGE_NAME

# Linux (x86_64-linux)
nix-build -A PACKAGE_NAME --argstr system x86_64-linux

# Darwin/macOS (x86_64-darwin)
nix-build -A PACKAGE_NAME --argstr system x86_64-darwin

# AArch64 Linux (aarch64-linux)
nix-build -A PACKAGE_NAME --argstr system aarch64-linux
```

**Note:** Cross-platform builds require binfmt or remote builders

### 6. Package-Specific Tests

**Run package tests:**
```bash
nix-build -A PACKAGE_NAME.tests
```

**Run passthru.tests:**
```bash
# Many packages have passthru tests
nix-build -A PACKAGE_NAME.tests.version
nix-build -A PACKAGE_NAME.tests.pytest
```

**Examples:**
```bash
# Python package tests
nix-build -A python311Packages.requests.tests

# NixOS module tests
nix-build -A nixosTests.nginx

# Application tests
nix-build -A firefox.tests
```

### 7. Dependency Impact Analysis

**Find reverse dependencies:**
```bash
# Packages that depend on your package
nix-store --query --referrers $(nix-build -A PACKAGE_NAME --no-out-link)

# Use nix-tree for visual exploration
nix run nixpkgs#nix-tree -- $(nix-build -A PACKAGE_NAME --no-out-link)
```

**Test reverse dependencies:**
```bash
# nixpkgs-review tests these automatically
nixpkgs-review rev HEAD
```

### 8. Static Analysis

**Check nix expression:**
```bash
# Syntax check
nix-instantiate --parse pkgs/path/to/package/default.nix

# Evaluate expression
nix-instantiate --eval -E 'with import ./. {}; PACKAGE_NAME.meta'

# Check meta attributes
nix-instantiate --eval -E 'with import ./. {}; PACKAGE_NAME.meta.description'
```

**Run nixfmt:**
```bash
nixfmt pkgs/path/to/package/default.nix
```

**Check with statix:**
```bash
statix check pkgs/path/to/package/
```

### 9. License Verification

```bash
# Check license
nix-instantiate --eval -E 'with import ./. {}; PACKAGE_NAME.meta.license'

# Verify license is in lib.licenses
nix-instantiate --eval -E 'with import ./. {}; lib.licenses'
```

### 10. Build Time Analysis

```bash
# Measure build time
time nix-build -A PACKAGE_NAME

# Check derivation size
nix-store --query --size $(nix-instantiate -A PACKAGE_NAME)

# Analyze build closure
nix-store --query --tree $(nix-build -A PACKAGE_NAME --no-out-link)
```

### 11. Runtime Testing

**Test the built package:**
```bash
# Build and keep result
nix-build -A PACKAGE_NAME

# Test binary
./result/bin/BINARY_NAME --version
./result/bin/BINARY_NAME --help

# Run program
./result/bin/BINARY_NAME

# Check shared libraries
ldd ./result/bin/BINARY_NAME

# Verify files
ls -la result/
tree result/ | head -20
```

**Install temporarily:**
```bash
# Install in temporary profile
nix-env -f . -iA PACKAGE_NAME

# Test
BINARY_NAME --version

# Remove
nix-env -e PACKAGE_NAME
```

### 12. Generate Test Report

Create comprehensive test report:

```markdown
# Test Report: PACKAGE_NAME

## Build Tests
- [x] nix-build successful
- [x] All outputs built
- [x] Package tests passed

## nixpkgs-review Results
- Changed packages: X
- Built packages: Y
- Failed packages: Z
- Reverse dependencies: N

## Platform Tests
- [x] x86_64-linux: Success
- [ ] x86_64-darwin: Not tested
- [ ] aarch64-linux: Not tested

## Runtime Tests
- [x] Binary runs: `./result/bin/program --version`
- [x] Help works: `./result/bin/program --help`
- [x] Basic functionality: Tested manually

## Meta Attributes
- Description: "Package description"
- License: mit
- Maintainers: @username
- Platforms: x86_64-linux, x86_64-darwin

## Issues Found
- None

## Recommendations
- Ready for PR
```

## Test Levels

### Level 1: Quick (1-5 minutes)
```bash
nix-build -A PACKAGE_NAME
./result/bin/BINARY --version
```

**Use when:**
- Quick iteration during development
- Checking if basic build works
- Testing small changes

### Level 2: Full (5-15 minutes)
```bash
nix-build -A PACKAGE_NAME --keep-going
nix-build -A PACKAGE_NAME.tests
nixpkgs-review rev HEAD
```

**Use when:**
- Preparing for PR
- Testing significant changes
- Verifying reverse dependencies

### Level 3: All (15-60 minutes)
```bash
nixpkgs-review rev HEAD
# + Cross-platform builds
# + Full reverse dependency testing
# + Runtime testing
# + Static analysis
```

**Use when:**
- Final PR verification
- Major package updates
- New package additions
- Security-sensitive changes

## Common Test Scenarios

### Scenario 1: Version Update

```bash
# 1. Update version in default.nix
# 2. Update hash

# 3. Test build
nix-build -A firefox

# 4. Test reverse dependencies
nixpkgs-review rev HEAD

# 5. Test runtime
./result/bin/firefox --version

# 6. Check for regressions
./result/bin/firefox  # Manual testing
```

### Scenario 2: New Package

```bash
# 1. Create package derivation
# 2. Add to all-packages.nix

# 3. Test build
nix-build -A mypackage

# 4. Verify outputs
ls -la result/

# 5. Test binary
./result/bin/myprogram --version

# 6. Run nixpkgs-review
nixpkgs-review rev HEAD

# 7. Check meta attributes
nix-instantiate --eval -E 'with import ./. {}; mypackage.meta'
```

### Scenario 3: Build Fix

```bash
# 1. Fix build issue in derivation

# 2. Quick test
nix-build -A package

# 3. Full test with dependencies
nixpkgs-review rev HEAD

# 4. Verify fix doesn't break others
# nixpkgs-review checks this automatically
```

### Scenario 4: Security Update

```bash
# 1. Update to patched version

# 2. Build all variants
nix-build -A package
nix-build -A package.override { enableFeature = true; }

# 3. Test reverse dependencies (critical for security)
nixpkgs-review rev HEAD

# 4. Document CVE fixes in commit message
git commit -m "package: 1.0 -> 1.0.1 (CVE-2024-XXXXX)"
```

## Integration with CI/CD

### Local Pre-Commit Testing

Create `.git/hooks/pre-push`:
```bash
#!/usr/bin/env bash
# Test changed packages before pushing

set -e

echo "Running pre-push tests..."

# Get changed packages
CHANGED=$(git diff --name-only HEAD origin/master | grep '\.nix$' || true)

if [ -n "$CHANGED" ]; then
  echo "Testing changed packages..."
  nixpkgs-review rev HEAD
else
  echo "No package changes detected"
fi
```

### GitHub Actions (ofborg)

After pushing, ofborg will automatically:
- Build on multiple platforms
- Test reverse dependencies
- Report results in PR

Check status:
```bash
gh pr view --json statusCheckRollup
```

## Troubleshooting

### Issue 1: Build Fails with "hash mismatch"

**Solution:**
```bash
# Update hash with correct value
nix-build -A PACKAGE_NAME 2>&1 | grep "got:"

# Copy the correct hash to your derivation
# Then rebuild
nix-build -A PACKAGE_NAME
```

### Issue 2: "infinite recursion"

**Solution:**
```bash
# Check for circular dependencies
nix-instantiate --eval -E 'with import ./. {}; PACKAGE_NAME' --show-trace

# Look for self-references in override
```

### Issue 3: nixpkgs-review fails

**Solution:**
```bash
# Clean environment
rm -rf ~/.cache/nixpkgs-review

# Try again
nixpkgs-review rev HEAD

# Check logs
cat ~/.cache/nixpkgs-review/*/report.md
```

### Issue 4: Cross-platform build fails

**Solution:**
```bash
# Setup binfmt for cross-compilation
boot.binfmt.emulatedSystems = [ "aarch64-linux" ];

# Or use remote builder
# See: https://nixos.org/manual/nix/stable/#chap-distributed-builds
```

### Issue 5: Tests timeout

**Solution:**
```bash
# Increase timeout
nix-build -A PACKAGE_NAME --timeout 7200  # 2 hours

# Or disable specific slow tests
nix-build -A PACKAGE_NAME --arg doCheck false
```

## Best Practices

### DO

1. **Always run nixpkgs-review before PR**
   ```bash
   nixpkgs-review rev HEAD
   ```

2. **Test the actual binary**
   ```bash
   ./result/bin/program --version
   ```

3. **Check meta attributes**
   ```bash
   nix-instantiate --eval -E 'with import ./. {}; package.meta'
   ```

4. **Test with clean environment**
   ```bash
   nix-build -A PACKAGE_NAME --option build-use-sandbox true
   ```

5. **Document test results**
   - Note platforms tested
   - Record any manual testing
   - List reverse dependencies checked

### DON'T

1. **Don't skip nixpkgs-review**
   - It catches reverse dependency breakage

2. **Don't ignore test failures**
   - Fix or document why tests are disabled

3. **Don't assume cross-platform compatibility**
   - Test on target platforms or document limitations

4. **Don't forget to test after rebasing**
   - Conflicts may break builds

5. **Don't skip runtime testing**
   - Binary might build but not run correctly

## Expected Output

### Successful Build
```
these 5 derivations will be built:
  /nix/store/...-package-1.0.drv
...
building '/nix/store/...-package-1.0.drv'...
...
/nix/store/...-package-1.0
```

### nixpkgs-review Success
```
Changed packages:
  package: 1.0 -> 1.1

Built packages (1):
  package

No failed packages

Reverse dependencies (5):
  dependent1
  dependent2
  ...

All tests passed!
```

### Runtime Test
```bash
$ ./result/bin/program --version
program version 1.1.0

$ ./result/bin/program --help
Usage: program [options]
...
```

## Integration with Other Commands

Works with:
- `/nixpkgs-fork` - Setup required first
- `/nixpkgs-pr` - Runs tests before creating PR
- `/nixpkgs-review` - Alternative PR testing method
- `/nix-build` - Basic build command

## Success Checklist

- [x] Build succeeds: `nix-build -A PACKAGE`
- [x] All outputs built
- [x] Package tests pass
- [x] nixpkgs-review passes
- [x] Runtime testing successful
- [x] Binary version correct
- [x] No reverse dependency breakage
- [x] Meta attributes correct
- [x] Static analysis clean
- [x] Test report generated

## Speed Optimization

Test timing depends on package size:
- **Quick test**: 1-5 minutes (single build)
- **Full test**: 5-15 minutes (with nixpkgs-review)
- **All tests**: 15-60 minutes (multi-platform)

**Optimization tips:**
- Use cachix for dependencies
- Enable parallel builds: `--max-jobs auto`
- Use `--keep-going` to continue past failures

Ready to test your nixpkgs package? Just tell me the package name and test level!
