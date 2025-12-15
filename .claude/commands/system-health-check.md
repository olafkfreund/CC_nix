# Comprehensive System Health Check

You are a NixOS infrastructure monitoring specialist. Perform comprehensive health check across all infrastructure components.

## Task Overview

Execute a thorough health assessment of the entire NixOS infrastructure, identifying issues and recommending actions.

## Pre-Check Setup

**Note**: This command provides a template for system health checks. Replace `HOST1`, `HOST2`, etc. with your actual hostnames, and adapt service checks to match your infrastructure.

```bash
# Ensure you have access to all hosts
for host in HOST1 HOST2 HOST3; do
  ping -c 1 $host >/dev/null && echo "$host: OK" || echo "$host: FAILED"
done

# Check monitoring services are running (if you use them)
systemctl status grafana
systemctl status prometheus
systemctl status node-exporter
```

## Health Check Components

### 1. Host Connectivity Check

```bash
# Test network connectivity to all hosts
# Replace with your actual hostnames/domains
for host in host1.domain.com host2.domain.com host3.domain.com; do
  ping -c 3 $host
done

# Test SSH connectivity
for host in host1 host2 host3; do
  echo "=== Testing $host ==="
  ssh -o ConnectTimeout=5 $host "hostname && uptime"
done

# Test VPN connectivity (if using Tailscale/WireGuard)
tailscale status  # or: wg show
```

**Record:**
- Which hosts are reachable
- Response times
- Any connectivity issues

### 2. System Services Status

```bash
# Check for failed systemd services on each host
for host in host1 host2 host3; do
  echo "=== $host ==="
  ssh $host "systemctl --failed --no-pager"
  echo
done
```

**For each failed service:**
1. Determine criticality (critical/high/medium/low)
2. Check logs: `journalctl -u SERVICE_NAME -n 50`
3. Determine if action needed

### 3. Critical Services Verification

Check essential services on each host (adapt to your services):

```bash
# Example: Check monitoring services on monitoring host
ssh monitoring-host "systemctl is-active prometheus grafana node-exporter"

# Example: Check application services
for host in app-host1 app-host2; do
  echo "=== $host ==="
  ssh $host "systemctl is-active YOUR_SERVICE_1 YOUR_SERVICE_2 node-exporter"
done

# List all active services to identify what to monitor
ssh YOUR_HOST "systemctl list-units --type=service --state=running"
```

### 4. Disk Usage Analysis

```bash
# Check disk usage on all hosts
for host in host1 host2 host3; do
  echo "=== $host Disk Usage ==="
  ssh $host "df -h / /nix/store /home 2>/dev/null | grep -v tmpfs"
  echo
done

# Check Nix store size and identify cleanup candidates
for host in host1 host2 host3; do
  echo "=== $host Nix Store Analysis ==="
  ssh $host "du -sh /nix/store 2>/dev/null"
  ssh $host "nix-store --gc --print-dead | wc -l | xargs echo 'Dead paths:'"
  echo
done
```

**Disk space warnings:**
- >90% usage: CRITICAL - immediate action required
- >80% usage: WARNING - cleanup recommended
- >70% usage: NOTICE - monitor closely

**Recommended actions if >80%:**
```bash
# Safe cleanup on affected host
ssh HOST "nix-collect-garbage -d"
ssh HOST "nix-store --gc"
ssh HOST "nix-store --optimize"
```

### 5. Memory and CPU Usage

```bash
# Current resource usage
for host in host1 host2 host3; do
  echo "=== $host Resources ==="
  ssh $host "free -h && echo && top -bn1 | head -20"
  echo
done
```

**Check for:**
- High memory usage (>90%)
- High swap usage (>50%)
- High CPU load (>80% sustained)
- Zombie processes

### 6. Boot Time Analysis

```bash
# Check boot times
for host in host1 host2 host3; do
  echo "=== $host Boot Analysis ==="
  ssh $host "systemd-analyze"
  ssh $host "systemd-analyze blame | head -10"
  echo
done
```

**Concerns:**
- Boot time >2 minutes: investigate slow services
- Any service >30 seconds: optimization candidate

### 7. Monitoring Stack Health (if using Prometheus/Grafana)

