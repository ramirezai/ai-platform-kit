# Security tiers (T0–T11)

Load this file for the tier algorithm, descriptive ladder, industry ranges, or entry lens.

Select the **lowest** tier that satisfies the customer's actual regulatory, data-sensitivity, and risk requirements. Higher is not automatically better; each added control carries cost, build complexity, and operational overhead.

## Determination algorithm

**Final tier = maximum of every triggered floor, then apply the CMK modifier.**

Tiers are **cumulative bundles**: each tier includes every control below it. A single high floor therefore pulls in all lower controls — for example, N3 Restricted-only egress (T8) also brings T6 front-end private connectivity and T4/T5 private storage and back-end paths, even if those were not separately requested. When one floor drives the tier well above the others, say so in the decision record rationale so the customer understands the added scope and cost, and confirm the driving requirement is real rather than assumed.

- **P2 data sensitivity:** Public → T1, Confidential → T3, Regulated → T6, Classified → T10.
- **N4 isolation mandate:** T10. Overrides N3 and applies regardless of other answers.
- **N1 front-end private access:** T6.
- **N2 existing SaaS private-connectivity pattern:** T4.
- **N3 egress policy:** Restricted-only → T8; No outbound internet → T10; Yes → no floor. N3=No and N4 both reach T10; N4 is the mandate/attestation layer on top of the same control.
- **T0 no-IaC sandbox:** Apply T0 only after computing the other floors, and only when all of these are true:
  - E2 is POC/evaluation, not production
  - T1a is No (not using Terraform today)
  - T1b is No (this kit will not deploy with Terraform)
  - S1 is serverless, not hybrid/classic — a customer-managed network cannot be T0
  - No floor above T1 fired (N1, N2, N3 restricted/no-internet, N4, Confidential/Regulated/Classified, S3 hybrid/classic)
  - The Public → T1 sensitivity floor does not block T0 when the conditions above hold
  - If any floor T3+ fired, or N1/N2/N3-restricted/N4/S3-hybrid fired, do not recommend T0. Keep the higher tier and treat IaC as required.
- **P3 customer-managed keys:** modifier, not a floor:
  - T6 → T7
  - T8 → T9
  - T10 → T11
  - At T0/T1/T2/T3/T4/T5, add CMK as an encryption control without changing the tier number.
- **S3 hybrid/classic (customer-managed network):** T3 floor. Hybrid/classic requires a customer-managed VPC/VNet and IaC, so it can never be T0–T2. This floor blocks the T0 sandbox rule even for a POC.
- **Serverless (S1 serverless):** T1 baseline; S2 stable egress or S4 private serverless connectivity selects T2. Serverless is the preferred implementation whenever it can meet the required posture. **The tier number reflects the security posture (the floors); the compute model is the implementation choice under it.** A floor above T3 no longer automatically forces hybrid/classic — serverless can satisfy several higher-tier controls without a customer-managed network: private egress to customer resources (NCC private endpoints — GA on AWS/Azure, Public Preview on GCP), restricted/firewalled egress (serverless network policies — GA), and private front-end access (inbound/front-end Private Link — GA and compute-agnostic, so it applies to serverless-only workspaces). Set the tier from the floors, then choose serverless as the implementation when its connectivity features meet that posture, stating the Preview/Beta caveat where one applies (GCP private egress, on-prem gateways, and advanced ingress controls are Preview/Beta; full "no public access" of the compute path also needs a classic compute-plane Private Link). Move to hybrid/classic only when the posture needs a GA guarantee serverless cannot yet provide, when compute must run in the customer tenant, or when strict residency requires it.

Do not let a directional industry range override explicit data classification, contractual obligations, or security policy.

## Worked examples

Regulated + private front end + CMK:

```text
Recommended Security Tier: T7
Triggered floors (final tier is the maximum):
  - P2 data sensitivity: T6
  - N1 front-end private access: T6
CMK modifier (P3=Yes): T6 -> T7.
```

Isolation mandate + CMK:

```text
Recommended Security Tier: T11
Triggered floors (final tier is the maximum):
  - P2 data sensitivity: T3
  - N4 network isolation mandate: T10
CMK modifier (P3=Yes): T10 -> T11.
```

Serverless requested and a regulated-data floor fired:

```text
Recommended Security Tier: T6 (posture)
Triggered floors (final tier is the maximum):
  - P2 data sensitivity: T6
Implementation: serverless can meet a T6 posture — inbound/front-end Private
Link (GA, compute-agnostic) for private UI/API, plus NCC private endpoints
(GA on AWS/Azure) for private egress. Note that advanced ingress controls
(context-based ingress, custom-URL access) are Beta, and full "no public access"
of the compute path also needs a classic compute-plane Private Link. Confirm
regional availability with the account team.
```

