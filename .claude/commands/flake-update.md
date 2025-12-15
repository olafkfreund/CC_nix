# NixOS Flake Update with Automated Safety Checks

You are a NixOS system maintenance specialist. Use the automated update subagent to safely update flake inputs and deploy changes with integrated safety checks.

## Overview

This command leverages the **update subagent** for safe, automated NixOS system updates with:
- Pre-update issue detection
- Automatic test rebuilds
- Intelligent problem resolution
- Desktop notifications
- Automatic rollback on failure

## Quick Start

### Option 1: Automated Update (Recommended)

Use the update subagent for a fully automated, safe update process:

```bash
# Invoke the update subagent
# This will:
# 1. Run nix flake update
# 2. Check for known issues (issue-checker)
# 3. Test rebuild
# 4. Fix any problems automatically
# 5. Switch to new configuration
# 6. Send desktop notification

# For manual trigger:
sudo systemctl start nixos-auto-update

# Or if you have the convenience command:
update-system

# Monitor the update process
journalctl -u nixos-auto-update -f
```

The update subagent handles the entire workflow automatically. See `.claude/agents/update.md` for full details.

### Option 2: Manual Update (For Custom Control)

If you need manual control over the update process:

```bash
# 1. Update flake inputs
nix flake update

# 2. Review changes
git diff flake.lock
nix flake metadata

# 3. Test rebuild
nixos-rebuild test

# 4. If tests pass, switch
sudo nixos-rebuild switch

# 5. Verify services
systemctl --failed

# 6. Commit changes
git add flake.lock
git commit -m "chore(flake): update inputs $(date +%Y-%m-%d)"
git push
```

## Understanding the Update Subagent

### What It Does

The update subagent performs an 8-step safety workflow:

1. **Pre-Update Preparation**
   - Backs up current generation
   - Checks disk space (5GB minimum)
   - Verifies network connectivity

2. **Flake Update**
   - Runs `nix flake update`
   - Logs version changes
   - Optional: auto-commits flake.lock

3. **Issue Detection**
   - Invokes issue-checker subagent
   - Identifies critical problems in updated packages
   - Generates risk assessment

4. **Risk Evaluation**
   - Reviews detected issues
   - Determines if safe to proceed
   - Applies mitigations if available

5. **Test Rebuild**
   - Runs `nixos-rebuild test`
   - Captures build errors
   - Identifies failing components

6. **Automatic Issue Resolution**
   - Analyzes build failures
   - Applies known fixes (up to 3 attempts)
   - Retries test rebuild

7. **System Switch**
   - Runs `nixos-rebuild switch` (only if tests pass)
   - Verifies critical services
   - Automatic rollback on failure

8. **Notification & Reporting**
   - Sends desktop notification
   - Generates update report
   - Logs to system journal

### Configuration Options

The update subagent can be configured via environment variables:

```nix
# In your NixOS configuration
systemd.services.nixos-auto-update = {
  serviceConfig = {
    Environment = [
      "NOTIFICATION_ENABLED=true"       # Desktop notifications
      "ROLLBACK_ON_FAILURE=true"        # Auto-rollback on errors
      "MAX_FIX_ATTEMPTS=3"              # Maximum fix retry attempts
      "AUTO_COMMIT_LOCK=false"          # Auto-commit flake.lock
      "AUTO_PROCEED_ON_CRITICAL=false"  # Abort on critical issues
    ];
  };
};
```

## Pre-Update Checks (Manual Process)

If you want to check the system state before triggering the update:

```bash
# 1. Check for open tasks/issues
/check_tasks

# 2. View current flake inputs
nix flake metadata

# 3. Ensure clean working directory
git status

# 4. Check available disk space
df -h /

# 5. Verify network connectivity
ping -c 1 cache.nixos.org
```

## Manual Update Process

For situations where you need manual control:

### Step 1: Update Flake Inputs

```bash
# Update all inputs
nix flake update

# Or update specific inputs only
nix flake lock --update-input nixpkgs
nix flake lock --update-input home-manager
```

### Step 2: Review Changes

```bash
# Show what changed in flake.lock
git diff flake.lock

# Compare metadata before/after
nix flake metadata --json | jq -r '.locks.nodes | keys[]'
```

**Review for:**
- Major version bumps
- Breaking changes in changelogs
- Deprecated package warnings
- Security updates

### Step 3: Test Rebuild (CRITICAL)

```bash
# Test the new configuration
sudo nixos-rebuild test

# If you have validation commands:
nix flake check
```

**Never skip testing!** Always test before switching.

### Step 4: Switch Configuration

```bash
# Only if tests pass
sudo nixos-rebuild switch
```

### Step 5: Verify System

```bash
# Check for failed services
systemctl --failed

# Verify critical services (customize for your system)
systemctl status sshd NetworkManager

# Check system logs for errors
journalctl -p err -b
```

### Step 6: Commit Changes

