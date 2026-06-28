# Investigation Example — SSH Brute Force Alert

A worked SOC investigation of an alert raised by this Wazuh deployment.

## Alert
| Field | Value |
|-------|-------|
| Rule ID | **5710** |
| Level | 5 |
| Description | sshd: Attempt to login using a non-existent user |
| Groups | syslog, sshd, authentication_failed, invalid_login |
| MITRE ATT&CK | **T1110.001** (Brute Force: Password Guessing), T1021.004 |
| Source IP | 45.83.122.10 |
| Target user | `admin` (non-existent) |

Raw event:
```
Jun 28 05:35:00 myhost sshd[2345]: Failed password for invalid user admin from 45.83.122.10 port 4444 ssh2
```
Verified with the Wazuh detection engine — see [`../artifacts/wazuh-logtest.txt`](../artifacts/wazuh-logtest.txt).

## Triage (1–2–3)
1. **Scope it.** How many failures from this IP, over what window, against which users/hosts?
   Repeated 5710s from one source escalate to **rule 5712** ("SSHD brute force"), level 10.
2. **Was anything successful?** Pivot to rule **5715** (successful login) from the same IP — a
   failure→success sequence = likely compromise.
3. **Enrich the source IP** (45.83.122.10): reputation, geo, known-scanner? Use `iocsift`
   (project 10) or the threat-intel brief (project 09).

## Decision & response
- If brute force only (no success): block the source IP at the firewall, confirm the targeted
  account doesn't exist / is locked. Automate this with the SOAR playbook (project 19).
- If a success followed: treat as **incident** → invoke the IR playbook (project 17), reset creds,
  hunt for persistence.

## Hardening
- Disable password SSH (keys only), restrict exposure, enforce fail2ban / lockout, MFA.
- Tune Wazuh to alert on the level-10 brute-force composite and on failure→success sequences.

## Takeaway
The SIEM turned a raw syslog line into a structured, ATT&CK-mapped, actionable alert — the
foundation of detection & response.
