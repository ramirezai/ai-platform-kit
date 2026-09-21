---
name: databricks-workspace-advisor
description: "Use when a customer does not yet know which Databricks workspace architecture they need, has conflicting requirements, wants help choosing a security tier (T0-T11), compute model, or network posture, or asks a specific workspace architecture question. Do not use when the customer says they know what to deploy or supplies a concrete architecture; route those requests directly to databricks-platform-provisioning."
---

# Databricks Workspace Advisor

Identify the workspace architecture the customer needs:

1. If the customer says they know what to deploy or supplies a concrete architecture, route directly to `databricks-platform-provisioning`.
2. Otherwise, run the Workspace Deployment Advisor and produce a Unified Decision Record.
3. If the customer wants a Terraform deployment, hand approved decisions to the existing platform skills without repeating settled questions.
4. If the customer does not want Terraform deployment, finish with the approved decision record and clearly state that no deployment was performed.

This skill decides **what to build**. It does not deploy workspaces. The other AI Platform Kit skills own **how to build and verify them with Terraform**.

## How to interact with the customer

**Pushback level: HIGH for deployment and remote mutations; MODERATE for architecture advice.**

- Ask one architecture question at a time. Never dump the entire question bank.
- Recommend the lowest security tier that satisfies actual requirements. Higher tiers add cost, complexity, and operational burden.
- Surface conflicting answers and ask one targeted follow-up. Never silently choose between incompatible requirements.
- Calibrate explanations to the respondent:
  - Executive or sponsor: risk, cost, ownership, and timeline.
  - Security or compliance lead: controls, mandates, and evidence.
  - Platform or network architect: CIDRs, endpoints, DNS, IAM, and implementation constraints.
- Suggest a safer or more maintainable choice once, with a brief reason. If the alternative remains safe and the customer insists, record the decision and proceed.
- Hard-stop on impossible combinations, missing critical identifiers, placeholder account IDs, or unapproved remote mutations.
- Never run `terraform apply` from the assessment. The implementation skill must present a plan and receive explicit approval.
- Always confirm destructive or access-removing actions such as `terraform destroy`, disabling public access, deleting a metastore, or changing private DNS.

## Resource naming convention

Use **`<prefix>`** only as a documentation placeholder for the customer-specific workspace or project identifier.

- Derive a unique prefix from the customer, project, or workspace name.
- Confirm it before provisioning.
- Never use `<prefix>` literally in HCL, shell commands, SQL, cloud resource names, or profiles.
- Never reuse a prefix where the cloud requires globally or account-unique names.

## Companion files

Load only the detail needed for the current phase:

- `question-bank.md` — exact questions, flow, conflicts, RACI, readiness checklist, and Unified Decision Record.
- `security-tiers.md` — deterministic T0–T11 algorithm, tier ladder, industry ranges, and worked examples.
- `networking-by-cloud.md` — gated network questions, subnet sizing, and AWS/Azure/GCP requirements.
- `terraform-and-dr.md` — customer-led Terraform maturity and disaster-recovery gates. Open only when triggered.

These files are authoritative for advisor detail. Do not use files outside this skill directory for the assessment.

## Workflow

### Step 1: Ask the architecture gate

If the customer says they already know what to deploy, or the opening request supplies a concrete architecture, skip this question and continue to Step 2. Do not require every architecture field before honoring the customer's statement that the design is already known.

Otherwise ask exactly:

> **Do you already know the workspace architecture you need — the security tier or isolation level, cloud, and networking posture? Or would it help to work that out first?**

Interpret the answer:

- Knows it or supplies a concrete specification → Step 2.
- Unsure, requests advice, or supplies contradictory requirements → Step 3.
- Asks one specific architecture question → answer it directly from the relevant companion file. Do not force a full assessment.

### Step 2: Known architecture

1. Stop the advisor assessment and read `../platform-provisioning/SKILL.md`.
2. Pass all stated decisions into its intake:
   - cloud and region
   - new or existing Databricks account
   - single workspace or multiple environments
   - POC or production
   - serverless or hybrid/classic compute
   - security tier and network posture
   - data sensitivity and compliance requirements
   - customer-managed key requirement
   - Unity Catalog, identity, and governance requirements
   - resource naming prefix and required tags
3. Ask only for information that remains unknown. Do not re-litigate a complete specification.
4. Follow the provisioning skill's auth checks, Terraform plan review, approval gate, apply, and mandatory verification.

### Step 3: Architecture assessment

Read `question-bank.md`, then follow its recommended flow:

