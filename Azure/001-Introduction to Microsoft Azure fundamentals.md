The data-analysis feature is temporarily unavailable (it may be available again in about 24 hours). This does not prevent me from reading your GitHub transcript, so I can still prepare the notes here.

I've reviewed the 2 November transcript again. This version includes only Rochak's teaching sections and excludes participant questions and the discussions that followed them. I've also corrected obvious speech-to-text errors and technical inaccuracies.

# Azure Fundamentals — Session Notes

2 November 2026

Instructor-only notes

Instructor: Rochak Agrawal\
Topics covered: Azure global infrastructure, regions, Availability Zones, resources, Resource Groups, Azure Resource Manager, subscriptions and Azure Portal.

## 1. Introduction to Cloud Platforms

Rochak introduced the three major cloud platforms:

- AWS
- Microsoft Azure
- Google Cloud Platform (GCP)

Although their terminology and services differ, many of their core cloud concepts are similar.

Correction: These are public cloud platforms, not private clouds.

The session begins by explaining how cloud infrastructure is organised geographically.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub



## 2. Azure Global Infrastructure

Azure infrastructure is organised into geographical regions, with data centres and networking infrastructure providing the physical foundation.

### Important terms

| Component         | Meaning                                                                                                     |
| ----------------- | ----------------------------------------------------------------------------------------------------------- |
| Geography         | A broader geographical area that can contain multiple Azure regions and serves as a data residency boundary |
| Region            | A geographical area containing one or more data centres                                                     |
| Availability Zone | An isolated grouping of one or more data centres within a region                                            |
| Data Centre       | Physical facility containing servers, storage and networking equipment                                      |

### Azure infrastructure hierarchy

Azure Geography

Example: India, Europe or United States

Azure Region

Example: Central India

Zone 1

One or more data centres

Zone 2

One or more data centres

Zone 3

One or more data centres

Illustrative region with three Availability Zones. Not every Azure region supports Availability Zones.

A region consists of one or more data centres. In regions that support Availability Zones, each zone is an isolated grouping of data centres.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://learn.microsoft.com\&sz=32)

Microsoft Learn+1



### Why do we need different regions?

Rochak explained two important reasons:

1\. Reduce latency

Deploy resources closer to your customers.

Example:

- An Indian company has customers in the United States.
- Hosting the application in a suitable US region can reduce network latency for those customers.

2\. Data residency and compliance

Some organisations must keep customer data within a particular geographical boundary.

Example:

- A financial organisation may have regulatory requirements for Indian customer data.
- It must select an appropriate region and configure its services to meet those requirements.

### Factors to consider when choosing a region

- Customer location and latency.
- Data residency and compliance.
- Availability of required Azure services.
- Availability and resiliency requirements.

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+2

## 3. Availability Zones

Availability Zones provide protection against failures affecting an individual zone.

Each zone has independent:

- Power.
- Cooling.
- Networking.

Zones within a region are connected through Microsoft's network infrastructure.

### Single-zone vs multi-zone deployment

Single-zone deployment

Azure Region

Availability Zone 1

Application VM

Workload deployed in one zone

If the zone experiences an outage, the workload may become unavailable.

Multi-zone deployment

Azure Region

Zone 1

VM 1

Zone 2

VM 2

Zone 3

VM 3

Workload distributed across zones

If one zone fails, instances in other zones can continue serving traffic, provided the application is designed for resilience.

Important correction: Azure does not automatically transfer every single VM to another data centre when a failure occurs. For zone resilience, the application generally needs appropriately deployed instances or a service that provides zone redundancy.

### Availability SLA

SLA means Service Level Agreement — the contractual availability commitment for a service under specified conditions.

Rochak used 99.9% as an example.

| Availability | Approximate downtime per year   |
| ------------ | ------------------------------- |
| 99%          | 3 days, 15 hours, 36 minutes    |
| 99.9%        | 8 hours, 45 minutes, 36 seconds |
| 99.99%       | 52 minutes, 34 seconds          |

99.9% availability example

# 8h 45m 36s

Maximum approximate downtime in a 365-day year at 99.9% availability.

Remember: An SLA depends on the exact Azure service and deployment configuration. Do not assume that simply selecting an Availability Zone guarantees a particular SLA.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+2



