# Project Reflection & Interview Preparation

This document captures the strategic reasoning, trade-offs, and lessons behind this project — the "why" behind the "what" documented in `scenarios/` and `runbooks/`.

## 1. Problem Framing

**What problem does this project solve?**

This project investigates a question every security team eventually faces: *"We deployed a SIEM — does it actually detect attacks, or does it just look like it does?"* Rather than assuming Wazuh's default configuration provides complete coverage, this lab deliberately tests that assumption against four distinct attack types.

**Why does this matter in the real world?**

Organizations frequently deploy SIEM tools and treat the deployment itself as "done," without validating actual detection coverage. A SIEM with undetected blind spots creates a false sense of security — arguably more dangerous than having no SIEM at all, because it leads to under-investment in compensating controls. Anyone relying on this deployment (a SOC team, an organization's leadership assuming they're "covered") is directly affected by undiscovered gaps.

**What constraints shaped this project?**

- Single-laptop lab environment (VirtualBox, shared RAM/CPU across 3-4 VMs simultaneously)
- Solo execution — no team to divide installation, attack execution, and documentation (see "Individual vs Group PBL" comparison in main project doc)
- Limited time window (approx. 1-2 weeks of active work, part-time alongside coursework)
- No budget for commercial tooling or cloud infrastructure — entirely free/open-source stack

## 2. Tool Selection Rationale

**Why Wazuh, not Splunk or the ELK stack?**

- **Cost**: Wazuh is fully open-source with no licensing cost, critical for a self-funded student lab. Splunk's free tier has strict data ingestion limits; ELK is also free but requires significantly more manual integration work to get SIEM-equivalent functionality (Wazuh ships with XDR capabilities — FIM, vulnerability detection, active response — built in, which ELK alone does not).
- **Trade-off accepted**: Wazuh's community and documentation, while solid, is smaller than Splunk's enterprise ecosystem — meaning some troubleshooting (as extensively documented in this repo) required first-principles debugging rather than readily available community answers.

**What can this tool do, and what can't it do, for this use case?**

This is the core finding of the project, documented scenario-by-scenario:

- **Can do well**: host-based authentication monitoring (SSH brute force — Scenario 02, detected reliably with zero configuration)
- **Cannot do by default**: network-layer detection (port scanning — Scenario 01), application-layer detection (SQL injection via web logs — Scenario 03)
- **Partial capability**: file integrity tracking works at the data layer but alerting is selective, not exhaustive (Scenario 04)

**What would be the alternative if Wazuh weren't available?**

