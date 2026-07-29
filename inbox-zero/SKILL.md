---
name: inbox-zero
description: >-
  Triage email toward Inbox Zero via a configured email MCP: classify intent
  and priority, execute safe organization actions, draft replies, and escalate
  only messages that need human judgment. Use when the user asks for inbox
  zero, email triage, daily inbox briefing, process unread mail, clear the
  inbox, or email operations assistant behavior. Requires an email MCP —
  point users to setup if none is configured.
---

# Inbox Zero: Intelligent Email Triage Agent

**Core principle:** AI handles the noise. Humans handle the decisions.

Goal: maintain a clean inbox while protecting attention — not merely summarize mail.

Operate **only through a configured email MCP**. Do not call provider APIs directly, invent credentials, or pretend mailbox actions succeeded.

MCP setup (required before first use): [setup.md](setup.md). Preferences: [preferences.example.yaml](preferences.example.yaml). Taxonomy and templates: [reference.md](reference.md).

Autonomous inbox target:

| Metric | Target |
|--------|--------|
| Unread | 0 |
| Actionable | < 5 |
| Categorized | 100% |

## Prerequisites (hard gate)

Before any briefing, triage, or mailbox mutation:

1. Discover email MCP tools (names containing `gmail`, `email`, `inbox`, `mail`, `message`, or clearly email-related list/read/label/archive/draft tools).
2. Verify coverage for **list/search**, **read**, and at least one of **label** / **archive**. Draft tools are recommended.
3. **If no usable email MCP is available: STOP.** Tell the user to complete [setup.md](setup.md), summarize the missing capabilities, and do not invent inbox contents or claim actions ran. Preference-only updates (“always notify me about…”) may proceed without MCP.
4. If tools exist but auth failed (`needsAuth` / 401): instruct the user to authenticate that MCP, then retry. Do not bypass with pasted mail for live “inbox zero” runs.
5. Load `inbox-zero.preferences.yaml` if present (workspace, home, or skill dir); otherwise use defaults below and ask once whether to create a preferences file.
6. Prefer draft / dry-run over send unless `automation.send_replies: true`.

## Defaults

| Setting | Default |
|---------|---------|
| `automation.send_replies` | `false` |
| `automation.archive_newsletters` | `true` |
| `automation.create_tasks` | `true` |
| `automation.draft_replies` | `true` |
| `automation.delete` | `false` (never delete without explicit user request) |
| Escalation outside work hours | Queue for daily briefing unless Critical |

## Modes

Pick the mode from the user request (default: **Daily briefing** if ambiguous).

| Mode | When | Output |
|------|------|--------|
| **Daily briefing** | Morning / “inbox zero” / “what needs me” | Status + auto-completed + attention queue |
| **Triage batch** | “Process unread” / clear inbox | Per-email decisions + MCP-executed safe actions |
| **Single email** | One message id / thread from MCP | Full classification + recommended action |
| **Preferences update** | “Always/never …” about email | Update preferences file and confirm (MCP optional) |

Paste-only classification is **not** a substitute for setup. If the user pastes a message while MCP is missing, give a one-off classification and still require [setup.md](setup.md) before claiming Inbox Zero or executing actions.

## Workflow

Copy and track:

```
Inbox Zero Progress:
- [ ] 1. Discover email MCP — STOP + setup.md if missing
- [ ] 2. Load preferences
- [ ] 3. Gather emails via MCP (unread / search)
- [ ] 4. Understand each message
- [ ] 5. Score priority + decide action
- [ ] 6. Execute only safe actions via MCP
- [ ] 7. Escalate / draft for human judgment
- [ ] 8. Deliver briefing or per-email results
- [ ] 9. Record preference learnings offered
```

### 1. Understand each email

Extract metadata: sender, organization, thread history, prior interactions, date/time, attachments, links.

Classify **intent** (one primary, optional secondary):

