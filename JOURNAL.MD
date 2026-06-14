## 2026-06-13 — End-to-End Brute Force Detection (T1110)

### What I did
- Built custom Wazuh dashboard visualizations for MITRE ATT&CK T1110 (Brute Force):
  - Vertical bar chart: alert volume over time, split by rule description
  - Data table: agent ID, timestamp, and rule description for each alert
- Simulated a brute force attack from Kali Linux using Hydra against the agent VM's SSH service
- Confirmed detection: Wazuh generated T1110-tagged alerts that populated both visualizations in real time

### Attack details
- Tool: Hydra (`hydra -l <username> -P <wordlist> ssh://<agent-IP> -t 4`)
- Target: SSH service on agent VM
- Result: Multiple failed auth attempts → Wazuh sshd rules → MITRE T1110 alert

### What I learned
- [add your own notes here — e.g., observations on bucket aggregations, MITRE field mapping, anything that surprised you]

### Screenshots
![Kali attack](../assets/screenshots/2026-06-13-kali-attack.png)
![Wazuh dashboard](../assets/screenshots/2026-06-13-dashboard.png)

### Next steps
- [ ] Build recon dashboard (T1046/T1018) — Kali nmap scans against agent
- [ ] Investigate whether plain port scans generate Wazuh alerts without additional log sources (auditd, firewall, Suricata)
