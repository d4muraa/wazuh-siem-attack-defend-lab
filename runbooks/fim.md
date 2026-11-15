\# Runbook: File Integrity Monitoring (FIM) Alert



\## Trigger

A tracked file change is visible in the Wazuh FIM Inventory module. Note: not all tracked changes generate a formal alert — see Finding below.



\## Known Limitation (from Scenario 04)

Wazuh's FIM correctly tracks file changes at the data layer (visible in Inventory with full checksum/metadata), but does NOT generate alerts for all monitored file changes — only for a subset matching specific default rules (typically security-critical files like `/etc/passwd`, `/etc/shadow`). A changed file with no alert does not mean no change occurred — check Inventory, not just Events.



\## Response Steps

1\. \*\*Detect\*\*: Alert fires (for critical files) OR manual review of FIM Inventory reveals an unexpected change (for non-critical files, which won't alert automatically).

2\. \*\*Triage\*\*:

&#x20;  - Is this a known/scheduled change (patch, config deployment)? → Log and close.

&#x20;  - Is this to a security-critical file (`/etc/passwd`, `/etc/shadow`, SSH keys, etc.)? → Escalate immediately to step 4.

&#x20;  - Is this an unexpected change to a non-critical file? → Proceed to step 3.

3\. \*\*Investigation\*\*: Review the file's checksum history, modification timestamp, and owning user/process (from FIM Inventory details) to determine if the change is benign or suspicious.

4\. \*\*Escalation (critical file or confirmed malicious change)\*\*:

&#x20;  - Isolate the host.

&#x20;  - Compare the file against a known-good baseline or backup.

&#x20;  - Investigate the process/user responsible for the change.

5\. \*\*Document\*\*: Record file path, checksums (before/after if available), timestamp, and responsible user/process.

6\. \*\*Lesson learned\*\*: Review whether `local\_rules.xml` needs broader coverage beyond the default critical-file list, to ensure alerting matches organizational monitoring needs.



\## Severity

\*\*Low-Medium\*\* for non-critical files, \*\*Critical\*\* for security-critical system files (passwd, shadow, SSH config).



\## Related Scenario

See `scenarios/04-fim-detection.md` — change was successfully tracked at the database and Inventory level but did not escalate to a formal alert, highlighting the gap between data collection and alert generation.

