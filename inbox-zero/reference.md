# Inbox Zero Reference

Supporting detail for [SKILL.md](SKILL.md). Read when classifying edge cases, formatting outputs, or explaining future capabilities.

**MCP required:** Live triage and mailbox actions need a configured email MCP. See [setup.md](setup.md). If tools are missing, stop and send the user there — do not fake an inbox.

## Intent cues

| Intent | Typical signals |
|--------|-----------------|
| Request | Asks you to do something; action verbs; deadlines |
| Question | Direct questions; needs an answer to unblock |
| Update | FYI, status, “no action needed” |
| Notification | Automated system mail; receipts; CI; shipping |
| Marketing | Promo language; % off; sales CTAs |
| Newsletter | Digest, unsubscribe footer, periodic content |
| Invoice | Amount due, payment link, PO, billing |
| Appointment | Calendar invite, reschedule, confirm time |
| Personal | Friends/family; non-work tone |
| Automated alert | Monitoring, uptime, threshold breach |
| Potential security concern | Password reset, new login, MFA, suspicious activity, domain spoofing |

## Priority scoring heuristics

Raise priority when any apply:

- Sender in `important_contacts` or domain in `important_domains`
- Security / legal / money / production language
- Explicit deadline within 48 hours
- Customer escalation or executive sender
- Thread already marked High/Critical

Lower priority when:

- Bulk unsubscribe headers / List-Unsubscribe
- No-reply automated marketing
- Pure FYI with no ask
- Sender in `never_notify_senders`

If heuristics conflict with preferences, **preferences win**.

## Decision examples

**Newsletter from vendor** → Low → label + archive (if enabled) → count in daily report.

**Customer: production down** → Critical → immediate escalation + draft acknowledgment (do not promise fixes unless user said to send).

**Colleague: “which approach should we pick?”** → High → briefing + draft recommendation, do not send.

**Scheduler: “Tuesday or Wednesday?”** → Medium → draft reply with proposed times; create calendar hold only if prefs allow and times are clear.

**Contract revision request** → High/Critical by deadline/value → escalate; never auto-approve.

## Escalation template (Critical / High)

```
IMPORTANT EMAIL
From: Acme Corporation (billing@acme.com)
Priority: High
Summary: They request a contract revision before Friday.
Why this matters: Impacts a $50k renewal.
Recommended action: Review contract terms today.
AI recommendation: Accept the requested timeline extension.
Draft (if useful):
---
Thanks — I've received the revision request and will confirm by EOD Thursday.
---
```

## Critical interrupt card

```
HIGH PRIORITY EMAIL
From: Customer X
Reason: Customer reporting production outage.
Recommended action: Respond within 30 minutes.
Suggested response:
---
{draft}
---
```

## Safe vs unsafe actions

| Action | Auto OK? | Notes |
|--------|----------|-------|
| Label / categorize | Yes | Per prefs |
| Archive newsletters | Yes | If `archive_newsletters` |
| Create task / reminder | Yes | If `create_tasks` |
| Draft reply | Yes | If `draft_replies` |
| Send reply | Only if `send_replies: true` | Default off |
| Acknowledge receipt | Only if `send_acknowledgments: true` | Still skip legal/conflict |
| Calendar event | Yes if tools + prefs | Confirm ambiguous times with user |
| Unsubscribe | Suggest only by default | |
| Delete | Never auto | |
| Approve contract / spend | Never auto | |
| Share secrets / attachments outward | Never auto | |

## Daily report example

```
Good morning.
Inbox status: 312 → 8 actionable

Completed automatically:
✓ Archived 47 newsletters
✓ Categorized 23 notifications
✓ Created 3 tasks
✓ Drafted 5 replies (not sent)

Needs your attention:
1. John Smith — Contract renewal
   Priority: HIGH
2. AWS Security Alert
   Priority: CRITICAL
3. Sarah — Project decision
   Priority: MEDIUM

Recommended focus: Respond to John first (deadline Friday), then AWS alert.
```

## Preferences learning examples

User says → write to preferences:

| Utterance | Preference update |
|-----------|-------------------|
| Always notify me about emails from my accountant | Add to `important_contacts`; escalation immediate |
| Never notify me about LinkedIn messages | `never_notify_senders` or `escalation.linkedin: none` |
| Archive all software newsletters | `always_archive_intents` includes newsletter; optional sender rules |
| Draft responses but don’t send | `draft_replies: true`, `send_replies: false` |

## Future capabilities

Not required for daily triage; offer only if user asks:

- **Relationship intelligence** — “You haven’t responded in 45 days.”
- **Commitment tracking** — “You promised pricing three weeks ago.”
- **Email memory** — prior thread outcomes (e.g. churn risk).

Philosophy: the best Inbox Zero system does not make email disappear. It creates a decision layer:

Email arrives → AI understands → AI decides → Human focuses only where judgment matters.
