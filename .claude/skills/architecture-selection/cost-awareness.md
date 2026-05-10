# Cost Awareness

Cost is an architectural constraint that becomes load-bearing late — usually after the bill arrives and someone asks why infrastructure spend grew 5x while traffic grew 1.5x. By then, the architecture has assumed the cheap version is fine, and unwinding it is expensive.

This skill is for cost as an engineering signal, not as a finance discipline. When does an architecture decision have a meaningful cost implication? When does an agent generate code that will be expensive to run? What patterns waste money silently?

---

## The cost surface area

Software costs come from a small set of categories:

**Compute** — CPU, memory, time. Pay-per-second on cloud, per-machine on dedicated hardware. Often dominated by background work (cron jobs, async workers, ML inference) rather than user-facing requests.

**Storage** — bytes at rest. Cheap per GB; expensive in volume. The cost grows with the *retention policy*, not just the current volume.

**Network egress** — bytes leaving your cloud. Almost always more expensive than bytes within the cloud, often more expensive than storage. The "free CDN traffic" is usually only free within the same provider.

**Per-API-call costs** — third-party services (LLM API, geocoding, payment, email, SMS) charge per call. Cost dominates when calls are made in loops, fan out without limits, or cache nothing.

**Data transfer between services** — cross-region or cross-AZ traffic has its own cost line. A microservice architecture that ignores network topology can produce surprising bills.

**Human cost** — engineering time spent operating, debugging, and patching. Often invisible in the bill but the largest cost in early-stage systems.

---

## Patterns that waste money silently

**API calls in loops without caching.** Calling a paid API once per item in a 10000-item list, when the result for most items would be the same. Common with geocoding, currency conversion, LLM calls. Fix: cache by input, or batch the requests.

**Unlimited fan-out.** A request triggers N downstream calls where N is unbounded. A user with 1000 items triggers 1000 API calls; a user with 100000 triggers 100000. Cost scales with the worst user, not the average. Fix: bound the fan-out, paginate, or use bulk APIs.

**Polling loops with short intervals.** A worker polls a queue or a database every second. 99% of polls return nothing. The cost: continuous compute, continuous database load, continuous network. Fix: use long-polling, push notifications, or webhooks.

**Forever-running background jobs.** A job scheduled to run every minute that should have ended after the migration completed. A scaled-up worker pool that never scaled back down. A debug log shipper consuming a terabyte per day in development. Fix: review what is actually running and why.

**Oversized instances.** A service deployed on a 16-core, 64GB instance because "we might need it." 90% of the time, 90% of the resources are idle. Fix: right-size based on actual usage, with autoscaling for spikes.

**LLM calls without context limits.** Sending the entire conversation history (or entire document) on every call. Cost scales with input tokens; large inputs are expensive. Fix: summarise, truncate, or use embedding-based retrieval.

**Unbounded log retention.** Logs stored forever at high resolution. Most logs are useful for hours or days, not years. Fix: tiered retention (hot for 7 days, warm for 30, cold for 90, deleted after).

**Cross-AZ database queries.** A service in zone A queries a database in zone B. Each query has cross-zone network cost. At scale, this is a real bill. Fix: deploy services in the same zone as their primary database, or pay deliberately for redundancy.

---

## When cost is an architectural decision

These decisions have order-of-magnitude cost implications. Get them right early; changing them later is expensive.

**Sync vs. async.** A synchronous request holds a connection, a thread, and possibly a database transaction for the entire duration. An async pattern (job queue, message bus) decouples the request from the work, allowing cheaper processing. For long-running work, async is often 10x cheaper.

**Pull vs. push.** Pulling (polling) data on a schedule has continuous cost. Push (webhooks, change notifications) has cost only when something changes. For low-frequency updates, push is dramatically cheaper.

**Cached vs. computed.** Re-computing a value on every request that needs it can be expensive (LLM calls, complex aggregations, external API calls). Caching trades memory for compute. The economic question: how often is the cached value used? If used >5x per cache lifetime, caching usually wins.

**Dedicated vs. shared.** Dedicated capacity is predictable but pays for idle time. Shared capacity (serverless, multi-tenant) pays per use. For low-utilisation workloads, shared wins. For high-sustained workloads, dedicated wins.

**Region choice.** Cloud regions have different prices for the same instance. US-east is usually cheapest. The cheapest region may be in a different country, with latency and data sovereignty implications.

**Build vs. buy.** A managed service (managed database, managed queue, managed search) is more expensive per unit than running your own — but cheaper than the engineering time to operate your own at production reliability. The cost question is total: dollar cost + engineering time. For small teams, buy almost always wins. For large teams at scale, build sometimes wins.

---

## When to optimise for cost vs. when to ignore it

**Ignore cost when:**
- The total spend is small enough that engineering time to optimise costs more than the savings (a $50/month service is not worth a week of engineering)
- The product is unvalidated — optimising the cost of a feature nobody uses is wasted work
- A faster implementation matters more than a cheaper one (high-stakes user-facing latency)

**Optimise cost when:**
- A specific cost line is growing faster than the value it produces
- A clearly wasteful pattern is identified (no need for measurement; just fix it)
- The cost of a feature exceeds its value to the business
- Hardware utilisation is low enough that scaling down is safe

**The signal of "engineer, not finance":** if the cost optimisation requires understanding the system to do safely (without breaking functionality, performance, or reliability), it is engineering work. If it is purely about turning off unused resources, it is operations work.

---

## Build-vs-buy quick framework

When evaluating whether to build something or adopt a third-party service:

**Build cost = (engineering time to build) + (engineering time to operate per year × N years) + (opportunity cost of not building something else)**

**Buy cost = (subscription / per-use cost per year × N years) + (integration time) + (lock-in risk)**

Compare honestly. Most teams underestimate the operating cost of building (it does not stop after launch — patching, on-call, upgrades, reliability work) and overestimate the lock-in risk of buying (most services can be migrated away from with weeks-to-months of work, not years).

**Default to buy** for non-core capabilities. Build only for things that are differentiated to your business.
