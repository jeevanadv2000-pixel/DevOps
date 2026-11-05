Azure Fundamentals: Regions, Resource Hierarchy & Management Tools
Overview

This session introduced the core Azure architecture, including Regions, Availability Zones, Resource Groups, Subscriptions, Management Groups, Azure Resource Manager (ARM), and Azure management tools.

Azure Global Infrastructure
Key Concepts
Component	DescriptionRegion	Geographic area containing one or more datacenters
Availability Zone (AZ)	Physically separate datacenters within a region
Datacenter	Physical infrastructure where resources run
Resource	Any Azure service instance (VM, Storage, DB, etc.)
Region Selection Factors
Factor	Why It MattersUser Location	Reduces latency
Compliance	Data residency requirements
Cost	Pricing may vary slightly by region
Service Availability	Not all services are available everywhere
Examples of Compliance
Region RequirementRBI: Indian customer data should remain in India
EU: Customer data should remain within EU boundaries
Azure Infrastructure Hierarchy
flowchart TD
    Region --> AvailabilityZone
    AvailabilityZone --> Datacenter
    Datacenter --> Resources

Availability Zones

Availability Zones provide protection against datacenter failures.

SLA Comparison
Deployment Type	SLASingle VM	99.9%
Availability Zone Enabled	Higher Availability
Multi-Region DR	Disaster Recovery
Why Availability Zones?
flowchart LR
    DC1[Datacenter 1] --> VM
    DC1 -.Failure.-> DC2[Datacenter 2]
    DC2 --> VM2[Recovered VM]

Benefits
Protects against datacenter failures
Automatic failover
Independent:
Power
Cooling
Network
Connected via private fiber network
Azure Resource Groups

A Resource Group (RG) is a logical container used to manage Azure resources.

Characteristics
Property	ValueCost	Free
Purpose	Organize resources
Permissions	Can be assigned at RG level
Policies	Can be applied at RG level
Resource Membership	One resource can belong to only one RG
Example

Application resources:

Web Server
App Server
Database
Load Balancer

can all reside inside a single Resource Group.

Resource Group Structure
flowchart TD
    RG[Resource Group]
    RG --> VM
    RG --> Database
    RG --> Storage
    RG --> LoadBalancer

Important Notes

✅ Resources inside a resource group can exist in different regions.

✅ Resources can be moved between resource groups.

✅ Resource group metadata is stored in its selected region.

Azure Resource Manager (ARM)

Azure Resource Manager is the management layer for Azure.

Responsibilities
Authentication
Authorization
Resource Creation
Resource Updates
Resource Deletion
ARM Request Flow
flowchart LR
    User --> Portal
    User --> CLI
    User --> PowerShell
    User --> RESTAPI

    Portal --> ARM
    CLI --> ARM
    PowerShell --> ARM
    RESTAPI --> ARM

    ARM --> AzureServices

Azure Subscription

A Subscription is a:

Billing Boundary
Access Control Boundary
Subscription Responsibilities
Function	PurposeBilling	Azure cost tracking
Access Control	Permission management
Resource Ownership	Resources belong to subscriptions
Important Rules
Every resource must belong to a subscription.
Every resource group belongs to a subscription.
One account can have multiple subscriptions.
AWS vs Azure
AWS	AzureBilling at Account Level	Billing at Subscription Level
AWS Account	Azure Subscription
Azure Management Hierarchy
flowchart TD
    MG[Management Group]
    MG --> SUB[Subscription]
    SUB --> RG[Resource Group]
    RG --> RES[Resources]

Management Groups

Used for managing multiple subscriptions centrally.

Use Cases
Organization-wide policies
Governance
Access Management
Compliance Controls
Example Structure
flowchart TD

ORG[My Organization]

ORG --> HR
ORG --> Admin
ORG --> Engineering

HR --> SUB1
Admin --> SUB2
Engineering --> SUB3

SUB1 --> RG1
SUB2 --> RG2
SUB3 --> RG3

When to Use Management Groups
Scenario	RecommendationSingle Subscription	Not Required
Multiple Teams	Recommended
Organization Policies	Recommended
Central Governance	Recommended
Azure Resource Organization Best Practices
Typical Enterprise Layout
flowchart TD

MG[Management Group]

MG --> PROD[Production Subscription]
MG --> NONPROD[Non-Production Subscription]

NONPROD --> DEV[Dev RG]
NONPROD --> TEST[Test RG]
NONPROD --> STAGE[Stage RG]

PROD --> PRODRG[Production RG]

Segregation Guidelines
Level	Common Segregation StrategyManagement Group	Department / Business Unit
Subscription	Application / Environment
Resource Group	Logical Workload
Azure Management Tools

Azure resources can be managed using multiple interfaces.

Tool	PurposeAzure Portal	GUI Management
Azure CLI	Command-Line Management
Azure PowerShell	PowerShell-Based Management
Cloud Shell	Browser-Based CLI/PowerShell
REST APIs	Programmatic Access
SDKs	Application Integration
Mobile App	Limited Operations
Azure CLI
Command Structure
az <service> <action> <arguments>

Examples

List Resource Groups

az group list


Create Resource Group

az group create \
  --name demo-rg \
  --location eastus


Delete Resource Group

az group delete \
  --name demo-rg

Azure PowerShell
Login
Connect-AzAccount

Create Resource Group
New-AzResourceGroup `
-Name demo-rg `
-Location EastUS

Delete Resource Group
Remove-AzResourceGroup `
-Name demo-rg

Azure CLI vs PowerShell
Azure CLI	Azure PowerShellPreferred by Linux Users	Preferred by Windows Users
Shell Based	PowerShell Based
Starts with az	Uses Az Cmdlets
Cross Platform	PowerShell Ecosystem
Recommendation
Linux Background → Azure CLI
Windows Background → Azure PowerShell

Both provide equivalent functionality.

Cloud Shell

Cloud Shell provides a preconfigured environment inside Azure Portal.

Features
No local installation required
Azure CLI support
PowerShell support
File upload/download
Integrated editor
flowchart LR
    AzurePortal --> CloudShell
    CloudShell --> CLI
    CloudShell --> PowerShell

Key Interview Takeaways
Question	AnswerWhat is an Azure Region?	Geographic location containing one or more datacenters
Why use Availability Zones?	Protection against datacenter failure
What is a Resource Group?	Logical container for Azure resources
Can a resource belong to multiple Resource Groups?	No
Why do we need Subscriptions?	Billing and Access Control
What is ARM?	Azure Resource Manager, the control plane of Azure
What is Management Group used for?	Managing multiple subscriptions centrally
Difference between Azure CLI and PowerShell?	Linux-oriented CLI vs Windows-oriented PowerShell
Can resources inside one Resource Group belong to different regions?	Yes
Summary
mindmap
  root((Azure Fundamentals))
    Regions
      Datacenters
      Availability Zones
      Compliance
    Resource Groups
      Resource Organization
      Policies
      Permissions
    Subscription
      Billing
      Access Control
    Management Groups
      Governance
      Multiple Subscriptions
    ARM
      Authentication
      Authorization
      Resource Management
    Tools
      Portal
      CLI
      PowerShell
      Cloud Shell
      SDKs


Source: Azure(Morning) transcript_2026-11-02_09.24.16.txt