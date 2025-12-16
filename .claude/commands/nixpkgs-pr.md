# Create Nixpkgs Pull Request

Automate PR creation for nixpkgs following all contribution guidelines.

## Quick Start

Just tell me:
1. PR description/commit message (or I'll detect from commits)
2. Package(s) being updated or added
3. Testing level completed (quick, full, or all)

I'll handle the entire PR workflow automatically.

## What I'll Do

### 1. Pre-PR Validation (Automatic)

Check current state:
- Verify we're in nixpkgs directory
- Check git status and current branch
- Identify changed files
- Verify commits exist
- Check remote configuration

### 2. Run nixpkgs-review

**Comprehensive testing before PR:**
```bash
nixpkgs-review rev HEAD
```

This automatically:
- Builds all changed packages
- Builds reverse dependencies
- Identifies potential breakage
- Creates isolated test environment
- Generates detailed report

**What nixpkgs-review checks:**
- Package builds successfully
- All outputs produced correctly
- Tests pass (if present)
- Reverse dependencies still work
- No unexpected breakage

**Example output:**
```
Changed packages:
  firefox: 120.0 -> 121.0

Built packages (1):
  firefox

Reverse dependencies (8):
  firefox-bin
  firefox-devedition
  firefox-esr
  firefox-unwrapped
  ...

All tests passed!
```

### 3. Commit Message Validation

**Check format compliance:**

Nixpkgs requires specific commit message format:
```
packageName: brief description

Optional longer description explaining the change.
Motivation and context.

Fixes #issue_number (if applicable)
```

**Examples of good commit messages:**
```
firefox: 120.0 -> 121.0

python311Packages.requests: 2.31.0 -> 2.32.0

postgresql_15: fix CVE-2024-12345

Applies security patch for critical vulnerability.

See https://www.postgresql.org/about/news/... for details.
```

**Examples of bad commit messages:**
```
# Too vague
update packages

# Missing package name
Updated to latest version

# Wrong format
[firefox] version bump to 121.0
```

**Auto-fix:**
If commit message doesn't follow convention, I'll:
1. Detect the issue
2. Suggest correct format
3. Offer to amend commit with proper message

### 4. Meta Attributes Check

**Verify package quality:**

Required meta attributes:
```nix
meta = with lib; {
  description = "Brief description";  # REQUIRED
  homepage = "https://...";          # REQUIRED
  license = licenses.mit;             # REQUIRED
  maintainers = with maintainers; [ username ]; # REQUIRED
  platforms = platforms.linux;        # REQUIRED (or all, darwin, etc.)
};
```

**Check for:**
- Missing required fields
- Description quality (not just package name)
- Valid license identifier
- Maintainer attribution
- Platform specification

**Common issues:**
```nix
# BAD: Description is just the package name
description = "Firefox";

# GOOD: Descriptive
description = "A free and open-source web browser";

# BAD: Unknown license
license = "MIT";

# GOOD: From lib.licenses
license = licenses.mit;
```

### 5. Package Quality Checks

**Automated quality verification:**

**Build reproducibility:**
```bash
# Build twice, compare outputs
nix-build -A firefox
nix-build -A firefox --check
```

**Output validation:**
```bash
# Check all outputs exist
nix-instantiate --eval -E 'with import ./. {}; firefox.outputs'

# Verify binaries work
./result/bin/firefox --version
```

**Dependency analysis:**
```bash
# Check runtime dependencies
nix-store --query --tree $(nix-build -A firefox --no-out-link)

# Verify no unexpected dependencies
nix-store --query --references $(nix-build -A firefox --no-out-link)
```

**Security check:**
```bash
# Check for known vulnerabilities (if available)
nix-instantiate --eval -E 'with import ./. {}; firefox.meta.knownVulnerabilities or []'
```

### 6. Create Pull Request

**Generate PR with proper template:**

```bash
# Push changes to your fork
git push origin update-firefox

# Create PR using GitHub CLI
gh pr create \
  --title "firefox: 120.0 -> 121.0" \
  --body "$(cat <<'EOF'
### Description of changes

Update Firefox from version 120.0 to 121.0.

### Testing

- [x] Built successfully on x86_64-linux
- [x] All outputs produced (out, dev, man)
- [x] Package tests passed
- [x] Runtime tested: `firefox --version`
- [x] nixpkgs-review passed (8 reverse dependencies checked)

### Checklist

- [x] Builds on x86_64-linux
- [x] Builds on aarch64-linux (ofborg)
- [x] Commit message follows guidelines
- [x] Meta attributes verified
- [x] Tests pass
- [ ] Built on x86_64-darwin (ofborg will verify)

### Additional context

Security and bug fix release.
See: https://www.mozilla.org/en-US/firefox/120.0/releasenotes/

CC: @maintainer1 @maintainer2
EOF
)"
```

**PR template sections:**

**1. Description of changes**
- What changed
- Why it changed
- Impact of changes

**2. Testing performed**
- Build tests
- Runtime tests
- nixpkgs-review results
- Manual testing

**3. Checklist**
- Platform builds
- Commit format
- Meta attributes
- Tests status

**4. Additional context**
- Links to upstream changes
- Security advisories (if applicable)
- Breaking changes (if any)
- Migration guide (if needed)

### 7. Add Labels and Reviewers

**Automatic label assignment:**

Based on package path:
```
pkgs/applications/          → "6.topic: applications"
pkgs/servers/               → "6.topic: servers"
pkgs/development/libraries/ → "6.topic: libraries"
pkgs/development/python-modules/ → "6.topic: python"
```

Based on change type:
```
Version update    → "8.has: package (update)"
New package       → "8.has: package (new)"
Build fix         → "10.rebuild-linux: 1-10"
Security fix      → "1.severity: security"
```

**Reviewer assignment:**

```bash
# Based on CODEOWNERS or maintainers
# Automatically CC package maintainers
```

**Example:**
```bash
gh pr create \
  --title "firefox: 120.0 -> 121.0" \
  --label "6.topic: applications" \
  --label "8.has: package (update)" \
  --label "10.rebuild-linux: 1-10" \
  --assignee @maintainer1
```

### 8. Handle Edge Cases

**Existing PR check:**
```bash
# Check if PR already exists for this branch
gh pr list --head update-firefox

# If exists, offer to update or create new
```

**Upstream conflicts:**
```bash
# Check if upstream changed since branch creation
git fetch upstream
git diff upstream/master..HEAD

# If conflicts, offer to rebase
git rebase upstream/master
```

**Multiple commits:**
```bash
# If multiple commits, check if they should be squashed
git log upstream/master..HEAD --oneline

# Offer to squash if needed
git rebase -i upstream/master
```

**Large rebuilds:**
```bash
# Check rebuild impact
nix-instantiate --eval -E '
  with import ./. {};
  let changed = [ firefox ];
  in lib.length (builtins.filter
    (p: builtins.elem firefox (p.buildInputs or []))
    (lib.attrValues pkgs))
'

# Warn if > 100 rebuilds
echo "⚠️ This change triggers X rebuilds"
echo "Consider coordinating with staging team"
```

## Complete Workflow Example

### Example 1: Version Update

```bash
# 1. We're on update-firefox branch with commits
git status
# On branch update-firefox
# Changes to be committed:
#   modified: pkgs/applications/networking/browsers/firefox/default.nix

# 2. Run command
User: "/nixpkgs-pr Create PR for firefox update to 121.0"

# 3. I perform:

# a) Run nixpkgs-review
echo "Running nixpkgs-review to test changes..."
nixpkgs-review rev HEAD

# b) Verify commit message
git log -1 --pretty=%B
# firefox: 120.0 -> 121.0
# ✓ Commit message follows convention

# c) Check meta attributes
nix-instantiate --eval -E 'with import ./. {}; firefox.meta'
# {
#   description = "A free and open-source web browser";
#   license = licenses.mpl20;
#   maintainers = [ maintainers.eelco ];
#   platforms = platforms.unix;
# }
# ✓ All required meta attributes present

# d) Verify build
nix-build -A firefox
# /nix/store/...-firefox-121.0
# ✓ Build successful

# e) Runtime test
./result/bin/firefox --version
# Mozilla Firefox 121.0
# ✓ Runtime check passed

# f) Push to fork
git push origin update-firefox

# g) Create PR
gh pr create \
  --title "firefox: 120.0 -> 121.0" \
  --body "[generated template with all test results]" \
  --label "6.topic: applications" \
  --label "8.has: package (update)"

# 4. Report success
echo "✓ Pull request created: https://github.com/NixOS/nixpkgs/pull/12345"
echo ""
echo "Next steps:"
echo "1. ofborg will test on additional platforms"
echo "2. Respond to any review comments"
echo "3. Monitor build status"
echo "4. PR will be merged by maintainers"
```

### Example 2: New Package

```bash
User: "/nixpkgs-pr Create PR for new package: mycli"

# 1. Detect new package
git diff --name-only upstream/master..HEAD
# pkgs/tools/misc/mycli/default.nix
# pkgs/top-level/all-packages.nix

# 2. Run comprehensive tests
nixpkgs-review rev HEAD

# 3. Verify new package quality
nix-instantiate --eval -E 'with import ./. {}; mycli.meta'
# Check description, license, maintainers

# 4. Build and test
nix-build -A mycli
./result/bin/mycli --version

# 5. Create PR with "new package" template
gh pr create \
  --title "mycli: init at 1.0.0" \
  --body "$(cat <<'EOF'
### Description

New package: mycli - A command-line tool for X

### Package information

- Homepage: https://github.com/user/mycli
- License: MIT
- Platforms: x86_64-linux, aarch64-linux
- Maintainer: @username

### Testing

- [x] Built successfully
- [x] Binary works: `mycli --version`
- [x] Dependencies minimal and appropriate
- [x] Meta attributes complete
- [x] No unnecessary runtime dependencies

### Checklist for new packages

- [x] Package builds
- [x] Meta attributes complete
- [x] License verified
- [x] Maintainer added
- [x] No known vulnerabilities
- [x] Follows nixpkgs conventions
EOF
)" \
  --label "8.has: package (new)" \
  --label "6.topic: tools"
```

### Example 3: Security Fix

```bash
User: "/nixpkgs-pr Create PR for postgresql CVE fix"

# 1. Detect security fix
git log -1 --pretty=%B
# postgresql_15: fix CVE-2024-12345

# 2. Enhanced testing for security
nixpkgs-review rev HEAD

# 3. Verify CVE is addressed
nix-build -A postgresql_15
# Check patch applied

# 4. Create PR with security emphasis
gh pr create \
  --title "postgresql_15: fix CVE-2024-12345" \
  --body "$(cat <<'EOF'
### Security Advisory

Fixes CVE-2024-12345: SQL injection vulnerability

**Severity**: Critical (CVSS 9.8)
**Affected versions**: < 15.5
**Fix version**: 15.5

### Changes

- Applies upstream security patch
- Backports fix from PostgreSQL 16
- Adds regression test

### Testing

- [x] Built successfully
- [x] Existing tests pass
- [x] New regression test added
- [x] Verified patch applies cleanly
- [x] nixpkgs-review: 23 reverse dependencies checked

### References

- CVE: https://nvd.nist.gov/vuln/detail/CVE-2024-12345
- Upstream fix: https://git.postgresql.org/...
- Security announcement: https://www.postgresql.org/...

### Urgency

**High priority** - Request expedited review and merge.

CC: @postgresql-maintainers
EOF
)" \
  --label "1.severity: security" \
  --label "12.approvals: security" \
  --label "8.has: package (update)"
```

## PR Best Practices

### DO

1. **Run nixpkgs-review before creating PR**
   ```bash
   nixpkgs-review rev HEAD
   ```

2. **Test the actual binary**
   ```bash
   nix-build -A package
   ./result/bin/program --version
   ./result/bin/program --help
   ```

3. **Check reverse dependencies**
   ```bash
   # nixpkgs-review does this automatically
   nixpkgs-review rev HEAD
   ```

4. **Follow commit message format**
   ```
   packageName: brief change description
   ```

5. **Include test results in PR description**
   - Build success
   - Runtime testing
   - nixpkgs-review results

6. **CC relevant maintainers**
   ```bash
   # Check CODEOWNERS or package meta.maintainers
   gh pr create --assignee @maintainer
   ```

7. **Add appropriate labels**
   - Package category
   - Change type
   - Rebuild scope

### DON'T

1. **Don't skip nixpkgs-review**
   - It catches reverse dependency breakage
   - Required for most PRs

2. **Don't ignore test failures**
   - Fix all issues before creating PR
   - Document if tests must be disabled

3. **Don't create PRs from master branch**
   - Always use feature branch
   - Branch name: `update-packagename` or `add-packagename`

4. **Don't mix multiple unrelated changes**
   - One package per PR (usually)
   - Exception: related packages that must update together

5. **Don't forget to rebase**
   ```bash
   # Before creating PR
   git fetch upstream
   git rebase upstream/master
   ```

6. **Don't ignore merge conflicts**
   - Resolve before creating PR
   - Test again after resolving

7. **Don't spam maintainers**
   - Wait for ofborg to complete
   - Give reasonable time for review

## Commit Message Guidelines

### Version Updates

```
packageName: oldVersion -> newVersion

# Examples:
firefox: 120.0 -> 121.0
python311Packages.requests: 2.31.0 -> 2.32.0
postgresql_15: 15.4 -> 15.5
```

### New Packages

```
packageName: init at version

# Examples:
mycli: init at 1.0.0
python311Packages.awesome-lib: init at 0.5.0
```

### Build Fixes

```
packageName: fix build on platform

# Examples:
firefox: fix build on aarch64-linux
postgresql: fix darwin build
```

### Dependency Updates

```
packageName: update dependency

# Examples:
firefox: update to use gtk4
nginx: migrate to openssl_3
```

### Feature Changes

```
packageName: enable/disable feature

# Examples:
vim: enable python support
nginx: disable stream module by default
```

### Multiple Changes

```
packageName: brief summary

Longer description:
- Change 1
- Change 2
- Change 3

# Example:
postgresql_15: 15.4 -> 15.5, enable JIT

- Update to version 15.5
- Enable JIT compilation by default
- Add new outputs: dev, lib
- Fix CVE-2024-12345
```

## ofborg Integration

After creating PR, ofborg (nixpkgs CI) will:

### 1. Build on Multiple Platforms

```
# ofborg will test:
- x86_64-linux
- aarch64-linux
- x86_64-darwin
- aarch64-darwin (if applicable)
```

### 2. Check Mass Rebuilds

```
# ofborg detects rebuild scope:
- 0 rebuilds: ✓ Safe
- 1-10 rebuilds: ✓ Generally OK
- 10-100 rebuilds: ⚠️ Needs review
- 100-500 rebuilds: ⚠️ Consider staging
- 500+ rebuilds: 🚫 Use staging branch
```

### 3. Run Package Tests

```
# ofborg runs:
- Package-specific tests
- NixOS module tests (if applicable)
- Cross-compilation tests
```

### 4. Comment with Results

```
# Example ofborg comment:
Success on x86_64-linux:
  firefox

Success on aarch64-linux:
  firefox

3 packages to test on x86_64-darwin:
  firefox
  firefox-devedition
  firefox-esr

(ofborg will test and comment later)
```

## Monitoring and Updates

### Check PR Status

```bash
# View your PR
gh pr view

# Check CI status
gh pr checks

# View ofborg results
gh pr view --json statusCheckRollup
```

### Respond to Reviews

```bash
# Add commits to address feedback
git commit -m "firefox: address review comments"
git push origin update-firefox

# Reply to comments
gh pr comment --body "Fixed in latest commit"
```

### Rebase on Master

```bash
# If master advanced
git fetch upstream
git rebase upstream/master
git push --force-with-lease origin update-firefox
```

## Troubleshooting

### Issue 1: nixpkgs-review Fails

**Problem:**
```
Error: Build failed for firefox
```

**Solution:**
```bash
# Check build log
nix-build -A firefox

# Fix issues in package
vim pkgs/applications/networking/browsers/firefox/default.nix

# Test again
nixpkgs-review rev HEAD
```

### Issue 2: Commit Message Invalid

**Problem:**
```
Commit message doesn't follow convention
```

**Solution:**
```bash
# Amend commit
git commit --amend

# Fix message format:
# firefox: 120.0 -> 121.0

# Force push
git push --force-with-lease origin update-firefox
```

### Issue 3: Merge Conflicts

**Problem:**
```
Your branch has conflicts with master
```

**Solution:**
```bash
# Fetch and rebase
git fetch upstream
git rebase upstream/master

# Resolve conflicts
# Edit conflicted files
git add .
git rebase --continue

# Test build still works
nix-build -A firefox

# Force push
git push --force-with-lease origin update-firefox
```

### Issue 4: Large Rebuild Count

**Problem:**
```
This change triggers 150 rebuilds
```

**Solution:**
```bash
# Check if change can be staged
# Contact staging team

# Or break into smaller PRs
# Update dependencies separately

# Document in PR:
"This change requires staging due to rebuild count.
Can be merged during next staging merge window."
```

### Issue 5: Test Failures

**Problem:**
```
Package tests fail on ofborg
```

**Solution:**
```bash
# Run tests locally
nix-build -A firefox.tests

# Fix test issues or disable
doCheck = false;  # Only if justified

# Document why tests disabled
# Must have good reason and issue filed
```

### Issue 6: Platform-Specific Failures

**Problem:**
```
Build fails on darwin but succeeds on linux
```

**Solution:**
```bash
# Add platform-specific fixes
meta.broken = stdenv.isDarwin;

# Or fix darwin build
buildInputs = [ ... ]
  ++ lib.optionals stdenv.isDarwin [ darwin.apple_sdk.frameworks.Security ];

# Document limitations
meta.platforms = platforms.linux;
```

## Expected Timeline

### Fast Track (1-2 days)
- Security fixes
- Critical bug fixes
- Small, well-tested changes
- No reverse dependencies affected

### Normal Track (3-7 days)
- Version updates
- Feature additions
- Moderate rebuild count
- Some reverse dependencies

### Staging Track (2-4 weeks)
- Large rebuilds (500+)
- Core package updates
- Breaking changes
- Requires staging branch

## Success Metrics

### Before Creating PR
- [x] nixpkgs-review passes
- [x] All builds successful
- [x] Runtime testing complete
- [x] Commit messages follow format
- [x] Meta attributes verified

### PR Quality
- [x] Clear description
- [x] Test results documented
- [x] Appropriate labels
- [x] Maintainers CC'd
- [x] No merge conflicts

### After Merge
- [x] Package in unstable channel
- [x] No regression reports
- [x] Reverse dependencies work
- [x] ofborg green on all platforms

## Integration with Other Commands

This command works with:
- `/nixpkgs-fork` - Must run first (setup)
- `/nixpkgs-test` - Should run before this (validation)
- `/nixpkgs-review` - Can use to review others' PRs

## Complete Contribution Workflow

```bash
# 1. Setup (one-time)
/nixpkgs-fork

# 2. Create branch and make changes
cd ~/nixpkgs
./scripts/new-package-branch.sh firefox
# Make changes to package...

# 3. Test thoroughly
/nixpkgs-test firefox full

# 4. Create PR (this command)
/nixpkgs-pr "Update firefox to 121.0"

# 5. Monitor and respond to feedback
gh pr view
# Address review comments...

# 6. Merge (maintainers do this)
# Your contribution is now in nixpkgs!
```

Ready to create your nixpkgs pull request? Just tell me what you're updating or adding!
