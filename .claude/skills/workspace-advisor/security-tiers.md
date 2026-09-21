# Security tiers (T0–T11)

Load this file for the tier algorithm, descriptive ladder, industry ranges, or entry lens.

Select the **lowest** tier that satisfies the customer's actual regulatory, data-sensitivity, and risk requirements. Higher is not automatically better; each added control carries cost, build complexity, and operational overhead.

## Determination algorithm

**Final tier = maximum of every triggered floor, then apply the CMK modifier.**

- **P2 data sensitivity:** Public → T1, Confidential → T3, Regulated → T6, Classified → T10.
- **N4 isolation mandate:** T10. Overrides N3 and applies regardless of other answers.
- **N1 front-end private access:** T6.
- **N2 existing SaaS private-connectivity pattern:** T4.
- **N3 Restricted-only egress:** T8.
- **T0 no-IaC sandbox:** Apply T0 only after computing the other floors, and only when all of these are true:
  - E2 is POC/evaluation, not production
  - T1a is No (not using Terraform today)
  - T1b is No (this kit will not deploy with Terraform)
  - No floor above T1 fired (N1, N2, N3 restricted/no-internet, N4, Confidential/Regulated/Classified)
  - The Public → T1 sensitivity floor does not block T0 when the conditions above hold
  - If any floor T3+ fired, or N1/N2/N3-restricted/N4 fired, do not recommend T0. Keep the higher tier and treat IaC as required.
- **P3 customer-managed keys:** modifier, not a floor:
  - T6 → T7
  - T8 → T9
  - T10 → T11
  - At T0/T1/T2/T3/T4/T5, add CMK as an encryption control without changing the tier number.
- **S1–S3 serverless:** resolves to T1–T2 only when no floor above T3 fired. If a higher floor exists, surface a conflict and reconcile it with the customer. S2 stable egress or private serverless connectivity selects T2 rather than T1 when the serverless path is valid.

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

Serverless requested but a regulated-data floor fired:

```text
Recommended Security Tier: T6
Triggered floors (final tier is the maximum):
  - P2 data sensitivity: T6
CONFLICT: serverless was requested but a floor above T3 applies.
Do not default to serverless; reconcile the mismatch with the customer.
```

POC with no Terraform and no higher floor:

```text
Recommended Security Tier: T0
Triggered floors (final tier is the maximum):
  - E2 POC + T1a No + T1b No, and no T1+ floor: T0
CMK modifier (P3=Yes): record CMK as an encryption control; tier stays T0.
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

- **Serverless → T1–T2:** fastest to launch and no infrastructure to manage, but may not satisfy compute-in-tenant or strict residency requirements.
- **Customer-managed network + SCC → T3:** common first-production landing point; retains a path to private connectivity and controlled egress.
- **Customer-managed network + private connectivity → T4–T6:** higher control with greater deployment and operating complexity.
- **CMK additions → T7/T9/T11:** use the modifier logic above. At T0–T5, record CMK without changing the tier number.
- **Managed locked network → T0:** only for POC/sandbox with no Terraform and no higher floor. Cannot be retrofitted to customer-managed networking without redeploying.
