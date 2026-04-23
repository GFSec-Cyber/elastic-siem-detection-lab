# Elastic SIEM Home Lab — Live Threat Detection

A fully functional security information and event management (SIEM) 
lab built on Elastic Stack 8.19, featuring real-time log ingestion, 
custom detection rules, and live attack simulations mapped to the 
MITRE ATT&CK framework.

---

## What this project demonstrates

- Deploying and configuring a production-grade SIEM from scratch
- Shipping endpoint logs to Elasticsearch using Filebeat
- Writing custom KQL detection rules in Kibana Security
- Simulating real adversary techniques using Kali Linux tools
- Mapping detections to MITRE ATT&CK tactics and techniques

---

## Lab architecture

| Component     | Details                        |
|---------------|-------------------------------|
| SIEM server   | Ubuntu 22.04, Elastic Stack 8.19 (Elasticsearch + Kibana + Logstash) — 172.16.109.129 |
| Endpoint      | Ubuntu 24.04, Filebeat 8.19 — 172.16.109.130 |
| Attack machine | Kali Linux 2025.3 — 172.16.109.128 |
| Network       | VMware host-only isolated network |

---

## Tools used

- Elastic Stack 8.19 (Elasticsearch, Kibana, Logstash)
- Filebeat 8.19
- Kali Linux (nmap, Hydra)
- VMware Fusion
- KQL (Kibana Query Language)

---

## Detection rules

All rules were authored in Kibana Security SIEM and exported as 
NDJSON. See the /rules folder.

| Rule name | Type | Severity | MITRE Tactic | Technique |
|-----------|------|----------|--------------|-----------|
| SSH Brute Force Attempt | Threshold (≥5 from same IP) | High | Credential Access | T1110.001 |
| Potential Port Scan — Invalid SSH User | Query | Medium | Discovery | T1046 |
| Root Login Attempt via SSH | Query | High | Privilege Escalation | T1078.001 |
| New Local User Account Created | Query | Medium | Persistence | T1136.001 |

---

## Attack simulations

All attacks were conducted inside the isolated lab network 
from the Kali Linux VM against the Ubuntu endpoint.

### 1. Network reconnaissance — T1046 (Discovery)
```bash
nmap -sV -p 22,80,443,3389 172.16.109.130
```
Purpose: identify open ports and running services on the endpoint.

### 2. SSH brute force — T1110.001 (Credential Access)
```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt \
  ssh://172.16.109.130 -t 4 -V
```
Purpose: simulate automated password guessing against the root account.

### 3. Lateral movement — T1021.004 (Lateral Movement)
```bash
ssh user@172.16.109.129
ssh garrett@172.16.109.129
```
Purpose: simulate an attacker moving between systems using stolen credentials.

### 4. Backdoor account creation — T1136.001 (Persistence)
```bash
sudo useradd backdooruser
```
Purpose: simulate an attacker establishing persistence via a new local account.

---

## Results

43 total alerts generated across 3 detection rules within a 
single simulated attack session.

| Rule | Alerts fired | Severity |
|------|-------------|----------|
| Potential Port Scan — Invalid SSH User | 41 | Medium |
| Root Login Attempt via SSH | 1 | High |
| New Local User Account Created | 1 | Medium |

---

## Key troubleshooting challenges

This section documents real issues encountered and resolved 
during the build — included because troubleshooting is a 
core SOC analyst skill.

**Issue 1 — Kibana encryption key length**
Kibana 8.x requires a minimum 32-character encryption key 
for xpack.encryptedSavedObjects. An initial 31-character key 
caused a fatal startup error. Fixed by generating a correct 
key with: openssl rand -hex 16

**Issue 2 — Filebeat 401 Unauthorized**
Filebeat on the endpoint was rejecting connections to 
Elasticsearch with a 401 error. Root cause: incorrect 
credentials in filebeat.yml. Fixed by resetting the elastic 
user password and updating the Filebeat config.

**Issue 3 — Log field parsing**
Filebeat 8.19 ships auth logs as raw message strings rather 
than parsed ECS fields, meaning system.auth.ssh.event did 
not exist as a queryable field. Detection rules were 
rewritten to query the message field directly using wildcard 
matching.

---

## Repository structure

elastic-siem-detection-lab/
├── README.md
├── MITRE_COVERAGE.md
├── lab-setup.md
├── rules/
│   ├── ssh_brute_force.ndjson
│   ├── port_scan_detection.ndjson
│   ├── root_login_attempt.ndjson
│   └── new_user_created.ndjson
├── screenshots/
│   ├── alerts_firing.png
│   ├── security_rules_created.png
│   ├── ssh_brute_force_rule.png
│   ├── root_login_rule.png
│   ├── port_scan_rule.png
│   ├── new_user_rule.png
│   ├── hydra_attack.png
│   ├── nmap_scan.png
│   ├── kibana_discover_logs.png
│   └── elasticsearch_kibana_logstash_active.png
└── attack-simulations/
    └── simulation_playbook.md

---

## Author

Garrett Fegley
CompTIA Security+
https://linkedin.com/in/[your-handle]
