# Infrastructure as Code Signals

Heuristics for IaC (Terraform, Pulumi, CloudFormation, etc.) — not a tutorial for any specific tool. Signals for when IaC is being misused or when it is the right choice.

---

## When IaC adds value

**The infrastructure is recreated or replicated.** If you provision the same infrastructure multiple times (dev/staging/prod environments, regional replicas, per-tenant isolation), IaC gives you deterministic reproduction. Manual provisioning diverges over time.

**The infrastructure change needs to be reviewable.** An IaC diff is a code review artefact. A manual console click is not. If infrastructure changes need audit trail, approval, and rollback, IaC is the correct tool.

**The infrastructure has lifecycle that outlasts any one person.** If the person who clicked the console buttons leaves, their intent is gone. IaC is self-documenting infrastructure.

**Does not add value when:** The infrastructure is truly one-off and will never be recreated, the team lacks IaC expertise and the learning cost exceeds the benefit, or the tooling overhead exceeds the scale of the infrastructure.

---

## Common IaC signals

### Drift
The actual infrastructure differs from what the IaC describes. Signals: `terraform plan` shows unexpected changes, manual changes were made in the console, multiple tools are managing the same resource.

**Ask:** Was this a deliberate manual change? If yes, import it into the IaC. If it was accidental, revert it. Drift is not a one-time fix — it recurs unless manual changes are eliminated.

### State file problems (Terraform-specific)
The state file is the source of truth for what Terraform manages. Signals: state conflicts in CI (two pipelines running `apply` simultaneously), state stored locally (not shared), state desync after a manual resource deletion.

**Hard rules:**
- Store state remotely (S3+DynamoDB, Terraform Cloud, etc.), never locally
- Lock state during apply to prevent concurrent modification
- Never manually edit the state file — use `terraform state` commands

### Module sprawl
Dozens of Terraform modules with deeply nested calls, duplicated variables, and unclear ownership.

**Signal:** A change to one environment requires updating 7 module files. Root cause is premature abstraction — modules were created before the pattern was stable.
**Ask:** Does this module exist because the same infrastructure is genuinely reused, or because someone thought it was "cleaner"? A module boundary should align with a reuse boundary or an ownership boundary, not a conceptual preference.

### Plan / apply separation not enforced
`terraform apply` runs without a reviewed `terraform plan`. A plan shows exactly what will change; skipping it is how unexpected resource deletions happen.

**Hard rule in CI:** `plan` runs on PR, `apply` runs only after merge (or explicit approval). Never auto-apply without a reviewed plan in production environments.

---

## IaC and secrets

IaC often needs to pass secrets (database passwords, API keys) to resources. The failure modes:

**Secrets in state files:** Terraform state can contain sensitive values in plaintext. If state is stored in S3, ensure the bucket is encrypted and access-controlled. Use `sensitive = true` for output values.

**Secrets in source code:** Never hardcode secrets in `.tf` files. Use variable references, and source the values from a secrets manager (AWS SSM Parameter Store, HashiCorp Vault, etc.) at apply time.

**Secrets in CI environment variables:** Acceptable for non-production, but ensure CI logs do not print secrets (`sensitive` flag in Terraform, masked variables in CI).

---

## The "infrastructure is just code" misunderstanding

IaC is reviewed like code, versioned like code, and tested like code — but it is not code. A bad code change produces a bug. A bad IaC change can delete a production database.

**Implications:**
- Test IaC changes in a non-production environment before applying to production
- Use `terraform plan` output in code review, not just the `.tf` diff
- Treat resource deletions and replacements in the plan as high-risk requiring explicit approval
- Know the blast radius of each `apply` — not all IaC changes are equal risk

**Signals of high-risk IaC changes:**
- `destroy` in the plan for any non-ephemeral resource
- `forces replacement` for a stateful resource (database, storage bucket)
- Changes to IAM policies or security group rules
- Changes to networking (VPC, subnets, routing)

These require more scrutiny than adding a new tag or updating an instance type.
