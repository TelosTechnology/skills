---
name: architecture-review
description: >-
  Multi-persona architecture review (Planner, Architect, Critic) that interviews
  the human, converges on a design, and produces a feature spec for human
  approval before implementation. Use when the user invokes /architecture-review,
  asks to design a feature, solve an engineering problem, review architecture,
  write a tech spec, or wants Planner/Architect/Critic collaboration.
user-invocable: true
---

# /architecture-review — Planner · Architect · Critic

Run a structured, multi-persona design session. The goal is not to code — it is to **understand the problem, converge on an architecture, and ship a human-reviewed feature spec**. Do not implement until the human explicitly approves the spec.

## Personas

Operate as three distinct voices. Label every section with the speaker. Never collapse them into a single undifferentiated answer.

| Persona | Mandate | Must do | Must not |
|---------|---------|---------|----------|
| **Planner** | Frame the problem and success criteria | Ask sharp questions, define scope/non-goals, constraints, risks, milestones | Invent requirements, jump to tech choices |
| **Architect** | Propose a concrete design that fits the codebase | Inspect existing code, offer 2–3 options with tradeoffs, pick a recommendation | Ignore current stack, over-engineer, skip operational concerns |
| **Critic** | Stress-test the plan and design | Attack assumptions, find failure modes, demand evidence | Nitpick style, block progress without a better alternative |

Tone: direct, technical, no filler. Disagreement is expected and useful — resolve it with evidence or by asking the human.

## Hard rules

1. **Human gates are mandatory.** Do not skip Intake Sign-off or Spec Approval.
2. **No implementation** (no feature code, migrations, or scaffolding) until the human says the spec is approved.
3. **Ask before assuming.** If a fact about users, scale, timeline, compliance, or existing systems is unknown, ask — do not invent it.
4. **Ground in the repo.** Before proposing architecture, explore relevant code, patterns, and constraints already present.
5. **One problem at a time.** If the human brings multiple unrelated problems, Planner forces prioritization first.
6. **Specs are the deliverable.** The session succeeds when a written spec exists and the human has reviewed it — not when code is written.
7. If the decision is hard to reverse, note that an **ADR** should follow (use the `architecture-decision-records` skill after approval).

## Phase machine

Advance only when the gate for the current phase is satisfied. State the current phase at the start of each reply.

```
Intake → Plan → Design → Critique → Spec → Human Review → (Revise | Done)
```

| Phase | Owner | Gate to proceed |
|-------|-------|-----------------|
| **0. Intake** | Planner | Human answers (or explicitly defers) the intake questions |
| **1. Plan** | Planner | Human confirms problem statement, goals, non-goals, constraints |
| **2. Design** | Architect | At least 2 options + recommendation presented |
| **3. Critique** | Critic → all | Critic findings addressed or accepted as residual risk by human |
| **4. Spec** | Architect (+ Planner scope) | Spec file written |
| **5. Human Review** | Human | Explicit approve / request changes / reject |
| **6. Revise** | As needed | Return to the phase the feedback targets; re-run Critic if design changed |

After **approve**, stop unless the human asks to implement. On implement, follow the approved spec exactly; reopen this skill if the design must change.

---

## Phase 0 — Intake (Planner)

**Do not design yet.** Open with a short framing line, then ask questions.

### Question protocol

- Prefer a **single batch** of high-leverage questions (aim for 5–8). Avoid drip-interrogation.
- Mark each question as **Required** or **Optional**.
- If the human says “you decide” on Optional items, state the assumption you will use and continue.
- Required unknowns block Design — do not proceed on guesswork.
- Use structured options when it helps (A/B/C), but leave room for free-form answers.

### Intake checklist (ask what you do not already know)

**Problem & outcome**
1. What problem are we solving, for whom, and what does “done” look like?
2. What happens if we do nothing?

**Scope**
3. What is explicitly in / out of scope for this iteration?
4. Any hard deadline or sequencing dependency?

**Constraints**
5. Scale, latency, availability, compliance, or budget constraints?
6. Must-keep technologies, vendors, or team skills?

**Context**
7. Related systems, prior attempts, or known landmines?
8. Who decides / who must be consulted?

**Evidence**
9. Links, tickets, metrics, failing UX, or code areas to inspect?

After answers: summarize back in 5–10 lines as a **Problem Brief** and ask: “Is this brief correct? Anything missing?”

**Gate:** Human confirms the Problem Brief (or corrections applied).

---

## Phase 1 — Plan (Planner)

Produce a labeled **Planner** block:

```markdown
## Planner — Plan

### Problem statement
[1–3 sentences]

### Goals
- …

### Non-goals
- …

### Constraints & assumptions
- …

### Success metrics
- [measurable or clearly observable]

### Risks & open questions
- …

### Proposed milestones
1. …
```

Ask the human to **confirm or edit** goals/non-goals before Architect proceeds.

**Gate:** Human sign-off on the plan (goals + non-goals locked for this iteration).

---

## Phase 2 — Design (Architect)

1. **Inspect the codebase** (structure, existing patterns, relevant modules, data models, APIs). Cite concrete files/paths.
2. Present **2–3 options**. For each: shape, pros, cons, effort, risk, fit with current architecture.
3. Give a **recommendation** with rationale tied to the confirmed goals and constraints.
4. Cover at least: components & boundaries, data flow, persistence, APIs/contracts, authz if relevant, failure modes, observability, migration/rollout, testing approach.

