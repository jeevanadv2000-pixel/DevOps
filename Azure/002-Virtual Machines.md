# Azure Notes – 3 November 2026

Topic: Azure Virtual Machines (VMs) – Planning, Pricing, Storage and Creation\
Instructor: Rochak Agrawal

These notes use Rochak's main teaching sections only. I've excluded participant questions and the discussions that followed them, including instructor replies prompted by those questions. I've also corrected obvious transcript errors and clarified a few Azure-specific details.

## 1. Cloud service models and shared responsibility

Azure services generally fall into three service models:

| Model | Meaning                     | What you manage                                                   |
| ----- | --------------------------- | ----------------------------------------------------------------- |
| IaaS  | Infrastructure as a Service | OS, runtime, applications, data and access                        |
| PaaS  | Platform as a Service       | Applications, data and access; Azure manages more of the platform |
| SaaS  | Software as a Service       | Primarily how you use and configure the software                  |

With IaaS, Azure manages the physical datacentre, servers, storage and networking. You manage the virtual machine's operating system and the software running on it.

Shared responsibility – simplified

On-premises

You manage the entire stack, including physical infrastructure.

IaaS – Azure Virtual Machine

Azure: physical infrastructure

You: OS and everything above it

PaaS / SaaS

Azure manages progressively more of the underlying stack.

Lift and shift: When migrating an application from an on-premises environment to Azure with minimal application changes, IaaS VMs are often a suitable option. This migration approach is also called rehosting.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+1



## 2. Common use cases for Azure VMs

- Development and testing environments.
- Hosting websites and applications.
- Backup and recovery workloads.
- High-performance computing and data analysis.
- Extending an organisation's datacentre capacity.

A VM is useful when you need control over the operating system and the software environment.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+1



## 3. Plan before creating a VM

A VM should not be created by choosing a size and clicking Create immediately. Consider these factors first:

| Planning area    | What to decide                                                                 |
| ---------------- | ------------------------------------------------------------------------------ |
| Network          | Which VNet and subnet will the VM use? Which systems must it communicate with? |
| Name             | Use a meaningful naming convention to identify its purpose and environment.    |
| Region           | Consider cost, compliance, user location and proximity to related resources.   |
| Size             | Choose sufficient CPU, RAM and capacity for the workload.                      |
| Pricing          | Pay-as-you-go or a reservation, depending on expected usage.                   |
| Storage          | OS disk, data disks, capacity and performance requirements.                    |
| Operating system | Linux, Windows or a custom image.                                              |

Network planning matters: The VM may need to communicate with a database, another application server, the internet or external APIs. Check network placement and connectivity before deployment.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+2



## 4. VM location, size and pricing

### 4.1 Choosing a region

Azure allows you to deploy resources in different geographical regions. Consider:

- Cost: Pricing can vary by region.
- Compliance: Some organisations must keep data in specific geographical locations.
- User base: Choose a region that provides suitable latency for users.
- Related resources: Keeping communicating resources in compatible regions can help reduce latency and simplify architecture.

For example, if an application uses a load balancer in East US, consider deploying its VM in the same region, subject to the service's regional requirements.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub



### 4.2 Choosing VM size

VM size determines the compute capacity available to the workload, including CPU and memory. Choose a size based on:

- Application requirements.
- Expected number of users and requests.
- Development versus production workload.
- Cost and performance requirements.

For example, a development VM with little traffic may need fewer resources than a production VM serving many users.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+1



### 4.3 VM pricing models

| Pricing model             | Explanation                                                           | Suitable for                        |
| ------------------------- | --------------------------------------------------------------------- | ----------------------------------- |
| Pay-as-you-go (on-demand) | Pay for compute usage without making a long-term commitment.          | Variable or short-term workloads    |
| Reservations              | Commit to eligible VM usage for a term to receive discounted pricing. | Predictable, long-running workloads |

The instructor mentioned that reservations can offer substantial discounts compared with on-demand pricing. The actual discount depends on the VM, region, term and purchase conditions; do not treat the quoted maximum as guaranteed.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+1



## 5. VM naming convention

Use meaningful names so administrators can identify a VM's purpose when an environment contains many machines.

Example from the instructor's explanation:

`India-01-App-Payroll-Dev`

| Part    | Meaning                   |
| ------- | ------------------------- |
| India   | Location                  |
| 01      | Instance number           |
| App     | Workload type             |
| Payroll | Application or department |
| Dev     | Environment               |

This is a recommended naming approach, not a mandatory Azure naming format.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub



## 6. VM storage

A VM commonly uses an OS disk and can have additional data disks.

Azure Virtual Machine storage

