---
name: security-engineering
description: Security design, threat modelling, and compliance engineering. Use when designing authentication or authorisation flows, handling user input or external data, storing or transmitting sensitive data, identifying trust boundaries in a new feature, threat-modelling a new integration, reasoning about access control, designing audit trails, implementing data retention or right-to-erasure, or handling PII in logs. Covers STRIDE threat modelling, secure design principles, common vulnerability signals, and the engineering subset of compliance (PII handling, audit trails, data retention/deletion, encryption at rest, key management). Does not cover code review security checklists (use the security-review command) or legal interpretation of regulations.
---

# Security Engineering

Security problems are cheaper to prevent than to fix. A vulnerability introduced at design time — a missing trust boundary, an over-privileged component, an unauthenticated internal path — is orders of magnitude harder to remediate after deployment than before it.

Security engineering means asking security questions at design time, not only at review time.

For threat modelling with STRIDE see [threat-modeling.md](threat-modeling.md).
For secure design principles see [secure-design-principles.md](secure-design-principles.md).
For common vulnerability signals see [common-vulnerabilities.md](common-vulnerabilities.md).
For compliance engineering (PII in logs, audit trails, data retention, right to erasure, encryption at rest) see [compliance-engineering.md](compliance-engineering.md).

## Safety vs. security

**Safety** is protection against hazards that originate from the operation of the system — unintended harm caused by the system itself (a brake system that fails, a dosing algorithm that miscalculates).

**Security** is protection against external threats — deliberate malicious action by an attacker who is trying to make the system behave in a way it was not designed to.

Both matter. They are not the same. Security engineering focuses on adversarial threats.

## The CIA triad

The three properties a system must preserve against attack:

**Confidentiality** — information is accessible only to those authorised to access it. Violated by data leaks, overly broad permissions, unencrypted storage or transmission.

**Integrity** — information and system state can only be modified by authorised parties through authorised channels. Violated by injection, tampering, unauthorised writes.

**Availability** — the system functions when legitimate users need it. Violated by denial-of-service, resource exhaustion, destructive attacks.

When evaluating a design, ask which CIA properties are relevant to each component and what the consequences are if each is violated.

## Trust boundaries

A trust boundary is a point in the system where data crosses from one trust level to another — from the internet into your API, from one microservice to another, from user input into a database query.

**Rule:** Every trust boundary is an attack surface. Data crossing a trust boundary must be validated, sanitised, and authorised before use.

**Ask when designing:**
- Where does data enter the system from outside the trust boundary?
- What is the trust level of each component? Does component A trust component B's input without validation?
- Are there internal trust boundaries (admin vs. regular user, service A vs. service B) that are not enforced?

Draw trust boundaries explicitly before implementing. Implicit trust is a design vulnerability.
