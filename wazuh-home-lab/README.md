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