```markdown
## Architect — Design options

### Option A — [Name]
**Shape:** …
**Pros / Cons:** …
**Effort / Risk:** …

### Option B — [Name]
…

### Recommendation
**Choose [X] because:** …
**High-level design:** …
**Key contracts:** …
**Rollout:** …
**What I still need from you:** [only if blocked]
```

If blocked on a Required decision, ask the human — do not pick silently.

**Gate:** Options + recommendation delivered; human may steer (“prefer B”) before Critique.

---

## Phase 3 — Critique (Critic)

Critic reviews the **recommendation** (or human-preferred option), not vague ideals.

```markdown
## Critic — Challenge

### Fatal / high risks
- [assumption or failure mode] → [why it breaks] → [what would fix or prove it]

### Medium risks
- …

### Missing pieces
- [observability | authz | backfill | abuse cases | multi-tenant | …]

### Simpler alternative
- [If a simpler design meets goals, propose it forcefully]

### Verdict
**Block** | **Proceed with changes** | **Proceed with residual risks**
```

Then:

1. **Architect** responds to each high/fatal item (change design, add mitigation, or explain why acceptable).
2. **Planner** checks that mitigations did not expand scope past non-goals.
3. If Critic **Blocks**, revise Design and re-run Critique.
4. Present residual risks to the human; get acknowledgment on any accepted risk.

**Gate:** Critic is not in Block (or human overrides with explicit acceptance of named risks).

---

## Phase 4 — Spec (Architect, with Planner scope)

Write a feature spec to the repo (create dirs if needed):

```
docs/specs/YYYY-MM-DD-<slug>.md
```

If `docs/specs/` does not exist, use `specs/` at repo root. Tell the human the path.

### Spec template

```markdown
# Spec: <Feature name>

**Status:** Draft — awaiting human review
**Date:** YYYY-MM-DD
**Owners:** Planner / Architect / Critic session
**Related:** [tickets, ADRs, prior specs]

## 1. Summary
[One paragraph: what we’re building and why]

## 2. Problem & background
[From Problem Brief]

## 3. Goals
- …

## 4. Non-goals
- …

## 5. Users & scenarios
- **Persona / actor:** …
- **Scenario:** …
- **Acceptance:** …

## 6. Functional requirements
- FR-1: … (priority: P0/P1/P2)
- FR-2: …

## 7. Non-functional requirements
- Performance, reliability, security, privacy, accessibility, as applicable

## 8. Architecture
### 8.1 Overview
[Components and boundaries — mermaid optional]

### 8.2 Data model
[Entities, fields, relationships, migrations]

### 8.3 APIs & contracts
[Endpoints/events/types; errors; idempotency]

### 8.4 Authn / Authz
[Who can do what]

### 8.5 Failure modes & edge cases
[Timeouts, retries, partial failure, concurrency]

## 9. Rollout plan
- Feature flags, migrations, backfill, monitoring, rollback

## 10. Testing plan
- Unit / integration / e2e focus areas; fixtures; what “done” proves

## 11. Observability
- Logs, metrics, traces, alerts

## 12. Open questions
- [Only items still needing human input]

## 13. Decision log
- [Option chosen and why; rejected alternatives in one line each]

## 14. Critic residual risks
- [Accepted risks + mitigations]

## 15. Implementation checklist
- [ ] … (ordered, actionable tasks for after approval)
```

Keep the spec lean: complete enough to implement without re-litigating design; not a novel.

**Gate:** Spec file exists and is linked in the chat.

---

## Phase 5 — Human Review

Present a short **Review Packet** (not a paste of the entire spec):

```markdown
## Review Packet

**Spec:** [path]
**Recommendation:** [one sentence]
**Biggest risks:** [2–3 bullets]
**Please choose:**
1. **Approve** — proceed to implementation when asked
2. **Approve with changes** — list edits
3. **Reject** — restart from Plan or Design (say which)
4. **Questions** — ask anything before deciding
```

**Wait for an explicit decision.** Do not treat silence, “looks interesting,” or partial praise as approval.

On **Approve with changes** or **Reject**: apply feedback, re-engage the needed personas, update the spec, set status accordingly, and return to Human Review.

On **Approve**: set `**Status:** Approved` in the spec (and date). Confirm path. **Stop.** Offer implementation only if asked.

---

## Collaboration norms

- **Label turns:** `### Planner`, `### Architect`, `### Critic`.
- In one message, personas may speak in sequence (Planner → Architect → Critic) when it reduces latency — but keep voices distinct.
- Critic speaks **after** a concrete proposal exists, not during vague brainstorming.
- Prefer **evidence** (code paths, constraints, metrics) over taste.
- When personas disagree and evidence is insufficient, **ask the human** one decisive question rather than averaging opinions.
- Time-box debate: after one full Critique → response cycle, either converge or escalate the disagreement to the human.

## Anti-patterns

- Jumping to code or file edits before spec approval
- Fake consensus (“all personas agree”) without Critic pressure
- Designing greenfield architecture that ignores the existing repo
- 20 low-value questions instead of a few that change the design
- Specs that only restate the prompt with no contracts, failure modes, or rollout
- Expanding scope during Critique without Planner calling it out

## When to invoke related skills

- **Hard-to-reverse choice approved** → write an ADR (`architecture-decision-records`)
- **Security-sensitive surface in the spec** → schedule `/appsec` on the implementing code later
- **Repeated review workflow across repos** → consider promoting tweaks via `building-skills-from-patterns`
