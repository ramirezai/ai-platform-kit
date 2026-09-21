# Question bank, flow, conflicts, and decision record

This file is the authoritative assessment flow. Ask **one question at a time**.

Before asking anything, **harvest what the customer already provided** and mark those IDs as provisionally answered. Then ask only the open IDs — never re-ask a settled question. Maintain a running captured / open / conflicting ledger across phases.

**Accept batched answers.** If the customer volunteers several answers in one message, record all of them; one-question-at-a-time governs how you ask, not how much you accept.

**Explain when the customer is unsure.** Each technical question below carries an *If unsure* gloss — a plain-language explanation, the tradeoff, and the typical choice. Offer it whenever the customer answers "I don't know," asks what a term means, or seems confused. Never record "unknown" without first offering the explanation.

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
- **S2** (serverless): Do you need stable egress IPs to allowlist on downstream firewalls? (Private *reach* to your own resources is S4 — do not ask it here.)
  - *If unsure:* "Stable egress IPs" means outbound traffic leaves from a fixed set of addresses you can allow-list on other systems' firewalls. Needed when a partner or database restricts access by IP. If nothing downstream filters by IP, you don't need it. Selecting this moves serverless from T1 to T2 (adds an NCC). These are shared CIDR pools / service tags, not a single static IP.
- **S3** (hybrid/classic): Do you accept managing a customer VPC/VNet?
  - *If unsure:* Hybrid/classic runs compute inside a network you own and operate — more control, but your team maintains the VPC/VNet, subnets, and routing. If you don't want that operational burden, serverless avoids it. Required for hybrid/classic; drives T3+.
- **S4** (serverless): Must serverless privately reach a resource in your own cloud network or on-premises?
  - *If unsure:* Whether serverless jobs must connect privately (not over the public internet) to a database or service in your own network. If your data sources are all cloud-managed or internet-reachable, the answer is usually No. The status differs by target:
    - **Your cloud VPC/VNet resources:** NCC private endpoints — GA on AWS (PrivateLink+NLB) and Azure (private endpoint rules), Public Preview on GCP (outbound PSC).
    - **On-premises:** Preview — Azure via the Private Network Gateway (ExpressRoute/VPN); GCP via Google NCC + Cloud Interconnect/VPN; AWS via an NLB target that routes onward. Apply the Preview-explanation rule for the on-prem paths.
    Do not treat any of these as a hard blocker. (If on-prem reach is the need, N5 captures the on-prem requirement — reconcile with it rather than asking twice.)
- **S5** (S4=Yes): What is the exact target domain and internal IP, and is a load balancer (AWS NLB / Azure Standard LB / GCP internal LB or service attachment) already present?
  - *If unsure:* We need the hostname and internal IP of the resource serverless must reach, and whether a load balancer already fronts it (serverless private reach is published through an internal LB / service attachment). If you don't know, your network team will — flag it as a gap rather than guessing.
- **S6** (serverless and N1=Yes): Do you need the workspace UI/API reachable privately while staying serverless?
  - *If unsure:* Inbound (front-end) Private Link/PSC keeps the login/API off the public internet, and it is **independent of the compute model — so it works with a serverless-only workspace (GA)**. This lets serverless satisfy N1 without a customer-managed network. Caveats: advanced ingress controls are Beta (context-based ingress policies; private access via a custom "general access" URL), and full "no public access" of the compute-to-control-plane path additionally requires a classic compute-plane Private Link. Front-end privacy for users is GA (Enterprise/Premium plan).
- **S7** (serverless and N3=Restricted): Do you want restricted egress enforced on serverless by domain/FQDN?
  - *If unsure:* Serverless network policies (egress firewall) restrict outbound traffic to an allowlist of domains and storage destinations, with a dry-run mode to audit first. This is **GA** and lets serverless satisfy restricted egress without a customer-managed firewall. Limits: ~100 FQDNs and 100 storage destinations per policy.

For S4–S7, prefer the serverless path when it meets the need. Only recommend hybrid/classic when the requirement needs a GA guarantee that serverless cannot yet provide, or when another floor already requires a customer-managed network. See the serverless connectivity matrix in `networking-by-cloud.md`.

### Network and connectivity

- **N1:** Must users access the workspace UI and API over a private network only?
  - *If unsure:* This decides whether the workspace login/API is reachable from the public internet or only from your corporate network (VPN/private link). Private-only is more secure but adds DNS and connectivity setup. Most regulated customers require it; most others start public with IP allow-listing. Drives a T6 floor.
- **N2:** Does the organization require VPN/Private Link to other SaaS applications today?
  - *If unsure:* Asking whether you already connect to other SaaS tools over private links rather than the open internet — a sign your network team expects the same here. If they already do this elsewhere, the answer is usually Yes. Drives a T4 floor.
- **N3:** Does the workspace need outbound internet? (Yes / Restricted-only / No)
  - *If unsure:* Whether workloads can reach the internet for package installs, APIs, and data sources. "Yes" is simplest; "Restricted-only" routes egress through a firewall (T8); "No" means fully isolated (T10). Most production picks Yes or Restricted-only.
- **N4:** Is full network isolation mandated (air-gapped, sovereign, or classified)?
  - *If unsure:* "Air-gapped" means no internet path at all; usually driven by a government, defense, or sovereignty mandate. If no regulator or contract requires it, the answer is No. Forces T10 and overrides other network choices.
