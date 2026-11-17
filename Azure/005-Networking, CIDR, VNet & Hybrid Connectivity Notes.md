Azure Networking Notes (Crisp Version)

Source: Azure(Morning) transcript_2026-11-09_09.28.16.txt

Executive Summary
Topic	Key TakeawayCIDR Notation	Used to define network ranges (e.g., /16, /24, /28). Applicable across Azure, AWS, GCP, and on-prem environments.
Azure Reserved IPs	Azure reserves 5 IP addresses per subnet. This must be considered during subnet sizing.
VNet & Subnet Sizing	Common practice: VNet /16, Subnet /24 (depends on application requirements).
Cloud Networking	Azure replaces many networking functions traditionally handled by routers, switches, and firewalls.
Hybrid Connectivity	On-premises networks connect to Azure using VPN Gateway or ExpressRoute.
NSG	Acts as a virtual firewall controlling inbound and outbound traffic.
NSG Scope	Can be attached to both Subnet and NIC (VM).
NSG Rules	Lower priority number = higher precedence.
Effective Rules	Traffic must be allowed at both Subnet NSG and NIC NSG levels.
Interview Focus	CIDR calculations, subnet sizing, NSG priorities, Hub-Spoke Architecture.
1. CIDR Notation
Formula
CIDR	Total IPs/24	256
/28	16
/29	8
/16	65,536
Important Concept
Larger CIDR number → Smaller subnet.
Smaller CIDR number → Larger subnet.
Example
10.0.0.0/16  -> Large Network
10.0.0.0/24  -> Smaller Network
10.0.0.0/28  -> Very Small Network

Exam / Interview Question

Question: Need to deploy 12 VMs.

Can you use a /28 subnet?

Calculation
Item	CountTotal IPs in /28	16
Azure Reserved	5
Available	11
Needed for VMs	12

Result:

❌ /28 Not Enough

✅ Use/27

2. Azure Networking vs Traditional Networking
Traditional Network Device	Azure EquivalentRouter	Route Tables
Firewall	Azure Firewall / NSG
Switches	Virtual Network
Physical Infrastructure	Managed by Azure
Key Idea

When applications fully run on Azure:

No need to manage physical routers/switches.
Azure handles underlying networking infrastructure.
3. Hybrid Connectivity

When Azure needs to communicate with On-Premises infrastructure.

Connectivity Options
Method	UsageVPN Gateway	Most common
ExpressRoute	Enterprise-grade private connection
Hybrid Connectivity Flow
graph LR

A[On-Prem Network]
B[Gateway Public IP]
C[VPN Tunnel]
D[Azure VPN Gateway]
E[Azure VNet]

A --> B
B --> C
C --> D
D --> E

Azure Admin Responsibility
Azure Team	On-Prem TeamAzure VPN Configuration	Cisco/Juniper Devices
VPN Gateway Setup	Routing
VNet Configuration	Internal Device Connectivity
4. VPN Tunnel Relationship
Basic Design
graph LR

A[On-Prem Network]
B[VPN Tunnel]
C[Azure VNet]

A --> B --> C

Key Point
One VPN tunnel generally connects one VNet to one on-prem network.
Multiple VNets may require additional design patterns.
Preferred large-scale solution: Hub-Spoke Architecture.
5. Network Security Group (NSG)
What is NSG?

Virtual Firewall used to:

Allow traffic
Deny traffic
Secure VMs and Subnets
NSG Placement
graph TD

A[VNet]
B[Subnet + NSG]
C[VM + NIC NSG]

A --> B
B --> C

6. Default NSG Rules
Inbound
RuleAllow VNet Inbound
Allow Azure Load Balancer
Deny All Other Inbound
Outbound
RuleAllow VNet Outbound
Allow Internet Outbound
7. Common Ports
Service	PortSSH	22
HTTP	80
HTTPS	443
RDP	3389
8. NSG Rule Processing
Important Rule

Lower Priority Number = Higher Precedence

Example:

Rule	Priority	ActionBlock SSH	290	Deny
Allow SSH	300	Allow

Result:

❌ SSH Blocked

Because 290 is evaluated before 300.

Rule Evaluation Flow
flowchart TD

A[Incoming Traffic]
B[Check Lowest Priority Number]
C[Match Found?]
D[Apply Action]
E[Stop Processing]

A --> B
B --> C
C --> D
D --> E

9. Effective NSG Rules

Traffic is evaluated at both layers:

Subnet NSG
NIC/VM NSG
Effective Rule Logic
flowchart TD

A[Traffic]
B[Subnet NSG]
C[NIC NSG]
D[Allowed]
E[Blocked]

A --> B

B -->|Allow| C
B -->|Deny| E

C -->|Allow| D
C -->|Deny| E

Rule

✅ Allow must exist at BOTH levels.

❌ Block at ANY level = Traffic Blocked.

Interview Questions
Question	Expected AnswerWhat is CIDR?	Method of defining IP address ranges.
Why can't 12 VMs fit in /28 subnet?	16 total IPs minus 5 reserved = 11 usable.
Difference between NSG and Azure Firewall?	NSG works at subnet/NIC level; Azure Firewall operates at network level.
Can NSG be applied to VM?	Yes, via NIC association.
Can NSG be applied to Subnet?	Yes.
How are NSG rules processed?	Lower priority number first.
What is Hub-Spoke Architecture?	Centralized networking model for multiple VNets.
How do you connect Azure to On-Prem?	VPN Gateway or ExpressRoute.
Key Takeaways

✅ CIDR sizing is critical due to Azure's 5 reserved IPs.
 ✅ Typical design: VNet /16, Subnet /24.
 ✅ VPN Gateway is the common hybrid connectivity solution.
 ✅ NSG is Azure's first line of network protection.
 ✅ NSG rules follow priority-based processing.
 ✅ Traffic must pass both Subnet NSG and NIC NSG.
 ✅ Hub-Spoke Architecture is a frequently asked networking interview topic.