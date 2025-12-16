# Review Nixpkgs Pull Requests

Review and test nixpkgs PRs from other contributors using nixpkgs-review.

## Quick Start

Just tell me:
1. PR number to review (e.g., 12345)
2. Review depth: "quick", "thorough", or "contribute" (default: thorough)

I'll test the PR and provide a comprehensive review report.

## What I'll Do

### 1. Fetch PR Information

**Gather PR details:**
```bash
# Get PR information
gh pr view 12345

# Show:
# - Title and description
# - Changed files
# - Commits
# - Labels
# - Status checks
```

**Identify changes:**
```bash
# List changed packages
gh pr diff 12345 | grep '^+' | grep -E '(pname|version)'

# Detect change type:
# - Version update
# - New package
# - Build fix
# - Feature addition
```

### 2. Run nixpkgs-review

**Comprehensive PR testing:**

```bash
nixpkgs-review pr 12345
```

**What this does:**

1. **Fetches PR branch**
   ```bash
   # Creates isolated environment
   # Merges PR into current master
   # Identifies all changed packages
   ```

2. **Builds changed packages**
   ```bash
   # Builds each modified package
   # Tests all outputs (out, dev, lib, etc.)
   # Runs package-specific tests
   ```

3. **Checks reverse dependencies**
   ```bash
   # Finds packages that depend on changed packages
   # Builds them to verify no breakage
   # Reports any failures
   ```

4. **Creates test report**
   ```bash
   # Summary of builds
   # Failed packages (if any)
   # Reverse dependency status
   # Links to build logs
   ```

**Example output:**
```
Changed packages:
  firefox: 120.0 -> 121.0

Built packages (1):
  firefox

Reverse dependencies (8):
  ✓ firefox-bin
  ✓ firefox-devedition
  ✓ firefox-esr
  ✗ firefox-wrapper (build failed)
  ✓ tor-browser-bundle-bin
  ✓ mullvad-browser
  ✓ librewolf
  ✓ waterfox

Status: 7/8 reverse dependencies successful
```

### 3. Test Package Runtime

**Verify actual functionality:**

```bash
# Enter test environment
cd ~/.cache/nixpkgs-review/pr-12345

# Test the binary
./result/bin/firefox --version
./result/bin/firefox --help

# Basic functionality test
./result/bin/firefox

# Check dependencies
ldd ./result/bin/firefox

# Verify files
ls -la result/
tree result/ | head -20
```

### 4. Review Code Quality

**Examine package changes:**

```bash
# View diff
gh pr diff 12345

# Check common issues:
```

**Meta attributes:**
```nix
# Verify required fields present
meta = with lib; {
  description = "...";  # Not just package name
  homepage = "...";     # Valid URL
  license = licenses.mit;  # Proper license
  maintainers = [ ... ];   # Maintainer list
  platforms = platforms.linux;  # Platform spec
};
```

**Build configuration:**
```nix
# Check for common problems
- hardcoded paths
- evaluation-time secrets
- missing dependencies
- incorrect phases
- platform-specific issues
```

**Code style:**
```bash
# Run nixpkgs-fmt
nix-shell -p nixpkgs-fmt --run "nixpkgs-fmt --check ."

# Run statix
nix-shell -p statix --run "statix check"
```

### 5. Security Review

**Check for security issues:**

**Secret handling:**
```nix
# BAD: Evaluation time
password = builtins.readFile "/secrets/pass";

# GOOD: Runtime
passwordFile = "/path/to/secret";
```

**Service hardening:**
```nix
systemd.services.myservice = {
  serviceConfig = {
    # Check for security settings
    DynamicUser = true;
    ProtectSystem = "strict";
    NoNewPrivileges = true;
    PrivateTmp = true;
  };
};
```

**Known vulnerabilities:**
```bash
# Check CVE databases if security-related
# Verify patches applied correctly
# Check meta.knownVulnerabilities
```

### 6. Platform Testing

**Verify cross-platform compatibility:**