Virtual Machine

OS Disk

Contains the operating system and boot files.

Required

Data Disk

Additional storage for application files and data.

Optional

| Feature                     | OS disk                                          | Data disk                                                |
| --------------------------- | ------------------------------------------------ | -------------------------------------------------------- |
| Purpose                     | Stores the operating system                      | Stores application and other data                        |
| Required                    | Yes                                              | No                                                       |
| Can be detached and reused? | Possible, subject to OS disk and VM requirements | Yes, subject to disk compatibility and attachment limits |
| Deletion with VM            | Depends on delete option                         | Depends on delete option                                 |

Important correction: The instructor described the OS disk as automatically deleted with the VM. In practice, Azure's delete behaviour depends on the disk's configured delete option and how the VM was created. Check the setting rather than assuming every disk is deleted. Data disks can also remain after VM deletion and continue to incur charges.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+2



### Data disk example

Think of a data disk as an external hard drive:

1. Create or attach a data disk to a VM.
2. Store application data on it.
3. Detach it when required.
4. Attach it to another compatible VM.

A data disk can be created empty or from a snapshot. A snapshot is a point-in-time copy of a disk that can be used to create or restore a disk.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+1



## 7. Understanding VM costs

The instructor highlighted two major cost components:

Total VM-related cost

Compute cost

VM processing capacity and running time

## +

Storage cost

OS disks, data disks and their provisioned capacity/performance

### Running, stopped and deallocated

This is an important interview topic. Azure distinguishes between a VM that is merely stopped and one that is deallocated.

| VM state              | Compute charges | Disk/storage charges |
| --------------------- | --------------- | -------------------- |
| Running               | Yes             | Yes                  |
| Stopped (allocated)   | Yes             | Yes                  |
| Stopped (deallocated) | No              | Yes                  |

Remember: Stopping a VM from inside its operating system does not necessarily deallocate it. To stop compute billing, ensure its state is Stopped (deallocated) in Azure. Disks and some networking resources can still incur charges.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+2



## 8. Azure VM size families

Azure offers different VM families for different workloads. Choose based on the workload's resource requirements.

| Family type                | Main strength                                    | Typical workloads                                       |
| -------------------------- | ------------------------------------------------ | ------------------------------------------------------- |
| General purpose            | Balanced CPU and memory                          | Development, testing, web servers                       |
| Compute optimized          | High CPU-to-memory ratio                         | Application servers, batch processing                   |
| Memory optimized           | High memory-to-CPU ratio                         | Large caches, in-memory analytics, relational databases |
| Storage optimized          | High disk throughput and I/O                     | Large databases, Big Data, data warehousing             |
| GPU optimized              | GPU acceleration                                 | Graphics, video rendering, compute-intensive workloads  |
| High-performance computing | High-performance CPU and networking capabilities | Scientific computing and demanding compute workloads    |

Examples of Azure VM families discussed in the class include:

- B and D: General purpose.
- F: Compute optimized.
- E: Memory optimized.
- L: Storage optimized.
- N: GPU optimized.

These are broad family associations, not an exhaustive or permanent list. Always check the current VM size documentation because available series and features change.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+2



How to choose a VM family

Understand the workload

Balanced needs

General purpose

CPU intensive

Compute optimized

Memory intensive

Memory optimized

Disk intensive

Storage optimized

GPU and HPC families are specialised options for accelerated and high-performance workloads.

You do not need to memorise every VM series. Understand the workload categories and use Microsoft's VM size documentation to select a suitable size.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+1



## 9. Creating a VM through Azure Portal

The instructor demonstrated the VM creation wizard. The main configuration stages are:

1. Basics
   - Select subscription and Resource Group.
   - Enter VM name.
   - Choose region.
   - Select availability options.
   - Choose an operating system image.
   - Select VM size.
   - Configure administrator credentials.
   - Configure inbound ports.
2. Disks
   - Review the OS disk.
   - Create and attach a new data disk or attach an existing disk.
   - Configure disk size and source.
   - Decide whether a data disk should be deleted with the VM.
   - Review encryption and caching options.
3. Networking
   - Select an existing VNet and subnet, or create new ones.
   - Review the Network Interface.
   - Configure public IP and Network Security Group settings as required.
4. Management
   - Configure backup.
   - Consider disaster recovery.
   - Configure guest updates and monitoring.
   - Set up alerts where needed.
5. Review + Create
   - Azure validates the configuration.
   - Review warnings and configuration details.
   - Select Create to start deployment.

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+4

### Operating system images

An image provides the operating system and its initial configuration used to create a VM.

Examples include:

