# AI Agent Security

This section of my SOC analyst portfolio covers the security of AI agents from a SOC point of view. The two labs here work as a pair. One asks how an analyst monitors agents and tells whether something unusual was authorised. The other asks where outside text could steer an agent in the first place, and whether the logs would even show it.

Agents can cause real damage without a single failed call, because they use accounts and tools they are allowed to use. Alerts built around failures have nothing to react to, so both labs focus on comparison work instead: behaviour against a baseline, actions against the records that should exist for them, and an agent's configuration against what it can reach.

## What These Labs Cover

**Monitoring and investigation (Agentic Security Monitoring)**

- Reading agent audit log rows: who acted, which tool, with what details, under which credential, and what started it
- Building a behavioural baseline for an agent (tools, call frequency, credential) and spotting deviations
- Checking whether an unusual action was authorised: a service desk request, an approval that names it, and an approval logged before the action
- Spotting slow behavioural drift that a rolling average hides
- Recognising a missing record, such as a schedule that runs with no creation record
- Reading control decisions in the log (allow, deny, modify, not_evaluated) and understanding checkpoints and kill switches
- Investigating an agent incident end to end and reporting a verdict to a manager

**Exposure review (Agentory)**

- Mapping an agent estate from configuration files: which agents take in outside text, read shared memory, convert audio or images, or can raise their own identity
- Following untrusted text through the system, including the shared queue and shared memory
- Finding the guardrail rules that are switched off and what each would have prevented
- Seeing an outside email change an agent's action in the Inspector
- Asking whether the audit log would show any of it, and what to alert on

## Labs

| Lab | Platform | Focus Area |
|-----|----------|------------|
| [Agentic Security Monitoring](https://github.com/angeline-infosec/SOC-Analyst-Lab-Portfolio/blob/main/AI-Agent-Security/Agentic%20Security%20Monitoring.md) | TryHackMe | AI agent audit logs, baselining, drift, missing records, controls, and incident investigation |
| [Agentory](https://github.com/angeline-infosec/SOC-Analyst-Lab-Portfolio/blob/main/AI-Agent-Security/Agentory.md) | TryHackMe | Agent exposure review from configuration, untrusted text, queue and memory trust, and switched-off controls |

The two labs use the same fictional company, TryTrainMe, and I recommend reading them in this order: monitoring first, then exposure. Agentory builds on the records and controls the first lab introduces.

## Platforms & Tools

- TryHackMe: simulated company (TryTrainMe) running eight AI agents
- Agent chat and Inspector: used to follow each request through the agents' tool calls
- Audit log queries: searched through the log agent to pull rows by session, actor, and event type
- Service desk requests: checked against approvals to confirm authorisation
- Configuration files (`agents.json`, `tools.json`, `queue.yaml`): read in a terminal to map each agent's exposure

## How These Labs Are Documented

Each writeup follows the same investigation format as my other labs:

- **Lab overview:** what the room is about, in plain language, and how it maps to normal SOC work
- **Task walkthroughs:** what I did in each task, the evidence I found, and what each piece meant
- **Verdicts and checks:** whether an action was authorised, or which controls were switched off
- **Detection ideas:** what I would alert on after seeing the findings
- **Loose ends:** what I could not confirm, and where my own reading of the evidence goes beyond what the room demonstrates

This format is intentional. It reflects how I approach investigations: showing the evidence behind a conclusion, and being clear about what the logs and files do and do not prove.
