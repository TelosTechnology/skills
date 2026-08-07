# Engineering review modes

Pick one mode (router in SKILL.md). Follow that playbook, then apply all four lenses.

---

## change-review

**Goal:** Defect-first review of a PR/diff/commit. Gate on unresolved highs.

1. Resolve the change set (`git diff`, PR files, or user-named paths). Prefer merge-base vs target branch when reviewing a branch.
2. Read enough surrounding code and existing tests to validate each claim.
3. Scan S/R/P/O on **introduced or worsened** issues.
4. Emit ≤6 findings + suggested tests; verdict Block / Proceed…
5. Pre-existing issues → Residual only (unless user asked for full audit).

**Output:** Standard review format.

---

## incident

**Goal:** Restore service, then harden. Speed over perfection; still evidence-based.

1. Capture symptom, start time, blast radius, recent deploys/changes.
2. Form 1–3 hypotheses; seek confirming/disconfirming evidence (logs, metrics, traces, configs).
3. Recommend **mitigate now** (rollback, flag, scale, failover) vs **fix forward** with rationale.
4. After mitigation path is clear: findings for root cause + prevent recurrence.
5. Define verify-recovery checks and follow-up tickets.

**Do not** redesign the system mid-incident unless required for mitigation.

**Output:** Incident brief format.

---

## debug

**Goal:** Find the causal defect for a bug or failing test.

1. Reproduce or pin the failing assertion / stack / behavior.
2. Bisect: recent diff, dependency, data, env. Cite the smoking gun.
3. One primary finding (usually reliability or security); optional related findings.
4. Patch recommendation + regression test (required).

**Output:** Debug brief, then findings schema for hardenings.

---

## infra-audit

**Goal:** Posture report for cloud/K8s/IaC scope the user names (account, region, stack, cluster).

1. Confirm scope and read-only access (CLI, IaC files, MCP). Do not mutate infra unless asked.
2. Analyze with S/R/P/O (+ cost hygiene under performance/operability).
3. Cite resource identifiers and attributes; note analyzer gaps / failed API calls.
4. Prioritize actions (high first); group by blast radius.

**Typical surfaces:** IAM, network exposure, storage encryption/public access, logging/audit, DB HA/backups, compute waste, Kubernetes RBAC/NetworkPolicy when in scope.

**Output:** Infra posture format.

---

## design-critique

**Goal:** Stress-test a proposed design (or help choose among options) using S/R/P/O. Self-contained — no other skill required.

1. Restate goals/constraints in 3–5 lines; ask only if a required unknown blocks critique.
2. Inspect relevant existing code/infra so the critique is grounded.
3. Attack: failure modes, authz, data integrity, operability, scale cliffs.
4. Offer **2 options** when the shape is still open (or critique the given proposal); pick a recommendation with rationale.
5. Offer a simpler alternative if it meets goals.
6. Verdict: Block / Proceed with changes / Proceed with residual risk.
7. If the user needs a durable artifact to approve before build → write the **Design brief** from SKILL.md and wait for explicit approval. Do not refuse or redirect to a missing skill.

**Output:** Standard review (findings = design risks) + options/recommendation + “Simpler alternative” when relevant. Design brief only when asked or clearly needed for an approval gate.

---

## test-harden

**Goal:** Produce copy-pasteable tests that lock behavior and catch abuse/edge cases.

1. Inspect diff + nearest existing tests (framework, fixtures, patterns).
2. Propose 2–6 tests: at least one `edge_case` and one `security` or `fuzz` when warranted; include `regression` for known bugs.
3. Each item: title, rationale, category, framework, targetPath, insertHint, code, optional notes.
4. Code must be syntactically valid for the framework; mock I/O; use plausible imports.

**Output:** Suggested-tests list (can stand alone or attach to a review).

---

## rollout

**Goal:** Safe migrate/deploy plan for a risky change.

1. Classify risk: expand/contract schema, dual-write, backfill, API compatibility, flag strategy.
2. Ordered steps: deploy order, migration, verification, rollback.
3. Findings for missing observability, irreversible steps, or insufficient rollback.
4. Define abort criteria and monitors.

**Output:** Standard review + ordered rollout checklist.

---

## prod-readiness

**Goal:** Ship/no-ship checklist for a feature or service.

Scan:

- [ ] Security: authz, secrets, threat-relevant inputs
- [ ] Reliability: failure modes, migrations, idempotency
- [ ] Performance: expected load, budgets, pagination
- [ ] Operability: dashboards, alerts, runbooks, rollback
- [ ] Tests: critical path + regression for prior bugs
- [ ] Compliance/privacy if in scope (ask if unknown)

**Verdict:** Ship / Ship with conditions / No-ship — with findings backing any condition.

---

## reconcile

**Goal:** Update the ledger of prior review findings after new work.

1. Ingest prior open findings (ids/titles/severities).
2. Inspect current evidence.
3. Partition: **resolved** (concrete fix visible) / **still open** / **new**.
4. Veto false resolutions: if a fresh scan still reports the same issue, keep open.
5. Recompute verdict from unresolved highs.

**Output:** Standard review with Fixed / Still open / New sections.

---

## intake

Use when the request is vague (“make it better”, “any issues?”).

Ask in one batch (max ~5):

1. What artifact? (PR, path, stack, incident link)
2. What decision do you need? (merge gate, fix plan, ship call, posture)
3. Any constraints? (time, compliance, no-downtime)
4. Prior review thread / findings to continue?
5. Implement after review, or review-only?

Then re-enter the matching mode.
