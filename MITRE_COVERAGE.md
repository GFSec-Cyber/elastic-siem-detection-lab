# MITRE ATT&CK Coverage

This document maps each custom detection rule built in this lab 
to its corresponding MITRE ATT&CK tactic and technique.

| Rule Name | Tactic | Technique ID | Technique Name | Severity |
|-----------|--------|-------------|----------------|----------|
| SSH Brute Force Attempt | Credential Access | T1110.001 | Brute Force: Password Guessing | High |
| Potential Port Scan — Invalid SSH User | Discovery | T1046 | Network Service Discovery | Medium |
| Root Login Attempt via SSH | Privilege Escalation | T1078.001 | Valid Accounts: Default Accounts | High |
| New Local User Account Created | Persistence | T1136.001 | Create Account: Local Account | Medium |

---

## Why MITRE ATT&CK mapping matters

MITRE ATT&CK is a globally recognized framework that catalogs 
the tactics and techniques real adversaries use in the wild. 
Mapping detection rules to ATT&CK techniques means every alert 
that fires tells you not just *what* happened, but *why an 
attacker would do it* and where it fits in their overall 
kill chain.

---

## Coverage summary

| Tactic | Techniques Covered |
|--------|-------------------|
| Credential Access | T1110.001 |
| Discovery | T1046 |
| Privilege Escalation | T1078.001 |
| Persistence | T1136.001 |
