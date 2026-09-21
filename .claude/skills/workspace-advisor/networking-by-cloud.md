# Networking and permissions by cloud

Load this file when the assessment involves a customer-managed network (tier T3+), subnet sizing, private connectivity, egress, CMK, or deployer IAM.

Always use the customer's cloud vocabulary. Never assume parity.

Every cloud consumes approximately **2 IPs per classic compute node**. Serverless compute in a hybrid workspace does not consume customer-subnet addresses.

## Gated network questions

Run the gated NI questions when the tier is T3+, hybrid/classic is selected, or C3 indicates an existing landing zone. Ask only the applicable IDs, one at a time. Never dump NI1–NI12.

Skip gates:

- **NI1:** skip for serverless-only.
- **NI6:** ask only if N5=Yes.
- **NI8:** ask only if the customer already raised DR. Do not mention a second region or secondary CIDR otherwise.
- **NI11:** ask only at T4+.

- **NI1:** Roughly how large is classic compute at a busy time? Ask for S/M/L/XL, not an exact node count unless the customer already has one.
- **NI2:** Do you have a VPC/VNet CIDR and subnet plan, or do you need a recommendation?
- **NI3:** Explain that subnet size is a hard compute ceiling and can be disruptive to change. What VPC/VNet CIDR and subnet sizes should be used?
- **NI4:** What literal CIDR should be used, or should one be proposed?
- **NI5:** Which cloud, hub/transit, and on-premises ranges must not overlap? Is there an IPAM registry?
- **NI6** (N5=Yes): What are the on-premises CIDRs?
- **NI7:** Is this the only workspace, or should ranges be reserved for dev/staging/prod or other workspaces?
- **NI8** (DR in scope): What non-overlapping CIDR will the secondary region use?
- **NI9:** Central hub/transit with shared egress firewall, or standalone outbound?
- **NI10:** Who owns DNS, and can they create/delegate private zones or forwarding?
- **NI11** (tier T4+): Are dedicated private-endpoint subnets reserved separately from compute?
- **NI12:** Can the deployer create network resources, or is a separate cloud/network team required?

## Workload bands

Use B4 to suggest a band, then confirm:

| Band | Recognition | Planning nodes |
| --- | --- | ---: |
| S | A handful of analysts; mostly SQL/dashboards | 32 |
| M | One data team; mixed notebooks and jobs | 128 |
| L | Several teams or concurrent ETL | 200 |
| XL / unknown | Platform-wide or leave room to grow | 500 |

Skip NI1 for serverless-only. If the customer already has a numeric peak, use it. Subnet size is a hard ceiling; bias upward.

## Subnet sizing rules

- **AWS:** five reserved IPs per subnet; workspace subnet netmask /17–/26; at least two subnets across separate availability zones; VPC netmask is unconstrained by Databricks but must contain all subnets; do not use `10.139.0.0/16`.
- **Azure:** delegated host and container subnets of equal size; /26 or larger recommended, /28 hard minimum; VNet /16–/24; add approximately /27 for private endpoints at T4+; five reserved IPs per subnet.
- **GCP:** one subnet and one primary IP range; no secondary ranges or delegation; Google reserves four addresses. Approximate maximum nodes: /25→60, /24→120, /23→250, /22→500, /21→1000, /20→2000, /19→4000.

## Worked sizing examples: 200 classic nodes

AWS:

```text
200 nodes x 2 IPs + 5 reserved = 405 IPs per workspace subnet.
Recommend /23 per subnet.
AWS requires at least two subnets in different AZs.
A /23 supports approximately 253 nodes.
```

Azure:

```text
200 nodes x 2 IPs + 5 reserved = 405 IPs per delegated subnet.
Recommend equal /23 host and container subnets.
Use a /16–/24 VNet and add a private-endpoint subnet at tier T4+.
```

GCP:

```text
For 200 nodes, use at least a /23 primary range (up to approximately 250 nodes).
GCP uses one subnet with one primary range; no secondary ranges or delegation.
```

## AWS requirements

- **Network model:** customer-managed VPC with VPC injection and at least two subnets across AZs.
- **Private connectivity (T4+):** AWS PrivateLink with workspace and SCC relay interface endpoints. Set no-public-IP/NPIP and disable public access when required.
- **Browser authentication:** no separate endpoint; browser and REST share the workspace endpoint.
- **Private DNS:** Route 53 private hosted zones/records associated with the VPC.
- **Egress:** NAT for standard classic control-plane traffic; for T8+ use Transit Gateway plus an inspection VPC or AWS Network Firewall.
- **CMK:** AWS KMS; the cross-account role requires decrypt, data-key generation, and key-description permissions.
- **Deployer IAM:** VPC, subnet, security group, VPC endpoint, Route 53, PassRole, and KMS permissions when in scope.

## Azure requirements

- **Network model:** VNet injection with host and container subnets delegated to `Microsoft.Databricks/workspaces`.
- **Private connectivity (T4+):** `databricks_ui_api` and `browser_authentication` private endpoints.
- **Browser authentication:** only one browser-authentication endpoint per region per private DNS zone.
- **Private DNS:** `privatelink.azuredatabricks.net`.
- **Public access:** disable Public Network Access when private-only access is required.
- **Egress:** Azure Firewall/NVA in a hub VNet with UDRs for T8+. New private VNets need an explicit outbound method.
- **NSGs:** retain the Microsoft-provided Databricks rules on delegated subnets.
- **CMK:** Key Vault plus an Access Connector managed identity with the required crypto role.
- **Deployer IAM:** VNet, NSG, private endpoint, private DNS, Databricks workspace, and Key Vault write permissions when in scope.

## GCP requirements

- **Network model:** customer-managed VPC injection with one subnet and one primary IP range.
- **Private connectivity (T4+):** Private Service Connect service attachments, reserved internal addresses, forwarding rules, and private Cloud DNS.
- **Browser authentication:** DNS A record, not a separate endpoint object.
- **Serverless private reach:** GCP serverless cannot privately reach the customer's VPC or on-premises network. If S4=Yes, raise a hard conflict and offer hybrid/classic or an approved public-endpoint allow-list pattern.
- **Egress:** Cloud NAT for stable egress; Cloud NGFW/firewall policy for T8+ restricted egress.
- **CMK:** Cloud KMS; the workspace service account needs encryption/decryption access to the key.
- **Deployer IAM:** VPC, subnet, firewall, address, forwarding-rule, Cloud DNS, service-account-policy, and KMS-policy permissions when in scope.

## DNS is the primary failure source

For private connectivity, the workspace, relay, storage, and browser-auth hostnames must resolve to private endpoint addresses from the correct networks.

Before implementation, confirm:

- named DNS owner
- private-zone creation/delegation rights
- forwarding between on-premises, hub, and spoke resolvers
- no split-horizon records pointing clients to public endpoints
- endpoint records exist before public access is disabled

The `private-networking` skill owns the implementation details and remote-mutation approval gate.