A combination of the ELK stack (Elasticsearch, Logstash, Kibana) for log aggregation, paired with a separate NIDS (Suricata) for network visibility, and a separate FIM tool (OSSEC standalone, Wazuh's predecessor) — essentially rebuilding Wazuh's XDR feature set from separate components, at the cost of significantly more integration work.

## 3. Architecture & Design Rationale

**Why this architecture?**

The lab uses a flat, single-subnet topology (Bridged network, 192.168.1.0/24) rather than a segmented network, because the goal was to isolate the *detection* variable — removing network segmentation as a confounding factor let every test focus purely on whether Wazuh's agent/manager pipeline caught the attack, not on whether the attack could even reach the target.

**What's the most critical/fragile point in this design?**

The single Wazuh server hosting manager, indexer, and dashboard together (all-in-one deployment) is the most fragile point — any resource contention (as repeatedly observed: indexer/manager startup timeouts, RAM pressure from running 4 VMs simultaneously) risks taking down the entire monitoring pipeline at once. In a production environment, these components would be separated across nodes specifically to avoid this single point of failure.

**What assumptions were made, and what happened when they were wrong?**

- **Assumption**: Metasploitable2 (legacy Ubuntu 8.04, 32-bit) could be fully integrated as a monitored target.
- **Reality**: Modern Wazuh agent builds (4.14.x) no longer ship 32-bit packages. This assumption failure led to a mid-project pivot — Metasploitable2 was kept only for unmonitored network-level attack practice, while a new Debian-based target (DVWA-target) was provisioned specifically for agent compatibility. This pivot is documented transparently rather than hidden, and the topology diagram was updated to reflect the actual final architecture rather than the originally planned one.

## 4. Configuration Decisions

**Why specific parameters were chosen (or not):**

This project largely used **Wazuh's default detection thresholds and rule IDs** (e.g., rule 5760 for SSH auth failure) rather than custom-tuned thresholds, by design — the goal was to measure *default* coverage first, before any tuning. The one exception was the custom rule development for network reconnaissance (Scenario 01), where two rule iterations were attempted:

- Iteration 1 (`<decoded_as>auditd</decoded_as>`) — too broad, matched by a higher-priority built-in rule before reaching the custom logic.
- Iteration 2 (`<if_sid>80700</if_sid>` + `<field name="audit.type">SYSCALL</field>`) — more specific, but the underlying `auditctl` rule itself was too broad (monitoring all `connect`/`accept` syscalls, including local Unix socket traffic), drowning the actual network signal in noise.

**What would happen if a specific config line were removed?**

Removing the `<localfile>` block that points Wazuh to `/var/log/audit/audit.log` would silently disable the entire auditd-based detection attempt — no error would be thrown, Wazuh would simply never see that log source. This is itself a documented lesson: Wazuh fails silently on missing log sources rather than warning the operator, making source configuration verification an essential manual step.

## 5. Testing & Validation

**How was "it works" actually proven, not just assumed?**

Rather than trusting the dashboard UI alone, verification was pushed to the data layer wherever a dashboard result was ambiguous. Example: for Scenario 04 (FIM), when no alert appeared in the Events view, the investigation went directly to the manager's SQLite FIM database (`fim_entry` table) via `sqlite3` queries to confirm whether the underlying data collection had actually occurred — separating "no alert generated" from "no detection at all occurred."

**What was measured, concretely?**

- Scenario 02: 12 authentication failures + 1 authentication success recorded, rule 5760, alert level 5
- Scenario 04: file change tracked with full forensic metadata (MD5/SHA1/SHA256 checksums, exact modification timestamp) at the database layer, confirmed via direct query
- Scenarios 01 & 03: zero alerts generated despite 100% successful exploitation (confirmed via Nmap scan completion and sqlmap's confirmed injection points) — a quantified "detection coverage gap," not just "it didn't work"

**What's missing from the measurements:**

This project did not measure false positive rate (no benign/legitimate traffic was run to compare noise levels) or response time (time from attack execution to alert visibility was observed qualitatively, not precisely timestamped and compared). This is a known gap for future iterations.

## 6. Failures & Limitations

**Where did this project fail or produce wrong results during the work?**

- Multiple failed Wazuh installations due to dpkg state corruption after interrupted installs, requiring full OS reinstallation to resolve.
- Metasploitable2 kernel panics (`MP-BIOS bug: 8254 timer not connected to IO-APIC`) on modern VirtualBox, requiring CPU core reduction and IO-APIC disabling.
- Two failed custom rule iterations for network reconnaissance detection (see Section 4).
- `git push` conflicts from inconsistent local/remote editing — resolved by establishing a single-source-of-truth editing workflow (laptop only, never GitHub web editor).

**What was tried to fix these, and why that approach specifically?**

Each failure was debugged using the lowest-level available evidence rather than guessing at higher-level fixes: direct log inspection (`ossec.log`, `audit.log`) over trusting dashboard state; direct SQLite queries over trusting the UI; `wazuh-logtest` for rule validation before live re-testing. This bottom-up debugging approach is consistent throughout the project and is itself a transferable skill demonstrated by the work, not just a means to an end.

**Current limitations — what's NOT handled yet:**

- No Active Response configured (recommended in Scenario 02 but not implemented) — detections currently generate alerts but take no automated containment action.
- Network-layer and application-layer detection gaps (Scenarios 01, 03) remain open — documented but unresolved.
- No privilege escalation scenario tested (Scenario 05 was scoped but not executed, given time constraints).
- No false positive / noise baseline established.
- Single-node deployment only — no resilience/failover testing.

## 7. Reflection & Next Steps

**If given more time, what would be improved?**

1. Resolve the auditd rule scoping issue (filter by `saddr_fam=inet`, exclude local socket traffic) to close the reconnaissance detection gap properly.
2. Extend Active Response beyond SSH brute force to other scenario types (e.g., automated containment for confirmed FIM alerts on critical files).
3. Enable Apache log monitoring and retest SQL injection detection with that log source in place.
4. Establish a false-positive baseline by running benign traffic patterns before/after each attack scenario.

**What concept still feels unclear despite the project being "complete"?**

1. How to find out the log directory of the currently running program?
2. How do you enter a rule into a program in Wazuh, and what actually does the program run and show the results?
3. There are times when Wazuh displays the results as if it were brute force, while others don't, but when checked into the target VM, it actually goes in, and why is that?

**What new skills transfer to other projects?**

- Reading raw system/application logs directly rather than trusting dashboard abstractions
- Root-cause debugging methodology (bottom-up: data layer → transport layer → presentation layer)
- Git workflow discipline under real failure conditions (merge conflicts, push rejections)
- Writing technical documentation that honestly reports negative/partial results, not just successes

## 8. Storytelling for Interviews

**Explaining this project to a non-technical person (3 sentences):**

"I built a mini security monitoring lab to test whether a popular security tool actually catches hackers trying different attack methods, or if it just creates a false sense of security. I simulated four real attack types against a deliberately vulnerable practice target. Two of the four attacks went completely undetected by default — which is exactly the kind of gap that causes real security breaches when organizations assume their tools are watching everything."

**Explaining this project to a fellow security engineer (3 sentences):**

"I deployed Wazuh as an all-in-one SIEM/XDR and ran four MITRE ATT&CK-mapped attack scenarios (T1046 recon, T1110 brute force, T1190 SQLi, T1565 FIM) against a DVWA target to empirically map default detection coverage. SSH brute force was reliably caught via built-in auth-failure rules, but Nmap recon and SQLi produced zero alerts without additional log sources — I attempted to close the recon gap via auditd integration but hit a signal-to-noise problem with overly broad syscall auditing that I've documented as unresolved future work. The repo includes full custom rule iterations, root cause analysis, and incident response runbooks per scenario."

**A likely "trap" interview question, and the answer:**

*Q: "If Wazuh detected everything perfectly in your lab, would that make you confident it's production-ready?"*

A: "No — and that's actually the opposite of what I'd want to see. A tool that detects everything in a clean lab environment with no tuning usually means the lab is too simple, not that the tool is complete. The fact that my default configuration had real, specific blind spots (network recon, SQLi) is more useful information than a false 'it caught everything' result would have been — it tells me exactly what additional log sources and rules a real deployment would need before going into production."

---

*This document is intended to be read alongside the main project documentation and complements, rather than duplicates, the detailed technical reports in `scenarios/`.*