## 4. Azure Resources

In Azure, every instance of a service that you create is called a resource.

Examples:

| Azure service | Resource example               |
| ------------- | ------------------------------ |
| Compute       | Virtual Machine                |
| Networking    | Load Balancer, Virtual Network |
| Storage       | Storage Account                |
| Data          | Azure Data Factory             |

For example, when you create a Virtual Machine, Azure creates a VM resource.

When you create a Storage Account, Azure creates a Storage Account resource.

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub

## 5. Resource Groups

A Resource Group is a logical container used to organise and manage related Azure resources.

Think of it as a folder that groups resources belonging to an application or workload.

### Example: Application architecture

Suppose an application requires:

- Web server.
- Application server.
- Database.
- Load Balancer.

These resources can be organised in one Resource Group.

Resource Group — Payroll Application

Web Server

App Server

Database

Load Balancer

### Why use Resource Groups?

- Organise related resources.
- Apply access permissions at Resource Group level.
- Manage resources together.
- Organise resources by application, business unit or environment.

### Important Resource Group concepts

| Concept             | Explanation                                                                              |
| ------------------- | ---------------------------------------------------------------------------------------- |
| Resource membership | Each resource belongs to exactly one Resource Group                                      |
| Region              | A Resource Group has a metadata location                                                 |
| Resource locations  | Resources inside it can reside in different Azure regions                                |
| Cost                | Creating a Resource Group itself does not incur a resource usage charge                  |
| Moving resources    | Supported resources can be moved between Resource Groups, subject to service limitations |

### Can one Resource Group contain resources from different regions?

Yes.

Example:

- VM 1 is deployed in East US.
- VM 2 is deployed in Australia East.
- Both VMs can belong to the same Resource Group.

The Resource Group's selected region stores its metadata; it does not force all its resources to be deployed in that region.

### Ways to organise Resource Groups

- By location.
- By application functionality.
- By business unit.
- By environment — Dev, Test and Production.

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+3

## 6. Azure Resource Manager (ARM)

Azure Resource Manager is the management layer used to create, update and delete Azure resources.

Whenever you create a VM or a Load Balancer, the request goes through ARM.

ARM performs authentication and authorisation checks before processing the management request.

### ARM architecture

Ways to manage Azure

Azure Portal

GUI

Azure CLI

Command line

PowerShell

Scripting

REST API / SDK

Programmatic access

Azure Resource Manager

Authentication, authorisation and resource management

Azure Resources

VMs, storage, networking and other services

The important point is that all these management methods use ARM as the management layer.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+1



## 7. Azure Portal

Azure Portal is the graphical user interface (GUI) used to manage Azure resources.

Rochak demonstrated the main areas of the portal.

| Portal area       | Purpose                                                     |
| ----------------- | ----------------------------------------------------------- |
| Home              | Access frequently used services                             |
| All Services      | Browse Azure services by category                           |
| Create a Resource | Create a new Azure resource                                 |
| Notifications     | View resource deployment notifications                      |
| Cloud Shell       | Run Azure CLI or PowerShell                                 |
| Settings          | Access account, directory and subscription-related settings |

Azure services are organised into categories such as:

- AI and Machine Learning.
- Compute.
- Networking.
- Storage.
- Databases.

### Creating a resource

The general workflow demonstrated in class:

1. Open Azure Portal.
2. Select Create a Resource.
3. Choose the service, such as Virtual Machine.
4. Select or create a Resource Group.
5. Configure the required settings.
6. Review and create the resource.

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+2

## 8. Azure Cloud Shell

Cloud Shell is a browser-accessible terminal available through Azure Portal.

It allows you to execute Azure management commands without setting up the CLI locally.

Azure provides two command-line environments:

| Tool             | Common preference                         |
| ---------------- | ----------------------------------------- |
| Azure CLI        | Often preferred by Linux users            |
| Azure PowerShell | Often preferred by Windows administrators |

Both can manage Azure resources.

Azure Cloud Shell

Azure Portal

Cloud Shell

Azure CLI

PowerShell

Manage Azure Resources

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub

## 9. Azure Subscription

