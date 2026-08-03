# Wazuh SOC Lab — Agent Auto-Start & File Integrity Monitoring

**Environment**
- Wazuh v4.9.2 (Docker Desktop, single-node deployment)
- Agent: Ubuntu 26.04 (`ubuntu2604`) on VirtualBox
- Manager/Indexer/Dashboard: Docker containers on host

## 1. Agent Auto-Start on Boot

**Problem:** Agent required manual `start` after every VM reboot; did not appear in dashboard automatically.

**Root cause:** Service was started but not enabled — `start` vs. `enable` are separate systemd actions.

**Fix:**
```bash
sudo systemctl enable --now wazuh-agent
sudo systemctl is-enabled wazuh-agent   # verify: should return "enabled"
```

**Result:** Confirmed working across a full reboot. Dashboard recognizes the agent as Active roughly 60–90 seconds after boot (normal delay — keep-alive interval + manager→indexer→dashboard pipeline).

## 2. File Integrity Monitoring (FIM) Setup

**Goal:** Monitor key Linux directories for file changes and surface alerts in the dashboard.

**Approach:** Centralized config via agent group (`agent.conf`), rather than local `ossec.conf`, to mirror real SOC practice of pushing policy from the manager.

**Config added (Server Management → Groups → group config):**
```xml
<agent_config>
  <syscheck>
    <disabled>no</disabled>
    <frequency>43200</frequency>

    <directories realtime="yes" report_changes="yes">/etc</directories>
    <directories realtime="yes" report_changes="yes">/root</directories>
    <directories realtime="yes" report_changes="yes">/usr/bin,/usr/sbin</directories>

    <ignore>/etc/mtab</ignore>
    <ignore>/etc/hosts.deny</ignore>
    <ignore type="sregex">.log$|.swp$</ignore>

    <alert_new_files>yes</alert_new_files>
    <scan_on_start>yes</scan_on_start>
  </syscheck>
</agent_config>
```

**Verification steps used:**
1. Confirmed agent status and config sync from the manager container:
   ```bash
   docker exec -it <manager_container_name> bash
   /var/ossec/bin/agent_control -l          # list agents, get ID
   /var/ossec/bin/agent_control -i 001      # detailed status for this agent
   ```
   Output confirmed `Status: Active`, plus `Syscheck last started/ended` timestamps showing the baseline scan completed (~3.5 min for first full hash pass).

2. Triggered a live test event on the agent VM:
   ```bash
   sudo touch /etc/test_fim_file.txt
   sudo rm /etc/test_fim_file.txt
   ```

3. Confirmed the alert in the dashboard: **Integrity Monitoring → Events** tab (not the summary Dashboard tab — that view aggregates and doesn't show individual file paths). Events tab shows `syscheck.path`, `syscheck.event` (added/modified/deleted) per row.

**Result:** Realtime FIM alerts confirmed working end-to-end, with full file paths visible.

## Notes / Next Steps
- Tune `<ignore>` list further based on normal system noise observed over the next few days.
- Test `report_changes="yes"` diff output by modifying (not just creating/deleting) a monitored file's contents.
- Default dashboard/indexer credentials (`admin`/default password) still need to be rotated — documented separately once completed.
- Consider scoping monitored directories tighter for a production-like configuration (currently broad `/etc` for learning purposes).