```bash
# Replace MONITORING_HOST with your monitoring server hostname/IP
MONITORING_HOST="monitoring-server"

# Prometheus targets status
curl -s http://$MONITORING_HOST:9090/api/v1/targets | jq '.data.activeTargets[] | {job: .labels.job, instance: .labels.instance, health: .health}'

# Grafana status
systemctl status grafana

# Alertmanager status
curl -s http://$MONITORING_HOST:9093/api/v2/status | jq '.uptime, .cluster.peers[]'

# Check for firing alerts
curl -s http://$MONITORING_HOST:9093/api/v2/alerts | jq '.[] | select(.status.state == "active")'
```

**Verify:**
- All exporters reporting (should be 4+ per host)
- No gaps in metrics (check Grafana dashboards)
- Alert rules loading correctly
- No firing alerts (unless expected)

### 8. Application-Specific Health Checks

```bash
# Example: Check your specific application services
# Replace with your actual services and hosts

# Web application status
ssh app-server "systemctl status your-web-app"

# Database status
ssh db-server "systemctl status postgresql"

# Cache status
ssh cache-server "systemctl status redis"

# List all services to identify what to monitor
systemctl list-units --type=service --state=running
```

### 9. Network Stability

```bash
# VPN status (if using Tailscale/WireGuard)
ssh your-host "tailscale status"  # or: wg show

# DNS resolution check
for host in host1 host2 host3; do
  echo "=== $host DNS ==="
  ssh $host "resolvectl status | grep 'DNS Servers'"
  ssh $host "nslookup google.com | grep Server"
  echo
done

# Check for DNS conflicts
ssh your-host "resolvectl status"
```

### 10. Security Audit

```bash
# Check for security updates
for host in host1 host2 host3; do
  echo "=== $host Security ==="
  ssh $host "nixos-rebuild dry-run 2>&1 | grep -i security || echo 'No security updates'"
  echo
done

# Check fail2ban status (if configured)
ssh your-host "systemctl is-active fail2ban && fail2ban-client status"

# Check firewall rules
for host in host1 host2 host3; do
  echo "=== $host Firewall ==="
  ssh $host "iptables -L -n | grep -c 'ACCEPT\\|DROP\\|REJECT' | xargs echo 'Rules:'"
  echo
done
```

### 11. Backup Status

```bash
# Check backup configurations
for host in host1 host2 host3; do
  echo "=== $host Backup Status ==="
  ssh $host "systemctl list-timers | grep backup || echo 'No backup timers configured'"
  echo
done
```

### 12. GitHub Issues Review

```bash
# Check open issues
/check_tasks

# Focus on:
# - Critical priority issues
# - Blocked issues
# - Stale issues (>30 days)
```

## Generate Health Report

Create a comprehensive report with the following sections:

### Executive Summary

```markdown
# Infrastructure Health Report - [DATE]

## Overall Status: [HEALTHY/DEGRADED/CRITICAL]

- Total Hosts: [YOUR_HOST_COUNT]
- Hosts Online: X/[YOUR_HOST_COUNT]
- Critical Issues: X
- Warnings: X
- Open Tasks: X
```

### Host Status Summary

```markdown
## Host Status

| Host | Status | Uptime | Failed Services | Disk Usage | Memory | CPU Load |
|------|--------|--------|----------------|------------|--------|----------|
| P620 | 🟢/🟡/🔴 | Xd | X | X% | X% | X.XX |
| Razer | 🟢/🟡/🔴 | Xd | X | X% | X% | X.XX |
| P510 | 🟢/🟡/🔴 | Xd | X | X% | X% | X.XX |
| Samsung | 🟢/🟡/🔴 | Xd | X | X% | X% | X.XX |

Legend: 🟢 Healthy | 🟡 Warning | 🔴 Critical
```

### Critical Issues

```markdown
## 🔴 Critical Issues (Immediate Action Required)

1. **[Issue Description]**
   - Affected: [Host(s)]
   - Impact: [User impact]
   - Action: [Required action]
   - ETA: [Time to fix]

[Repeat for each critical issue]
```

### Warnings

```markdown
## 🟡 Warnings (Action Recommended)

1. **[Warning Description]**
   - Affected: [Host(s)]
   - Risk: [Potential impact]
   - Recommendation: [Suggested action]
   - Priority: [High/Medium/Low]

[Repeat for each warning]
```

### Monitoring Status