An Azure Subscription is an important boundary for billing and access management.

Rochak highlighted two major purposes:

1\. Billing boundary

Azure tracks resource usage and costs against a subscription.

For example, if you create 100 resources under a subscription, their applicable charges are included in that subscription's billing.

2\. Access control boundary

Permissions can be assigned at the subscription level so users can manage resources within the permitted scope.

### Subscription and Resource Group relationship

Azure Subscription

Billing and access boundary

Resource Group A

VM 1

Storage 1

Resource Group B

VM 2

Network 1

One subscription can contain multiple Resource Groups, each containing multiple resources.

Important: You need an active and appropriately authorised subscription to deploy Azure resources.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+1



## 10. Azure RBAC — brief introduction

Rochak briefly introduced RBAC (Role-Based Access Control) while demonstrating subscription access.

RBAC determines what actions a user, group or application can perform on Azure resources.

For example, the Owner role provides broad management access, including the ability to assign roles.

Detailed RBAC configuration was deferred to a later class, so it is not expanded here.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub



## 11. Core Azure areas to focus on

Rochak explained that Azure has many services, but cloud administration and architecture require a strong understanding of the core areas.

| Core area  | What it covers                           |
| ---------- | ---------------------------------------- |
| Compute    | Virtual Machines and application hosting |
| Networking | Virtual Networks and connectivity        |
| Storage    | Storage services and data                |
| Databases  | Database services                        |
| Security   | Access, permissions and protection       |
| Operations | Monitoring and administration            |

The instructor's key point was that architects should be able to design solutions that are:

- Highly available.
- Scalable.
- Efficient.
- Cost-effective.

Specialised services can be learned when a project requires them; the core cloud concepts are the foundation.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+1



## 12. Quick revision

| Question                                                       | Answer                                                                  |
| -------------------------------------------------------------- | ----------------------------------------------------------------------- |
| What is an Azure Region?                                       | A geographical area containing one or more data centres.                |
| What is an Availability Zone?                                  | An isolated grouping of data centres within a region.                   |
| Why do we choose different regions?                            | Latency, data residency, compliance and service availability.           |
| What is an Azure Resource?                                     | An instance of an Azure service that you create.                        |
| What is a Resource Group?                                      | A logical container for organising Azure resources.                     |
| Can a Resource Group contain resources from different regions? | Yes.                                                                    |
| Can one resource belong to multiple Resource Groups?           | No.                                                                     |
| What is ARM?                                                   | Azure's management layer for creating, updating and deleting resources. |
| What is Azure Portal?                                          | The GUI for managing Azure services.                                    |
| What is Cloud Shell?                                           | A browser-accessible terminal for Azure CLI and PowerShell.             |
| What is an Azure Subscription?                                 | A billing and access-management boundary.                               |
| What is RBAC?                                                  | Role-Based Access Control.                                              |

## 13. Final mental model

Azure Cloud

Geography

Data residency

Regions

Deployment location

Availability Zones

Resilience

Data Centres

Physical infrastructure

Azure Subscription

Billing and access management

Resource Groups

Logical organisation of resources

Azure Resources

VMs, Storage, Networking, Databases

Management layer

Azure Resource Manager (ARM)

Portal • CLI • PowerShell • REST API • SDK

Remember the difference:

- Region and Availability Zone describe where Azure infrastructure is located.
- Subscription and Resource Group describe how resources are organised and managed.
- ARM describes how management requests reach Azure resources.

Original transcript: [Azure Morning — 2 November 2026](https://github.com/jeevanadv2000-pixel/python/blob/main/Azure%28Morning%29%20transcript_2026-11-02_09.24.16.txt?utm_source=chatgpt.com)

Reference documentation: [Azure Global Infrastructure](https://learn.microsoft.com/azure/reliability/regions-overview?utm_source=chatgpt.com) · [Availability Zones](https://learn.microsoft.com/azure/availability-zones/az-overview?utm_source=chatgpt.com) · [Moving Azure Resources](https://learn.microsoft.com/azure/azure-resource-manager/management/move-resources-overview?utm_source=chatgpt.com)

These are the revised notes based on Rochak's teaching sections only. Participant questions and their discussion threads have been left out.