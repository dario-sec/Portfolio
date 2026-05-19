# 🔬 Wazuh SIEM Deployment & Threat Detection Laboratory

This laboratory demonstrates the implementation of a centralized monitoring infrastructure inside an isolated virtual enterprise network. The primary focus is investigating telemetry pipeline ingestion, log parsing, and mapping system audits to the worldwide standard **MITRE ATT&CK® framework**.

---

## 🏗️ Deployment Architecture

The environment is strictly orchestrated within an isolated hypervisor network framework to prevent any out-of-band traffic leakage:

* **SIEM Controller Node:** Deployed a self-hosted **Wazuh Stack** (Manager, Indexer, and Dashboard interface).
* **Audited Endpoint:** Kali Linux endpoint running the integrated host-level Wazuh agent daemon.
* **Network Topology:** Isolated VirtualBox `NAT Network` with strict static IP assignments (`10.0.2.x/24`) to enforce consistent log source correlation.

---

## ⚔️ Threat Scenarios, Log Decoders & Analysis

The security baseline was validated by executing distinct threat vectors to analyze how Unix daemons handle authentication and system state modifications.

### 1. Reconnaissance: Service Discovery (`T1046`)
* **Vector:** Network scanning via target port interrogation using Nmap.
* **Analysis:** Validated network footprint detection and telemetry logging through standard firewall/connection drop logs in syslog pipelines.

### 2. Credential Access: SSH Brute-Force (`T1110`)
* **Vector:** Automated multi-threaded dictionary attack targeting port 22.
* **Telemetry Path:** Handled by the Linux Pluggable Authentication Modules (`PAM`) subsystem and logged into `/var/log/auth.log` under the `sshd` session context.
* **SIEM Correlation:** Triggered **Wazuh Rule ID 2502** (*"syslog: User missed the password more than one time"*).
* **Evidence Payload (JSON structure analyzed):**
```json
{
  "agent": { "id": "001", "name": "LinuxAgent", "ip": "10.0.2.4" },
  "decoder": { "name": "sshd" },
  "rule": {
    "id": "2502",
    "level": 10,
    "mitre": { "id": "T1110", "technique": "Brute Force" }
  }
}
```
### 3. Persistence: Unauthorized Account Creation (`T1136`)
* **Vector:** Adversary attempting to create a local user backdoor command (`useradd hacker`) to maintain administrative hold.
* **Telemetry Path:** Logged directly by core system hooks and ingested through systemd service managers via `journald`.
* **SIEM Correlation:** Triggered **Wazuh Rule ID 5902** (*"New user added to the system"*), isolating parameters such as UID, GID, and new directory setups.
* **Evidence Payload (JSON structure analyzed):**
```json
{
  "decoder": { "name": "useradd" },
  "full_log": "new user: name=hacker, UID=1001, GID=1001, home=/home/hacker",
  "rule": {
    "id": "5902",
    "level": 8,
    "mitre": { "id": "T1136", "tactic": "Persistence" }
  }
}
```
---

## 📊 Lab Findings & Incident Response Takeaways

* **Log Centralization Works:** The unified core agent smoothly managed data streaming from the host subsystem to the dashboard with low resource usage.
* **Structured Data is Key:** Parsing raw server text logs into clear JSON telemetry allows security tools to read, filter, and alert on incidents instantly.
* **Defensive Baseline:** Studying the raw rule logic inside the SIEM proved that proper system defense requires deep knowledge of OS log structures.

---
*Developed as part of the High School Capstone Academic Portfolio ("Capolavoro").*