1. Collect B1–B9 for business context, audience depth, ownership, and approval.
2. Lock C1, then ask C2 if needed, C4 (region), and C3 so all subsequent questions use the chosen cloud's vocabulary.
3. Collect E1–E2 for single vs multi-environment and POC vs production.
4. Ask T1a and T1b one at a time: whether the customer uses Terraform today and whether they want this kit to deploy the approved design with Terraform. Do not initiate the maturity deep-dive.
5. Keep DR silent unless the customer raises it.
6. Resolve S1–S5 for serverless versus hybrid/classic.
7. Walk N1–N5 and P1–P6.
8. Read `security-tiers.md`; calculate the tier from triggered floors, the T0 sandbox rule, and the CMK modifier.
9. If T3+, hybrid/classic, or an existing landing zone applies, read `networking-by-cloud.md` and ask the gated NI questions. Skip NI8 unless DR is already in scope.
10. Ask I1 for SSO/SCIM readiness. If T1b is Yes, confirm the resource `<prefix>` if still unknown.
11. Run the conflict checks from `question-bank.md`.

Hard behavior rules:

- **Terraform is customer-led.** Ask T1a and T1b by default. Read `terraform-and-dr.md` and ask T1–T3 only if the customer asks about modules, remote state, CI/CD, or multi-environment reuse.
- **DR is customer-led.** Never mention DR, regional outages, failover, RTO/RPO, or a second region unless the customer raises the topic. Then read `terraform-and-dr.md`.
- **Security tier is deterministic.** Do not eyeball it. Calculate all floors, choose the maximum, then apply the CMK modifier.
- **Cloud vocabulary is locked after C1.** Do not imply feature parity where none exists.

### Step 4: Present the decision record

Use the Unified Decision Record template in `question-bank.md`.

Include:

- customer/workspace and named owners
- whether the customer wants Terraform deployment after approval
- industry, cloud, region, and landing-zone constraints
- environment and compute model
- data classification
- every triggered tier floor and the CMK modifier
- final T0–T11 recommendation with driving question IDs
- CIDR, subnet plan, workload band, and overlap status when gated
- Unity Catalog, identity, logging, encryption, tagging, and `<prefix>`
- unresolved conflicts and readiness gaps

Include Terraform maturity only if the customer requested that deep-dive. Include DR only if the customer raised DR.

Ask the customer to correct or approve the record. If T1b is No, deliver the approved record and stop; do not load deployment skills.

### Step 5: Hand off without re-asking

After approval, only when T1b confirms that the customer wants Terraform deployment:

1. Read `../platform-provisioning/SKILL.md`.
2. Pre-fill its intake from the Unified Decision Record. Do not re-ask Round 2 network isolation, encryption, or compliance questions already settled by the record.
3. Treat T6+, N1=Yes, or an explicit private-connectivity mandate in the record as an explicit request for the matching private-networking pattern. Do not fall back to a public-front-end default.
4. Load only the implementation skills needed:
   - `../private-networking/SKILL.md` for Private Link, Private Endpoints, PSC, NCC, hub-spoke, DNS, or restricted egress.
   - `../unity-catalog-setup/SKILL.md` for metastores, catalogs, storage credentials, and external locations.
   - `../identity-governance/SKILL.md` for account groups, service principals, workspace assignments, and grants.
   - `../workspace-config/SKILL.md` for policies, warehouses, secrets, tokens, and IP access lists.
5. Ask only implementation details not settled by the record, such as the exact account ID or final `<prefix>`.
6. Follow the implementation skill's approval gate for every remote mutation.
7. Do not declare completion until `../deployment-verification/SKILL.md` has run every applicable required path.

If T1b is No, do not hand off to platform provisioning. State that the architecture decision is complete and no infrastructure was deployed.

## Examples

**Known architecture**

> Set up an AWS Databricks workspace with VPC injection, Unity Catalog, no PrivateLink, and a single production environment.

Skip the advisor and go directly to platform provisioning. Pre-fill its intake with the supplied architecture, gather only missing identifiers, then plan, approve, apply, and verify.

**Architecture advice required**

> A regulated-industry customer wants Databricks, but we have not selected the architecture.

Run the guided assessment one question at a time, calculate the tier, produce the decision record, and obtain approval. Hand it to platform provisioning only if the customer wants Terraform deployment.

**Reference lookup**

> What subnet size do I need for 200 classic nodes on AWS?

Answer directly from `networking-by-cloud.md`. Do not force the architecture gate or full assessment.

## Cross-links

- For Terraform generation, deployment, and auth checks, see **platform-provisioning**.
- For Private Link, Private Endpoints, PSC, NCC, DNS, and hub-spoke networking, see **private-networking**.
- For metastores, catalogs, storage credentials, and external locations, see **unity-catalog-setup**.
- For account groups, service principals, RBAC, and Unity Catalog grants, see **identity-governance**.
- For SQL warehouses, cluster policies, secrets, tokens, and workspace settings, see **workspace-config**.
- For mandatory post-deployment testing, see **deployment-verification**.
