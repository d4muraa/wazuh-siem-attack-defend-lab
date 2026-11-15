\# Runbook: Brute Force Attack (SSH/Authentication)



\## Trigger

Wazuh rule 5760 (`sshd: authentication failed`) firing repeatedly from a single source IP, or any spike in `authentication\_failure` events.



\## Response Steps

1\. \*\*Detect\*\*: Alert fires on repeated failed login attempts to SSH (or other authenticated service).

2\. \*\*Triage\*\* — this is the critical decision point:

&#x20;  - All attempts failed → \*\*Medium\*\* severity, proceed to step 3.

&#x20;  - \*\*Any attempt succeeded\*\* after failures → \*\*Critical\*\* severity, skip to step 5 immediately.

3\. \*\*Containment (failed attempts only)\*\*: Verify active response (if configured) has auto-blocked the source IP. If not configured, manually block via `iptables`:

&#x20;  ```bash

&#x20;  sudo iptables -A INPUT -s <attacker\_ip> -j DROP

&#x20;  ```

4\. \*\*Document\*\*: Record source IP, target username(s), number of attempts, time window.

5\. \*\*Escalation (successful compromise)\*\*:

&#x20;  - Isolate the affected host from the network immediately.

&#x20;  - Force password reset for the compromised account.

&#x20;  - Review logs for any post-login activity (lateral movement, privilege escalation, data access) during the compromised session window.

&#x20;  - Notify system owner / team lead.

6\. \*\*Lesson learned\*\*: Review whether password authentication should be disabled in favor of SSH key-based auth, and whether MFA is feasible.



\## Severity

\*\*Medium\*\* (failed attempts only) escalating to \*\*Critical\*\* (successful authentication following brute force pattern).



\## Related Scenario

See `scenarios/02-bruteforce-ssh.md` — this attack was successfully detected by Wazuh's default ruleset with no custom configuration required.

