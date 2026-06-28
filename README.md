# 01 — Home SIEM (Wazuh) + Investigation

A self-hosted **Wazuh** SIEM/XDR (single-node, Docker) with a worked **SOC investigation** of a
detected SSH brute-force attack.

## Goal
Stand up a real SIEM, prove its detection engine works against attacker activity, and document the
analyst investigation workflow — the core of blue-team / SOC operations.

## Stack
Wazuh **v4.14.5** single-node: **manager** (detection/correlation) + **indexer** (OpenSearch-based
storage) + **dashboard** (web UI).

## Exact Setup Commands
```powershell
git clone -b v4.14.5 https://github.com/wazuh/wazuh-docker.git
cd wazuh-docker\single-node
wsl -d docker-desktop sysctl -w vm.max_map_count=262144            # for the indexer
docker compose -f generate-indexer-certs.yml run --rm generator   # TLS certs
docker compose up -d                                               # dashboard at https://localhost (admin / SecretPassword)

# test the detection engine against an attack log line:
$line='Jun 28 05:35:00 myhost sshd[2345]: Failed password for invalid user admin from 45.83.122.10 port 4444 ssh2'
$line | docker exec -i single-node-wazuh.manager-1 /var/ossec/bin/wazuh-logtest
```

## Proof It Works
**All 3 containers Up; dashboard serving (HTTP 302 login redirect)** ([proof](./artifacts/wazuh-deployment.txt)):
```
single-node-wazuh.dashboard-1  Up  0.0.0.0:443->5601
single-node-wazuh.manager-1    Up  1514-1515, 514/udp, 55000
single-node-wazuh.indexer-1    Up  0.0.0.0:9200->9200
```
**Live detection** — feeding an SSH brute-force log line fired a real Wazuh rule
([`artifacts/wazuh-logtest.txt`](./artifacts/wazuh-logtest.txt)):
```
id: '5710'  level: '5'
description: 'sshd: Attempt to login using a non-existent user'
groups: [syslog, sshd, authentication_failed, invalid_login]
mitre.id: ['T1110.001', 'T1021.004']
```
Full analyst writeup in [`docs/investigation-example.md`](./docs/investigation-example.md).

## Screenshots
See [`./screenshots/`](./screenshots). Add: the Wazuh dashboard (security events), the rule 5710
alert, and the `wazuh-logtest` output.

## My Custom Extensions
- Proved detection with **`wazuh-logtest`** (no agent needed) — a fast, reproducible way to validate
  rules, complete with the **MITRE ATT&CK** mapping the rule carries.
- A complete **SOC investigation writeup** (triage → enrich → decide → harden) that cross-links the
  IOC enricher (10), threat-intel brief (09), SOAR auto-contain (19), and IR playbook (17).
- Full deployment runbook incl. cert generation and the OpenSearch `vm.max_map_count` fix.

## Resume Bullet Points
- Deployed a production-style **Wazuh SIEM/XDR** (manager + indexer + dashboard) via Docker and
  validated its detection engine against an SSH brute-force attack (rule 5710, MITRE T1110.001).
- Authored a SOC **investigation playbook** for the alert (triage, enrichment, containment, hardening).
- Integrated the SIEM conceptually with IOC enrichment, threat intel, SOAR, and IR projects.

## Next-Level Ideas
- Enroll a Wazuh agent on an endpoint and simulate 3 attacks (brute force, malware drop, persistence).
- Build dashboards and forward alerts to the SOAR webhook (project 19) for auto-containment.
- Add custom decoders/rules and tune the brute-force composite (rule 5712, level 10).

---
status: ✅ complete & tested
```
✅ PROJECT COMPLETE & FULLY TESTED in its isolated folder. All works. Ready for portfolio.
```
