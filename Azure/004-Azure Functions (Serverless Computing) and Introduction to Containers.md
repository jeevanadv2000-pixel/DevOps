# Azure Notes – 5 November 2026

Main topics: Azure Functions (Serverless Computing) and Introduction to Containers\
Instructor: Rochak Agrawal

These notes include Rochak's independent teaching and demonstration only. Participant questions and the discussions prompted by them have been excluded.

## 1. Azure compute services – three approaches

The instructor begins by comparing three ways to run applications in Azure.

| Compute service        | Model             | What you manage                                                                 |
| ---------------------- | ----------------- | ------------------------------------------------------------------------------- |
| Azure Virtual Machines | IaaS              | VM, OS, runtime and application                                                 |
| Azure App Service      | PaaS              | Application and its configuration                                               |
| Azure Functions        | Serverless / PaaS | Function code and its configuration; Azure manages the execution infrastructure |

Three ways to run applications

1\. Virtual Machine

Manage the VM, OS, runtime and application.

2\. App Service

Deploy your application to a managed hosting platform.

3\. Azure Functions

Run code in response to events without managing servers.

The key difference is how much infrastructure management is required and how the application executes.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+1



## 2. What is Azure Functions?

Azure Functions is a serverless computing service provided by Microsoft Azure.

It allows developers to execute small units of code called functions in response to specific events or inputs.

You write the code and configure when it should execute. Azure manages the underlying execution infrastructure.

### What does serverless mean?

Serverless does not mean there are no physical servers.

It means:

- You do not manage the underlying servers.
- You do not need to provision a VM for each function.
- Azure manages the infrastructure on which your functions execute.
- Depending on the hosting plan, instances can scale automatically based on demand.

AWS equivalent: AWS Lambda.

| Microsoft Azure | Amazon Web Services |
| --------------- | ------------------- |
| Azure Functions | AWS Lambda          |

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+1

## 3. Event-driven architecture

Azure Functions is commonly used for event-driven applications.

An event is something that happens in a system. When that event occurs, the function executes the required code.

### Example 1: File uploaded to a Storage Account

Suppose you upload a file to an Azure Storage Account and want to compress it automatically.

1. A user uploads a file.
2. The file upload generates an event.
3. Azure Functions detects the event.
4. The function executes the compression code.
5. The compressed file can be stored in another location.

File processing using Azure Functions

File uploaded

Azure Storage Account

Trigger detects upload event

Azure Function executes

Compress the file

Save compressed file

Storage Account / other destination

### Example 2: VM stopped

Suppose a VM unexpectedly stops and you want to execute recovery logic.

- Event: VM stopped.
- Function: Executes the configured code.
- Action: Initiates a restart, if the required permissions and conditions are satisfied.

This is useful for automation because code executes in response to an event rather than continuously running on a server.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+1



## 4. Triggers and Bindings

Two important concepts in Azure Functions are Triggers and Bindings.

### 4.1 Triggers

A trigger defines the event that causes a function to execute.

Examples:

| Trigger               | When does the function execute?             |
| --------------------- | ------------------------------------------- |
| HTTP trigger          | When an HTTP request is received            |
| Timer trigger         | At a configured time or schedule            |
| Blob Storage trigger  | When a blob-related event occurs            |
| Queue Storage trigger | When a message becomes available in a queue |
| Event Hub trigger     | When events are received from Event Hubs    |

For example, if you configure an HTTP trigger, the function can be invoked through an HTTP endpoint.

### 4.2 Bindings

Bindings connect a function to other Azure services or external resources.

They allow a function to receive input from or send output to a service without writing all the connection-handling code yourself.

Examples:

- Read a file from Blob Storage.
- Write a message to Queue Storage.
- Read or write data in Cosmos DB.
- Interact with Azure SQL.

Important clarification: Bindings are not only outputs. They can be input bindings or output bindings. A trigger is a special kind of input that causes the function to execute.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+1



### Trigger vs Binding

| Trigger                           | Binding                                          |
| --------------------------------- | ------------------------------------------------ |
| Determines when the function runs | Connects the function to data or another service |
| Initiates function execution      | Provides input or handles output                 |
| Example: File uploaded            | Example: Read or save a file                     |

Putting them together

Trigger