```bash
# Stage flake.lock
git add flake.lock

# Create detailed commit message
git commit -m "chore(flake): update inputs $(date +%Y-%m-%d)

Updated packages to latest versions.
All tests passed, no service failures detected.

$(nix flake metadata --json | jq -r '.locks.nodes | to_entries[] | select(.value.locked.rev) | "- \(.key): \(.value.locked.rev[0:7])"')
"

# Push to remote
git push
```

## Scheduled Automated Updates

Set up the update subagent to run on a schedule:

```nix
# configuration.nix
{
  # Enable the update service
  systemd.services.nixos-auto-update = {
    description = "Automated NixOS System Update";
    # ... (see .claude/agents/update.md for full config)
  };

  # Schedule updates (weekly on Sunday at 3 AM)
  systemd.timers.nixos-auto-update = {
    wantedBy = [ "timers.target" ];
    timerConfig = {
      OnCalendar = "Sun *-*-* 03:00:00";
      Persistent = true;
    };
  };
}
```

**Other useful schedules:**
- Daily: `OnCalendar = "*-*-* 02:00:00";`
- Weekly: `OnCalendar = "Sun *-*-* 03:00:00";`
- Monthly: `OnCalendar = "*-*-01 04:00:00";`

## Monitoring Updates

### Real-Time Monitoring

```bash
# Follow update process live
journalctl -u nixos-auto-update -f

# View update logs
tail -f /var/log/nixos-auto-update.log

# Check update status
systemctl status nixos-auto-update
```

### Review Update History

```bash
# View all updates in last month
journalctl -u nixos-auto-update --since "1 month ago"

# Check last update report
cat /var/lib/nixos-auto-update/last-update.txt

# View detected issues (if any)
cat /var/lib/nixos-auto-update/issues.json | jq
```

### Success Metrics

```bash
# Count successful updates
journalctl -u nixos-auto-update | grep -c "SUCCESS"

# Count failed updates
journalctl -u nixos-auto-update | grep -c "FAILED"

# View last 10 update results
journalctl -u nixos-auto-update | grep -E "(SUCCESS|FAILED)" | tail -10
```

## Rollback Procedures

### Automatic Rollback

The update subagent automatically rolls back on failure. No manual intervention needed.

### Manual Rollback

If you need to rollback manually:

```bash
# Rollback to previous generation
sudo nixos-rebuild switch --rollback

# Or rollback to specific generation
nixos-rebuild list-generations
sudo nixos-rebuild switch --switch-generation NUMBER
```

### Revert Flake Update

```bash
# Restore previous flake.lock
git checkout HEAD~1 flake.lock

# Verify rollback
nix flake metadata

# Rebuild with old inputs
sudo nixos-rebuild switch
```

## Troubleshooting

### Update Fails to Start

```bash
# Check timer is enabled
systemctl list-timers | grep nixos-auto-update

# Manually trigger update
sudo systemctl start nixos-auto-update

# Check for errors
systemctl status nixos-auto-update
```

### Notifications Not Showing

```bash
# Test notification system
notify-send "Test" "Testing notifications"

# Check notification service
systemctl --user status notification-daemon

# Verify environment variables
systemctl show nixos-auto-update | grep NOTIFICATION
```

### Build Failures

```bash
# View failure logs
cat /var/lib/nixos-auto-update/test-failure.log

# Check what the subagent tried to fix
journalctl -u nixos-auto-update | grep "Fix attempt"

# Manually investigate and fix
nixos-rebuild test --show-trace
```

### Critical Issues Detected

```bash
# View detected issues
cat /var/lib/nixos-auto-update/issues.json | jq

# Review issue-checker recommendations
# Fix critical issues manually before retrying update
```

## Best Practices

### 1. ✅ Use Automated Updates for Regular Maintenance

The update subagent is ideal for:
- Development systems (daily updates)
- Test environments (frequent updates)
- Low-risk production systems (weekly updates)

### 2. ✅ Manual Updates for Critical Systems

Use manual process for:
- Production servers with strict change windows
- Systems requiring extensive validation
- Complex multi-host deployments

### 3. ✅ Monitor Update Success Rates

```bash
# Weekly review of update status
journalctl -u nixos-auto-update --since "1 week ago" | grep -E "(SUCCESS|FAILED)"

# Investigate patterns in failures
journalctl -u nixos-auto-update | grep ERROR | sort | uniq -c
```

### 4. ✅ Test Schedule on Non-Production First

```nix
# Enable on development systems first
config = mkIf (config.networking.hostName == "dev-system") {
  systemd.timers.nixos-auto-update.enable = true;
};
```

### 5. ✅ Maintain Update Documentation

Document any manual interventions needed:
```bash
# Create UPDATE_LOG.md with encountered issues and fixes
echo "$(date): Updated to nixpkgs commit abc123, required manual fix for package X" >> UPDATE_LOG.md
```

### 6. ✅ Keep Multiple Generations

