# TryHackMe: SOC Fundamentals

**Platform:** ![TryHackMe](https://img.shields.io/badge/TryHackMe-1abc9c?style=flat-square)

## Overview

This room is mostly theory, but I consider it one of the most important rooms for building a base in SOC work. It explains what a Security Operations Center is, why organizations need one, and how its People, Process and Technology fit together. It finishes with a practical exercise where I work an alert as a Level 1 Analyst using the 5 Ws.

**Learning objectives**

- Building a baseline for the SOC (Security Operations Center)
- Detection and response in a SOC
- The role of People, Processes, and Technology
- A practical exercise


## Task 1: Introduction to SOC

Technology has made life more efficient, and that efficiency comes with more responsibility. Modern fears have moved well beyond the exploitation of physical assets. Critical data, referred to in the room as secrets, is no longer kept in physical files. Organizations hold large amounts of confidential data in their networks and systems, and any unauthorized disruption, loss or modification of that data can cause them serious damage. Threat actors find and exploit new vulnerabilities in these networks and systems every day, which makes them a major concern in cyber security. Traditional security practices are often not enough to stop many of these threats, so dedicating a whole team to managing an organization's security is important.

A **SOC (Security Operations Center)** is a dedicated facility run by a specialized security team. The team continuously monitors an organization's network and resources and looks for suspicious activity so that damage can be prevented. It works 24 hours a day, seven days a week.


## Task 2: Purpose and Components

The main focus of the SOC team is keeping **Detection and Response** intact. The team uses security solutions that integrate the whole company's network and systems so everything can be monitored from one centralized location. Continuous monitoring is what makes it possible to detect and respond to a security incident.

### Detection

**Detect vulnerabilities.** A vulnerability is a weakness an attacker can exploit to do things beyond their permission level. It can exist in the software of any device, including the operating system and programs on a server or computer.

*Example:* the SOC might find a set of Microsoft Windows computers that need to be patched against a specific published vulnerability.

Strictly speaking, vulnerabilities are not necessarily the SOC's responsibility, but unfixed vulnerabilities lower the security level of the entire company.

**Detect unauthorized activity.** Unauthorized activity has to be caught quickly, before it causes damage.

*Example:* an attacker finds the username and password of one of the employees and uses them to log in to the company system. Many clues, such as geographic location, can help detect this.

**Detect policy violations.** A security policy is a set of rules and procedures created to help protect a company against security threats and to ensure compliance. What counts as a violation differs from company to company.

*Examples:* downloading pirated media files, and sending confidential company files insecurely.

**Detect intrusions.** Intrusions are unauthorized access to systems and networks.

*Examples:* an attacker successfully exploiting the company's web application, or a user visiting a malicious site and getting their computer infected.

### Response

**Support with incident response.** Once an incident is detected, certain steps are taken to respond to it. The response includes minimizing the impact and performing a root cause analysis of the incident. The SOC team also helps the incident response team carry out these steps.

### The three pillars of a SOC

A SOC becomes mature, and detects and responds to different incidents efficiently, when it has three pillars: **People, Process and Technology**. They coexist in a SOC environment. A mature SOC is a team of professionals working on state-of-the-art security tools within proper processes.

<img width="825" height="801" alt="image" src="https://github.com/user-attachments/assets/199560f2-3dca-49dc-ae61-8b4254c79761" />


**Question:** What are the three pillars of a SOC?

**Answer:** People, Process, Technology


## Task 3: People

Even as more security tasks are automated, the people in a SOC stay important. A security solution can generate a huge number of red flags, and that produces a lot of noise.

*Analogy from the room:* imagine you are on a fire brigade team with centralized software that has every fire alarm in the city integrated into it. You get many fire notifications at once, all for different places. When your team gets to those locations, most turn out to have been triggered by excessive smoke from cooking. In the end, all that effort is a waste of time and resources.

A SOC with security solutions and no human intervention ends up focusing on irrelevant issues in the same way. The people are the ones who help the solution identify truly harmful activity and make a prompt response possible. These people are the SOC team, and it has the following roles.


| Role | Responsibility |
|---|---|
| **SOC Analyst (Level 1)** | Anything detected by the security solution passes through these analysts first, so they are the first responders to any detection. They perform basic alert triage to decide whether a detection is harmful, and they report detections through the proper channels. |
| **SOC Analyst (Level 2)** | Some detections need deeper investigation than Level 1 can give. Level 2 Analysts help dig deeper and correlate data from multiple data sources to perform a proper analysis. |
| **SOC Analyst (Level 3)** | Experienced professionals who proactively look for threat indicators and support incident response. Critical severity detections reported by Level 1 and Level 2 are often security incidents that need a detailed response, including containment, eradication and recovery. This is where Level 3 experience is needed. |
| **Security Engineer** | Analysts work with security solutions, and those solutions need to be deployed and configured. Security Engineers do this and keep the solutions running smoothly. |
| **Detection Engineer** | Security rules are the logic built into security solutions to detect harmful activity. Level 2 and Level 3 Analysts often create these rules, but a SOC team can also use the detection engineer role independently for this responsibility. |
| **SOC Manager** | Manages the processes the SOC team follows and provides support. The SOC Manager also stays in contact with the organization's CISO (Chief Information Security Officer) and updates them on the SOC team's current security posture and efforts. |

> **Note:** The roles in a SOC team can increase or decrease depending on the size and criticality of the organization.

**Question:** Alert triage and reporting is the responsibility of?

**Answer:** SOC Analyst (Level 1)

**Question:** Which role in the SOC team allows you to work dedicatedly on establishing rules for alerting security solutions?

**Answer:** Detection Engineer


## Task 4: Process

Each role in the SOC has its own processes. For example, Level 1 Analysts act as first responders, carry out alert triage and decide whether an alert is harmful. The room covers three important processes.

### Alert Triage

Alert triage is the basis of the SOC team, because the first response to any alert is to triage it. Triage focuses on analyzing the specific alert, which determines its severity and helps prioritize it. It is all about answering the **5 Ws**.

<img width="740" height="627" alt="image" src="https://github.com/user-attachments/assets/ce85ce4c-5e65-44c6-879c-fed8cd5c026f" />

*Example from the room:*

**Alert: Malware detected on Host: GEORGE PC**

| 5 Ws | Answer |
|---|---|
| **What?** | A malicious file was detected on one of the hosts inside the organization's network. |
| **When?** | The file was detected at 13:20 on June 5, 2024. |
| **Where?** | The file was detected in the directory of the host "GEORGE PC". |
| **Who?** | The file was detected for the user George. |
| **Why?** | After the investigation, it was found that the file was downloaded from a pirated software-selling website. The investigation with the user revealed that they downloaded the file because they wanted to use a piece of software for free. |

### Reporting

Harmful alerts need to be escalated to higher-level analysts so they can be handled in a timely way. Alerts are escalated as tickets and assigned to the relevant people. A report should cover all of the 5 Ws along with a thorough analysis, and screenshots should be used as evidence of the activity.

### Incident Response and Forensics

Sometimes the reported detections point to highly malicious and critical activity. In those cases, higher-level teams start an incident response. In some cases, a detailed forensics activity also has to be carried out. Its aim is to determine the root cause of the incident by analyzing artifacts from a system or network.

**Question:** At the end of the investigation, the SOC team found that John had attempted to steal the system's data. Which 'W' from the 5 Ws does this answer?

**Answer:** Who

**Question:** The SOC team detected a large amount of data exfiltration. Which 'W' from the 5 Ws does this answer?

**Answer:** What


## Task 5: Technology

The right People and Processes are not enough without security solutions for detection and response. The Technology pillar refers to those solutions, which reduce the manual effort the SOC team needs to detect and respond to threats.

An organization's network has many devices and applications. Detecting and responding to threats on each one individually would take significant effort and resources. Security solutions centralize the information from the devices and applications on the network and automate detection and response.

### SIEM

Security Information and Event Management (SIEM) is used in almost every SOC environment. It collects logs from various network devices, called **log sources**. Detection rules configured in the SIEM contain the logic for identifying suspicious activity. The SIEM correlates events across multiple log sources and alerts the team when something matches one of the rules. Modern SIEM solutions go beyond rule-based detection and offer user behavior analytics and threat intelligence capabilities, supported by machine learning algorithms that strengthen detection.

> **Note:** The SIEM solution only provides the Detection capability in a SOC environment.

### EDR

Endpoint Detection and Response (EDR) gives the SOC team detailed real-time and historical visibility into the activity of devices. It works at the endpoint level and can carry out automated responses. It has extensive detection capabilities for endpoints, which lets you investigate them in detail and respond with a few clicks.

### Firewall

A firewall works purely for network security and acts as a barrier between the internal network and external networks such as the Internet. It monitors incoming and outgoing network traffic and filters unauthorized traffic. Firewalls also have detection rules deployed, which help identify and block suspicious traffic before it reaches the internal network.

### Other solutions

Several other security solutions play their own roles in a SOC, such as Antivirus, EPP, IDS/IPS, XDR, SOAR and more. Which technology to deploy comes after careful consideration of the organization's threat surface and available resources.

**Question:** Which security solution monitors the incoming and outgoing traffic of the network?

**Answer:** Firewall

**Question:** Do SIEM solutions primarily focus on detecting and alerting about security incidents? (yea/nay)

**Answer:** Yea


## Task 6: Practical Exercise of SOC

This exercise brings People, Process and Technology together and walks through the role of a Level 1 Analyst.

### Scenario

I am the Level 1 Analyst on the organization's SOC team. An alert comes in saying port scanning activity has been observed on one of the hosts in the network. I have access to the SIEM, where all the logs associated with the alert are visible. The task is to go through the logs individually and answer the 5 Ws.

> **Note:** The vulnerability assessment team told the SOC they were running a port scan inside the network from the host `10.0.0.8`.

### Findings

| Question | Answer |
|---|---|
| **What:** Activity that triggered the alert? | Port Scan |
| **When:** Time of the activity? | June 12, 2024 17:24 |
| **Where:** Destination host IP? | 10.0.0.3 |
| **Who:** Source host name? | Nessus |
| **Why:** Reason for the activity? (Intended/Malicious) | Intended |
| **Additional Investigation Notes:** Has any response been sent back to the port scanner IP? (yea/nay) | yea |

**Question:** What is the flag found after closing the alert?

**Answer:** `THM{000_INTRO_TO_SOC}`

### Analysis

The alert looked like reconnaissance, since a port scan is a common early step in an attack. The deciding context was the note from the vulnerability assessment team, which had announced a scan from `10.0.0.8`. The source host in the logs was Nessus, which matches that announcement, so the activity was intended and not malicious. This is why the Why question matters in triage: the same activity can be harmless or hostile depending on whether it is expected, and the answer comes from correlating the alert with what the organization already knows.

## Key Takeaways

- A SOC exists to keep **detection and response** running continuously, and it operates 24/7.
- Detection covers vulnerabilities, unauthorized activity, policy violations and intrusions. Response supports the incident response team with impact reduction and root cause analysis.
- **People:** Level 1 triages and reports, Level 2 investigates deeper and correlates sources, Level 3 hunts threats and leads critical responses. Security Engineers deploy and configure the tools, Detection Engineers write the rules, and the SOC Manager runs the processes and reports to the CISO.
- **Process:** alert triage through the 5 Ws, reporting through tickets with evidence, and incident response with forensics for critical cases.
- **Technology:** SIEM for detection through log correlation, EDR for endpoint visibility and response, and firewalls for filtering network traffic.
- Context decides whether an alert is malicious. The port scan in the practical was benign because it was a scheduled assessment from a known source.