```bash
# Check supported platforms
nix-instantiate --eval -E 'with import ./. {}; firefox.meta.platforms'

# If claims multi-platform, verify:
# - Conditional dependencies for darwin
# - Platform-specific patches
# - No linux-only assumptions
```

**Common platform issues:**
```nix
# Missing darwin dependencies
buildInputs = lib.optionals stdenv.isDarwin [
  darwin.apple_sdk.frameworks.Security
];

# Platform-specific patches
patches = lib.optionals stdenv.isDarwin [
  ./darwin-fix.patch
];

# Platform limitations
meta.platforms = platforms.linux;  # If truly linux-only
```

### 7. Generate Review Report

**Create comprehensive review:**

```markdown
## Review: PR #12345 - firefox: 120.0 -> 121.0

### Build Results ✓

- [x] Package builds successfully on x86_64-linux
- [x] All outputs produced (out, dev, man)
- [x] Package tests pass
- [x] 7/8 reverse dependencies successful
- [x] 1 reverse dependency needs fix (firefox-wrapper)

### Runtime Testing ✓

- [x] Binary executes: `firefox --version` → 121.0
- [x] Help works: `firefox --help`
- [x] Basic functionality: Starts correctly
- [x] No missing shared libraries

### Code Quality ✓

- [x] Meta attributes complete and accurate
- [x] Commit message follows convention
- [x] No obvious security issues
- [x] Code formatted correctly
- [x] No hardcoded paths

### Issues Found

#### firefox-wrapper Build Failure

The update breaks firefox-wrapper due to changed output structure.

**Error:**
```
error: attribute 'gtk3' missing
```

**Cause:** Firefox 121.0 changed from gtk3 to gtk4

**Suggested fix:**
```nix
# In firefox-wrapper/default.nix
buildInputs = [
  - gtk3
  + gtk4
];
```

### Recommendations

1. **Approve with conditions:**
   - Fix firefox-wrapper compatibility
   - Test wrapper after fix
   - Then ready to merge

2. **Testing on additional platforms:**
   - ofborg will test darwin
   - Appears compatible based on code review

3. **Documentation:**
   - Consider noting gtk4 migration in commit message

### Verdict: ✅ Approve after fix

Great work! Just needs firefox-wrapper compatibility fix.
```

### 8. Provide Feedback

**Post review to GitHub:**

```bash
# Approve PR
gh pr review 12345 --approve --body "$(cat <<'EOF'
Tested on x86_64-linux, builds successfully!

Build results:
- ✓ firefox builds
- ✓ All outputs present
- ✓ Runtime tests pass
- ✓ 7/8 reverse dependencies work

One issue found:
- firefox-wrapper needs gtk4 update (see detailed comment)

Otherwise ready to merge!
EOF
)"

# Or request changes
gh pr review 12345 --request-changes --body "$(cat <<'EOF'
Found an issue that needs addressing:

### Build Failure

firefox-wrapper fails due to gtk3 → gtk4 migration.

See detailed review for suggested fix.

Otherwise the change looks good!
EOF
)"

# Or just comment
gh pr comment 12345 --body "$(cat <<'EOF'
Successfully tested on NixOS 23.11:

```
$ nix-build -A firefox
$ ./result/bin/firefox --version
Mozilla Firefox 121.0
```

Works great, thanks!
EOF
)"
```

## Review Depths

### Quick Review (5-10 minutes)

**What to check:**
- Build succeeds
- Binary runs
- No obvious issues

**Commands:**
```bash
nixpkgs-review pr 12345
./result/bin/program --version
gh pr review 12345 --approve -b "LGTM, tested on x86_64-linux"
```

**Use when:**
- Simple version bump
- Trusted contributor
- No reverse dependencies
- Clear change

### Thorough Review (15-30 minutes)

**What to check:**
- Build and runtime testing
- Code quality review
- Reverse dependencies
- Security considerations
- Meta attributes

**Commands:**
```bash
nixpkgs-review pr 12345
./result/bin/program --version
gh pr diff 12345
# Manual code review
gh pr review 12345 --approve -b "[detailed review]"
```

**Use when:**
- New contributor
- Complex changes
- Many reverse dependencies
- Security-sensitive package