File uploaded to Blob Storage

Azure Function

Executes compression logic

Output Binding

Save compressed file to storage

## 5. Azure Functions hosting plans

During the portal demonstration, the instructor showed the available hosting options.

The important concepts are:

- Flex Consumption: Serverless, event-driven scaling with pay-as-you-go billing.
- Premium: Provides pre-warmed instances and more predictable performance for workloads that need them.
- App Service / Dedicated plan: Functions can run using an App Service Plan alongside web applications.

The instructor selected Flex Consumption for the demonstration.

Current Azure guidance: Flex Consumption is the recommended serverless hosting option for new Function Apps. The older Consumption plan is now considered legacy.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://learn.microsoft.com\&sz=32)

Microsoft Learn+1

### 5.1 What is scale to zero?

Scale to zero means that when there are no incoming events or requests, the hosting plan can scale its on-demand function instances down to zero.

For example:

- You create a function that compresses files uploaded to Blob Storage.
- No files are uploaded for a period.
- There are no function executions during that period.
- The on-demand instances can scale down to zero.

When a new event arrives, Azure can provision an instance and execute the function.

Scale-to-zero behaviour

No events

# 0

On-demand instances

Event received

# 1+

Instances execute function

Conceptual illustration; actual instance counts depend on workload, concurrency and hosting-plan settings.

Cost clarification: The instructor simplified this as no execution means no cost. For Flex Consumption, on-demand execution charges are based on execution count and resource consumption while functions execute. However, always-ready instances, storage accounts and other associated resources can still incur charges.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+2



## 6. Creating an Azure Function App through the Portal

The instructor demonstrated the Function App creation process.

### 6.1 Function App vs Function

These are two different things:

| Term         | Meaning                                                                                          |
| ------------ | ------------------------------------------------------------------------------------------------ |
| Function App | Azure resource that provides the hosting environment and configuration for one or more functions |
| Function     | The individual code unit that executes when triggered                                            |

Think of a Function App as a container that hosts multiple functions.

### 6.2 Creation steps

1. Search for Function App

   Open Azure Portal and search for Function App. Select Create.
2. Select hosting plan

   Choose Flex Consumption for the demonstration.
3. Configure Basics
   - Subscription
   - Resource Group
   - Function App name
   - Region
   - Runtime stack
   - Runtime version
   - Instance memory size
4. Configure zone redundancy

   Review whether zone redundancy is supported and enabled for the selected plan and region.
5. Configure Storage

   Select or create the Storage Account associated with the Function App.
6. Configure Networking

   Review public access and virtual network connectivity options.
7. Configure Monitoring and Deployment

   Review monitoring, continuous deployment and authentication settings.
8. Review + Create

   Validate the configuration and deploy the Function App.

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+1

### 6.3 Runtime stack and instance memory

The instructor selected Python during the demonstration.

Azure Functions supports multiple programming languages, including:

- C#
- Java
- JavaScript / TypeScript
- Python
- PowerShell

The available languages and versions depend on the hosting environment and runtime support.

Instance memory: This determines the memory capacity available to each function execution instance. It is a hosting configuration, even though the service is serverless.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+1



### 6.4 Storage Account

A Storage Account is associated with a Function App.

It supports runtime operations such as:

- Managing triggers.
- Storing runtime-related information.
- Supporting scaling and execution infrastructure.
- Logging and other hosting operations.

It is not necessarily the business database for your application. You can separately connect a function to Azure SQL, Cosmos DB or other data services.

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+1

### 6.5 Networking

The instructor demonstrated public access and virtual network integration options.

The important distinction is:

- Public access: Controls whether the Function App can be reached through its public endpoint.
- Virtual Network integration: Allows supported outbound connectivity from the Function App to resources in a virtual network.
- Private Endpoint: Provides private inbound access to the Function App through a private IP in your virtual network.

Important correction

VNet integration does not simply give the Function App a private IP for inbound access. Use Private Endpoint when you need private inbound connectivity. The available networking features depend on the hosting plan.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+2



## 7. Deploying function code

Creating a Function App does not automatically create the actual function code.

The instructor explained the sequence:

1. Create the Function App.
2. Open the Function App.
3. Create a function using a supported template and trigger.
4. Implement the required code.
5. Configure bindings.
6. Deploy the code.

