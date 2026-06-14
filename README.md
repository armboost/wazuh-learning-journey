# Wazuh Learning Journey 🛡️

A personal homelab documentation project tracking my hands-on experience with Wazuh SIEM/XDR as part of my cybersecurity SOC Analyst career path.

---

## About This Repo

This repository documents my real-world lab work with Wazuh, including setup, configuration, troubleshooting, and security exercises. Each entry reflects actual work done in my home lab environment.

**Lab Environment:**
- **Manager:** Windows bare metal (Wazuh 4.9.2 dashboard)
- **Agent:** Ubuntu 22.04 VM running in VirtualBox
- **Host Machine:** Asus Nitro AN515 (8GB RAM, GTX 1050, DDR4 2667MHz)
- **Network:** Bridged Adapter (VirtualBox) — Agent IP reaching Manager at `192.168.1.x`

---

## Session 1 — Wazuh Agent Setup & First Alerts

**Date:** June 2026

### What I Did

#### 1. Installed Wazuh Agent on Ubuntu VM
- Set VirtualBox network adapter to **Bridged Mode** so the VM could reach the Windows host manager
- Installed Wazuh agent via apt package manager
- Configured `/var/ossec/etc/ossec.conf` with manager IP address

**ossec.conf client block:**
```xml
<client>
  <server>
    <address><Wazuh IP Address></address>
    <port>1514</port>
    <protocol>tcp</protocol>
  </server>
</client>
```

#### 2. Version Mismatch Problem & Fix
**Problem:** Agent installed at version `4.14.5.1` but manager was running `4.9.2`. Wazuh requires the agent version to be equal to or lower than the manager.

**Error seen:**
```
Agent version must be lower or equal to manager version
```

**Fix:**
```bash
sudo apt remove wazuh-agent
sudo apt purge wazuh-agent
sudo apt install wazuh-agent=4.9.2-1
```

**Lesson learned:** Always check manager version before installing agent. After purging, config files are wiped — remember to re-add manager IP to `ossec.conf`.

#### 3. Invalid Server Address Error
After reinstalling, the agent failed to start with:
```
Invalid server address found
```

**Cause:** Manager IP was not re-added to `ossec.conf` after purge.

**Fix:** Re-added manager IP block to `/var/ossec/etc/ossec.conf`, then:
```bash
sudo systemctl restart wazuh-agent
sudo systemctl status wazuh-agent
```

#### 4. Agent Connected Successfully
Confirmed agent appeared as **Active** in Wazuh Dashboard under Endpoints.

---

### First Security Exercise — Brute Force Simulation

Simulated authentication failures to trigger Wazuh alerts:
```bash
su wrongpassword
# Repeated 5-6 times
```

**Result:** Authentication failure alerts appeared in real time in the Wazuh dashboard, mapped to **MITRE ATT&CK T1110 — Brute Force**.

**Key insight:** This is exactly what a SOC Analyst monitors for in a real environment.

---

## Concepts Learned

### File Integrity Monitoring (FIM)
Wazuh's FIM module watches for file changes across monitored directories. Configured in `ossec.conf` under the `<syscheck>` block.

**Default monitored paths:**
- `/etc` — system configuration
- `/usr/bin`, `/usr/sbin` — binaries
- `/bin`, `/sbin` — essential commands

**Custom FIM configuration:**
```xml
<syscheck>
  <directories realtime="yes" report_changes="yes">/etc</directories>
  <ignore>/etc/mtab</ignore>
  <frequency>43200</frequency>
</syscheck>
```

**Next exercise:** Create a test file in `/etc` and observe FIM alert in dashboard.

---

### MITRE ATT&CK Integration
Wazuh maps alerts natively to MITRE ATT&CK techniques. No separate API setup needed — metadata is baked into detection rules.

**Rule fields:**
- `rule.mitre.id` — e.g. T1110
- `rule.mitre.tactic` — e.g. Credential Access
- `rule.mitre.technique` — e.g. Brute Force

**Dashboard:** Threat Intelligence → MITRE ATT&CK heatmap

---

### WSL2 Memory Configuration
To prevent OOM (Out of Memory) issues when running resource-heavy tools, WSL2 resources can be tuned via `.wslconfig`:

**Location:** `C:\Users\YourUsername\.wslconfig`

**Current config (8GB RAM):**
```ini
[wsl2]
memory=3GB
processors=2
swap=2GB
```

**After RAM upgrade to 16GB:**
```ini
[wsl2]
memory=6GB
processors=4
swap=4GB
```

Apply changes:
```powershell
wsl --shutdown
```

---

### OOM (Out of Memory) Diagnosis & Fix

**Diagnose:**
```bash
sudo dmesg | grep -i "out of memory"
sudo journalctl -k | grep -i "oom"
free -h
top
```

**Fix — Add swap space:**
```bash
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

---

## Linux Directory Reference

| Directory | Purpose |
|---|---|
| `/etc` | System configuration files |
| `/bin` `/sbin` | Essential system binaries |
| `/usr/bin` `/usr/sbin` | User/admin installed programs |
| `/var/log` | System logs |
| `/var/ossec` | Wazuh agent files |
| `/tmp` | Temporary files (clears on reboot) |
| `/home` | User directories |
| `/root` | Root user home |
| `/proc` | Live kernel/process info |
| `/dev` | Device files |

> **Security note:** `/tmp` is world-writable — a common attacker drop location. Monitor it.

---

## Lab Hardware

| Component | Spec |
|---|---|
| Laptop | Asus Nitro AN515 (2017) |
| RAM | 8GB SK Hynix DDR4 2667MHz (upgrading to 16GB) |
| GPU | NVIDIA GTX 1050 |
| VM Platform | VirtualBox |
| Agent OS | Ubuntu 22.04 |

---

## Certifications & Background

- CompTIA Security+
- GIAC GCTI
- CompTIA A+
- Network+ (in progress)
- 20+ years Deaf Education / ASL
- Prior Public Trust clearance (Skookum/USCG)

---

## Next Steps

- [ ] FIM test — create file in `/etc` and observe alert
- [ ] Customize `syscheck` block in `ossec.conf`
- [ ] Explore MITRE ATT&CK heatmap in dashboard
- [ ] Install Ollama locally after RAM upgrade
- [ ] Begin Wazuh query language practice
- [ ] Document all lab exercises for SOC portfolio

---

*This repo is part of my ongoing preparation for the DEAFCYBER Hiring Village (October 2026) and SOC Analyst roles in the NoVA/DC corridor.*
