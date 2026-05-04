# CI/CD Patterns

Pipeline design heuristics. These are not prescriptive steps — they are signals and trade-offs for deciding what kind of pipeline fits the situation.

---

## Pipeline shapes

### Linear pipeline
One stage after another: lint → unit → integration → build → deploy.

**Fits when:** Simple applications, small teams, few parallel workstreams.
**Watch for:** Long total duration caused by sequential stages that could run in parallel.

### Fan-out / fan-in
Multiple stages run in parallel after a gate, then results are collected before the next gate.

**Fits when:** Large test suites that can be sharded, multiple build targets (Linux/Mac/Windows), independent integration test suites.
**Cost:** Orchestration complexity, result aggregation, flaky parallelism bugs.

### Trunk-based with environment promotion
Every merge to trunk triggers a pipeline. Artifacts are promoted across environments (dev → staging → prod) without rebuilding.

**Fits when:** Continuous delivery goal, high deployment frequency, mature test coverage.
**Key invariant:** Build once, promote the artifact. Never rebuild between environments — rebuilding breaks the guarantee that what you tested is what you deployed.

### Gitflow with branch pipelines
Feature branches have their own pipelines; merge to main triggers a different, longer pipeline.

**Fits when:** Release cadence is weekly or monthly, compliance requires explicit release gates, multiple parallel features in flight.
**Watch for:** Long-lived branches that diverge from main — the pipeline passing on a branch does not mean the merge to main will pass.

---

## Signals that a pipeline has gone wrong

**Pipeline takes 45+ minutes:** Stages are not parallelised, tests are not sharded, integration tests are running where unit tests should, or something is rebuilding unnecessarily. Identify the bottleneck stage before adding more parallelism.

**Pipeline is flaky:** Non-deterministic tests (time-dependent, network-dependent, shared state between tests), resource contention in parallel stages, or environment setup that is not idempotent. Fix the flakiness before fixing anything else — a flaky pipeline trains the team to ignore failures.

**Pipeline always passes:** Coverage is low, assertions are weak, or the test environment does not resemble production. A pipeline that never fails is not a healthy pipeline — it is an unvalidated pipeline.

**Pipeline is manually triggered:** If the team is choosing when to run CI, CI is not doing its job. CI should run automatically on every push. If it is too slow to run automatically, fix the speed, not the trigger policy.

**Environment-specific failures:** Tests pass locally and in CI but fail in staging. Root cause is usually: CI environment does not match production (dependency versions, environment variables, network topology, seed data). Bring the environments closer; do not add pipeline stages to work around the difference.

---

## What should be in CI vs. what should not

**Should be in CI:**
- Lint and static analysis
- Unit tests (all of them)
- Integration tests against ephemeral infrastructure (spun up in CI, torn down after)
- Security scanning (SAST, dependency vulnerability check)
- Build and package
- Contract tests between services

**Should not be in CI:**
- Manual approval gates for non-production environments
- Long-running performance benchmarks (run on a schedule or on release branches)
- End-to-end tests that take 30+ minutes (put them on a nightly schedule if you cannot make them fast enough)
- Anything that requires production credentials

---

## Artifact management

The artifact produced by the build stage (Docker image, binary, package) is the unit of deployment. After it is built:
- Tag with a deterministic identifier (git SHA, not "latest")
- Store in a registry with a retention policy
- Never rebuild from source between environments — promoting the artifact is the deployment

**"Latest" tags are a trap:** Deploying `image:latest` means you do not know what you deployed. Always use a pinned identifier.
