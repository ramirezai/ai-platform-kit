# Question bank, flow, conflicts, and decision record

This file is the authoritative assessment flow. Ask **one question at a time**.

Terraform T1–T3 and DR1–DR5 are customer-led. By default ask T1a and T1b one at a time and never raise DR. See `terraform-and-dr.md` only after the corresponding trigger.

## Default question bank

### Business context

- **B1:** What is your company or project name?
- **B2:** What industry best describes your organization? (FINS / HLS / CMEG / MFG / RCG / PubSec / DNB)
- **B3:** Which subvertical applies?
- **B4:** How many people or teams will use this workspace?
- **B5:** Is this greenfield, an on-premises migration, or a cloud-platform migration?
- **B6:** Do you already have Databricks workspaces deployed? If yes, how are they deployed and managed?
- **B7:** What is your role/title? (Executive/Sponsor / Security or Compliance / Platform or Infrastructure / Data or Platform Engineer / Cloud or Network Architect / Other)
- **B8:** Who are the key leaders or stakeholders sponsoring and approving this initiative?
- **B9:** Will the respondent execute Terraform and own the network build? If not, who is the technical owner?

B7 changes explanation depth, not the recommendation. B8–B9 populate named ownership.

### Cloud platform

- **C1:** Which cloud provider will host this workspace? (AWS / Azure / GCP / Multiple)
- **C2** (multiple): Are these independent deployments or a unified multi-cloud strategy?
- **C4:** Which cloud region should host this workspace?
- **C3:** Must the workspace fit into an existing landing zone?

Lock C1 before downstream network/security questions and use only that cloud's vocabulary. Ask C4 after C1 (and C2 when C1 is Multiple).

### Environment strategy

- **E1:** Is this a single workspace, or do you need separate environments such as dev, staging, and production?
- **E2:** Is this a proof-of-concept or evaluation, or a production workspace that must be hardened?

### Terraform baseline

- **T1a:** Are you using Terraform to deploy infrastructure today?
- **T1b:** After the architecture is approved, do you want this kit to deploy it with Terraform, or do you only need the architecture decision record?

Ask T1a and T1b one at a time. T1b controls the implementation handoff: Yes routes the approved record to `platform-provisioning`; No ends the workflow after record approval with no deployment. Do not ask T1–T3 unless the customer requests Terraform-specific guidance.

### Compute model

- **S1:** Would you like a serverless workspace or a hybrid workspace?
  - Azure: say **serverless or hybrid**.
  - AWS/GCP: say **serverless or classic**, then explain classic is hybrid: classic clusters run in the customer's VPC/VNet and serverless remains available.
  - Serverless means managed compute/networking and no customer VPC/VNet for classic clusters.
  - Hybrid means customer-managed networking plus classic and serverless compute.
- **S2** (serverless): Do you need stable egress IPs or private connectivity for serverless?
- **S3** (hybrid/classic): Do you accept managing a customer VPC/VNet?
- **S4** (serverless): Must serverless privately reach a resource in a peered customer network or on-premises?
- **S5** (S4=Yes on AWS/Azure): What is the exact target domain and internal IP, and is an AWS Network Load Balancer or Azure Standard Load Balancer already present?

If S4=Yes on GCP, stop and raise the hard conflict: GCP serverless cannot privately reach the customer's own VPC or on-premises network.

### Network and connectivity

- **N1:** Must users access the workspace UI and API over a private network only?
- **N2:** Does the organization require VPN/Private Link to other SaaS applications today?
- **N3:** Does the workspace need outbound internet? (Yes / Restricted-only / No)
- **N4:** Is full network isolation mandated (air-gapped, sovereign, or classified)?
- **N5:** Is on-premises connectivity required?

### Security posture

- **P1:** Must the workspace reach private source-code repositories?
- **P2:** What data sensitivity will it process? (Public / Confidential / Regulated / Classified)
- **P3:** Are customer-managed encryption keys required?
- **P4:** Are IP access lists needed in addition to or instead of private connectivity?
- **P5:** Must audit or billable-usage logs be delivered to the customer's SIEM?
- **P6:** Which compliance frameworks or residency requirements apply? (HIPAA / PCI-DSS / FedRAMP / SOC 2 / data residency / None)

### Identity

- **I1:** Are SSO/SAML and SCIM provisioning configured today?

I1 creates an identity workstream but does not change the tier.

### Gated network detail

When T3+, hybrid/classic, or C3=Yes applies, read `networking-by-cloud.md` and ask the gated NI questions. Do not dump NI1–NI12. Skip NI1 for serverless-only. Ask NI6 only if N5=Yes. Ask NI8 only if DR is already in scope. Ask NI11 only at T4+.

### Customer-led Terraform

Only after the customer raises Terraform-specific topics, read `terraform-and-dr.md` and ask:

- **T1:** Does the organization have experienced Terraform engineers?
- **T2:** Is remote state with locking and CI/CD available?
- **T3:** Must configuration be reusable across environments or business units?

### Customer-led disaster recovery

Only after the customer raises DR, read `terraform-and-dr.md` and ask:

- **DR1:** Would a multi-hour/day regional outage cause unacceptable impact?
- **DR2:** Are business-approved RTO and RPO documented?
- **DR3:** Is cross-region recovery contractually or regulatorily required?
- **DR4:** Is near-zero RTO required, with dual-run capacity?
- **DR5:** Can the organization fund, operate, and regularly test a second workspace?

## Recommended flow