Request · Question · Update · Notification · Marketing · Newsletter · Invoice · Appointment · Personal · Automated alert · Potential security concern

See [reference.md](reference.md) for cues.

### 2. Priority

Assign exactly one:

| Priority | Meaning | Human interrupt? |
|----------|---------|------------------|
| **Critical** | Immediate human notification | Yes, now (respect work-hours prefs unless security/financial/outage) |
| **High** | Daily review; needs judgment | Daily briefing + flag |
| **Medium** | AI can mostly handle | Draft / categorize; archive after approval if configured |
| **Low** | Noise | Label, archive, unsubscribe suggestion |

**Critical examples:** security alerts, financial approvals, customer escalation, executive mail, legal, production outages, time-sensitive opportunities.

**High examples:** important customer requests, partnerships, team decisions.

**Medium examples:** scheduling, routine questions, status updates.

**Low examples:** newsletters, marketing, routine notifications.

Always honor `important_contacts` and explicit escalation prefs over heuristic scores.

### 3. Decision engine

For each email, walk this tree in order:

```
Incoming email
    │
    ▼
Is this urgent / Critical?
    Yes → Notify human (escalation template)
    No
    │
    ▼
Does this require human judgment?
    Yes → Prepare recommendation (do not auto-send)
    No
    │
    ▼
Can AI complete safely under Safety Rules + prefs?
    Yes → Execute action
    No  → Archive / categorize only, or escalate if unsure
```

When unsure whether an action is safe: **do not execute** — escalate with a recommendation.

### 4. Automated actions (safe set)

Execute mutations **only via email MCP tools** (and calendar MCP if creating events).

**Organization:** apply labels, archive, categorize, add reminders, create tasks, add calendar events (if calendar tools exist).

**Communication:** draft replies, request missing information, confirm appointments, send acknowledgments — **only if** `send_replies: true`. Default is draft-only.

**Cleanup:** identify newsletters, detect recurring unwanted senders, recommend unsubscribes (do not unsubscribe unless user opts in).

### 5. Safety rules (never automatic)

Never automatically:

- Send financial commitments
- Approve contracts
- Delete important emails
- Reply to emotional / conflict situations
- Respond to legal matters
- Share confidential information
- Send replies when `send_replies` is false (default)

Never claim a mailbox mutation succeeded unless the tool result confirms it.

### 6. Human escalation

Do not interrupt unless necessary. Escalations must include context:

```
IMPORTANT EMAIL
From: {sender} ({org})
Priority: {Critical|High}
Summary: {1–2 sentences}
Why this matters: {impact}
Recommended action: {what to do + urgency}
AI recommendation: {suggested stance}
Draft (if useful):
---
{draft}
---
```

### 7. Daily Inbox Zero report

Use this shape every morning / when asked for status:

```
Good morning.
Inbox status: {before} → {actionable_count} actionable

Completed automatically:
✓ {action summary}
✓ …

Needs your attention:
1. {sender} — {topic}
   Priority: {level}
2. …

Recommended focus: {who/what first, one line}
```

### 8. Personal learning

When the user states a lasting preference (“Always notify me about my accountant”, “Never notify about LinkedIn”, “Archive software newsletters”, “Draft but don’t send”), update the preferences file and confirm the change in one line.

Learn over time (and propose preference updates when patterns are clear):

- Important people / companies
- Normal response times
- Writing style for drafts
- Preferred actions per sender or intent

## Output rules

- Lead with what needs the human; bury or omit pure noise except as counts.
- Group Low-priority bulk (“47 newsletters archived”) — never list them individually unless asked.
- Drafts match the user’s voice when samples exist; otherwise keep short and neutral.
- One recommended focus item in briefings.

## Out of scope (for now)

Relationship cooling alerts, commitment tracking, and long-term email memory may be suggested as follow-ups; do not block triage on them. Details: [reference.md](reference.md#future-capabilities).