Deployment can be done through supported development tools or deployment methods. The portal also provides deployment options, including manual package/code deployment and repository-based workflows.

Demonstration limitation: The instructor encountered a portal UI issue while trying to create the function. The Function App was created, but the function code and trigger were not successfully completed during the demonstration.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub



## 8. Introduction to Containers

After the Azure Functions discussion, Rochak starts introducing the concept of containers.

He begins by comparing containers with virtual machines.

### 8.1 What is a physical machine?

A laptop or desktop is a physical machine. It has actual hardware components such as:

- CPU
- RAM
- Storage
- Motherboard
- Network interface

For example, when you shut down your laptop, your Zoom connection stops because the physical machine is no longer operating.

### 8.2 What is a Virtual Machine?

A Virtual Machine is a software-based computer created using the resources of a physical server.

A physical server uses virtualization software, commonly a hypervisor, to create multiple virtual machines.

Each VM gets an allocated portion of the physical server's resources.

Examples:

- Azure Virtual Machines.
- AWS EC2 instances.

Physical server and virtual machines

Physical Server

CPU · RAM · Storage

Hypervisor / Virtualization Software

VM 1

Guest OS

Allocated resources

VM 2

Guest OS

Allocated resources

VM 3

Guest OS

Allocated resources

VM 4

Guest OS

Allocated resources

The number of VMs that can be created depends on the physical server's total capacity and the resources allocated to each VM.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub



### 8.3 What about containers?

Rochak introduces containers as the next concept and starts setting up a comparison with VMs. However, the available transcript excerpt ends while he is still explaining the physical-server and virtualization foundation.

Therefore, the detailed container architecture and comparison are not covered in these notes rather than being filled in with unrelated material.

## 9. Quick revision – interview questions

1\. What is Azure Functions?

A serverless compute service that executes code in response to events or triggers.

2\. What is the AWS equivalent of Azure Functions?

AWS Lambda.

3\. What does serverless mean?

You do not manage the underlying servers. The code still executes on physical infrastructure managed by the cloud provider.

4\. What is event-driven architecture?

An application design where code executes in response to events, such as a file upload or an HTTP request.

5\. What is a trigger?

A trigger defines the event that invokes a function.

6\. What is a binding?

A binding connects a function to another service for input or output operations.

7\. What is scale to zero?

The ability of an eligible hosting plan to reduce on-demand instances to zero when there is no workload.

8\. What is a Function App?

An Azure resource that hosts and manages one or more individual functions.

9\. Why does Azure Functions need a Storage Account?

It uses associated storage for runtime operations, trigger management, scaling and related infrastructure requirements.

10\. What is a hypervisor?

Virtualization software that allows multiple virtual machines to run on a physical server.

## 10. Final mental model

Azure compute services

Virtual Machine

Manage server and OS

App Service

Managed application hosting

Azure Functions

Execute code when events occur

Azure Functions execution flow

Event / Trigger

Function executes code

Binding / Output

Virtualization foundation

Physical Server

Hypervisor

Multiple Virtual Machines

Key takeaway: Azure Functions is useful when you want code to execute in response to events without managing servers. Containers are the next topic, introduced through the foundation of physical servers and virtualization.

## Reference links

- [Original class transcript – 5 November 2026](https://github.com/jeevanadv2000-pixel/python/blob/main/Azure%28Morning%29%20transcript_2026-11-05_09.29.19.txt?utm_source=chatgpt.com)
- [Microsoft Learn – Azure Functions overview](https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview?utm_source=chatgpt.com)
- [Microsoft Learn – Triggers and bindings](https://learn.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings?utm_source=chatgpt.com)
- [Microsoft Learn – Azure Functions hosting options](https://learn.microsoft.com/en-us/azure/azure-functions/functions-scale?utm_source=chatgpt.com)
- [Microsoft Learn – Azure Functions best practices](https://learn.microsoft.com/en-us/azure/azure-functions/functions-best-practices?utm_source=chatgpt.com)
- [Microsoft Learn – Virtual machines and hypervisors](https://learn.microsoft.com/en-us/azure/virtual-machines/overview?utm_source=chatgpt.com)

5 November session notes complete.