```nix
# Retain last 10 generations for rollback
boot.loader.grub.configurationLimit = 10;
```

### 7. ✅ Backup Before Major Updates

```bash
# Before major version updates, backup important data
# The update subagent handles generation backups automatically
```

### 8. ✅ Review Changes in Flake.lock

Even with automated updates, periodically review:
```bash
# What packages were updated this week
git log --since="1 week ago" --oneline flake.lock
git show <commit> -- flake.lock
```

## Integration with Other Workflows

### With GitHub Workflow

```bash
# Create update tracking issue
/new_task "Review and validate weekly flake updates"

# After automated update
git commit -m "chore(flake): automated update $(date +%Y-%m-%d) (#ISSUE_NUMBER)"
```

### With Monitoring

The update subagent integrates with:
- **Grafana**: Track update frequency and success rates
- **Prometheus**: Monitor system health after updates
- **Alertmanager**: Alert on update failures

### With issue-checker Subagent

The update subagent automatically uses issue-checker:
```bash
# issue-checker runs before test rebuild
# Identifies critical issues in updated packages
# Provides recommendations (PROCEED, CAUTION, DELAY)
```

## Advanced Configuration

### Custom Fix Patterns

Add project-specific fixes to the update script:

```bash
# In nixos-auto-update.sh, add to fix_build_issues()
if echo "$error_log" | grep -q "your-specific-error"; then
    log_info "Applying custom fix"
    # Your fix here
fi
```

### Multi-Host Updates

```nix
# Update multiple hosts sequentially
systemd.services.nixos-auto-update-fleet = {
  script = ''
    for host in host1 host2 host3; do
      ssh $host "systemctl start nixos-auto-update"
      sleep 300  # Wait 5 minutes between hosts
    done
  '';
};
```

### Notification Customization

```bash
# Add Slack/Email notifications
send_slack_notification() {
    curl -X POST -H 'Content-type: application/json' \
        --data "{\"text\":\"NixOS Update: $1\"}" \
        "$SLACK_WEBHOOK_URL"
}
```

## Success Criteria

After an update (manual or automated), verify:

- [ ] Flake inputs updated to latest versions
- [ ] System rebuilt successfully
- [ ] No failed systemd services
- [ ] Critical services verified operational
- [ ] Desktop environment functional (if applicable)
- [ ] Network connectivity working
- [ ] Changes committed to git
- [ ] No unexpected errors in logs

## Common Issues

### Hash Mismatch Errors

```bash
# Clear evaluation cache
nix-collect-garbage -d

# Retry update
sudo nixos-rebuild test
```

### Service Won't Start After Update

```bash
# Check service logs
journalctl -xeu SERVICE_NAME

# Compare service configurations
systemctl cat SERVICE_NAME
git diff HEAD~1 -- path/to/service/config

# Rollback if needed
sudo nixos-rebuild switch --rollback
```

### Package Conflicts

```bash
# Check for conflicts
nix flake check --show-trace

# Review package versions
nix-env -qaP | grep package-name

# May need to pin specific package version
```

## Documentation References

- `@.claude/agents/update.md` - Full update subagent documentation
- `@.claude/agents/issue-checker.md` - Issue detection details
- `@.claude/NIX_ANTIPATTERNS.md` - Patterns to avoid
- NixOS Manual: https://nixos.org/manual/nixos/stable/

## Example Workflows

### Workflow 1: Fully Automated Weekly Updates

```nix
# Set up in configuration.nix
{
  # Enable auto-update service and timer
  # Updates run every Sunday at 3 AM
  # Desktop notification on completion
  # Automatic rollback on failure
}
```

Monitor via:
```bash
# Check weekly results
journalctl -u nixos-auto-update --since "1 week ago"
```

### Workflow 2: Manual with Safety Checks

```bash
# 1. Update inputs
nix flake update

# 2. Let issue-checker review
# (Invoke manually or wait for update subagent)

# 3. Test carefully
nixos-rebuild test

# 4. Deploy if safe
sudo nixos-rebuild switch

# 5. Verify and commit
systemctl --failed
git add flake.lock && git commit && git push
```

### Workflow 3: Emergency Security Update

```bash
# 1. Update specific package
nix flake lock --update-input nixpkgs

# 2. Quick test
nixos-rebuild test

# 3. Emergency switch
sudo nixos-rebuild switch

# 4. Verify critical services
systemctl status sshd NetworkManager

# 5. Quick commit
git add flake.lock
git commit -m "security: emergency nixpkgs update for CVE-XXXX-YYYY"
git push
```

## Summary

**For most users:** Use the automated update subagent for safe, hands-off updates with integrated safety checks and notifications.

**For advanced users:** Use manual process when you need precise control over the update workflow.

**For production systems:** Start with manual updates, transition to automated updates after validating the subagent on test systems.

The update subagent provides the best balance of automation and safety for regular NixOS system maintenance.
