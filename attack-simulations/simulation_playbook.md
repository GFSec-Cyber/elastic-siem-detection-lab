# Attack Simulation Playbook

All simulations were conducted inside an isolated host-only 
VM network. The attack machine (Kali Linux) targeted the 
Ubuntu endpoint (172.16.109.130) only. No external systems 
were involved.

---

## Simulation 1 — Network reconnaissance
**MITRE Technique:** T1046 — Network Service Discovery  
**Tactic:** Discovery  
**Tool:** nmap  
**Detection rule triggered:** Potential Port Scan — Invalid SSH User

```bash
nmap -sV -p 22,80,443,3389 172.16.109.130
```

**What this simulates:**  
An attacker identifying open ports and running services on a 
target before deciding how to proceed. This is typically one 
of the first things an attacker does after gaining initial 
access to a network.

**What the SIEM caught:**  
Connection attempts from an unknown source IP using invalid 
usernames, consistent with automated scanning behavior.

---

## Simulation 2 — SSH brute force attack
**MITRE Technique:** T1110.001 — Brute Force: Password Guessing  
**Tactic:** Credential Access  
**Tool:** Hydra  
**Detection rule triggered:** SSH Brute Force Attempt

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt \
  ssh://172.16.109.130 -t 4 -V
```

**What this simulates:**  
An attacker using an automated tool to guess the root 
password by cycling through a wordlist. The rockyou.txt 
wordlist contains over 14 million common passwords leaked 
from real data breaches.

**What the SIEM caught:**  
Repeated failed SSH authentication attempts from the same 
source IP targeting the root account, triggering the 
threshold rule at 5+ failures.

---

## Simulation 3 — Root login attempts
**MITRE Technique:** T1078.001 — Valid Accounts: Default Accounts  
**Tactic:** Privilege Escalation  
**Tool:** SSH  
**Detection rule triggered:** Root Login Attempt via SSH

```bash
for i in {1..10}; do ssh root@172.16.109.130; done
```

**What this simulates:**  
An attacker attempting to authenticate directly as root 
over SSH. Direct root SSH access should be disabled in 
any hardened environment — any attempt is immediately 
suspicious regardless of whether it succeeds.

**What the SIEM caught:**  
Failed SSH login attempts specifically targeting the root 
account, flagged as High severity with a risk score of 73.

---

## Simulation 4 — Backdoor account creation
**MITRE Technique:** T1136.001 — Create Account: Local Account  
**Tactic:** Persistence  
**Tool:** useradd  
**Detection rule triggered:** New Local User Account Created

```bash
sudo useradd backdooruser
```

**What this simulates:**  
An attacker who has already gained access creating a new 
local user account to maintain persistence. Even if their 
initial access method is discovered and patched, the 
backdoor account lets them return.

**What the SIEM caught:**  
A useradd event in auth.log fired the persistence detection 
rule immediately, generating a Medium severity alert.

---

## Results summary

| Simulation | Tool | Alerts Generated | Rule Triggered |
|------------|------|-----------------|----------------|
| Network recon | nmap | 41 | Potential Port Scan |
| SSH brute force | Hydra | — | SSH Brute Force Attempt |
| Root login attempts | SSH | 1 | Root Login Attempt via SSH |
| Backdoor account | useradd | 1 | New Local User Account Created |

**Total alerts: 43**
