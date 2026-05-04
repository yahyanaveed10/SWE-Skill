# Secure Design Principles

Principles that make a system harder to attack by design. These are reasoning tools — apply them to specific decisions, not as abstract ideals.

---

## Least Privilege

**Principle:** Every component, process, and user should have the minimum permissions needed to do its job — and no more.

**Ask:**
- What is the minimum set of permissions this service/user/process actually needs?
- If this component is compromised, what can an attacker do with the permissions it has?
- Are these permissions scoped as narrowly as possible (read-only where writes are not needed, table-specific where full database access is not needed)?

**Trade-off:** Least privilege requires more thought at design time and more operational overhead (managing fine-grained permissions). The payoff is containment — when a component is compromised, the blast radius is bounded.

---

## Defence in Depth

**Principle:** Do not rely on a single security control. Layer controls so that if one fails, others remain.

**Ask:**
- What happens to security if this control is bypassed? Is there a next layer?
- Are authentication and authorisation applied at every layer (API gateway AND service AND database), or only at one?

**Trade-off:** Multiple controls add complexity and performance overhead. The right investment is proportional to the consequence of a breach.

---

## Fail Secure

**Principle:** When a component fails, it should fail in a way that denies access rather than grants it.

**Ask:**
- If the authentication service is unavailable, does the system allow requests through or deny them?
- If a permission check throws an exception, does the system default to allow or deny?
- What is the secure default for every security-sensitive decision?

**Trade-off:** Failing secure can cause availability impact (legitimate users are denied when the auth service is down). This is the correct trade-off when confidentiality or integrity matters more than availability.

---

## Separation of Duties

**Principle:** No single component or person should have the ability to perform a sensitive operation end-to-end without a second control.

**Ask:**
- Can a single user or service initiate and approve a sensitive operation (money transfer, production deployment, account deletion)?
- Are high-stakes operations gated on a second factor or second approver?

**Trade-off:** Separation of duties adds friction to legitimate operations. The right investment is proportional to the consequence of a unilateral error or malicious action.

---

## Minimise Attack Surface

**Principle:** The less code, functionality, and exposure that exists, the less there is to attack.

**Ask:**
- Is every exposed endpoint, port, and API necessary?
- Are unused features disabled rather than left dormant?
- Are internal services exposed externally when they do not need to be?

**Trade-off:** Minimising attack surface requires actively removing functionality rather than just not using it. This takes ongoing discipline.

---

## Do Not Trust Input

**Principle:** Treat all input from outside the current trust boundary as potentially malicious, regardless of source.

**Ask:**
- Is input validated and sanitised at every trust boundary, including internal service-to-service calls?
- Is the validation based on allowlisting (what is permitted) rather than denylisting (what is blocked)?
- Are input size, type, format, and range all validated?

**Trade-off:** Thorough input validation adds code and processing overhead. The alternative — trusting input — is the root cause of injection, overflow, and parsing attacks.
