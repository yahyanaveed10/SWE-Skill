# Common Vulnerability Signals

What to ask before building — not a post-hoc checklist. Each entry describes how to recognise a vulnerability risk in a design and what questions to ask before it becomes code.

---

## Injection (SQL, command, LDAP, template)

**Design signal:** User-controlled input is concatenated with a query, command, or expression rather than parameterised.

**Ask:**
- Where does user input enter a query or command? Is it parameterised (placeholder-based) or concatenated?
- Are ORM escape mechanisms being bypassed anywhere (raw queries with string formatting)?
- Are shell commands constructed from user input?

**Before building:** Identify every place user data is used in a query or command. Verify parameterisation is used. There is no safe way to sanitise input for injection — parameterisation is the only reliable control.

---

## Broken Authentication

**Design signal:** Authentication state is managed in a way that can be bypassed, stolen, or forged.

**Ask:**
- Where are session tokens or JWTs stored? (localStorage is vulnerable to XSS; httpOnly cookies are not.)
- Are session tokens invalidated on logout and on password change?
- Is there a mechanism to invalidate all sessions for a user (account takeover recovery)?
- Are password reset flows resistant to guessing (short-lived tokens, one-time use)?

---

## Insecure Direct Object Reference (IDOR)

**Design signal:** Resource IDs in requests are used to access data without verifying the requester owns or is authorised to access that resource.

**Ask:**
- If a user changes the ID in this request to another user's ID, what happens?
- Is ownership or authorisation checked against the server-side session, or against a parameter in the request?
- Are sequential or predictable IDs used where an attacker could enumerate resources?

**Before building:** For every endpoint that accesses a resource by ID, explicitly verify that the authenticated user is authorised to access that specific resource. Do not rely on obscurity (non-sequential IDs).

---

## Server-Side Request Forgery (SSRF)

**Design signal:** The server fetches a URL or connects to a host specified by user input.

**Ask:**
- Does this endpoint make outbound requests based on user-supplied URLs or hostnames?
- Can the URL target internal services (metadata endpoints, internal APIs, local services) that are not intended to be accessible?
- Is the set of valid destination hosts allowlisted rather than just denylisted?

---

## Path Traversal

**Design signal:** User input is used to construct a file path, and the path is not validated or constrained to a safe directory.

**Ask:**
- Is user input used to construct file paths? Are `../` sequences stripped or rejected?
- Is the constructed path validated to be within the intended base directory before use?
- Can the path construction be replaced with an indirect reference (an ID that maps to a path on the server) rather than a direct path?

---

## Sensitive Data Exposure

**Design signal:** Sensitive data is stored, logged, or transmitted without appropriate protection.

**Ask:**
- What data is logged by this code path? Could it include passwords, tokens, personal data, or payment information?
- Is sensitive data encrypted at rest? Is the encryption key managed separately from the data?
- Is sensitive data transmitted over unencrypted channels?
- Are secrets (API keys, database credentials) stored in a way that limits access to only the processes that need them?

---

## Cross-Site Scripting (XSS)

**Design signal:** User-controlled content is rendered in a browser context without escaping.

**Ask:**
- Is user-supplied content rendered as HTML anywhere? Is it escaped before rendering?
- Are content security policies in place to limit the impact of a successful XSS?
- Are `httpOnly` and `Secure` flags set on session cookies?