### Contribute Review (30-60 minutes)

**What to check:**
- Everything in thorough review
- Multi-platform testing
- Documentation improvements
- Additional test cases
- Performance comparison
- Help fix issues

**Actions:**
```bash
nixpkgs-review pr 12345

# Test on multiple systems
# Check performance
# Add tests if missing
# Fix found issues
# Push fixes to PR branch

gh pr review 12345 --comment -b "
Added tests and fixed issues.
Commits pushed to your branch.
"
```

**Use when:**
- Want to help improve PR
- Know package well
- Have time to contribute
- PR needs extra help

## Common Review Scenarios

### Scenario 1: Simple Version Update

```bash
# PR: firefox: 120.0 -> 121.0

# 1. Quick test
nixpkgs-review pr 12345

# 2. Verify version
./result/bin/firefox --version
# Mozilla Firefox 121.0 ✓

# 3. Check reverse deps
# nixpkgs-review already did this
# 8/8 successful ✓

# 4. Approve
gh pr review 12345 --approve -b "
Tested on x86_64-linux, works perfectly!
All reverse dependencies successful.
"
```

### Scenario 2: New Package

```bash
# PR: mycli: init at 1.0.0

# 1. Test build
nixpkgs-review pr 12345

# 2. Thorough runtime testing
./result/bin/mycli --version
./result/bin/mycli --help
# Try actual functionality

# 3. Review code quality
gh pr diff 12345

# Check:
- Meta attributes complete
- License correct
- Dependencies appropriate
- No unnecessary runtime deps
- Follows nixpkgs conventions

# 4. Security review
# - Service hardening (if applicable)
# - No hardcoded secrets
# - Proper user isolation

# 5. Provide detailed feedback
gh pr review 12345 --comment -b "
Great new package! A few suggestions:

### Tested Successfully
- ✓ Builds on x86_64-linux
- ✓ Binary works correctly
- ✓ All features functional

### Suggestions
1. Add dev output for headers
2. Consider adding passthru.tests
3. Meta.description could be more descriptive

Otherwise looks great!
"
```

### Scenario 3: Build Fix

```bash
# PR: postgresql: fix build on darwin

# 1. Review the fix
gh pr diff 12345

# 2. Test build
nixpkgs-review pr 12345

# 3. Verify fix doesn't break linux
nix-build -A postgresql

# 4. Check if proper solution
# - Is this the right approach?
# - Are there side effects?
# - Is it platform-specific enough?

# 5. Review
gh pr review 12345 --approve -b "
Fix works correctly!

Verified:
- ✓ Builds on x86_64-linux (still works)
- ✓ Reverse dependencies unaffected
- ✓ Platform-specific fix is appropriate

Thanks for the fix!
"
```

### Scenario 4: Security Fix

```bash
# PR: postgresql_15: fix CVE-2024-12345

# 1. Verify CVE information
# Check CVE database
# Verify severity

# 2. Review patch
gh pr diff 12345

# Check:
- Patch from upstream?
- Covers all affected versions?
- No other changes included?

# 3. Test build and reverse deps
nixpkgs-review pr 12345

# 4. Extra testing for security
./result/bin/postgres --version
# Verify patched version

# 5. High priority review
gh pr review 12345 --approve -b "
Security fix verified!

CVE: CVE-2024-12345 (Critical)
Patch: Applied from upstream PostgreSQL 15.5
Testing: ✓ Builds, ✓ 23 reverse dependencies successful

Recommend expedited merge. 🔴 High priority
"
```

### Scenario 5: Breaking Change