- **N5:** Is on-premises connectivity required?
  - *If unsure:* Whether the workspace must reach systems in your own data center (databases, file shares) over VPN or a dedicated link. Yes if data or apps still live on-prem. If Yes, we will also need your on-premises CIDR ranges (NI6).

### Security posture

- **P1:** Must the workspace reach private source-code repositories?
  - *If unsure:* Whether notebooks/jobs pull code from an internal Git server that isn't on the public internet (e.g. self-hosted GitHub/GitLab). Yes if your repos are behind the firewall. Interacts with the egress policy (N3).
- **P2:** What data sensitivity will it process? (Public / Confidential / Regulated / Classified)
  - *If unsure:* Public = open/marketing data; Confidential = internal business data; Regulated = data under a law or contract (PHI, cardholder, PII); Classified = government-classified. Pick the highest class any data in the workspace falls under. This is the biggest tier driver: Public→T1, Confidential→T3, Regulated→T6, Classified→T10.
- **P3:** Are customer-managed encryption keys required?
  - *If unsure:* By default Databricks encrypts data with keys it manages. Customer-managed keys (CMK) let you own and revoke the key yourself — usually required for sovereignty or strict compliance, at the cost of extra key-management setup. If no policy demands it, default keys are fine. CMK is a modifier (e.g. T6→T7), not its own tier.
- **P4:** Are IP access lists needed in addition to or instead of private connectivity?
  - *If unsure:* IP allow-lists restrict workspace access to known office/VPN IP ranges. They're a lighter-weight control than private networking and often used together with it, or on their own for public workspaces. Most customers want at least this.
- **P5:** Must audit or billable-usage logs be delivered to the customer's SIEM?
  - *If unsure:* Whether your security team needs Databricks audit and cost logs streamed into their monitoring system (Splunk, Sentinel, etc.). Common in regulated orgs; creates a logging workstream but doesn't change the tier.
- **P6:** Which compliance frameworks or residency requirements apply? (HIPAA / PCI-DSS / FedRAMP / SOC 2 / data residency / None)
  - *If unsure:* Name any regulation or contract that governs this data or where it may physically reside. HIPAA (health), PCI-DSS (payment cards), FedRAMP (US gov), SOC 2 (general security attestation), data residency (must stay in a region/country). If none applies, answer None. Should line up with the data class in P2.

### Identity

- **I1:** Are SSO/SAML and SCIM provisioning configured today?
  - *If unsure:* SSO/SAML lets users log in with your corporate identity provider (Okta, Entra ID); SCIM auto-syncs users and groups so access is granted and revoked centrally. "Configured today" means already set up for other apps. Most enterprises have this; if not, it becomes a setup task.

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

0. Harvest everything already provided, mark those IDs captured, reflect them back for confirmation, and skip them wherever they appear below. Ask only open IDs.
1. Collect B1–B9. Use B7 for depth and B8–B9 for ownership.
2. Collect C1, C2 if needed, C4, then C3. Lock cloud vocabulary after C1.
3. Collect E1–E2 for environment strategy.
4. Ask T1a and T1b one at a time. Do not initiate the Terraform maturity assessment.
5. Skip DR entirely unless the customer raised it.
6. Ask S1. For serverless, ask S2, then S4 (and S5 when S4=Yes). For hybrid/classic, confirm S3. Prefer serverless where it meets the need; GCP private reach is Public Preview, not a hard conflict.
7. Ask N1–N5 and P1–P6. If serverless is the path, ask S6 when N1=Yes and S7 when N3=Restricted, and confirm whether a serverless connectivity feature satisfies N1, N3, or private resource reach before concluding a customer-managed network is required.
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
| Serverless private reach on GCP | GCP serverless-to-VPC private reach is Public Preview (outbound PSC): confirm GA/region and account-preview enablement, or use hybrid/classic for a GA path today. Not a hard conflict. | S4, C1 |
| Terraform Level 0 but requests modular multi-environment IaC | Start with a template and grow into modules, or bring support to build modules from day one? | T1, T3 |
| Compliance framework differs from industry expectation | Is the requirement driven by a contract, subsidiary, or specific data type? | B2, P6 |
| DR considered but no approved RTO/RPO | Define business-approved objectives before selecting a strategy. | DR1, DR2 |
| On-premises required but CIDR unknown | Obtain the range before finalizing CIDR and routing. | N5, NI6 |
| Small CIDR conflicts with workload band | Size up now, or confirm the lower long-term capacity ceiling? | NI1, NI3 |
| POC/no-IaC implies T0 but a higher floor fired | Keep the higher tier and use IaC, or confirm the workload is truly a non-sensitive sandbox? | E2, T1a, T1b, P2 |
| Hardened production depends on a Preview/Beta serverless feature | Confirm GA and regional availability with the account team; accept the Preview risk, or use the classic/private GA path for that control. | E2, S4, S6, S7 |

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
| Compute: Serverless / Serverless+NCC / Hybrid (S1–S7) | |
| Serverless connectivity: private egress, egress firewall, front-end PL (S4–S7) | |
| Preview/Beta features relied on, and GA/region confirmation status | |
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
