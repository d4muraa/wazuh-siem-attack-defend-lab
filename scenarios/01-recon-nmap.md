# Scenario 01: Reconnaissance - Nmap Scan

## Objective

Perform network service discovery against the target to identify open ports and running services, and evaluate whether Wazuh's default ruleset detects basic reconnaissance activity.

## MITRE ATT&CK Mapping

- **Technique**: T1046 - Network Service Discovery

- **Tactic**: Discovery

## Environment

- **Attacker**: Kali Linux (192.168.1.11)

- **Target**: DVWA-target (192.168.1.10)

- **Monitoring**: Wazuh Server (wazuh-server)

## Attack Command

nmap -sV -A 192.168.1.10
## Execution Timeline

| Time | Event |

|---|---|

| 2026-11-05 10:13 (Kali local time) | Scan executed from Kali |

| ~32.64 seconds | Scan completed — 1 host scanned |

## Scan Result Summary

- Port 80/tcp open — Apache httpd 2.4.25 (Debian)

- OS fingerprint attempted (no exact match found)

- HTTP headers and cookie flags enumerated (DVWA login redirect detected)

## Detection Result

- **Alert triggered**: No

- **Alerts reviewed**: Dashboard checked ~2 hours after scan execution; no alerts correlating to network scanning activity from the attacker IP were found in that window

- **Note**: Timezone discrepancy between Kali (scan timestamp) and dashboard display was identified during this test — relevant for accurately correlating future scenario timestamps

## Screenshot Evidence

![Nmap scan output from Kali](../screenshots/01-nmap-scan.png)

![Wazuh dashboard - no related alerts found](../screenshots/01-wazuh-dashboard-empty.png)

## Analysis

Wazuh's default ruleset does not include detection logic for basic TCP port scanning. This is expected behavior — Wazuh is primarily a host-based intrusion detection system (HIDS) that relies on log sources (auth logs, file integrity, syscall monitoring) rather than raw network traffic inspection. A tool like Nmap performing a service scan does not generate log entries on the target host by default, so there is nothing for Wazuh's log-based rules to match against.

This finding highlights an important architectural gap: **Wazuh alone is not sufficient for network-level reconnaissance detection**. Closing this gap would require either a complementary NIDS/NDR tool (e.g., Suricata or Zeek) feeding data into Wazuh, or a custom rule built on connection-level logs (e.g., via auditd syscall monitoring or firewall/iptables logging) that Wazuh could then correlate.

## Severity Classification

Based on project severity matrix: **Low**

- Justification: reconnaissance activity alone (pre-exploitation, no indication of compromise) — matches the Low criteria in the project runbook, warranting monitoring/review rather than immediate response.

## Recommendation

1. **Do not rely on Wazuh alone for network scan detection** — this is a known limitation of host-based SIEM/XDR tools, not a misconfiguration.

2. **Future improvement**: integrate a lightweight NIDS (Suricata) on the target network segment to detect scanning behavior at the network layer, forwarding alerts into Wazuh for unified visibility.

3. **Alternative**: build a custom Wazuh rule based on auditd connection logs or SSH/service connection attempt logs if network-layer tooling is out of scope for this lab.

## Active Response

*Not applicable for this scenario — no detection occurred, so no response action was tested.*
