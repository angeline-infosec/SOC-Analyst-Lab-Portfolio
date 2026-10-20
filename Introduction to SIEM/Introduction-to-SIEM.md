# [Introduction to SIEM](https://tryhackme.com/room/introtosiem) - TryHackme

Platform: TryHackMe | Difficulty: Easy

### Lab Overview

This room covers the fundamentals of Security Information and Event Management (SIEM), the core platform a SOC analyst works out of day to day. I went through how logs get generated across a network, why no single log source tells the full story on its own, and how a SIEM pulls everything together into something an analyst can actually act on. The room closed with a short simulated incident inside a SIEM dashboard, identifying a malicious process, the user and host behind it, and the correct response action.

### What I Learned

What counts as a log Before anything else, it's worth nailing down the basic unit all of this is built on. A log is a record of an event that occurred within a system or network, a timestamped entry noting that something happened: a file was opened, a login was attempted, a process ran. On its own a single log is just a fact. The value comes later, once logs from different places get lined up against each other.

## Log Sources 

Every device on a network produces logs constantly, and they fall into two broad categories:

| Category | Examples | Log Types |
| --- | --- | --- |
| Host-Centric | Windows, Linux, servers | File access, authentication attempts, process execution, registry changes, PowerShell activity |
| Network-Centric | Firewalls, IDS/IPS, routers | SSH connections, FTP transfers, web traffic, VPN sessions, file sharing activity |

The key thing I took from this: individual logs are rarely enough on their own to call something malicious. It's the correlation across multiple sources that turns a pile of individually boring events into a visible attack pattern.

### Challenges With Isolated Log Analysis: 

Before SIEM existed, analysts were stuck with a handful of real problems:

- No centralization. Logs live on whatever machine generated them, so investigating anything means SSHing or RDPing into each device one at a time.
- Scale. Devices throw off hundreds of events per second. Manually reviewing that volume isn't realistic.
- Limited context. A single file-access event looks completely normal in isolation. The same event, correlated with lateral movement from another host, is suddenly a strong indicator of compromise.
- Format inconsistency. Windows Event Logs, Linux syslogs, and web server logs all use different structures, so an analyst has to know multiple formats just to read them.

### What SIEM Actually Is: 

**SIEM is a security tool that collects and analyzes logs and data from across an organization's systems in one place. It centralizes logs from different sources, detects suspicious activity, and generates alerts. 
Examples: Splunk, Wazuh, Microsoft Sentinel, IBM QRadar, Log360.**


### What it actually solves:

| Challenge | SIEM Capability |
| --- | --- |
| Scattered logs | Centralized collection via agents, syslog, APIs, or port forwarding |
| Inconsistent formats | Parsing and normalization into one consistent field-value structure |
| Limited context | Correlation across sources to surface attack patterns |
| Manual analysis | Detection rules that fire real-time alerts automatically |
| Reporting | Dashboards summarizing alerts, failed logins, ingestion counts, top domains, etc. |

<img width="733" height="729" alt="image" src="https://github.com/user-attachments/assets/bf45fe2e-1caf-4f1f-a1c3-be3a7181a668" />

<img width="850" height="561" alt="image" src="https://github.com/user-attachments/assets/5df3ea3e-3e96-48b2-84be-889118705923" />

### Log Sources and Ingestion

Windows logs everything through Event Viewer, tagging each event type with a unique Event ID, which is what makes it possible to build detection rules against specific activity later. Linux keeps its logs in a handful of predictable locations:

| Path | Contents |
| --- | --- |
| /var/log/httpd (or /var/log/apache) | HTTP requests, responses, errors |
| /var/log/cron | Cron job execution events |
| /var/log/auth.log or /var/log/secure | Authentication events |
| /var/log/kern | Kernel-level events |

Sample cron log entry, just to see the actual shape of it:

```
May 28 13:04:20 ebr crond[2843]: /usr/sbin/crond 4.4 dillon's cron daemon, started with loglevel notice
```

And an Apache access log entry:

```
192.168.21.200 - - [21/March/2022:10:17:10 -0300] "GET /cgi-bin/try/ HTTP/1.0" 200 3395 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/98.0.4758.102 Safari/537.36"
```

A SIEM pulls logs like these in through a few standard methods:

