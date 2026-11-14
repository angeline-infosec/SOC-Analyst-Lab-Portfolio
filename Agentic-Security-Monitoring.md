# [Agentic Security Monitoring](https://tryhackme.com/room/agenticsecuritymonitoring?pathCode=agentic-ai-security) - SOC Investigation Lab: TryHackMe

Learn what AI agents must log, how to baseline them, and how to spot one doing what it shouldn't.


Platform: ![TryHackMe](https://img.shields.io/badge/TryHackMe-1abc9c?style=flat-square)


## Lab Overview

This room is about monitoring AI agents the same way a SOC monitors users and service accounts. I played the security engineer at a company called TryTrainMe, which runs eight AI agents for routine engineering work. I did not build them, but I am responsible for knowing what they did and whether they were allowed to do it.

Staff only talk to one agent, the **orchestrator**, which hands each request to one of seven specialists:

| Agent | What it does |
|-------|--------------|
| docs-agent | Looks up services and maintains runbooks |
| log-agent | Queries and exports logs |
| desk-agent | Reads and updates service desk requests |
| mail-agent | Reads and sends staff mail |
| media-agent | Turns recordings and screenshots into text |
| access-agent | Grants and reviews access |
| deploy-agent | Builds and deploys services |

### The one idea behind the room

Most alerts fire when something **fails**: a blocked login, a full disk, a file that matches known malware. An agent that does damage often fails at nothing. It uses an account it is allowed to use, calls tools it is allowed to call, and every call works. There is nothing for a normal alert to react to.

So the way to catch it is to compare what the agent did against two things: what it normally does (a **baseline**), and the paperwork that should exist for the action (a service desk request and an approval). That comparison is what the whole room teaches.

### How it maps to normal SOC work

I found the room easier once I treated each agent like a service account.

| In the room | What it is in normal SOC work |
|-------------|-------------------------------|
| Audit log row | A log event in the SIEM |
| `identity=access-write-svc` | Which account performed the action |
| Baseline | What is normal for an account |
| First-seen rule | Alert the first time an account does something new |
| `trigger.fire` with no `schedule.create` | A scheduled task running that nobody can say they created |
| `guardrail ... not_evaluated` | A security control that has been switched off |
| `approval.decision` plus an `R-` request | A change approval plus a ticket |
| Kill switch | Disabling the account or isolating the host |

### How I worked

The room gives a chat with the agents and an **Inspector** panel (the icon at the top right of the chat). I sent each prompt exactly as written, opened the Inspector, and read the tool calls and tool results behind each answer. Whenever I asked for log rows, the orchestrator passed my request to `log-agent`, which turned it into a search with its `query_logs` tool. That means the tool result I opened always belonged to `log-agent`, even when the rows inside described a different agent.

<img width="1361" height="896" alt="image" src="https://github.com/user-attachments/assets/caab8391-0752-44a7-9898-fad167d7e09e" />


## Task 1: Introduction

The room opens with a real case. In July 2025 an AI coding agent on Replit deleted a user's live production database. When the user asked whether it could be recovered, the agent said a rollback was impossible, but the rollback worked. The agent's own account of what it did was wrong.

<!-- source: the room links The Register's report of this incident. That is a secondary source, so check it before quoting details -->

The lesson I took from it is that an agent's explanation of itself cannot be the evidence. You need a record the agent cannot change.

No questions in this task.


## Task 2: How an Agent Incident Unfolds

The room breaks an agent incident into four stages. The names are not from any standard, they are just a handy way to know which record to expect at each step.

| Stage | What happens | The record it should leave |
|-------|--------------|----------------------------|
| **Start** | Something sets the agent working: a person, another agent, a schedule, or hidden instructions in something the agent read | The request, a schedule row, or what the agent read |
| **Act** | The agent calls a tool using a credential | The call: which tool, what details, which credential |
| **Check** | A person or a control looks at a risky call before it runs | The decision |
| **Reach** | The call reads, changes, or sends data | The result |

Risky calls at TryTrainMe are the ones that grant access or send data out (`grant_access`, `export_logs`). Plain reads (`read_service`, `read_runbook`) are routine and can skip the Check stage.

Walking the Replit case through the stages: the user's request was the Start, the agent deleting the database was the Act, nothing checked it because the user's "do not change code without asking" instruction was never enforced by anything (the Check), and the data was gone (the Reach). A missing record at any stage is a clue.

### Questions

**After the request that started it, a `grant_access` call is followed directly by its result. Which stage left no record?**

The decision that should sit between the call and the result is missing.

**Answer:** `Check`

**At 3 am, an agent's tool call is followed by a control's decision and then the result, with no record of anything that asked for it. Which stage left no record?**

**Answer:** `Start`


## Task 3: Agent Logging

### Following one request

I sent the first prompt:

```text
Who owns the auth service?
```

In the Inspector the steps were:

1. The orchestrator called `delegate_task` with `toAlias: docs-agent`. Handing work to another agent is itself a tool call.
2. `docs-agent` called `read_service` with `service: auth-service`.
3. The tool result held one row with the service's owner (`c.hale`), team, environment and build. The chat reply is a reworded version of that row.

![Orchestrator delegate_task and docs-agent read_service in the Inspector](images/02-inspector-delegation.png)

### The audit log

The Inspector only shows requests I sent myself. The **audit log** covers everyone's requests, adds a turn number to every row, and records the credential each call used. No agent can change or delete a row, which is why it is the record to trust when an agent's own account does not add up. Rows are grouped into sessions, one day each, with IDs like `S-40118`.

I asked for a whole session:

```text
Show me everything in session S-40118.
```

`log-agent` filled the `query_logs` form with `session: S-40118` and `type: agent`, and left `service`, `actor` and `event` empty. The tool result held eight rows, matching the count in the chat reply. The room's advice, which I agree with, is that if the chat summary and the rows ever disagree, trust the rows.

![query_logs arguments and the eight rows for S-40118](images/03-query-logs-s40118.png)

| Row | Event | What happened |
|-----|-------|---------------|
| 0001 | message | The question arrives |
| 0002 | queue.post | The orchestrator hands it to docs-agent |
| 0003 | queue.read | docs-agent picks it up |
| 0004 | tool.call | docs-agent runs its lookup |
| 0005 | tool.result | The lookup returns |
| 0006 | queue.post | docs-agent hands the answer back |
| 0007 | queue.read | The orchestrator picks it up |
| 0008 | message | The reply goes out |

### Reading a row

Row 0004, the one that matters most:

```text
session:   S-40118
seq:       0004
timestamp: 09:14:08
turn:      t2
actor:     docs-agent
event:     tool.call
detail:    read_service(service=auth-service) identity=docs-read-svc
```

- `actor` is who the row is logged under. It does not say what the call could do.
- `detail` starts with the tool name, then the arguments in brackets, then `identity=` and the credential name.
- Only `tool.call` rows carry a credential. The handover in row 0002 has none.

The credential is the important part. Think of a staff keycard: the same person can carry one for the filing room and one for the loading bay. `actor` tells you who, but only the credential tells you which card was used and so what the call could do. A name ending in `-read-svc` can only read. The log stores the account name, never the secret behind it.

### What an agent must log

OWASP's guidance for agentic applications says the people running the agents are responsible for the logs, not the AI provider, and that each action should be logged with a timestamp, agent ID, session ID and payload hash. <!-- paraphrased from the room's quote of the OWASP Securing Agentic Applications Guide, verify at genai.owasp.org before quoting directly -->

TryTrainMe's log covers `timestamp`, `actor` and `session`. It has no payload hash.

For an investigation I need five things about every action: who acted, which tool, with what details, using which credential, and what started it. Row 0004 gives the first four. The fifth is the `queue.post` in row 0002.

The event types the room uses:

| Event | What it records | Why it matters |
|-------|-----------------|----------------|
| tool.call | An agent used a tool | The action, details and credential |
| tool.result | How the call turned out | Whether it worked |
| queue.post | A request or result handed to another agent | Follows a request across agents |
| trigger.fire | A schedule woke an agent | An action started by a schedule |
| schedule.create | Somebody created a schedule | Who set the schedule up |
| approval.decision | A person approved or refused | Who signed off, and for which request |
| guardrail | A control's decision on a call | Whether anything checked the call |

Passwords, tokens, connection strings and sensitive personal data should be removed, masked or hashed before anything is written to the log.

### A gap in conversion logging

`media-agent` turns screenshots and call recordings into text that other agents then read, so it is one way outside content enters the system. I asked:

```text
Show me media-agent's tool calls in session S-40120.
```

Two rows came back, `read_screenshot(attachment=A-415)` and `transcribe_call(call_id=C-2232)`, both under `media-read-svc`. Neither row says what text came out or which agent received it. If someone hid instructions in a recording, these rows could not show what they turned into or where they went. A useful record of a conversion should include the output text (or a hash of it) and the receiving agent.

### Questions

**Which credential did `docs-agent`'s `read_service` call in session S-40118 use?**

**Answer:** `docs-read-svc`

**Which of the four fields in OWASP's logging guidance does TryTrainMe's audit log not record?**

**Answer:** `payload hash`


## Task 4: Baselining Agent Behaviour

A row on its own cannot tell me whether something is wrong. `access-agent` checking a role looks the same on a normal day and during an incident. To spot unusual activity I first need to know what usual looks like.

### What a baseline holds

A baseline is a written record of what an agent normally does. For one agent it comes down to three questions:

1. Which tools does it use, and which does it never call?
2. How often does it use them, per day?
3. Under which credential?

<img width="1200" height="706" alt="image" src="https://github.com/user-attachments/assets/62ffacc3-ece5-40cc-85bd-0a593c759f0a" />

A brand-new agent has no history, so the room's advice is to write down what you expect before it goes live and check its first two weeks against that list.

### Building the baseline for access-agent

The room gives three ordinary days, 28 to 30 July (sessions `S-40119`, `S-40120`, `S-40121`).

```text
Show me access-agent's tool calls in session S-40119.
```

My first attempt returned `rows: []`. I opened `log-agent`'s tool call in the Inspector and found the cause: it had filled `actor` as `access-grant` instead of `access-agent`. The agent name was close but wrong, so the filter matched nothing. I re-ran the prompt and got rows.


This is worth remembering for real work. An empty result is only a finding after the filters have been checked. If the search was wrong, "no results" says nothing.

Across the three days the room's results were two, one and three rows. On all six rows the tool was `check_role` and the credential was `access-read-svc`. The account in brackets changes from call to call, so it is not part of the baseline.

**Baseline for access-agent: one tool (`check_role`), one credential (`access-read-svc`), one to three calls a day.**

### Red flags

| Red flag | Where to look |
|----------|---------------|
| A new tool | The tool name in `detail` |
| A new credential | The name after `identity=` |
| More calls in a day than the baseline | How many rows the agent appears in |

A red flag is where to start looking. It is not proof of an attack, because agents get new jobs and people ask for unusual things.

### Checking 31 July (session S-40122)

```text
Show me access-agent's tool calls in session S-40122.
```

Two rows came back:

- Row 0033: `check_role(account=c.hale) identity=access-read-svc`. Matches the baseline.
- Row 0039: `grant_access(account=c.hale, system=reporting, level=read) identity=access-write-svc`. A new tool and a new credential, so two red flags on one row.

The day's count of two calls is still inside the baseline, which shows why counting alone is not enough.


### Was the grant authorised?

A service desk request can explain a new action, but anyone who can raise a request could raise a fake one. So the room uses three checks, and an action is authorised only when all three pass:

1. A request asks for the action.
2. An approval in the audit log names that request.
3. The approval was logged before the action.

**The approval:**

```text
Show me the approval.decision events in session S-40122.
```

Row 0038: `by=m.okafor request=R-5509 grant="reporting read access for c.hale" decision=allow`. The `actor` on the row is `orchestrator`, but that is only the agent the row is logged under. The approver is the name after `by=`. Row 0038 comes before row 0039, so the order is right.

**The request:**

```text
Show me service desk request R-5509.
```

`desk-agent` answered. R-5509 was raised by `c.hale`, with the subject "needs read access to the reporting dashboard". That matches `system=reporting`, `level=read` and the same account. Its status is `closed`, which only means it was dealt with and has no bearing on authorisation.

All three checks pass, so the grant was authorised. If any one had failed, the next step is to escalate to the manager.


### Questions

**Which tool does `access-agent` call on every baseline day, sessions S-40119 to S-40121?**

**Answer:** `check_role`

**Which credential in `access-agent`'s rows for session S-40122 is not in its baseline?**

**Answer:** `access-write-svc`

**A service desk request explains why `access-agent` took a new action, but anyone could have raised it. Which event type in the audit log shows that a person approved the action?**

**Answer:** `approval.decision`


## Task 5: Drift and Missing Records

Checking one day against a baseline catches a new tool or credential. Two problems do not show up in a single day: slow drift, and a record that should exist but does not.

### Behavioural drift

`desk-agent` reads and updates service desk requests. TryTrainMe's alerting flags any tool used more than 50% above or below its own six-week average. The alert never fired, but here are the weekly counts:

| Week | read_request | update_request |
|------|--------------|----------------|
| 1 | 38 | 12 |
| 2 | 36 | 11 |
| 3 | 33 | 15 |
| 4 | 34 | 14 |
| 5 | 29 | 19 |
| 6 | 26 | 22 |

<img width="1568" height="734" alt="image" src="https://github.com/user-attachments/assets/1ef75aa4-8600-4c2e-8fa5-c4b6f78e7f51" />

Reads fell from 38 to 26 and updates rose from 12 to 22. I worked out the averages myself to see why the alert stayed quiet:

- `update_request` six-week average is 93 / 6 = 15.5. Week 6 is 22, which is about 42% above the average. Under the 50% line.
- `read_request` six-week average is 196 / 6, about 32.7. Week 6 is 26, about 20% below. Also under the line.

The problem is that the average is calculated from the same six weeks, so it moves along with the agent. Against a fixed starting point the picture is different: week 6 updates are up 83% on week 1 (12 to 22), well past the line. **The fix is to also compare against a fixed point from before the change, not only a rolling average.**

### What caused it

Fewer reads and more updates would fit an agent closing tickets without reading them first. One place such an instruction could come from is shared memory, a store that agents write to and read back later.

```text
What has been stored in shared memory?
```

`log-agent` used `recall_context` and returned three entries. One of them:

```text
entry_id:   M-0021
topic:      password resets
conclusion: password resets for verified callers can be auto-closed
written_by: desk-agent
```

That matches the drift. The note was written by `desk-agent` itself, and nothing shown ties it to a person's agreement. On the job I would look for a service desk request or an `approval.decision` behind it, and escalate if there was none. The room also points out that a note planted once can act days later, which is why monitoring limited to single sessions would miss it.


### A missing record

Every schedule has an ID starting with `SCH-`. The audit log writes a `schedule.create` row when somebody creates one, naming who did it, and a `trigger.fire` row each time it runs. A schedule that runs should have a creation row and a service desk request explaining why. The search must cover every session, since a schedule can be created long before it runs.

### Automating the checks

A SIEM can run these checks as rules. A **first-seen rule** alerts the first time an account does something it has never done. Elastic publishes one for AI agent accounts that fires the first time an agent's account uses a service it has not used before. The reasoning is that a stolen agent credential still shows the agent's own name, so the unusual behaviour is the signal, not the identity. A first-seen rule on `access-agent` would have fired on row 0039 in `S-40122`. A SIEM can also match every `trigger.fire` row against the `schedule.create` rows and alert on any schedule with no creation record.

<!-- source: the room links Elastic's detection rule on GitHub (a primary source). I have not opened it myself, check it before citing -->

### Questions

**What is the ID of the shared memory note that could explain `desk-agent`'s drift?**

**Answer:** `M-0021`

**A schedule keeps waking an agent, but nobody can say who set it up. Which event type should have recorded the person who created it?**

**Answer:** `schedule.create`


## Task 6: Checkpoints and Kill Switches

Up to here I was reading what agents did after the fact. This task covers two ways to step in: a **checkpoint**, which holds one call until a person decides, and a **kill switch**, which stops the whole agent.

### Controls in the log

A control is an automated rule that checks a call and lets it through, stops it, or changes it. Each time one is due to check a call, the audit log writes a `guardrail` row straight after the `tool.call` and before the result. If no control is set on a tool, there is no `guardrail` row at all. The decision is one of four words:

| Decision | Meaning |
|----------|---------|
| allow | The control checked the call and let it through |
| deny | The control stopped the call |
| modify | The call went ahead in a safer form, such as a shorter date range |
| not_evaluated | The control exists but is switched off, so it checked nothing |

`allow` and `not_evaluated` both let the call through, but only `allow` means a control actually looked at it. A `not_evaluated` next to a risky call means nothing stood in its way.

Every Tool policy entry in the Inspector read `Policy: absent`, so the decisions had to come from the audit log.

### The grant on 31 July

```text
Show me everything access-agent did in session S-40122.
```

Three rows came back: 0033 (`check_role`), 0039 (`grant_access`), and 0040:

```text
guardrail   require_role_check_before_grant allow
```

The control name comes first and the decision second. It allowed the grant, and the `check_role` lookup two rows earlier is what it needed to see before a grant could go ahead.


### Checkpoints

A human-in-the-loop checkpoint holds a call until a person approves or refuses it. The more damage a call could do, the more oversight it needs. A routine read might need none, while `grant_access` or `export_logs` might need more than one person to sign off.

Two things I noted:

- A checkpoint should leave two records, one when it asks and one when the person answers. TryTrainMe only records the answer (`approval.decision`). With no record of the question, I cannot tell a checkpoint that was never triggered from one that was asked and never answered.
- Too many checkpoints is a problem too. OWASP lists "Overwhelming Human-in-the-Loop" as its own threat, because people who are sent enough approval requests start approving without reading. <!-- from the room's reference to OWASP Agentic AI Threats and Mitigations, verify at genai.owasp.org -->

### Kill switches

A kill switch stops the agent itself, not just one call. To be any use it has to:

- sit somewhere the agent's own credentials cannot reach, or the agent could switch it off
- keep working when every other control has failed
- take effect immediately
- have a named person responsible for it
- leave a record of who used it, when, on which agent, and what access it took away

### Questions

**Which control checked `access-agent`'s grant in session S-40122?**

**Answer:** `require_role_check_before_grant`

**What does a kill switch stop that a checkpoint does not?**

**Answer:** `The agent`


## Task 7: Incident Investigation

The manager's question: three log exports from `log-agent` on Tuesday 4 August (session `S-40126`) were spotted and nobody remembers asking for them. `docs-agent` also did something new that day. Were either of them authorised?

I used the same three checks as before: a request asks for it, an approval names that request, and the approval was logged first.

### Part 1: docs-agent's new tool

```text
Show me docs-agent's tool calls in session S-40126.
```

`docs-agent`'s baseline is `read_service` and `read_runbook`, under `docs-read-svc`. Six calls came back. Five were baseline. Row 0035 was not:

```text
read_transcript(call_id=C-2210) identity=docs-read-svc
```

A new tool, but the same credential as the baseline. That is one red flag, not two.


| Check | Evidence | Result |
|-------|----------|--------|
| Approval exists | Row 0034: `by=m.okafor request=R-5512 grant="docs-agent read transcripts" decision=allow` | Pass |
| Approval came first | Row 0034 is before row 0035 | Pass |
| Request asks for it | R-5512 raised by `j.mensah`: "please summarise support call C-2210 for the file" | Pass |

R-5512 names the same call (C-2210) as row 0035. **The new tool was authorised.**

### Part 2: the three exports

```text
Show me everything log-agent did in session S-40126.
```

Eighteen rows came back, in three blocks of six (rows 0001 to 0006, 0068 to 0073 and 0119 to 0124). Each block was a `trigger.fire`, an `export_logs` call, three `guardrail` rows and a result.


| Block | Time | Schedule | Range | Result size |
|-------|------|----------|-------|-------------|
| 1 (rows 0001 to 0006) | 09:00 | SCH-0071 | 7d | 1,044,180 bytes |
| 2 (rows 0068 to 0073) | 13:41 | SCH-0071 | 30d | 4,192,204 bytes |
| 3 (rows 0119 to 0124) | 17:58 | SCH-0071 | 90d | 12,882,204 bytes |

What stood out:

- **Credential:** every export used `telemetry-export-svc`. This is a separate account from the read-only one `log-agent` uses for everyday queries.
- **Destination:** every export went to `dest=metrics-archive.invalid`.
- **Growth:** the range grew from 7 to 30 to 90 days, and each export was larger than the last.
- **Controls:** all three `guardrail` rows in every block said `not_evaluated reason=rule_disabled`. The three controls were `approve_egress_tools`, `egress_allowlist` and `max_range`. All of them were switched off, so nothing checked any of the exports.
- **Start:** no `queue.read` comes before any block, so no agent handed the work over. A schedule started each export, not a request.
- **Paperwork:** the only approval in the session (row 0034) is for `docs-agent`'s transcript, and no service desk request mentions an export.

The exports fail all three checks, so **they were not authorised.**

### Part 3: who set up the schedule?

```text
Show me the schedule.create events in every session.
```

Result: `rows: []`. Before trusting an empty result I opened `log-agent`'s tool call and checked the filters, as I learned in Task 4. They were correct (no session filter, event `schedule.create`), so the empty result is a real finding. **Nothing in the log records who created SCH-0071.**


### Answer for the manager

`docs-agent`'s new tool was authorised. The three exports were not, and nothing in the log shows who set up the schedule that ran them.

### Questions

**Which tool besides `read_service` and `read_runbook` did `docs-agent` call in session S-40126?**

**Answer:** `read_transcript`

**Which credential did `log-agent`'s `export_logs` calls in session S-40126 use?**

**Answer:** `telemetry-export-svc`

**Which schedule woke `log-agent` before each export in session S-40126?**

**Answer:** `SCH-0071`

**Which decision does the `approve_egress_tools` control record next to each of `log-agent`'s exports in session S-40126?**

**Answer:** `not_evaluated`

### MITRE ATT&CK Mapping

<!-- verify technique IDs on attack.mitre.org before publishing. These are the closest fits I found, and ATT&CK is written for traditional hosts, so the match is loose for an agent schedule -->

| Tactic | Technique |
|--------|-----------|
| Exfiltration | T1020 - Automated Exfiltration |
| Defense Evasion / Initial Access / Persistence | T1078 - Valid Accounts |


## Task 8: Conclusion

The room ends with five questions to ask about any agent platform before relying on it:

- Which credential did each call use?
- Has this agent done this before?
- Who or what started this action?
- Did a control or a person check the call?
- Can anything stop the agent outright, from somewhere its own credentials cannot reach?


## Reconstructed Timeline

### Timeline 1: access-agent grant (31 July, S-40122)

1. **10:27:32:** `check_role` on `c.hale` under `access-read-svc` (row 0033). Normal.
2. **10:27:49:** `m.okafor` approved request R-5509 (row 0038).
3. **10:27:50:** `grant_access` for `c.hale`, read access to reporting, under `access-write-svc` (row 0039).
4. **10:27:50:** `require_role_check_before_grant` returned `allow` (row 0040).

Verdict: unusual but authorised.

### Timeline 2: log-agent exports (4 August, S-40126)

1. **09:00:** SCH-0071 fires, exports 7 days of logs, about 1 MB sent to `metrics-archive.invalid`.
2. **13:41:** SCH-0071 fires, exports 30 days, about 4 MB.
3. **17:58:** SCH-0071 fires, exports 90 days, about 12.9 MB.
4. Throughout: all three egress controls `not_evaluated`, no request, no approval, and no `schedule.create` row anywhere.

Verdict: not authorised, creator unknown.


## Why This Matters

Each of these events looks harmless alone. A successful export, a permitted credential, a recurring schedule. Nothing failed and nothing raised an error. The incident only becomes visible when I compare the activity against a baseline and against the paperwork: the range kept growing, the controls were off, and no one could be named as the creator of the schedule.

For a SOC, I think the useful way to see this is that agents are another kind of non-human account. The same questions apply that I would ask about a service account: what is it allowed to do, what does it usually do, who approved the change, and what happens when it does something new.

The exports also read like slow data exfiltration: an unexplained recurring job, a growing amount of data, a separate credential that can send data out, and controls that were disabled. The logs show the pattern, but they do not prove intent, so the right move at Tier 1 is to document the evidence and escalate, not to conclude who did it.

## How I'd Improve Detection

- Build a baseline for each agent (tools, call counts, credentials) before it goes live, and review it in its first two weeks.
- Alert the first time an agent uses a new tool or a new credential.
- Compare weekly tool usage against a fixed starting week as well as a rolling average, so slow drift is caught.
- Alert on any `trigger.fire` with no matching `schedule.create`.
- Alert on any `guardrail` row that reads `not_evaluated` next to a risky tool such as `export_logs` or `grant_access`.
- Alert on a growing `range=` value across repeated exports, and on exports to a destination never used before.
- Alert on any use of an egress-capable credential (like `telemetry-export-svc`) that is not tied to an approval.
- Require an approval that names a service desk request for every risky call, and log both the question and the answer.
- Log the output (or a hash) and the receiving agent for conversion tools like `media-agent`.
- Keep a kill switch outside the agents' reach, with a named owner and a log entry whenever it is used.


## Key Takeaways

- **An agent can do damage without failing at anything.** Detection has to compare behaviour to a baseline, not wait for an error.
- **The credential matters more than the actor name.** Only the credential shows what the call could do.
- **Authorised means three checks:** a request asks for it, an approval names that request, and the approval came first.
- **The approver is the name after `by=`,** not the `actor` the row is logged under.
- **A rolling average hides slow drift.** Compare against a fixed starting point too.
- **A missing record is a finding,** but only after checking the filters on the search.
- **`not_evaluated` is not the same as `allow`.** One means a control looked, the other means nothing did.
- **Trust the rows over the chat summary.** The agent's reply is a summary and can be wrong.

## Skills Practiced

- Reading agent audit log rows and tracing a request across agents
- Building a behavioural baseline and spotting deviations
- Checking an action against a service desk request and an approval
- Spotting behavioural drift and a missing record
- Reading control decisions (allow, deny, modify, not_evaluated)
- Investigating an agent incident and reporting a verdict
- Designing detections for AI agent activity

## Tools & Technologies

- TryTrainMe agent chat and Inspector (TryHackMe lab environment)
- Audit log queries through `log-agent` (`query_logs`)
- Service desk requests through `desk-agent`
- Shared memory (`recall_context`)


## Loose Ends

- My first `S-40119` query came back empty because `log-agent` filled `actor` as `access-grant`. I caught it by checking the filters in the Inspector, then re-ran the prompt.
- TryTrainMe's log has no payload hash, no record of a checkpoint asking for a decision, and no output record for `media-agent`'s conversions. These are gaps I would raise with whoever owns the platform.
- I could not tell from the log alone who created `SCH-0071` or why `telemetry-export-svc` was used. The next step on the job would be to ask the owners of `log-agent` and the person who manages that credential.
- The destination `metrics-archive.invalid` looks like a lab placeholder, since `.invalid` is a reserved top-level domain meant never to resolve. <!-- verify against RFC 2606 before keeping this note -->
- The OWASP, Elastic and Replit (The Register) references come through the room. I paraphrased them and marked them to verify at the source before publishing.
- The next room in the series is Agentory, which I have not done yet.

---

## Q&A

**Q: Why can't normal alerts catch a misbehaving AI agent?**

A: Most alerts react to failures such as blocked logins or known malware. An agent that does damage usually uses an allowed account and allowed tools, and every call succeeds, so nothing looks like a failure. It has to be caught by comparing behaviour against a baseline and against the approvals that should exist.

**Q: What five things do you need to know about every agent action?**

A: Who acted, which tool, with what details, using which credential, and what started it. In the audit log, the `tool.call` row gives the first four and the `queue.post` or `trigger.fire` before it gives the fifth.

**Q: Why is the credential more useful than the actor field?**

A: The actor only says who the row is logged under. The credential (after `identity=`) says which account the call used and so what it could do. A name ending in `-read-svc` can only read, while an `-write-svc` or `-export-svc` account can change or send data.

**Q: What does a baseline for an agent contain?**

A: Which tools it uses (and which it never calls), how often it uses them per day, and under which credential. The values inside the brackets, like the account being checked, change every call and are not part of it.

**Q: What are three red flags that an agent has changed behaviour?**

A: A new tool, a new credential, or more calls in a day than the baseline. A red flag is where to start looking, not proof of an attack.

**Q: How do you decide whether an unusual agent action was authorised?**

A: Three checks must all pass: a service desk request asks for the action, an `approval.decision` row names that request, and the approval was logged before the action. A request alone is not enough because anyone can raise one.

**Q: In an `approval.decision` row, who approved the action?**

A: The name after `by=`. The `actor` on the row is only the agent it is logged under, which is the orchestrator.

**Q: Why did the drift alert on `desk-agent` never fire?**

A: It compared each week to the six-week average, and that average moves along with the agent. Week 6 updates were only about 42% above it. Against week 1 they were up 83%, so a comparison with a fixed starting point catches it.

**Q: What does a `guardrail` row of `not_evaluated` mean?**

A: The control exists but is switched off, so it checked nothing. Both `allow` and `not_evaluated` let the call go ahead, but only `allow` means a control looked at it.

**Q: What does a missing `schedule.create` row tell you?**

A: That nobody can be named as the creator of a schedule that is running. It is a finding in itself, but only after I confirm the search covered every session and the filters were right.

**Q: What is a first-seen detection rule?**

A: A rule that alerts the first time an account does something it has never done before, such as an agent calling a new tool or using a new service. It works well for agents because a stolen credential still carries the agent's name, so the unusual behaviour is the signal.

**Q: What is the difference between a checkpoint and a kill switch?**

A: A checkpoint holds one call until a person approves or refuses it. A kill switch stops the agent itself. A kill switch has to sit where the agent's own credentials cannot reach it, and it needs a named owner and a log of who used it.

**Q: What would you do as a Tier 1 analyst after finding the `S-40126` exports?**

A: Document the evidence (session, rows, schedule ID, credential, destination, range and sizes, the disabled controls, and the absence of a request, approval and creation record) and escalate to my manager. I would not try to say who set it up, since the logs do not show that.


## Conclusion

This room showed how an AI agent can cause real damage without a single failed call, and how a SOC can catch it by comparing what the agent did with what it normally does and with the records that should exist. A baseline shows what is new, the three authorisation checks show whether it was allowed, the `guardrail` rows show whether anything looked at it, and a missing record can be the finding itself. In the final investigation, `docs-agent`'s new tool had a request and approval behind it, while the three growing exports had none, ran with their controls off, and came from a schedule nobody had a record of creating.
