# Terraform and disaster recovery (customer-led)

Open this file **only when the customer raises these topics**.

By default:

- Ask T1a: "Are you using Terraform to deploy infrastructure today?"
- Then ask T1b: "After the architecture is approved, do you want this kit to deploy it with Terraform, or do you only need the architecture decision record?"
- Use T1b to control handoff: Yes routes to platform provisioning after approval; No ends after the approved decision record without deployment.
- Never mention disaster recovery, regional outages, failover, RTO/RPO, or a second region.

## Terraform deep-dive

Trigger: the customer asks about module structure, remote state, CI/CD for infrastructure, or multi-environment reuse.

Then ask:

- **T1:** Does your organization have engineers experienced with Terraform?
- **T2:** Do you have a remote Terraform state backend with locking and a CI/CD pipeline?
- **T3:** Do you need this configuration reusable across environments or business units?

## Maturity model

| Level | Characteristics | Recommended path |
| --- | --- | --- |
| 0 — No Terraform | Manual UI/ad hoc scripts; no IaC | Build capability first or co-build the first modules |
| 1 — Ad hoc | Inconsistent use; local state; no review | Guided local deployment; establish remote state and a repository |
| 2 — Standardizing | Remote state; basic CI; named platform team | Git-controlled deployment; adapt a modular template |
| 3 — Managed practice | State locking; enforced reviewed plan/apply; secrets vault | Modular Terraform and policy-as-code gates |
| 4 — Optimizing | Shared versioned modules; drift detection | Internal module registry and self-service pipelines |

### Deployment strategy

Local Terraform is acceptable only for sandbox or learning work.

Anything with real data, tier T3+, or DR in scope should deploy through a Git-controlled pipeline with remote state and reviewed applies. Require this from Level 2 onward.

### Resource strategy

- **Pre-built template:** fastest path; best for a single workspace at Level 0–1.
- **Modularized Terraform:** maintainable across environments and landing zones; best for multiple or regulated workspaces at Level 2+.
- Existing landing zone (C3=Yes) strongly favors a modularized approach.

The `platform-provisioning` skill owns actual Terraform generation and deployment.

## Disaster recovery gate

Trigger: the customer raises DR, business continuity, regional outages, cross-region failover, or RTO/RPO.

Databricks provides in-region high availability. This gate decides whether **cross-region DR** is warranted.

Ask:

- **DR1:** Would a multi-hour/day regional outage cause unacceptable business, safety, or regulatory impact?
  - If No, stop and record: "Cross-region DR not warranted; in-region HA is sufficient."
- **DR2:** Do you have a documented, business-approved RTO and RPO?
- **DR3:** Is there a regulatory or contractual mandate for cross-region recovery?
- **DR4:** Is RTO near-zero, with the capacity and budget to run production in two regions concurrently?
- **DR5:** Can the organization fund and operate a second workspace and test failover regularly?
- **NI8** (when a customer-managed network is in scope): What non-overlapping CIDR will the secondary region use?

DR qualifies only when all are true:

- The workload is production-critical.
- A documented, business-approved RTO and RPO exist.
- At least one strong trigger exists: regulatory/contractual mandate or unacceptable revenue, safety, or business impact.
- The organization will fund and operate a second workspace and test failover.

Otherwise recommend in-region HA and record that cross-region DR was considered and intentionally not recommended.

## DR outcomes

Qualifies:

```text
DR gate outcome: QUALIFIES
Cross-region DR is warranted.
The secondary region must mirror the primary security tier.
Default to active-passive unless RTO is near-zero and dual-run budget exists.
Consider managed DR when operating replication and failover is the concern.
```

Does not qualify:

```text
DR gate outcome: DOES NOT QUALIFY
Default to in-region HA.
Record that cross-region DR was considered and intentionally not recommended.
List the blocking reasons.
```

## Strategy after the gate clears

- **Active-passive:** default; one active workspace and a synchronized idle secondary.
- **Active-active:** only for near-zero RTO with dual-run budget and operational maturity.
- **Managed DR:** raise when the customer qualifies but is concerned about operating replication and failover. This is a gated add-on; engage the account team.

The secondary region must:

- mirror the primary workspace's security tier
- use a non-overlapping CIDR
- receive equivalent identity, governance, networking, and data-replication controls
- participate in recurring failover tests