- Agent/Forwarder: a lightweight agent installed on the endpoint itself (Splunk's version is called a Universal Forwarder) that ships logs to the SIEM.
- Syslog: a widely supported protocol for real-time log forwarding from servers, applications, and network devices.
- Manual Upload: useful for offline analysis, supported by Splunk, ELK, and similar platforms.
- Port Forwarding: endpoints send logs directly to a port the SIEM is listening on.

<img width="1397" height="436" alt="image" src="https://github.com/user-attachments/assets/8b8c6441-0caf-4902-9ec5-3d7add64d0f8" />

Reference: Common Windows Event IDs This room only covered a couple of Event IDs directly, but since detection rules live and die by exact Event IDs, I put together a short reference of the ones that came up plus a few other commonly cited ones worth knowing for SOC work:

| Event ID | Meaning |
| --- | --- |
| 4688 | New process created (what a rule like the whoami detection hooks into) |
| 104 | Event log cleared, a classic post-exploitation anti-forensics move |
| 4624 | Successful logon |
| 4625 | Failed logon attempt |
| 4720 | A user account was created |
| 4732 | A member was added to a security-enabled local group |
| 21 | Successful RDP session logon |
| 23 | RDP session disconnected |
| 24 / 25 | RDP session disconnected / reconnected |
| 39 / 40 | RDP session logon/session connection |
| 1149 | Successful RDP network connection (TerminalServices-RemoteConnectionManager) |

### Detection Rules and the Alerting Process

A detection rule is just a logical expression evaluated against normalized log fields. If the condition is met, it fires an alert. Normalization matters here specifically because rules match against exact field-value pairs. If logs from different sources aren't broken down into the same consistent fields, a rule built for one source won't reliably catch the same activity from another.

Two examples straight from the room:

```
IF LogSource = WinEventLog AND EventID = 104
THEN Alert: "Event Log Cleared"
```

Event ID 104 fires when someone clears a Windows event log, a common way attackers try to cover their tracks after the fact.

```
IF LogSource = WinEventLog AND EventID = 4688 AND NewProcessName CONTAINS "whoami"
THEN Alert: "WHOAMI Command Execution Detected"
```

Event ID 4688 fires on process creation. Attackers commonly run `whoami` right after privilege escalation just to confirm what access level they've landed.

A few more examples worth having in mind, since detection rules in practice cover a much wider range than just these two:

- If a user gets 5 failed login attempts within 10 seconds, alert `Multiple Failed Login Attempts`.
- If a login succeeds immediately after a string of failed attempts, alert `Successful Login After Multiple Failed Attempts`, since that pattern often means a brute-force attempt that eventually landed.
- If a USB device is plugged into an endpoint where USB storage is restricted by policy, alert on it directly, since the activity itself is the violation regardless of what's on the device.
- If outbound traffic from a host exceeds a set threshold (the room used 25MB as an example), alert for potential data exfiltration. The actual threshold is really a company-policy decision, not a fixed number.

Once an alert fires, the actual work starts. Analysts spend most of their time in the dashboard, since it summarizes the state of the network without needing to dig through raw logs first. From there, the general flow is:

1. Pull up the events/flows tied to the alert and confirm which rule condition was actually met.
2. Decide if it's a true positive or false positive based on context, not just the rule firing.
3. If it's a false positive, that's usually a signal the rule itself needs tuning, not just something to dismiss and move on from.
4. If it's a true positive, dig further: who's the user, what's the host, what's the actual scope.
5. Depending on severity, options from there include contacting the asset owner to ask about the activity, isolating an infected host, or blocking a suspicious IP outright.

The key judgment call throughout all of this is context. The exact same binary or behavior can be completely normal in one setting and a confirmed incident in another, which is exactly what the lab scenario below demonstrates.

### RDP Connections 

Since RDP shows up constantly in both legitimate remote work and as an attacker's lateral movement method, it's worth knowing specifically where it gets logged. RDP session activity lands in Event Viewer under Applications and Services Logs > Microsoft > Windows > TerminalServices-LocalSessionManager > Operational, with a few Event IDs worth recognizing:

- 21: successful RDP logon
- 23: session disconnected
- 24 / 25: session disconnected/reconnected
- 39 / 40: session logon/session connection
- 1149: successful RDP connection, logged under TerminalServices-RemoteConnectionManager

Separately, Windows keeps a history of past RDP connections in the Registry at `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Terminal Server Client`. The `Default` key holds the 10 most recently used connections, while the `Servers` key holds the full history of every remote desktop connection that profile has ever initiated.

RDP is also completely normal for remote work. Employees working from home commonly use it to access their office desktop directly, seeing their actual work screen and using the office machine's resources rather than their own. Because exposing raw RDP to the internet is a serious security risk, organizations rarely allow it unprotected. The usual layers on top of it:

| Security Layer | What It Does |
| --- | --- |
| VPN | Employee connects to the company VPN first, creating an encrypted tunnel before the RDP session ever starts |
| RDP Gateway | A server that filters and authenticates RDP requests before they reach internal desktops |
| MFA | A second verification step required before the RDP session is granted |

The practical takeaway for an analyst: an RDP connection alone isn't suspicious. An RDP connection from an unfamiliar IP, outside normal hours, or bypassing the usual VPN/gateway path is what actually warrants a closer look.

### MITRE ATT&CK Mapping in a SIEM

Mapping alerts to MITRE ATT&CK means tying correlation rules and alerts to the specific tactics, techniques, and procedures defined in the framework, rather than leaving an alert as just a raw description of what happened.

Why this is worth doing rather than skipping:

- Context. An alert that just says "PowerShell Executed" doesn't tell you much on its own. Mapped to T1059.001 (Command and Scripting Interpreter: PowerShell) under the Execution tactic, it immediately tells an analyst what stage of an attack they might be looking at.
- Coverage gaps. Plotting alerts against the full ATT&CK matrix shows where detection is strong (often Initial Access) and where it's thin (often Exfiltration), which is useful for prioritizing what rules to build next.
- Prioritization. Alerts mapped to later-stage tactics like Lateral Movement or Impact generally deserve faster response than early-stage reconnaissance alerts, since they represent a more active, further-along compromise.

The general flow looks like:

```
[ Raw Event Log ]
       |
       v
[ SIEM Correlation Rule ] --> Detects: "Unexpected addition of user to Admin group"
       |
       v
[ MITRE TTP Mapping Added ] --> Tactic: Persistence
                                 Technique: Account Manipulation (T1098)
```

### Common tactics and how a SIEM rule might catch them

| Tactic | What It Means | Example Detection |
| --- | --- | --- |
| Initial Access | How the attacker got in | Successful RDP connection from an unusual foreign IP |
| Execution | Running malicious code | Unauthorized PowerShell or command-line script execution |
| Persistence | Maintaining a foothold | New local admin account created, or a registry run key modified |
| Credential Access | Stealing credentials | LSASS memory dump detected (e.g. Mimikatz-style activity) |
| Lateral Movement | Moving through the network | RDP connection established from a regular workstation straight to a Domain Controller |

## Hands-On Lab: 

### Scenario 

A suspicious activity alert fired on the SIEM dashboard. The task was to investigate it end to end: identify the responsible process and user, work out the alert's classification, and take the right response action.

**Step 1**: Identify the triggered alert. After starting the suspicious activity simulation, an alert showed up on the dashboard. Drilling into the associated events, the process behind it was `cudominer.exe`, a known cryptocurrency miner, malware that uses a compromised endpoint's CPU/GPU to mine cryptocurrency on the attacker's behalf.

**Step 2**: Identify the responsible user and host. Examining the event details further:

| Field | Value |
| --- | --- |
| Process | cudominer.exe |
| User | chris |
| Hostname | HR_02 |

The process ran under the account `chris`, on a workstation (`HR_02`) that belongs to the HR department specifically.

**Step 3**: Examine the detection rule. The rule that fired was keyword-based, matching on the term `miner`. It's configured to catch any process name containing that string, which covers known cryptominer executables regardless of the exact filename variant in use.

Step 4: Determine alert classification

| Field | Rationale |
| --- | --- |
| True Positive | `cudominer.exe` is a known cryptominer. There's no legitimate business reason for it to be running on an HR workstation. The alert correctly caught real malicious activity. |

**Step 5**: Response action: Selected the appropriate response action within the SIEM platform, confirming the incident and kicking off containment.

Flag obtained: `THM{000_SIEM_INTRO}`

### Investigation Summary

| Field | Value |
| --- | --- |
| Alert Type | Cryptominer Detected |
| Process | cudominer.exe |
| Rule Match | Process name contains "miner" |
| User | chris |
| Hostname | HR_02 |
| Classification | True Positive |
| Action Taken | Confirmed & Escalated |

## Key Takeaways

- Log correlation is fundamental. Individual events seldom tell the full story on their own. Building the real picture of an incident means looking across multiple log sources at once.
- Normalization is what makes detection possible at all. Rules depend on consistent field-value pairs, and poorly normalized logs quietly break detection coverage without that being obvious at a glance.
- Not every alert is malicious. Telling a true positive from a false positive is a core SOC skill, and a false positive is a signal to tune the rule, not just something to dismiss and forget.
- SIEM exists to cut down Mean Time to Detect. Automating correlation and alerting is the whole point, versus trying to manually review logs at a scale no human actually can.
- Context changes everything. The same binary, `cudominer.exe`, running on a developer's personal test machine might warrant a completely different response than the same file running on an HR workstation.

## Skills Practiced

- SIEM alert triage and investigation
- Log source identification (host-centric vs. network-centric)
- Detection rule logic and field-value matching
- True positive/false positive classification
- MITRE ATT&CK tactic mapping for alert context

## Tools & Technologies

- Splunk (SIEM platform used in the room)
- Windows Event Viewer / Windows Event IDs
- Linux system logs (/var/log/)
- Apache web server logs
- Syslog protocol

## Conclusion

This room was a solid grounding in why SIEM exists in the first place: logs are everywhere, but scattered, inconsistently formatted, and individually low-context, which is exactly the gap a SIEM is built to close. The hands-on piece reinforced that identifying a malicious process is really just the start, the actual analyst work is in the context (which user, which host, why does that combination matter) and in correctly classifying what you're looking at before deciding how to respond.
