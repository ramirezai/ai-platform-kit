# Starterjourney Analytics — what is still missing

**To:** Platform engineering (Gonzalo) and stakeholders  
**From:** Cloud Architect, Workspace Advisor session  
**Date:** 21 September 2026  
**Project:** Starterjourney and Analytics Project  

This briefing is for implementation planning. **Architecture is approved. Nothing has been deployed.**

## Approved architecture (do not re-open)

| Item | Decision |
| --- | --- |
| Cloud / region | AWS `us-west-2` |
| Environments | Dev, staging, production |
| Compute | Hybrid/classic in a customer VPC |
| Security tier | **T9** (private UI/API + restricted firewall egress + KMS) |
| Front-end access | PrivateLink; users via VPN; no public workspace URL |
| Egress | Central hub/transit, shared inspection firewall |
| Keys | Customer-managed **AWS KMS** |
| Data / compliance | Regulated; **PCI-DSS, COPPA, kids’ data** |
| Git | Private repositories |
| Terraform in this kit | **Decision record only** — Gonzalo’s team owns deployment |

Working names: `env_stater_journey` family. Suggested prefix (not confirmed): `starter-journey`.

---

## Still missing — blocking network design

These items must be resolved **before** CIDRs are allocated or PrivateLink is built.

| # | Gap | Why it matters | Who should close it | Status |
| --- | --- | --- | --- | --- |
| 1 | **IPAM / landing-zone CIDRs** | Proposed `10.60.0.0/21`, `10.60.8.0/21`, `10.60.16.0/21` are **placeholders**. Overlap is unverified. | Gonzalo / IPAM | Open |
| 2 | **On-premises CIDRs** | On-prem connectivity is required. Unknown ranges cannot be routed or guaranteed non-overlapping. | Gonzalo / network | Open |
| 3 | **Hub / transit CIDRs** | T9 egress goes through a shared firewall. Spoke VPCs must not collide with hub, TGW, or inspection VPC. | Gonzalo / network | Open |
| 4 | **DNS private-zone rights** | PrivateLink fails if workspace, relay, and storage names resolve publicly. Who can create/delegate Route 53 private zones and forwarding is unknown. | Gonzalo’s team (owner named; rights unknown) | Open |
| 5 | **Deployer IAM** | Unknown whether platform engineering can create VPC, subnets, SGs, and VPC endpoints, or another cloud team must. | Gonzalo + cloud/IAM | Open |
| 6 | **Private Git path** | Restricted egress + private repos. Need PrivateLink to the Git provider **or** an internal mirror. | Gonzalo + source-control owners | Open |

### Recommended subnet shape (pending IPAM)

Per environment, **do not use `10.139.0.0/16`**:

- VPC **/21**
- Two compute subnets, different AZs, each **/23** (band **M**, ~128 classic nodes)
- Two dedicated PrivateLink subnets, each **/27**, **separate from compute**

---

## Still missing — identity, account, and governance

| # | Gap | Why it matters | Who should close it | Status |
| --- | --- | --- | --- | --- |
| 7 | **Databricks account** | First deployment; **no account exists yet**. | Gonzalo + AWS account admin | Open |
| 8 | **SSO / SAML and SCIM** | Required; not live today. Does not change T9, but blocks production user onboarding. | Identity + Gonzalo | Open |
| 9 | **Unity Catalog sign-off** | Recommended for production; not explicitly approved in the interview. | Gonzalo + data platform | Open |
| 10 | **Resource naming prefix** | Cloud names must be unique. Confirm a real prefix (not the docs placeholder). Candidate: `starter-journey`. | Cloud Architect + Gonzalo | Open |
| 11 | **SIEM / audit log sink** | No SIEM today. T9 production should still land audit and usage logs in a customer bucket/bus even if SIEM comes later. | Security (unnamed) + Gonzalo | Open |
| 12 | **Named security approver** | RACI needs an accountable security owner for tier, KMS, and production apply. | Joel Ramirez (CEO) to name | Open |

---

## Asked of leadership

| Ask | Owner |
| --- | --- |
| Approve this gap list and name a **security approver** | Joel Ramirez, CEO |
| Pull CIDRs, IPAM, DNS, and IAM facts from the landing zone | Gonzalo, platform engineering |
| Confirm Unity Catalog, prefix, and Git connectivity pattern | Cloud Architect + Gonzalo |
| Stand up Databricks account, SSO/SCIM, and KMS key policy | Platform + IAM |

---

## What this kit will not do

- **No Terraform apply** from the advisor session (`T1b = No`).
- Gonzalo’s team implements with their pipeline after gaps 1–7 are closed enough to plan.
- If the kit is asked to deploy later, that is a new decision: re-approve, then a **reviewed plan** — never a blind apply.

---

## One-line summary for Slack

Starterjourney Analytics is **approved T9 on AWS us-west-2** (three envs, hybrid VPC, PrivateLink, hub firewall, KMS). **Blockers:** IPAM/on-prem CIDRs, DNS rights, deployer IAM, private Git path, Databricks account, SSO/SCIM, prefix, security owner, UC sign-off. **No infra has been created.**
