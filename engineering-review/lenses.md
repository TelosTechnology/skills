# Engineering review lenses — S / R / P / O

Scan **all four** on every engagement. Depth scales with mode (change-review: focus on the diff; infra-audit: resources; incident: failure path). Prefer confirmed issues over exhaustive theory.

---

## Security

### Always check

- Authn missing or bypassable; authz only on UI; IDOR / path traversal
- Injection: SQL/NoSQL/OS/LDAP/template; unsafe `eval` / deserialize
- Secrets: hardcoded values, private keys, live tokens in diff — not mere `process.env.FOO` references
- SSRF / open URL fetch; XSS / unsafe HTML; CSRF on state-changing routes
- Crypto: weak algos, custom crypto, plaintext sensitive fields
- Multi-tenant boundary leaks; mass assignment
- Supply chain: unpinned risky installs, postinstall scripts, unexpected binaries in CI
- IAM / network: `0.0.0.0/0` on sensitive ports, public buckets, overly broad roles

### Infra-specific

- Root account MFA / access keys; password policy
- S3: public access block, encryption, versioning
- Security groups / firewall rules; public RDS / Redis
- CloudTrail / audit log coverage and integrity
- Secrets Manager / KMS usage vs plaintext env in hosts

### High only if

Evidence shows an exploitable or clearly introduced hole (removed check, injectable sink with user control, committed secret material, publicly reachable sensitive resource).

---

## Reliability

### Always check

- Incorrect control flow; off-by-one; null/empty unchecked on new paths
- Error swallowing; missing timeouts; unbounded retries without backoff/jitter
- Non-idempotent writes on retryable APIs; duplicate side effects
- Partial failure / dual-write inconsistency; missing transactions where needed
- Race conditions; unsafe concurrent mutation; TOCTOU
- Schema migrations that lock or destroy data; missing backfill
- Timezones / clock skew; at-least-once message handlers that aren’t safe
- Feature flags defaulting unsafe; broken rollback path

### Infra-specific

- Multi-AZ / multi-region expectations vs single point of failure
- Backup retention; restore never tested
- Health checks that lie (process up, dependency down)
- Autoscaling / capacity headroom; noisy-neighbor quotas

### High only if

Clear data loss, corruption, crash on common path, or broken correctness in the change.

---

## Performance

### Always check

- N+1 queries; sync work on request path that should be async/batched
- Unbounded lists / missing pagination; loading whole tables into memory
- Hot-path allocations; O(n²) on realistic sizes; missing indexes (when schema visible)
- Cache stampede / no TTL / caching sensitive data incorrectly
- Huge payloads; chatty APIs; missing compression where appropriate
- Frontend: layout thrash, unbounded re-renders only when clearly in changed code
- Resource leaks: connections, file handles, goroutines/tasks

### Infra-specific

- Stopped/ idle oversized instances; unattached volumes
- Lifecycle policies missing on growing storage
- Over-provisioned DB / lack of read replicas when read-heavy (recommend with evidence)
- Cross-AZ chatter cost; idle load balancers

### High only if

Change clearly causes severe degradation or DoS under normal load (e.g. full table scan per request with evidence).

---

## Operability

### Always check

- Structured logs on new failure paths; no secrets in logs
- Metrics/traces for new critical operations; useful cardinality
- Alerts that fire on symptom + runbook link — or note absence as medium/low
- Config via env/flags; dangerous defaults; missing validation at boot
- Deploy: migrations ordered, backward-compatible APIs, rollback story
- Feature flags / kill switches for risky behavior
- Ownership: who gets paged; dashboards for the new surface
- Docs/runbooks for non-obvious ops steps introduced by the change

### Infra-specific

- Tagging / cost allocation gaps
- Drift: manual console changes vs IaC
- Least-privilege CI roles; long-lived keys vs OIDC
- Backup/restore and incident runbooks for the service

### High only if

Change makes the system undebuggable or unrecoverable in practice (e.g. removes sole audit trail, deletes backup path, deploy with no rollback and destructive migration) — still needs evidence.

---

## Cross-cutting prompts (use when stuck)

Ask of the artifact:

1. What is the trust boundary and who crosses it?
2. What happens on failure of each dependency?
3. What is the largest input this accepts?
4. How would on-call detect and undo this in 5 minutes?
5. What test would have caught this before merge?

If a finding spans lenses, pick the **primary** category; mention secondary in Risk.
