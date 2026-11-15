# Custom Rules

This folder documents the custom Wazuh rule development attempted during this project.

- `local_rules.xml` — two iterations of a custom rule to detect network reconnaissance (Nmap scanning) via `auditd` integration. Neither iteration achieved full detection; the rule is preserved (commented) along with root cause analysis for transparency and future iteration.

Why this is here despite being unresolved: a real SIEM tuning process involves failed iterations before success (or an honest conclusion that a different approach is needed). This documents that process rather than only showing a polished final state.

Full technical narrative: see [`../scenarios/01-recon-nmap.md`](../scenarios/01-recon-nmap.md).

## Update: Active Response (Successful)
In addition to the unresolved reconnaissance detection attempt above, this project also implements and verifies **Active Response** — automated IP blocking triggered by rule 5760 (SSH brute force). Unlike the reconnaissance rule, this configuration was fully successful and verified end-to-end (detection → automated block → confirmed connection failure from attacker). See the bottom of `local_rules.xml` for the configuration reference, and `scenarios/02-bruteforce-ssh.md` for full evidence.

This folder now reflects both outcomes of real-world rule tuning: an unresolved attempt (reconnaissance) and a verified success (active response) — representing the actual range of results in iterative SIEM tuning work.