1. Collect B1–B9. Use B7 for depth and B8–B9 for ownership.
2. Collect C1, C2 if needed, C4, then C3. Lock cloud vocabulary after C1.
3. Collect E1–E2 for environment strategy.
4. Ask T1a and T1b one at a time. Do not initiate the Terraform maturity assessment.
5. Skip DR entirely unless the customer raised it.
6. Ask S1. For serverless, immediately ask S4; on GCP, raise the hard conflict if S4=Yes. Then ask S2 and S5 as applicable. For hybrid/classic, confirm S3.
7. Ask N1–N5 and P1–P6.
8. Calculate the tier using `security-tiers.md`.
9. For T3+, hybrid/classic, or an existing landing zone, use `networking-by-cloud.md` for the gated NI questions. Never ask NI8 unless DR is already in scope.
10. Ask I1. If T1b is Yes, ask for the resource `<prefix>` if it is still unknown.
11. Run all conflict checks below.
12. Present the Unified Decision Record and obtain corrections or approval.

Customer-led detours:

- If the customer asks about Terraform implementation maturity, insert T1–T3, classify maturity, then return to the default flow.
- If the customer raises DR, insert the DR gate. If it clears, capture DR4–DR5 and NI8, then return to the default flow.

## Conflict checks

Treat answers as provisional. Surface a conflict and ask one targeted follow-up.

| Conflict | Clarifying question | IDs |
| --- | --- | --- |
| Declines serverless and customer-managed networking | A hybrid/classic workspace requires a customer-managed network. Reconsider serverless, or scope VPC/VNet ownership? | S1, S3 |
| No internet but needs private Git with no private path | Private Link to the Git provider, an internal mirror, or reconsider outbound policy? | N3, P1, N2 |
| Industry implies high tier but data is Public | Can you confirm the actual data classification and contractual requirements? | B2/B3, P2 |
| Full isolation but serverless with no NCC | Use serverless+NCC with no external access, or classic with no-internet networking? | N4, S1, S2 |
| Serverless private reach on GCP | Use hybrid/classic, or an explicitly approved public endpoint with IP allow-listing? | S4, C1 |
| Terraform Level 0 but requests modular multi-environment IaC | Start with a template and grow into modules, or bring support to build modules from day one? | T1, T3 |
| Compliance framework differs from industry expectation | Is the requirement driven by a contract, subsidiary, or specific data type? | B2, P6 |
| DR considered but no approved RTO/RPO | Define business-approved objectives before selecting a strategy. | DR1, DR2 |
| On-premises required but CIDR unknown | Obtain the range before finalizing CIDR and routing. | N5, NI6 |
| Small CIDR conflicts with workload band | Size up now, or confirm the lower long-term capacity ceiling? | NI1, NI3 |
| POC/no-IaC implies T0 but a higher floor fired | Keep the higher tier and use IaC, or confirm the workload is truly a non-sensitive sandbox? | E2, T1a, T1b, P2 |

If the customer disagrees after the recommendation, reopen only the related IDs rather than restarting the assessment.

## RACI

R = Responsible, A = Accountable, C = Consulted, I = Informed.

| Activity | Platform Eng. | Security | Cloud/IAM Ops | Sponsor/CDO |
| --- | --- | --- | --- | --- |
| Define tier, CMK, compliance, network posture | C | A/R | C | I |
| Own Terraform modules and repository | R | C | A | I |
| Approve production applies | I | A/R | C | I |
| Manage cloud identity/IAM | C | C | A/R | I |
| Own state backend and secrets vault | I | C | A/R | I |
| Fund and prioritize initiative | C | C | I | A/R |
| Qualify and own DR | C | A/R | C | A |

Replace role labels with the named people captured in B8–B9.

## Readiness checklist

Before committing to a deployment date, identify gaps in:

- named platform/infrastructure owner
- security representative in review and approval
- executive sponsor
- protected Git repository and mandatory review
- documented production change-management process
- remote state with locking
- CI/CD pipeline
- short-lived/federated pipeline credentials
- secrets vault
- shared/versioned module strategy where required
- approved security tier
- naming, tagging, landing-zone, IPAM, and DNS standards

## Unified Decision Record

Create one record per workspace:

| Field | Entry |
| --- | --- |
| Company / workspace name | |
| Primary contact and role (B7) | |
| Sponsors, approver, technical owner (B8, B9) | |
| Industry / subvertical (B2, B3) | |
| Cloud / region (C1, C4) | |
| Multi-cloud strategy (C2, if asked) | |
| Existing landing zone (C3) | |
| Environment strategy (E1, E2) | |
| Using Terraform today (T1a) | |
| Terraform deployment requested (T1b) | |
| Compute: Serverless / Serverless+NCC / Hybrid (S1–S3) | |
| Data sensitivity (P2) | |
| Triggered tier floors and CMK modifier | |
| Recommended T0–T11 tier | |
| Key rationale with driving question IDs | |
| Network posture | |
| CIDR, subnet plan, NI1 band/nodes, overlap result | |
| Unity Catalog requirements | |
| Identity / SSO / SCIM requirements (I1) | |
| Logging, encryption, compliance, and tags | |
| Confirmed resource `<prefix>` | |
| Outstanding readiness gaps and conflicts | |
| Named owners (RACI) | |
| Approver and date | |

Optional addenda, only when customer-led:

| Field | Entry |
| --- | --- |
| Terraform maturity Level 0–4 (T1–T3) | |
| Local or Git-controlled deployment | |
| Template or modularized resource strategy | |
| DR gate: Qualifies / Does not qualify | |
| DR strategy, RTO/RPO, secondary CIDR | |

Ask the customer to correct or explicitly approve the record. Hand it to implementation only when T1b is Yes.
