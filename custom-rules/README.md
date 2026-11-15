# Custom Rules

This folder documents the custom Wazuh rule development attempted during this project.

- `local_rules.xml` — two iterations of a custom rule to detect network reconnaissance (Nmap scanning) via `auditd` integration. Neither iteration achieved full detection; the rule is preserved (commented) along with root cause analysis for transparency and future iteration.

Why this is here despite being unresolved: a real SIEM tuning process involves failed iterations before success (or an honest conclusion that a different approach is needed). This documents that process rather than only showing a polished final state.

Full technical narrative: see [`../scenarios/01-recon-nmap.md`](../scenarios/01-recon-nmap.md).