```markdown
## 📊 Monitoring Stack

- **Prometheus:** [Status] - [X targets, Y series]
- **Grafana:** [Status] - [X dashboards]
- **Alertmanager:** [Status] - [X alerts firing]
- **Exporters:** [X/Y reporting]

### Firing Alerts

[List any active alerts]

### Missing Metrics

[List any hosts/services not reporting]
```

### Service Health

```markdown
## 🛠️ Service Health

### P620 (Monitoring Server)
- Prometheus: [Status]
- Grafana: [Status]
- AI Services: [Status]
- [List failed services]

### P510 (Media Server)
- Plex: [Status] - [X active streams]
- NZBGet: [Status] - [X active downloads]
- [List failed services]

### Razer/Samsung (Mobile)
- Node Exporter: [Status]
- [List failed services]
```

### Resource Utilization

```markdown
## 💾 Resource Utilization

### Disk Usage
- P620: [X%] - [Action: none/cleanup/urgent]
- Razer: [X%] - [Action: none/cleanup/urgent]
- P510: [X%] - [Action: none/cleanup/urgent]
- Samsung: [X%] - [Action: none/cleanup/urgent]

### Memory Usage
- P620: [X%] - [Swap: X%]
- Razer: [X%] - [Swap: X%]
- P510: [X%] - [Swap: X%]
- Samsung: [X%] - [Swap: X%]

### Nix Store
- Total size: [X GB]
- Dead paths: [X]
- Cleanup potential: [X GB]
```

### Recommendations

```markdown
## 💡 Recommendations

### Immediate Actions
1. [Action 1]
2. [Action 2]

### Short-term (This Week)
1. [Action 1]
2. [Action 2]

### Long-term (This Month)
1. [Action 1]
2. [Action 2]

### Preventive Measures
1. [Measure 1]
2. [Measure 2]
```

### Open Issues Summary

```markdown
## 📋 Open GitHub Issues

- Total: X
- Critical: X
- High Priority: X
- Blocked: X
- Stale (>30 days): X

### Top Priority Issues
1. #X - [Issue title] - [Priority]
2. #X - [Issue title] - [Priority]
```

## Action Items

Generate prioritized action items:

### 🔴 CRITICAL (Do Now)

- [ ] [Action item with specific command]
- [ ] [Action item with specific command]

### 🟡 HIGH (Do Today)

- [ ] [Action item with specific command]
- [ ] [Action item with specific command]

### 🟢 MEDIUM (Do This Week)

- [ ] [Action item with specific command]
- [ ] [Action item with specific command]

### ⚪ LOW (Monitor)

- [ ] [Action item with specific command]
- [ ] [Action item with specific command]

## Success Criteria

- [ ] All hosts connectivity verified
- [ ] Failed services identified and categorized
- [ ] Disk usage checked and cleanup performed if needed
- [ ] Monitoring stack verified operational
- [ ] Critical services confirmed running
- [ ] Resource utilization analyzed
- [ ] Security status reviewed
- [ ] Comprehensive report generated
- [ ] Action items prioritized
- [ ] Recommendations documented

## Follow-up Actions

```bash
# Schedule next health check
echo "$(date -d '+7 days' '+%Y-%m-%d'): Run /system-health-check" >> docs/scheduled-tasks.txt

# Create issues for critical items
/new_task  # For each critical issue identified

# Update monitoring if gaps found
# Edit modules/services/monitoring.nix
```

## Documentation

Save the health report to:

```bash
# Create report file
vim docs/health-reports/health-report-$(date +%Y-%m-%d).md

# Link to latest
ln -sf health-report-$(date +%Y-%m-%d).md docs/health-reports/latest.md
```

## Notes

- Run health checks weekly (or after major changes)
- Compare trends with previous reports
- Update monitoring alerts based on findings
- Document any recurring issues
- Track action item completion rates

## Example Output

```markdown
# Infrastructure Health Report - [DATE]

## Overall Status: HEALTHY ✅

- Total Hosts: X/X online
- Critical Issues: 0
- Warnings: 2
- Open Tasks: 5

## Summary
All systems operational. Minor cleanup recommended on host3 (disk 78%).
Monitoring stack fully functional. No critical alerts.

## Action Items
- [ ] Run cleanup on host3 (disk usage high)
- [ ] Investigate host2 DNS intermittent issues
- [ ] Update 3 packages with security fixes
```