POC with no Terraform and no higher floor:

```text
Recommended Security Tier: T0
Triggered floors (final tier is the maximum):
  - E2 POC + T1a No + T1b No, serverless, and no T1+ floor: T0
CMK modifier (P3=Yes): record CMK as an encryption control; tier stays T0.
```

Hybrid/classic POC — the customer-managed network blocks T0:

```text
Recommended Security Tier: T3
Triggered floors (final tier is the maximum):
  - P2 data sensitivity (Public): T1
  - S3 hybrid/classic customer-managed network: T3
T0 does not apply: hybrid/classic requires a customer-managed network and IaC.
```

No outbound internet without a formal isolation mandate:

```text
Recommended Security Tier: T10
Triggered floors (final tier is the maximum):
  - P2 data sensitivity (Confidential): T3
  - N3 egress policy (No outbound internet): T10
N4 was not asserted, but N3=No reaches the same T10 control.
```

## Tier ladder

| Tier | Architecture | Added control | Typical use |
| --- | --- | --- | --- |
| T0 | Manual / UI or managed locked network | No IaC | POC, sandbox, no Terraform capability |
| T1 | Serverless | Managed compute-plane networking | Fast time-to-value, non-sensitive data |
| T2 | Serverless + NCC | Stable egress or private connectivity | Serverless needing controlled connectivity |
| T3 | Customer-managed network | SCC/no-public-IP compute | Standard enterprise production |
| T4 | T3 + private storage connectivity | No public storage exposure | Sensitive data at rest |
| T5 | T4 + back-end private connectivity | Private compute-to-control-plane path | Regulated workloads |
| T6 | T5 + front-end private connectivity | Private workspace UI/API | No-public-ingress environments |
| T7 | T6 + CMK | Customer-owned encryption keys | Sovereignty/key-ownership requirements |
| T8 | T6 + egress firewall | Restricted outbound access | Controlled production egress |
| T9 | T8 + CMK | Restricted egress plus key ownership | Regulated production |
| T10 | T6 + no internet | Full isolation | Classified/sovereign workloads |
| T11 | T10 + CMK | Full isolation plus key ownership | Maximum-control environments |

The **Architecture** column describes the classic / customer-managed *reference* implementation of each posture. The tier number is the security posture; classic and serverless are two implementations of it. Serverless meets several of these postures by other means without a customer-managed network — inbound (front-end) Private Link for the front-end control (T6), NCC private endpoints for private storage/resource connectivity (T4), and serverless network policies for restricted egress (T8). The T3–T5 rungs that describe the customer *compute plane* (SCC/no-public-IP, the back-end compute-to-control-plane path) do not apply to serverless, where Databricks manages the compute plane. Postures that require compute in the customer tenant, or full air-gap / no-internet isolation (T10/T11), remain classic/isolated — serverless does not meet those.

## Industry alignment

Directional only:

| Industry | Typical range | Notes |
| --- | --- | --- |
| Financial Services | T7–T11 | PCI-DSS/SOX; private connectivity and CMK are common |
| Healthcare / Life Sciences | T6–T11 | PHI commonly drives private connectivity and CMK |
| Public Sector | T9–T11 | FedRAMP, sovereign, and isolation mandates |
| Manufacturing & Energy | T3–T9 | OT/IT separation and controlled egress |
| Retail & Consumer Goods | T2–T5 | PCI subsets; otherwise velocity and baseline controls |
| Media / Communications / Games | T1–T4 | Elasticity and time-to-value |
| Digital Natives | T0–T3 | Cloud-native; posture follows data sensitivity |

## Entry lens

- **Serverless → T1–T2 baseline, higher postures via NCC:** fastest to launch and no infrastructure to manage. With NCC private endpoints, serverless network policies, and inbound (front-end) Private Link (GA and compute-agnostic), it can meet many private-connectivity and restricted-egress requirements without a customer-managed network — prefer it when it meets the posture, and see the serverless connectivity matrix in `networking-by-cloud.md`. It may still not satisfy compute-in-tenant or strict residency requirements.
- **Customer-managed network + SCC → T3:** common first-production landing point; retains a path to private connectivity and controlled egress.
- **Customer-managed network + private connectivity → T4–T6:** higher control with greater deployment and operating complexity.
- **CMK additions → T7/T9/T11:** use the modifier logic above. At T0–T5, record CMK without changing the tier number.
- **Managed locked network → T0:** only for POC/sandbox with no Terraform and no higher floor. Cannot be retrofitted to customer-managed networking without redeploying.
