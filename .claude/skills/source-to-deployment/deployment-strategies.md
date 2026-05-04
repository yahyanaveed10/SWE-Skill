# Deployment Strategies

Trade-offs for each major deployment strategy. The goal is controlling blast radius — how many users are affected, for how long, if the deployment has a problem.

---

## Recreate (stop-then-start)
Stop the old version entirely, then start the new version.

**Fits when:** Downtime is acceptable, the database migration requires the old version to be fully stopped, a single-instance environment.
**Does not fit when:** Uptime requirements exist, the operation is user-facing.
**Blast radius:** 100% of users see downtime. Duration is the startup time of the new version.

---

## Rolling update
Replace instances one at a time (or in small batches). At any point during the rollout, old and new versions are running simultaneously.

**Fits when:** Multiple instances, no breaking API changes between old and new (both versions must be simultaneously compatible with clients and databases), moderate blast radius tolerance.
**Does not fit when:** The old and new versions cannot coexist (incompatible database schema, breaking API change).
**Blast radius:** Partial — only users hitting new instances see the new behavior during rollout.
**Key constraint:** During a rolling update, requests can hit either version. The new version must be able to read data written by the old version, and vice versa. Schema changes must be backwards-compatible.

---

## Blue/Green
Two identical environments (blue = current, green = new). Switch traffic from blue to green atomically (at the load balancer).

**Fits when:** Zero-downtime deployment is required, instant rollback is required (switch back to blue), the blast radius of a bad deployment must be minimised.
**Does not fit when:** The infrastructure cost of running two full environments simultaneously is not acceptable, stateful services make the switch non-atomic (e.g., two database write endpoints cannot be switched atomically).
**Blast radius:** Near-zero for users — the switch is fast. Database state after the switch is not rolled back automatically.
**Rollback:** Switch load balancer back to blue. Fast and reliable for stateless services.

---

## Canary
Route a small percentage of traffic to the new version. Monitor. Gradually increase or roll back.

**Fits when:** Confidence in the new version is uncertain, real production traffic is needed to validate the change, gradual confidence-building is more important than speed of rollout.
**Does not fit when:** The change must be atomic (A/B split of the same transaction is problematic), the team does not have observability infrastructure to compare canary vs. baseline metrics.
**Blast radius:** Controlled — only the canary percentage of users sees the new version.
**Requires:** Metrics comparison between canary and baseline (error rate, latency, business metrics). Without this, canary is just slow rolling deployment.

---

## Feature flags
Decouple code deployment from feature activation. Code ships to all instances; the feature is turned on for specific users or percentage of traffic via a flag.

**Fits when:** Long-running features in development need to be merged to trunk without being visible, graduated rollout by user segment is needed, instant rollback without redeployment is required.
**Does not fit when:** The team cannot manage flag lifecycle (flags that are never cleaned up become permanent complexity), the overhead of the flag service is not justified for a small team.
**Blast radius:** Controlled by flag configuration — can be turned off instantly without a deployment.
**Debt:** Feature flags must have an expiry plan. Flags that stay in the codebase after the feature is fully rolled out become permanent branches in the logic.

---

## Database migration during deployment

This is the highest-risk part of most deployments. The pattern:

1. **Expand phase:** Add new columns/tables. Old version ignores them; new version uses them. Both versions run simultaneously.
2. **Contract phase:** Remove old columns/tables after the old version is fully gone.

Never drop a column or rename a column in the same deployment that changes the code reading it. The old version will break immediately.

**Ask:** Can the old version and new version run simultaneously against the same database schema? If not, you need a more careful migration plan.

---

## Rollback signals

**Rollback is fast only if it was planned.** Ad-hoc rollback under production pressure is slow and error-prone. Before deploying:
- Know the rollback procedure
- Know whether the database migration is reversible
- Know what "rollback" means for in-flight requests

A blue/green switch is fast. Redeploying the previous version from a pipeline is slow. Feature flags are instant. Know which mechanism applies to the deployment.
