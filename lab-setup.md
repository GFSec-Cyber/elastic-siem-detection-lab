# Lab Setup Guide

A complete step-by-step guide to reproducing this SIEM lab 
environment from scratch.

---

## Requirements

| Component | Minimum Spec |
|-----------|-------------|
| Host machine RAM | 16GB recommended (12GB minimum) |
| Host machine storage | 100GB free |
| Virtualization software | VMware Fusion / Workstation or VirtualBox |
| Network type | Host-only / Internal (isolated) |

---

## VM configuration

### SIEM server (siem-server)
- OS: Ubuntu Server 22.04 LTS
- RAM: 6–8GB
- Storage: 50GB
- IP: 172.16.109.129
- Role: Runs Elasticsearch, Kibana, Logstash

### Endpoint (endpoint01)
- OS: Ubuntu Desktop 24.04 LTS
- RAM: 2–4GB
- Storage: 30GB
- IP: 172.16.109.130
- Role: Generates logs, runs Filebeat

### Attack machine
- OS: Kali Linux 2025.3
- RAM: 2–4GB
- Role: Runs nmap, Hydra, SSH attack simulations

---

## Phase 1 — Install Elastic Stack on the SIEM server

```bash
# Add Elastic GPG key
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch \
  | sudo gpg --dearmor -o \
  /usr/share/keyrings/elasticsearch-keyring.gpg

# Add Elastic apt repository
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] \
  https://artifacts.elastic.co/packages/8.x/apt stable main" \
  | sudo tee /etc/apt/sources.list.d/elastic-8.x.list

# Install all three services
sudo apt update && sudo apt install elasticsearch kibana logstash -y

# Enable and start
sudo systemctl enable elasticsearch kibana logstash
sudo systemctl start elasticsearch kibana logstash

# Verify all three are running
sudo systemctl status elasticsearch kibana logstash | grep Active
```

### Required kibana.yml changes
Open /etc/kibana/kibana.yml and set:

server.host: "0.0.0.0"
xpack.encryptedSavedObjects.encryptionKey: "your-32-char-key-here"
xpack.reporting.encryptionKey: "your-32-char-key-here"
xpack.security.encryptionKey: "your-32-char-key-here"

Generate a valid 32-character key with:
```bash
openssl rand -hex 16
```
---

## Phase 2 — Install Filebeat on the endpoint

```bash
sudo apt install filebeat -y
sudo nano /etc/filebeat/filebeat.yml
```

Set the following in filebeat.yml:
```yaml
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/auth.log
      - /var/log/syslog

output.elasticsearch:
  hosts: ["https://172.16.109.129:9200"]
  username: "elastic"
  password: "your-elastic-password"
  ssl.verification_mode: none
```

```bash
sudo systemctl enable filebeat
sudo systemctl start filebeat
sudo filebeat test output
```

---

## Phase 3 — Verify log ingestion

On the SIEM server, confirm documents are arriving:
```bash
curl -k -u elastic:<password> \
  https://172.16.109.129:9200/filebeat-*/_count
```

A count greater than 0 confirms the pipeline is working.

In Kibana, go to Discover, set index to filebeat-* 
and search: message: "Failed"

---

## Phase 4 — Import detection rules

1. Go to Security → Detection rules (SIEM) → Import rules
2. Import each .ndjson file from the /rules folder
3. Enable all four rules
4. Set each rule to run every 1 minute

---

## Common issues and fixes

**Kibana won't start — encryption key error**
The xpack encryption keys must be exactly 32 characters.
Generate one with: openssl rand -hex 16

**Filebeat 401 Unauthorized**
The elastic user password in filebeat.yml is wrong.
Reset it on the SIEM server:
```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password \
  -u elastic
```

**No logs in Kibana Discover**
Check Filebeat is running and can reach Elasticsearch:
```bash
sudo systemctl status filebeat
sudo filebeat test output
```