```bash
# PR: python311: remove deprecated modules

# 1. Test changed package
nixpkgs-review pr 12345

# 2. Check reverse dependencies carefully
# Many will likely break

# Example output:
# Changed: python311
# Reverse dependencies: 450
# Failed: 23 packages

# 3. Review failures
cd ~/.cache/nixpkgs-review/pr-12345

# Check each failure
# Determine if acceptable or needs fixing

# 4. Detailed feedback
gh pr review 12345 --comment -b "
Tested on x86_64-linux.

### Results
- ✓ python311 builds successfully
- ⚠️ 23 reverse dependencies fail

### Failed Packages
Critical:
- django (widely used)
- numpy (many dependents)

Can be updated:
- ancient-package (unmaintained)
- legacy-tool (deprecated)

### Recommendation
This needs to go through staging branch due to:
1. High rebuild count (450+ packages)
2. Breaking changes
3. Multiple failures need fixes

Suggest:
1. Move to staging
2. Fix critical failures first
3. Coordinate with affected maintainers
"
```

## Review Best Practices

### DO ✅

1. **Always test, don't just review code**
   ```bash
   nixpkgs-review pr 12345
   ./result/bin/program --version
   ```

2. **Check reverse dependencies**
   ```bash
   # nixpkgs-review does this automatically
   # But verify results carefully
   ```

3. **Provide actionable feedback**
   ```
   # BAD
   "Something seems wrong"

   # GOOD
   "The gtk3 dependency should be gtk4.
   Change line 23: gtk3 → gtk4"
   ```

4. **Test on your system**
   ```bash
   # Report your testing environment
   "Tested on x86_64-linux, NixOS 23.11"
   ```

5. **Be constructive and helpful**
   ```
   "Great work! Just one small suggestion..."
   ```

6. **Review security implications**
   ```nix
   # Check service hardening
   # Check secret handling
   # Check user isolation
   ```

### DON'T ❌

1. **Don't approve without testing**
   ```
   # Always run nixpkgs-review
   # Always test the binary
   ```

2. **Don't be vague**
   ```
   # BAD: "Doesn't work"
   # GOOD: "Build fails with error X on line Y"
   ```

3. **Don't forget reverse dependencies**
   ```bash
   # Check that nothing breaks
   nixpkgs-review pr 12345
   ```

4. **Don't ignore ofborg**
   ```
   # Wait for ofborg results
   # Check all platform builds
   ```

5. **Don't be harsh**
   ```
   # Remember: volunteers contributing free time
   # Be respectful and encouraging
   ```

## Using Review Results

### nixpkgs-review Output

**Location:**
```bash
~/.cache/nixpkgs-review/pr-12345/
```

**Contents:**
```
~/.cache/nixpkgs-review/pr-12345/
├── result/           # Built packages
├── report.md         # Test report
├── logs/            # Build logs
└── built-packages   # List of built packages
```

**Report format:**
```markdown
# nixpkgs-review report for PR #12345

## Changed packages
- firefox: 120.0 -> 121.0

## Built packages (1)
- firefox

## Reverse dependencies (8)
- firefox-bin ✓
- firefox-devedition ✓
- firefox-esr ✓
- firefox-wrapper ✗ (failed)
  Error: attribute 'gtk3' missing
  Log: /nix/store/...-firefox-wrapper.drv.log

## Summary
7/8 reverse dependencies successful
1 failure requires attention
```

### Sharing Results

**In GitHub PR:**
```bash
# Share full report
cat ~/.cache/nixpkgs-review/pr-12345/report.md | gh pr comment 12345 --body-file -

# Or summarize
gh pr comment 12345 --body "
nixpkgs-review results:
- ✓ Builds successfully
- ✓ 7/8 reverse deps work
- ✗ 1 failure (firefox-wrapper)

See full report: [attach report.md]
"
```

## Troubleshooting

### Issue 1: nixpkgs-review Fails to Start

**Problem:**
```
Error: Could not fetch PR #12345
```

**Solution:**
```bash
# Check PR exists
gh pr view 12345

# Check nixpkgs-review version
nixpkgs-review --version

# Update if needed
nix-env -f '<nixpkgs>' -iA nixpkgs-review

# Clear cache
rm -rf ~/.cache/nixpkgs-review

# Try again
nixpkgs-review pr 12345
```

### Issue 2: Build Failures

**Problem:**
```
Build failed for package
```