- Windows Server images.
- Linux distributions.
- Marketplace images from software vendors.
- Custom images created by an organisation.

For example, selecting a Windows image creates a Windows-based VM.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub



### Network security and ports

The VM's inbound access is controlled by network security rules. Allow only the ports required for the workload.

| Protocol / service           | Default port |
| ---------------------------- | ------------ |
| SSH – Linux administration   | 22           |
| RDP – Windows administration | 3389         |
| HTTP                         | 80           |
| HTTPS                        | 443          |

For a Linux VM, SSH is commonly required; for a Windows VM, RDP is commonly required. Avoid exposing management ports to the entire internet when a restricted source IP, VPN or other secure access method can be used.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+2



## 10. Backup vs Disaster Recovery

These two concepts serve different purposes.

| Feature      | Azure Backup                                                                     | Azure Site Recovery                             |
| ------------ | -------------------------------------------------------------------------------- | ----------------------------------------------- |
| Main purpose | Protect data and recover from accidental deletion, corruption or other data loss | Keep workloads recoverable during major outages |
| Approach     | Creates and retains backups                                                      | Replicates workloads to a recovery location     |
| Recovery     | Restore from a recovery point                                                    | Fail over to a replicated VM                    |
| Main goal    | Data protection                                                                  | Business continuity and disaster recovery       |

Backup

VM → Backup recovery point

Restore data or the VM when needed.

Disaster recovery

Primary VM → Replication → Secondary region

Initiate failover to run the workload at the recovery location.

Important correction: Site Recovery does not simply and automatically create a replacement VM whenever the primary region fails. Replication must be configured beforehand, and failover is an operation that must be initiated according to the recovery plan.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+3



## 11. Other resources created with a VM

Creating a VM can also create supporting Azure resources, depending on the selected configuration.

Examples demonstrated by the instructor:

- Network Security Group (NSG).
- Public IP address.
- Network Interface (NIC).
- Virtual Network and subnet.
- OS disk and any configured data disks.

This is why VM creation is not necessarily the creation of only one Azure resource.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub



## 12. Quick revision – interview questions

1\. Which cloud service model is generally suitable for lift-and-shift migration?

IaaS, because it provides control over the VM operating system and environment.

2\. What factors should you consider before creating an Azure VM?

Network, region, size, pricing, storage, operating system and security.

3\. What is the difference between OS disk and data disk?

The OS disk contains the operating system. Data disks provide additional storage for application and other data.

4\. Does stopping a VM always stop compute billing?

No. A VM in the Stopped (allocated) state can still incur compute charges. It must be deallocated to stop compute billing.

5\. What are the two major VM cost components?

Compute and storage.

6\. Which VM family would you select for a large in-memory cache?

A memory-optimized family.

7\. Which VM family is suitable for a workload requiring high disk I/O?

A storage-optimized family.

8\. What is the difference between Azure Backup and Site Recovery?

Backup protects data for restoration. Site Recovery replicates workloads to support disaster recovery.

9\. What is an Azure VM image?

A template containing an operating system and its initial configuration, used to create a VM.

10\. What is the purpose of an NSG?

It filters network traffic using security rules, controlling allowed and denied inbound and outbound communication.

## 13. Final mental model

Plan the VM

Region · Network · Size · Pricing · OS

Create the VM

Subscription · Resource Group · Image · Credentials

Configure supporting resources

Disks · VNet · Subnet · NIC · NSG · Public IP

Operate and protect

Monitoring · Backup · Disaster Recovery · Cost Management

The key takeaway: Creating an Azure VM is not just choosing CPU and RAM. You must plan its network, region, storage, security, pricing and recovery requirements.

## Reference links

- [Original class transcript – 3 November 2026](https://github.com/jeevanadv2000-pixel/python/blob/main/Azure%28Morning%29%20transcript_2026-11-03_09.31.46.txt?utm_source=chatgpt.com)
- [Microsoft Learn – Azure VM sizes](https://learn.microsoft.com/en-us/azure/virtual-machines/sizes/overview?utm_source=chatgpt.com)
- [Microsoft Learn – VM states and billing](https://learn.microsoft.com/en-us/azure/virtual-machines/states-billing?utm_source=chatgpt.com)
- [Microsoft Learn – Delete a VM and attached resources](https://learn.microsoft.com/en-us/azure/virtual-machines/delete?utm_source=chatgpt.com)
- [Microsoft Learn – Azure Site Recovery](https://learn.microsoft.com/en-us/azure/site-recovery/site-recovery-overview?utm_source=chatgpt.com)

3 November session notes complete. The notes above are ready to copy into your Azure Markdown file.