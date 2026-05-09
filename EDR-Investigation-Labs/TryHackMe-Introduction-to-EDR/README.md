
# [Introduction to EDR](https://tryhackme.com/room/introductiontoedrs) – SOC Investigation Lab
Platform:![TryHackMe](https://img.shields.io/badge/TryHackMe-1abc9c?style=flat-square)  | Difficulty: Beginner–Intermediate

## Lab Overview

This lab introduced me to Endpoint Detection and Response (EDR), a security technology that goes beyond traditional antivirus by monitoring how programs behave rather than just scanning for known malware signatures.

I worked through a simulated SOC analyst workflow by triaging alerts from an EDR dashboard, tracing attack chains through process trees, and making decisions about which detections were malicious and which were false positives.

---

# What I Learned

## Why EDR matters over traditional antivirus:

Traditional antivirus software relies on signature databases, meaning it can only catch threats it already knows. EDR instead records everything happening on an endpoint (process launches, file changes, network connections, registry edits) and flags unusual behavior. This makes it far more effective against modern attacks like fileless malware or living-off-the-land techniques.

## Key detection methods EDR uses:

- Behavioral detection — flags activity that looks suspicious regardless of the file involved
- IOC matching — checks known malicious hashes, IPs, or domains
- Anomaly detection — identifies deviations from normal baseline activity
- MITRE ATT&CK mapping — tags detections to known attacker tactics and techniques

---

## Investigation Scenarios

### Detection 1 — DESKTOP-HR01 | ⚠️ Malicious

Observed Process Chain:

```text
WINWORD.EXE
↓
CMD.EXE
↓
cURL.EXE
↓
install.exe
```

![DESKTOP-HR01 Detection](screenshots/DESKTOP-HR01.png)


What happened:

A user on DESKTOP-HR01 opened a Word document (invoice.docm) that contained a malicious macro. When opened, the macro silently launched CMD.EXE, which then used cURL.EXE to download a payload from a remote server and save it as C:\Users\Public\install.exe.

Why this is suspicious:

Microsoft Word should never need to open a command prompt under normal use. When WINWORD.EXE spawns CMD.EXE, it is a strong indicator that a macro has executed malicious code (this is a well-documented technique attackers use in phishing campaigns). The C:\Users\Public\ path is also a common drop location because it is writable by all users.

The EDR quarantined the payload before it could execute, preventing further damage.

### MITRE ATT&CK Mapping:

| Tactic         | Technique                            |
| -------------- | ------------------------------------ |
| Initial Access | T1566.001 - Spearphishing Attachment |

--- 

### Detection 2 — WIN-ENG-LAPTOP03 | ⚠️ Malicious

Observed Process Chain:

```
syncsvc.exe (unsigned, from Temp directory)
↓
lsass.exe (memory access attempt)
↓
Outbound upload to files-wetransfer.com
```


![WIN-ENG-LAPTOP03 Detection](screenshots/WIN-ENG-LAPTOP03.png)


![WIN-ENG-LAPTOP03 Additional Evidence](screenshots/WIN-ENG-LAPTOP03%20-2.png)


What happened:

An unsigned executable (syncsvc.exe) ran from C:\Users\haris.khan\AppData\Local\Temp\. It attempted to access the memory of lsass.exe, the Windows process responsible for storing user credentials(user authentication, password changes, and security policies). Shortly after, outbound traffic was detected to an external file-sharing site.

Why this is suspicious:

Legitimate software is almost always digitally signed. An unsigned binary running from the Temp folder is unusual and a common characteristic of malware. lsass.exe holds password hashes in memory, and tools like Mimikatz are specifically designed to extract credentials from it. This is a technique called credential dumping. The immediate outbound connection to a file transfer service suggests the attacker was trying to exfiltrate what they stole.

### MITRE ATT&CK Mapping:

| Tactic      | Technique                      |
| ----------- | ------------------------------ |
| Persistence | T1003.002 - Credential Dumping |


---

### Detection 3 — DESKTOP-DEV01 | ✅ False Positive

What triggered the alert:

       C:\Users\daniel.richards\AppData\Roaming\UpdateAgent.exe 

An unsigned binary with outbound connections.

![DESKTOP-DEV01 Detection](screenshots/DESKTOP-DEV01.png)


What I found:

After checking threat intelligence, the binary was identified as a known internal IT utility tool. While the behavior pattern looked suspicious on the surface (unsigned + network traffic), the context explained it completely.

### Why this matters:
Not every alert is an attack. This detection is a good example of why SOC analysts need to validate findings with threat intelligence and internal knowledge before taking action. Acting too quickly on a false positive can disrupt legitimate business operations.

---

## Key Takeaways

1. EDR provides significantly deeper endpoint visibility compared to traditional antivirus solutions.
  
3. Process relationships tell a story. WINWORD.EXE opening a command prompt is not normal. Understanding what processes should talk to each other is fundamental to spotting attacks.

4. Context is everything. The same behavior (unsigned binary + network traffic) was malicious in one case and benign in another. Threat intelligence and business context change the conclusion.

5. EDR enables containment, not just detection. The ability to quarantine files, isolate hosts, and access endpoints remotely means analysts can act on threats before they spread.

6. The MITRE ATT&CK framework is a shared language. Mapping detections to ATT&CK techniques helps teams communicate and compare against known attacker behavior patterns.

---

### Skills Practiced

- SOC alert triage and prioritization
  
- Parent-child process analysis
  
- MITRE ATT&CK technique mapping
  
- Threat intelligence validation and false positive analysis
  
- Endpoint telemetry investigation

---

## Tools & Technologies

- Endpoint Detection & Response (EDR) platform

- MITRE ATT&CK Framework

- Threat Intelligence enrichment

- Process tree analysis

---

## Screenshots

### EDR Dashboard Overview
![EDR Dashboard](screenshots/edr-dashboard.png.png)

Overview of endpoint detections, CrowdScore metrics, and detection trends inside the EDR console.

---

### Endpoint Detection Queue
![Endpoint Detections](screenshots/detection-console.png.png)

Detection queue showing multiple alerts with severity levels, tactics, techniques, and suspicious command-line activity.

---

### Process Tree Investigation
![Process Tree Analysis](screenshots/process-tree-analysis.png.png)

Visualization of malicious parent-child process relationships used during investigation and attack chain reconstruction.

---

### Falcon Real Time Response (RTR)
![Falcon RTR Console](screenshots/real-time-response.png.png)

Example of remote endpoint response capabilities available through the EDR platform for investigation and containment.

---

## Conclusion

This lab provided practical exposure to EDR investigations and modern SOC analyst workflows. The investigation demonstrated how endpoint telemetry, behavioral detections, and process analysis enable analysts to identify and investigate malicious activity effectively.

The room also reinforced the importance of contextual analysis, threat intelligence validation, and attack chain visibility during real-world incident investigations.
