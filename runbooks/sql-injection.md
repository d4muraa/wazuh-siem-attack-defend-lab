\# Runbook: SQL Injection / Web Application Attack



\## Trigger

Anomalous database query patterns, unexpected data exposure in web application responses, or (if configured) Wazuh alerts from Apache log monitoring.



\## Known Limitation (from Scenario 03)

Default Wazuh configuration does NOT monitor web server logs (Apache access/error logs) out of the box. SQL Injection attacks — even successful ones with confirmed data exfiltration — generate \*\*zero alerts\*\* unless `<localfile>` monitoring for Apache logs is explicitly configured, and relevant rules are tuned to match injection patterns.



\## Response Steps

1\. \*\*Detect\*\*: Since default detection does not exist, this typically surfaces via manual code review, external report, or WAF logs if present — not via Wazuh alerting in the current configuration.

2\. \*\*Triage\*\*: Determine scope of data exposure — what tables/records were accessible via the injection point?

3\. \*\*Containment\*\*:

&#x20;  - Take the vulnerable endpoint offline or apply an emergency input validation patch if in production.

&#x20;  - Review web server access logs for the full extent of the attacker's queries (if available).

4\. \*\*Document\*\*: Record the injection point (URL/parameter), payload used, and data exposed.

5\. \*\*Escalation\*\*: Treat any confirmed data exfiltration as a data breach — notify relevant stakeholders per organizational policy.

6\. \*\*Remediation\*\*:

&#x20;  - Patch the vulnerable code (parameterized queries / prepared statements).

&#x20;  - Enable Apache log monitoring in Wazuh (`<localfile>` with `log\_format: apache`).

&#x20;  - Deploy a WAF (e.g. ModSecurity) as a defense-in-depth measure.



\## Severity

\*\*High\*\* by default (active exploitation with data access), escalating to \*\*Critical\*\* if sensitive data (credentials, PII) was exposed.



\## Related Scenario

See `scenarios/03-sql-injection.md` — exploitation was successful via both manual testing and sqlmap, with zero detection under default Wazuh configuration.

