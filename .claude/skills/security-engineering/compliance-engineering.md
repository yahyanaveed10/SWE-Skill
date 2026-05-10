# Compliance Engineering

The engineering subset of compliance — the parts that require code or architecture decisions rather than legal or process decisions. Compliance has two faces: the legal/regulatory face (what GDPR requires, what HIPAA officer responsibilities are) and the engineering face (how the system actually enforces those requirements).

This skill covers the engineering face. The interpretation of any specific regulation belongs with whoever owns compliance in your organisation — not with the engineer or the agent.

---

## The hard rule

**Compliance requirements are non-negotiable in scope; the implementation is an engineering decision.**

If GDPR requires the right to erasure, the system must support it. *How* the system supports it (synchronous deletion vs. eventual deletion; hard delete vs. soft delete with retention; cascade strategy across replicas, backups, and caches) is engineering judgment. The requirement does not specify the implementation; the implementation must satisfy the requirement.

The agent failure mode: ignoring compliance requirements because they are "legal stuff," or implementing them naively in ways that violate the spirit of the requirement (a "delete user" endpoint that deletes the row in one table but leaves PII in 6 other tables, a backup, and the search index).

---

## PII in logs

The most common compliance failure that comes from engineering decisions, not legal interpretation.

**What counts as PII (personally identifiable information):** anything that can identify an individual either directly (name, email, government ID) or indirectly (combinations of attributes that uniquely identify a person). Some categories are sensitive PII (health, financial, biometric, location, sexual orientation) with stricter handling rules.

**Where PII leaks into logs:**
- Logging the full request body (which contains form data, including PII)
- Logging exception traces that include PII variables in the stack
- Logging API responses (which may contain user data)
- Debug logging in development that ships to production
- Third-party libraries that log internal state

**Mitigation patterns:**

**Deny-by-default logging.** Log structured fields explicitly; never serialise an entire object containing user data. If logging a User object, log `user_id` and `tenant_id`, not the whole record.

**Scrubbing at the logging layer.** A logging middleware that strips known PII fields before emitting. Works for common patterns but fails for unexpected PII shapes (a user-supplied free-text field that contains a credit card number).

**Sensitive-field marking.** Type-level marking of fields as sensitive (`type SensitiveString = ...`) so the logger refuses to serialise them. Requires discipline at type definition time but catches issues at compile time rather than at audit time.

**Log retention policy.** Even with scrubbing, logs accumulate over time and may include PII that slipped through. A retention policy (delete logs after N days) limits the blast radius of any single leak.

---

## Audit trail requirements

Many compliance regimes (HIPAA, PCI-DSS, SOX, SOC2) require audit trails for sensitive operations: who did what, when, and from where.

**What an audit log entry must contain:**
- **Who** — the authenticated user/service that performed the action (with their identity at the time of the action; if the user account is renamed later, the audit log still shows the original identity)
- **What** — the specific operation (not "user updated record" — "user X updated field Y from value A to value B on record Z")
- **When** — timestamp with sufficient precision (millisecond, in UTC, with timezone explicit)
- **Where** — IP address, request ID, session ID (so the audit log entries can be correlated to other telemetry)
- **Why** — when applicable, the business reason or ticket reference (less common; required for some regulated environments)

**The audit log must be tamper-resistant.** If the user who performed the action can also delete or modify the audit entry, the audit log is worthless. Implementation patterns:
- Audit log written to a separate datastore with restricted write access
- Append-only log with cryptographic chaining (each entry references the hash of the previous; tampering breaks the chain)
- Write-once storage (S3 Object Lock, append-only databases)
- Replicated to a system the operator cannot modify (compliance vault, third-party audit service)

**Retention.** Different regimes require different retention periods (often 1-7 years). The retention policy must be enforced — neither too short (compliance failure) nor too long (privacy concern).

---

## Data retention and right to erasure

Two opposing forces: retain data long enough to meet legitimate business and compliance needs; delete data when retention is no longer justified or when a user requests it.

**Right to erasure (GDPR Article 17 and equivalents):** users can request that their personal data be deleted. The system must support this. Engineering implications:

**Identify all locations of user data.** A user's data is rarely in one table. It may be in:
- Primary database (user table, plus every table that references the user)
- Caches (Redis, CDN, application-level caches)
- Search indexes (Elasticsearch, Algolia)
- Analytics warehouses (often slow-changing dimension tables that retain history)
- Backups (last 30/60/90 days)
- Logs
- Third-party services (CRM, email service, payment processor, support tools)
- Replicas, read-replicas, and standby instances

**Delete from all of them.** A right-to-erasure request that deletes from the primary database but leaves data in the search index is incomplete. Build a deletion path that touches every location, or document explicitly which locations are exempt and why.

**Soft delete vs. hard delete.** Soft delete (setting a `deleted_at` flag) preserves the record for application use cases (audit, reversibility) but does not satisfy erasure — the data is still there. For erasure, the record must be physically removed or the personal fields must be irreversibly nullified.

**Backups are the hardest part.** A backup taken before the deletion contains the deleted data. Restoring the backup brings it back. Common patterns:
- Backup retention windows that match the maximum allowable delay for erasure (e.g., backups expire after 30 days; deletion is considered complete after the last pre-deletion backup expires)
- Re-run deletion against restored backups
- Separately encrypted user data with per-user keys; "delete" by destroying the key, leaving encrypted data that is permanently unreadable

**Eventual consistency of deletion.** A deletion request may take time to propagate across all systems. Document the maximum time and ensure it meets regulatory requirements. Audit the propagation.

---

## Encryption at rest and key management

**Encryption at rest** means data on disk is encrypted; an attacker who steals the disk cannot read the data without the key. Most cloud providers offer this transparently (S3, EBS, RDS).

**The naive assumption:** "We use AWS, so our data is encrypted." Partially true. AWS encrypts at the storage layer with AWS-managed keys. For most threats, this is sufficient. For threats that include AWS itself (regulatory, sovereign-data) or for higher-stakes data, application-level encryption is required.

**Application-level encryption.** Data is encrypted by the application before being written to storage. The storage system never sees plaintext. The application holds the keys.

**Key management:**
- Keys stored in a key management service (KMS, Vault, HSM), not in the application code or environment variables
- Key rotation on a schedule (annual at minimum; quarterly for high-stakes systems)
- Separation of duties — the team that holds the keys is different from the team that holds the data
- Per-user or per-tenant keys for stronger isolation (lets you destroy access by destroying the key)

**The trap with field-level encryption:** encrypted fields cannot be queried directly. A user lookup by email needs the email to be searchable, but if it is encrypted, you cannot WHERE on it. Solutions: deterministic encryption (same input always produces same output — searchable but weaker), blind indexes (a hash of the value stored alongside, queryable), or separating the searchable identifier from the encrypted record.

---

## The engineering scope

What an engineer can decide:
- How to implement deletion across systems
- How to design the audit log schema
- What encryption pattern to use
- How to scrub PII from logs
- How to enforce data retention policies in code

What requires the compliance/legal owner:
- Whether a specific data type counts as PII under a specific regulation
- What retention period is legally required
- Whether a specific use case is permitted under a specific regulation
- How to respond to a regulator inquiry
- Whether a third-party processor is approved

The agent and the engineer should not interpret regulations. They should implement what the compliance owner specifies, and they should surface engineering questions where the regulation is silent.
