# Agentic Security Monitoring

This section of my SOC analyst portfolio covers monitoring AI agents the way a SOC monitors users and service accounts: reading what an agent did from its audit log, working out what is normal for it, and checking whether an unusual action was actually authorised.

Agents can cause real damage without a single failed call, because they use accounts and tools they are allowed to use. Alerts built around failures have nothing to react to, so this lab focuses on the comparison work a SOC does instead: behaviour against a baseline, and actions against the records that should exist for them.

## What This Lab Covers

- Reading agent audit log rows: who acted, which tool, with what details, under which credential, and what started it
- Building a behavioural baseline for an agent (tools, call frequency, credential) and spotting deviations
- Checking whether an unusual action was authorised: a service desk request, an approval that names it, and an approval logged before the action
- Spotting slow behavioural drift that a rolling average hides
- Recognising a missing record, such as a schedule that runs with no creation record
- Reading control decisions in the log (allow, deny, modify, not_evaluated) and understanding checkpoints and kill switches
- Investigating an agent incident end to end and reporting a verdict to a manager

## Labs

| Lab | Platform | Focus Area |
|-----|----------|------------|
| [Agentic Security Monitoring](https://github.com/angeline-infosec/SOC-Analyst-Lab-Portfolio/blob/main/Agentic-Security-Monitoring/Agentic%20Security%20Monitoring.md) | TryHackMe | AI agent audit logs, baselining, drift, missing records, controls, and incident investigation |

## Platforms & Tools

- TryHackMe: simulated company (TryTrainMe) running eight AI agents
- Agent chat and Inspector: used to follow each request through the agents' tool calls
- Audit log queries: searched through the log agent to pull rows by session, actor, and event type
- Service desk requests: checked against approvals to confirm authorisation

## How This Lab Is Documented

The writeup follows the same investigation format as my other labs:

- **Lab overview:** what the room is about, in plain language, and how it maps to normal SOC work
- **Task walkthroughs:** the prompts I sent, the log rows I found, and what each one meant
- **Authorisation checks:** the request, the approval, and the order they happened in
- **Timelines and verdicts:** what happened, in order, and whether it was authorised
- **Detection ideas:** what I would alert on after seeing the incident
- **Loose ends:** what I could not confirm from the logs alone

This format is intentional. It reflects how I approach investigations: showing the evidence behind a conclusion, and being clear about what the logs do and do not prove.
