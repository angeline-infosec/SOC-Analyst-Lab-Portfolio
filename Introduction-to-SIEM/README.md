# Introduction to SIEM 

Platform:![TryHackMe](https://img.shields.io/badge/TryHackMe-1abc9c?style=flat-square)  | Difficulty: Easy

## Lab Overview

This room introduces the foundational concepts behind Security Information and Event Management (SIEM) systems, the primary tool used by analysts in a Security Operations Center (SOC). The lab covers how logs are generated across a network, why centralised collection is critical, and how SIEM solutions enable threat detection through correlation and alerting.

The room concludes with a hands-on simulated incident investigation using a SIEM dashboard, requiring identification of a malicious process, the user responsible, and the appropriate response action.


## Concepts Covered

### Log Sources

Logs are generated continuously by every device on a network and fall into two primary categories:

| Category        | Examples                    | Log Types                                                                             |
| --------------- | --------------------------- | ------------------------------------------------------------------------------------- |
| Host-Centric    | Windows, Linux, Servers     | File access, authentication, process execution, registry changes, PowerShell activity |
| Network-Centric | Firewalls, IDS/IPS, Routers | SSH connections, FTP transfers, web traffic, VPN sessions, file sharing               |


**Key insight**: Individual logs are rarely sufficient to determine malicious intent. It is only through correlation across multiple log sources that the full picture of an attack becomes visible.


### Challenges With Isolated Log Analysis

Before SIEM, analysts faced several critical limitations:

* No centralisation — Logs lived on individual machines; investigation required SSH/RDP into each device separately.
* Scale — Devices generate hundreds of events per second; manual review is humanly impossible at scale.
* Limited context — A single file-access event appears benign. Correlated with lateral movement from another host, it becomes an indicator of compromise.
* Format inconsistency — Windows Event Logs, Linux syslogs, and web server logs each use different formats, complicating analysis.

**How SIEM Solves These Problems**

| Challenge            | SIEM Capability                                                                   |
| -------------------- | --------------------------------------------------------------------------------- |
| Scattered logs       | Centralised log collection via agents, syslog, APIs, or port-forwarding           |
| Inconsistent formats | Parsing & normalisation into a unified field-value structure                      |
| Limited context      | Log correlation across sources to surface attack patterns                         |
| Manual analysis      | Detection rules that trigger real-time alerts automatically                       |
| Reporting            | Dashboards summarising alerts, failed logins, ingestion counts, top domains, etc. |


### Log Ingestion Methods
SIEM solutions ingest logs through several methods:

* Agent/Forwarder — A lightweight agent installed on endpoints (e.g., Splunk Universal Forwarder) that ships logs to the SIEM.
* Syslog — A widely supported protocol for real-time log forwarding from servers, web applications, and network devices.
* Manual Upload — Useful for offline log analysis (supported by Splunk, ELK, etc.).
* Port Forwarding — Endpoints send logs directly to a port the SIEM is listening on.

### Common Linux Log Locations

| Path                                    | Contents                         |
| --------------------------------------- | -------------------------------- |
| `/var/log/httpd`                        | HTTP requests, responses, errors |
| `/var/log/cron`                         | Cron job execution events        |
| `/var/log/auth.log` / `/var/log/secure` | Authentication events            |
| `/var/log/kern`                         | Kernel-level events              |


### Detection Rules

SIEM detection rules are logical expressions that evaluate normalised log fields and trigger alerts when conditions are met.

Example rules from this room:

                                 IF LogSource = WinEventLog AND EventID = 104
                                 THEN Alert: "Event Log Cleared"

EventID 104 indicates a Windows event log was cleared — a common post-exploitation anti-forensics technique.


                                IF LogSource = WinEventLog AND EventID = 4688 AND NewProcessName CONTAINS "whoami"
                                THEN Alert: "WHOAMI Command Execution Detected"


EventID 4688 is generated on process creation. Adversaries commonly run whoami after privilege escalation to confirm their access level.


**Why normalisation matters here**: Rules match against specific field-value pairs. If logs are not normalised into consistent fields, rules cannot reliably match across different log sources.


## Hands-On Lab — Incident Investigation

Scenario

A suspicious activity alert was triggered on the SIEM dashboard. The task was to investigate the alert, identify the responsible process and user, evaluate the alert type, and take the appropriate response action.


Investigation

**Step 1 — Identify the triggered alert**

After initiating the suspicious activity simulation, an alert appeared on the SIEM dashboard. Examining the associated events, the process that triggered the alert was identified as:

cudominer.exe


This is consistent with a cryptocurrency miner, a common malware category where an attacker deploys mining software on compromised endpoints to generate cryptocurrency using the victim's CPU/GPU resources.


**Step 2 — Identify the responsible user and host**

Drilling into the event details:

| Field    | Value           |
| -------- | --------------- |
| Process  | `cudominer.exe` |
| User     | `chris`         |
| Hostname | `HR_02`         |


The process was executed under the account chris on workstation HR_02 (an HR department machine).


**Step 3 — Examine the detection rule**

The detection rule that fired was keyword-based. The term that matched the rule condition:

miner

The rule was configured to alert on any process name containing the string miner, catching known cryptomining executables regardless of exact filename variations.


**Step 4 — Determine alert classification**

| Classification  | Rationale                                                                                                                                                              |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ✅ True Positive | `cudominer.exe` is a known cryptominer. Its presence on an HR workstation has no legitimate business justification. The alert correctly identified malicious activity. |


**Step 5 — Response Action**

The appropriate response was selected within the SIEM platform, confirming the incident and initiating containment.

Flag obtained: THM{000_SIEM_INTRO}


## Investigation Summary


| Field          | Value                           |
| -------------- | ------------------------------- |
| Alert Type     | `Cryptominer Detected`          |
| Process        | `cudominer.exe`                 |
| Rule Match     | `Process name contains "miner"` |
| User           | `chris`                         |
| Hostname       | `HR_02`                         |
| Classification | `True Positive`                 |
| Action Taken   | `Confirmed & Escalated`         |


## Key Takeaways

1 **Log correlation is fundamental** — Individual events rarely tell the whole story. Analysts must look across multiple log sources to build a complete picture of an incident.

2 **Normalisation enables detection** — Rules depend on consistent field-value pairs. Poorly normalised logs break detection coverage.

3 **Not every alert is malicious** — Distinguishing True Positives from False Positives is a core SOC analyst skill. False positives require rule tuning, not just dismissal.

4 **SIEM reduces MTTD** — By automating correlation and alerting, SIEM significantly reduces Mean Time to Detect (MTTD) compared to manual log review.

5 **Context is everything** — cudominer.exe running on a developer's personal machine (hypothetically) might warrant a different response than the same binary on an HR workstation.


## Tools & Technologies Referenced

* Splunk (SIEM example used in room)
* Windows Event Viewer / Windows Event IDs
* Linux system logs (/var/log/)
* Apache web server logs
* Syslog protocol

Room completed on TryHackMe as part of an ongoing SOC analyst learning path and portfolio development.

