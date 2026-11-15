# Runbook: Network Reconnaissance (Port Scanning)

## Trigger

Suspected or confirmed network scanning activity (e.g. Nmap) detected via network monitoring, firewall logs, or manual observation. Note: default Wazuh configuration does NOT alert on this activity — see Finding below.

## Known Limitation (from Scenario 01)

Wazuh's default ruleset does not detect TCP port scanning out of the box. An attempt to close this gap via custom `auditd` integration was partially successful at the data-collection level but did not produce reliable alerts due to noise from local socket traffic. Treat any scan activity as **invisible to Wazuh by default** unless a dedicated NIDS (Suricata/Zeek) or tuned custom rule is in place.

## Response Steps

1. **Detect**: If using supplementary tooling (firewall logs, NIDS), identify source IP, scanned ports, and time window.
2. **Triage**: Is the source IP internal/known (vulnerability scanner, monitoring tool) or external/unknown?
   - Known source → log and close, no further action.
   - Unknown source → proceed to step 3.
3. **Monitor**: Do not block immediately — reconnaissance alone is not an active compromise. Increase monitoring on the scanned host for follow-up exploitation attempts.
4. **Document**: Record source IP, scan pattern (ports/services targeted), and timestamp.
5. **Escalate** only if scanning is followed by exploitation attempts (brute force, injection, etc.) from the same source.

## Severity

**Low** by default. Escalate to Medium if repeated/sustained scanning from the same source over time.

## Related Scenario

See `scenarios/01-recon-nmap.md` for full technical detail and the custom detection attempt.
