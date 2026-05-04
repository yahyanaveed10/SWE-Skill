# API Design and Contracts

---

## Backward compatibility as the hard constraint

An API change is backward compatible if existing clients continue to work without modification after the change is deployed. The discipline is: **assume you cannot coordinate with your clients**.

Safe changes (backward compatible):
- Adding a new optional field to a response
- Adding a new optional parameter to a request
- Adding a new endpoint
- Adding a new enum value (with caution — see below)

Breaking changes (require versioning or a deprecation cycle):
- Removing a field from a response
- Changing a field's type (string → integer, nullable → required)
- Changing a field's semantics (same name, different meaning)
- Removing an endpoint
- Making an optional parameter required

**Adding enum values is a subtle breaking change.** If clients switch on known enum values and have a default case, a new value hits the default. If clients validate against a closed set of known values, they reject the new value. Document whether your enums are open (clients must handle unknown values) or closed (the set is exhaustive and stable).

---

## The Tolerant Reader principle

Clients should ignore fields they do not recognise. Producers should not remove fields they have published.

This principle — from Martin Fowler — is what makes evolutionary API design possible. A client that rejects unknown fields breaks when the producer adds new fields. A producer that removes fields breaks clients that depend on them.

**Design implication:** validate only what you need. Do not reject requests because they contain extra fields. Do not assert that a response contains only the fields you expect.

---

## Idempotency

A write operation is idempotent if calling it multiple times with the same input produces the same result as calling it once. This matters for:

- Retry logic: if a network failure occurs after the server processes but before the client receives the response, the client must retry safely
- Duplicate submissions: users who click a submit button twice, mobile apps that replay requests on reconnect

**Idempotency keys:** the pattern for making non-idempotent operations safe to retry. The client generates a unique key per operation and sends it in a header (`Idempotency-Key`). The server stores the key and the result; if it sees the same key again, it returns the stored result without reprocessing. Stripe, PayPal, and most payment APIs use this pattern for exactly this reason.

**Ask for every write operation:** if this request is retried after a network failure, what happens? If the answer is "the operation runs twice," the operation needs an idempotency mechanism.

---

## API versioning strategies

**URL versioning (`/v1/users`, `/v2/users`):** Simple to implement, explicit, easy to route. Cost: the version is part of the URL contract; removing a version requires updating all clients. Most common approach for REST APIs.

**Header versioning (`Accept: application/vnd.myapi.v2+json`):** Keeps URLs clean. Less discoverable — harder for clients to understand what version they are using from the URL alone. Common in API-first products.

**Date-based versioning (Stripe's approach):** Each client pins to a date (`Stripe-Version: 2023-10-16`). Breaking changes are only introduced in new versions; old clients continue to see old behaviour indefinitely. Most backward-compatible but requires maintaining multiple behaviour paths. Expensive to operate; appropriate for APIs with many external clients.

**No versioning (continuous backward compatibility):** Never make a breaking change. The Stripe model applied without explicit versioning. Only feasible with strict discipline and long deprecation cycles.

---

## Pagination design

Never return unbounded collections. A list endpoint that returns all records works in development and breaks in production when the table has a million rows.

**Cursor-based pagination** over offset-based for large or frequently-changing datasets. Offset pagination (`?page=3&limit=20`) breaks when records are inserted or deleted between pages — you see duplicates or skip records. Cursor pagination (`?after=cursor_token`) is stable under concurrent writes.

**Include a `next_cursor` or `next_page` in every list response.** Clients should not need to construct the next page URL from their own logic.

**Document the maximum page size.** Enforce it server-side. A client requesting `limit=1000000` should get an error, not a timeout or an OOM.

---

## Null vs. absent vs. empty

Three different states that are often conflated:

- **Absent:** the field was not included in the request/response (the client did not send it; it is unknown)
- **Null:** the field was explicitly set to null (the value is known to be absent)
- **Empty string / zero / empty array:** a valid value that happens to be empty

Define which states your API supports for each field. A field that can be absent and can be null and can be empty string has three distinct states with different meanings — conflating them produces subtle bugs when clients update a field to "clear" it.
