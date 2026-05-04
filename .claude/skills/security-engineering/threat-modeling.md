# Threat Modelling

Threat modelling is the practice of systematically identifying what could go wrong before it does. STRIDE is a structured framework for doing this.

## How to use STRIDE

STRIDE is not a checklist — it is a set of questions to ask about each component and data flow in the system. Apply it to the design, not to the code.

Work through the system diagram (or a mental model of it). For each component, data store, and data flow, ask the six questions.

---

## S — Spoofing

**Question:** Can an attacker pretend to be something or someone they are not?

**Ask:**
- Can a caller claim to be an authenticated user without actually being one?
- Can a service claim to be another trusted service?
- Is the identity of callers verified at every trust boundary, or only at the entry point?

**Design signals:**
- Authentication is checked only at the API gateway but not at internal service-to-service calls
- Session tokens are stored in a way that allows them to be stolen and replayed
- Symmetric keys or shared secrets are used for service identity where mTLS would be appropriate

---

## T — Tampering

**Question:** Can an attacker modify data or code without authorisation?

**Ask:**
- Is data validated and sanitised at every trust boundary, or only at the first entry point?
- Are write operations authorised, or can a caller modify data they should only read?
- Is stored data protected from modification by unauthorised parties?

**Design signals:**
- Object IDs in requests are used directly to look up records without checking that the requester owns the record (see IDOR in [common-vulnerabilities.md](common-vulnerabilities.md))
- Data is passed between services without re-validation at each boundary
- Configuration or code can be modified through an insufficiently protected channel

---

## R — Repudiation

**Question:** Can an attacker perform an action and credibly deny having done it?

**Ask:**
- Are security-relevant actions logged with sufficient detail (who, what, when, from where)?
- Are logs tamper-evident and stored where the actor cannot modify them?
- Is there enough audit trail to reconstruct what happened in a security incident?

**Design signals:**
- Sensitive operations are not logged
- Logs are stored in the same system as the code, where a compromised application can modify them
- User IDs are not included in audit logs, only session tokens

---

## I — Information Disclosure

**Question:** Can an attacker read data they should not have access to?

**Ask:**
- Does the API return more data than the caller is authorised to see?
- Do error messages reveal internal system details (stack traces, database schema, file paths)?
- Is data encrypted in transit and at rest where needed?
- Are secrets (keys, tokens, passwords) stored in a way that limits who can access them?

**Design signals:**
- API responses include fields for all users that should only appear for the owning user
- Error responses in production include exception details
- Database connection strings or API keys in environment variables accessible to all processes
- Logging of sensitive data (passwords, credit card numbers, health information)

---

## D — Denial of Service

**Question:** Can an attacker prevent legitimate users from using the system?

**Ask:**
- Are resource-intensive operations rate-limited and authenticated?
- Can an attacker trigger expensive operations without proportionate cost to themselves?
- Are there resource limits on inputs (file size, request body size, query complexity)?

**Design signals:**
- No rate limiting on public-facing endpoints
- File upload endpoints that accept unlimited file sizes
- Operations that fan out (one request triggers many downstream requests) without load controls
- Unauthenticated endpoints that perform expensive computation

---

## E — Elevation of Privilege

**Question:** Can an attacker gain permissions they should not have?

**Ask:**
- Is authorisation checked on every operation, or only at login?
- Can a regular user access admin functionality by manipulating a parameter?
- Are roles and permissions enforced at the data layer, not just the UI layer?
- Is there a path from a low-privilege account to a high-privilege action?

**Design signals:**
- Role checks performed in UI components but not in the API
- Admin endpoints that check for a query parameter rather than a server-side role
- Privilege escalation paths through chained operations (change email → reset password → gain account)
- JWT or session data that includes the user's role and is trusted without server-side verification