**Solution:**
```bash
# Check build log
cat ~/.cache/nixpkgs-review/pr-12345/logs/package.log

# Try building directly
cd ~/.cache/nixpkgs-review/pr-12345
nix-build -A package

# If issue with PR, leave comment
gh pr comment 12345 --body "
Build failed on x86_64-linux:

\`\`\`
[error message]
\`\`\`

Build log attached.
"
```

### Issue 3: Out of Disk Space

**Problem:**
```
error: no space left on device
```

**Solution:**
```bash
# Clean old reviews
rm -rf ~/.cache/nixpkgs-review/*

# Garbage collect
nix-collect-garbage -d

# Check space
df -h

# Try again with less at once
nixpkgs-review pr 12345 --build-graph 10
```

### Issue 4: Reverse Dependencies Take Too Long

**Problem:**
```
Testing 500 reverse dependencies...
```

**Solution:**
```bash
# Limit reverse dependency testing
nixpkgs-review pr 12345 --no-rev-deps

# Or set a limit
nixpkgs-review pr 12345 --rev-deps-limit 50

# Review main package, note in comment:
"Main package tested successfully.
Reverse dependencies too numerous for full local test.
Recommending reliance on ofborg for reverse dep verification."
```

### Issue 5: Can't Reproduce ofborg Failure

**Problem:**
```
ofborg reports failure on darwin
You only have linux
```

**Solution:**
```
# Note in review:
"Tested successfully on x86_64-linux.
Cannot reproduce darwin failure locally.
Recommend darwin maintainers review ofborg results."

# Tag darwin maintainers if critical
```

## Review Workflow Integration

### Complete Review Process

```bash
# 1. Pick a PR to review
gh pr list --label "11.by: package-maintainer"

# 2. Check basic info
gh pr view 12345

# 3. Run nixpkgs-review
nixpkgs-review pr 12345

# 4. Test runtime
cd ~/.cache/nixpkgs-review/pr-12345
./result/bin/program --version

# 5. Review code
gh pr diff 12345

# 6. Check ofborg
gh pr checks 12345

# 7. Provide feedback
gh pr review 12345 --approve -b "[detailed review]"

# 8. Follow up if needed
# Check for updates
# Re-review after changes
```

### Finding PRs to Review

```bash
# PRs needing review
gh pr list --label "2.status: review needed"

# PRs in your area of expertise
gh pr list --label "6.topic: python"

# New package submissions
gh pr list --label "8.has: package (new)"

# Security fixes (high priority)
gh pr list --label "1.severity: security"

# Your maintained packages
gh pr list --search "firefox"
```

## Expected Review Time

### Simple Version Update
- **Review time**: 5-10 minutes
- **Actions**: Build test, runtime check, approve

### New Package
- **Review time**: 15-30 minutes
- **Actions**: Thorough testing, code review, suggestions

### Complex Change
- **Review time**: 30-60 minutes
- **Actions**: Full review, multi-platform, detailed feedback

### Security Fix
- **Review time**: 15-30 minutes
- **Actions**: CVE verification, testing, expedited review

## Success Checklist

- [x] PR built successfully
- [x] Runtime testing completed
- [x] Reverse dependencies checked
- [x] Code quality reviewed
- [x] Security implications considered
- [x] ofborg results reviewed
- [x] Constructive feedback provided
- [x] Verdict clear (approve/request changes/comment)

## Integration with Other Commands

Works with:
- `/nixpkgs-fork` - Setup required
- `/nixpkgs-test` - Can test same way locally
- `/nixpkgs-pr` - For creating your own PRs

## Impact on Community

### Why Review Matters

1. **Catch Issues Early**
   - Before merge
   - Before channels
   - Before users affected

2. **Help Contributors**
   - Guidance for new contributors
   - Improve code quality
   - Share knowledge

3. **Maintain Quality**
   - Nixpkgs quality standards
   - Security best practices
   - Consistency across packages

4. **Reduce Maintainer Load**
   - More reviewers = faster merges
   - Distributed expertise
   - Community involvement

### Recognition

- Reviews count as contributions
- Build reputation in community
- Path to becoming a committer
- Help shape nixpkgs

Ready to review a nixpkgs PR? Just tell me the PR number!
