# Engineering review examples — calibrate quality

Use these as the bar for severity wording and finding shape. Prefer sparse, sharp findings over volume.

---

## Severity calibration

### Correct: high (security)

**Evidence:** Diff adds `app.get('/admin/users', (req, res) => db.users.all())` with no auth middleware; neighboring routes use `requireAdmin`.

```markdown
### [SECURITY | HIGH] Admin user list exposed without authz
- **Evidence:** `src/routes/admin.ts` (+12–18) — new handler omits `requireAdmin` used by sibling routes
- **Risk:** Any caller can enumerate users/PII
- **Recommendation:** Attach `requireAdmin` (or equivalent) before the handler; add integration test expecting 401/403 without session
- **Verify:** `test/admin-users.auth.test.ts` — unauthenticated GET `/admin/users` → 401/403; admin session → 200
```

### Incorrect: speculative high (must downgrade)

> “This **might** allow injection if inputs are not sanitized. **Ensure** queries are parameterized.”

No sink shown, no user-controlled path cited → **omit** or **low** hardening note, never high.

### Correct: medium (reliability)

**Evidence:** Catch block logs and returns 200 on payment webhook failure, so the provider will not retry.

```markdown
### [RELIABILITY | MEDIUM] Webhook handler hides failures from the provider
- **Evidence:** `src/webhooks/stripe.ts` (+40–48) — `catch` logs error then `res.status(200).end()`; Stripe only retries on non-2xx
- **Risk:** Failed side effects (DB write, entitlement grant) are silently dropped; no automatic retry
- **Recommendation:** Return 500 (or 400 only for invalid signatures); keep 200 solely for successfully processed events; make handler idempotent
- **Verify:** `test/webhooks.stripe.test.ts` — simulated processing error expects non-2xx; duplicate delivery with same event id is safe
```

### Correct: high → downgrade (secrets topic, no material)

Diff only adds `const key = process.env.STRIPE_SECRET_KEY`. Title mentions “secret handling”.

→ Not high. At most **low/info** about validation at boot if missing; env references are not credential leaks.

### Correct: high (secrets material)

Diff adds `const key = "sk_live_51AbC…"` or a `BEGIN PRIVATE KEY` block.

→ **high**, category security; recommend purge from git history + rotate.

---

## Operability finding (good)

```markdown
### [OPERABILITY | MEDIUM] Destructive migration with no rollback path
- **Evidence:** `migrations/2026-08-01_drop_legacy_events.sql` drops `legacy_events` in one step; deploy runbook has no restore
- **Risk:** Bad deploy loses forensic/event data; cannot roll back binary and keep old reads
- **Recommendation:** Expand/contract: stop writes → dual-read → backfill → drop in a later release; document restore from backup
- **Verify:** Staging dry-run of rollback; backup restore drill timed; alert on migration failure
```

---

## Follow-up reconciliation

**Prior open:** `[abc] Missing authz on /admin/users (high)`

**New commit:** adds `requireAdmin` + test.

```markdown
### Fixed since last review
- ✅ [SECURITY | HIGH] Missing authz on /admin/users — `requireAdmin` wired; auth test added

### Still open
- None

### New findings
- No major findings

**Verdict:** Proceed with residual risk
```

**Anti-pattern:** Marking resolved because the file did not appear in the latest commit hunks while `requireAdmin` is still missing → keep **still open**.

---

## Test-harden suggestion (good)

Emit these fields in chat (do **not** wrap the whole suggestion in an outer `markdown` fence — that breaks nested code fences). Put runnable source in its own language fence after the metadata:

### Rejects forged webhook signatures
- **Category:** security · **Framework:** vitest
- **Target file:** `test/webhooks.stripe.test.ts`
- **Where to paste:** Append beside existing Stripe webhook tests
- **Rationale:** Diff changes signature verification; lock the failure path

```typescript
import { describe, it, expect } from "vitest";
import { handleStripeWebhook } from "../src/webhooks/stripe";

describe("stripe webhook", () => {
  it("rejects invalid signatures", async () => {
    const res = await handleStripeWebhook({
      rawBody: Buffer.from("{}"),
      signature: "t=1,v1=deadbeef"
    });
    expect(res.status).toBe(400);
  });
});
```

---

## Infra-audit finding (good)

```markdown
### [SECURITY | HIGH] SSH open to the world on prod ASG SG
- **Evidence:** `sg-0abc` allows `tcp/22` from `0.0.0.0/0` attached to `prod-api` ASG
- **Risk:** Internet-wide brute force / exploit of sshd
- **Recommendation:** Restrict to bastion / VPN CIDR; prefer SSM Session Manager; IaC the change
- **Verify:** `aws ec2 describe-security-groups` shows no `0.0.0.0/0` on 22; can still operate via bastion/SSM
```

---

## Bad vs good titles

| Bad | Good |
|-----|------|
| Improve security | SQL injection via unsanitized `q` in `/search` |
| Consider adding metrics | No latency metric on new checkout path |
| Might be slow | N+1 `User.find` inside `map` in `listOrders` |
| Ensure errors are handled | `catch` returns 200 on webhook failure, blocking retries |

Titles should be **imperative or diagnostic**, specific, and true under the cited evidence.

---

## Verdict examples

| Open findings | Verdict |
|---------------|---------|
| 1+ high | **Block** |
| 0 high, 2 medium | **Proceed with changes** (list must-fix mediums) |
| Only low/info | **Proceed with residual risk** |
| High marked fixed with evidence; no new high | **Proceed…** (say what fixed) |
