\# Incident Response Runbooks — Quick Reference



This is a condensed index of all incident response runbooks developed for this project. For full detail on each, see the individual files in this folder.



| Scenario | Trigger | Default Detection | Key Escalation Point | Full Runbook |

|---|---|---|---|---|

| Network Reconnaissance | Port scan activity | ❌ Not detected by default | Followed by exploitation attempt | \[recon.md](./recon.md) |

| Brute Force (SSH) | Repeated auth failures | ✅ Detected (rule 5760) | Any successful login after failures | \[bruteforce.md](./bruteforce.md) |

| SQL Injection | Anomalous DB queries / data exposure | ❌ Not detected by default | Confirmed data exfiltration | \[sql-injection.md](./sql-injection.md) |

| File Integrity (FIM) | Tracked file change | ⚠️ Partial (Inventory yes, Alert no for non-critical files) | Change to security-critical file | \[fim.md](./fim.md) |



\## General Escalation Principle

Across all four scenarios tested in this lab, a consistent pattern emerged: \*\*Wazuh's default configuration reliably detects authentication-based attacks (SSH brute force) but has significant blind spots for network-layer and application-layer attacks\*\* (port scanning, SQL injection) and partial gaps even for host-based monitoring (FIM alerting vs. tracking).



\*\*Practical implication for any analyst using this runbook set\*\*: do not assume "no alert" means "no incident." For reconnaissance and SQL injection scenarios in particular, active monitoring and periodic manual review (web server logs, FIM Inventory) remain necessary even with Wazuh deployed, until the documented detection gaps are closed through custom rule development or supplementary tooling (NIDS, WAF).



\## Severity Matrix Reference

See the project's main documentation for the full Severity Classification \& Escalation Matrix (Critical / High / Medium / Low definitions and target response times) applied consistently across all runbooks above.

