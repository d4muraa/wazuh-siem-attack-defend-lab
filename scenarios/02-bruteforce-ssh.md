# Scenario 02: Brute Force Attack - SSH

## Objective

Simulate a brute force attack against SSH authentication to test Wazuh's default detection and alerting capability for credential access attempts.

## MITRE ATT&CK Mapping

- **Technique**: T1110.001 - Brute Force: Password Guessing
- **Secondary Technique**: T1021.004 - Remote Services: SSH
- **Tactic**: Credential Access, Lateral Movement

## Environment

- **Attacker**: Kali Linux (192.168.1.11)
- **Target**: DVWA-target (192.168.1.10), agent ID 002, SSH service on port 22
- **Monitoring**: Wazuh Server (wazuh-server)

## Tools & Attack Commands

Two tools were used to validate detection consistency across different brute force utilities:

```bash
# Hydra
hydra -l target -P ~/small-wordlist.txt ssh://192.168.1.10

# Ncrack
ncrack -user target -P ~/small-wordlist.txt ssh://192.168.1.10
```

## Execution Timeline

| Time (WIB) | Event |
|---|---|
| ~15:40 | First wave of login attempts recorded |
| ~16:25–16:30 | Secondary spike in alert volume (second tool run) |
| 16:34:50 | Failed password attempt logged (source port 50442) |

## Detection Result

- **Alert triggered**: Yes
- **Rule ID**: 5760 — `sshd: authentication failed`
- **Alert level**: 5
- **Total alerts (last ~1 hour window)**: 8
- **Authentication failure count**: 12
- **Authentication success count**: 1 *(indicates one credential in the wordlist was valid — successful compromise simulation)*
- **Decoder**: sshd
- **Source IP**: 192.168.1.11 (Kali)
- **Destination user**: target

### Sample Log Entry (Raw)

```
<!-- TODO: paste raw Wazuh alert JSON / sshd log line here -->
```

### Compliance Mapping (from Wazuh rule metadata)

- **GDPR**: IV_35.7.d, IV_32.2
- **HIPAA**: 164.312.b
- **NIST 800-53**: AU.14, AC.7
- **PCI DSS**: 10.2.4, 10.2.5
- **TSC**: CC6.1, CC6.8, CC7.2, CC7.3

## Screenshot Evidence

![Ncrack attack execution](../screenshots/02-ncrack-command.png)
![Wazuh alert detail - rule 5760](../screenshots/02-wazuh-alert-detail.png)
![Wazuh dashboard - authentication failure spike](../screenshots/02-wazuh-dashboard.png)

## Analysis

Wazuh's default ruleset successfully detected the brute force attempt without any custom rule configuration. Rule 5760 (`sshd: authentication failed`) triggered reliably across both attack tools (Hydra and Ncrack), confirming the detection logic is tool-agnostic — it monitors the authentication log output itself (via the `sshd` decoder), not the attacking tool's signature.

Notably, **one authentication success** was recorded alongside 12 failures, simulating a realistic brute force outcome where an attacker eventually guesses a valid credential after multiple failed attempts. This is the most critical finding of this scenario — the alert volume alone would correctly flag this session as suspicious, but **the single success event is what should drive escalation to Critical severity**, not just the failure count.

Wazuh also auto-mapped this activity to MITRE ATT&CK techniques T1110.001 and T1021.004 without manual tagging, confirming the built-in MITRE module is functioning correctly for credential access scenarios.

## Severity Classification

Based on project severity matrix: **Critical**

- Justification: a successful authentication followed a string of failed attempts from the same source IP — this matches the Critical criteria defined in the project runbook (successful compromise after brute force), not just Medium (failed attempts only).

## Recommendation

1. **Enable Active Response** to automatically block the source IP via `iptables` after a defined threshold of failed attempts (e.g., 5 failures in 60 seconds) — this would have prevented the eventual successful login in this test.
2. **Enforce SSH key-based authentication** and disable password authentication (`PasswordAuthentication no` in `sshd_config`) for production-equivalent environments.
3. **Consider rate-limiting** via `fail2ban` as a complementary layer alongside Wazuh detection.
4. **Alert escalation rule**: configure a correlation rule that specifically triggers a higher-severity alert when an `authentication_success` event follows multiple `authentication_failed` events from the same source IP within a short time window — this pattern wasn't distinctly flagged beyond the standard success/failure counts.

## Active Response

*Not yet configured at the time of this test — flagged as a follow-up item (see Recommendation #1). To be implemented and re-tested in a future iteration.*
