# Azure Notes – 4 November 2026

I'll use the same format as the previous session: concise study notes, tables, native diagrams that render here, corrected technical details, and only Rochak Agrawal's own teaching. Participant questions and the discussions prompted by them will be excluded.

## 1. Recap: IaaS – Infrastructure as a Service

The instructor begins the new topic by revisiting the shared responsibility model from the previous VM session.

With IaaS, Azure manages the underlying physical infrastructure, but the customer still manages the virtual machine and its software environment.

| Responsibility                  | IaaS – Azure VM |
| ------------------------------- | --------------- |
| Physical datacentre             | Microsoft       |
| Physical servers and networking | Microsoft       |
| Virtual machine OS              | Customer        |
| Runtime and middleware          | Customer        |
| Application                     | Customer        |
| Application data                | Customer        |

For example, when you create an Azure VM:

1. You select an operating system, such as Windows or Linux.
2. Azure provisions the VM.
3. You install the required runtime and applications.
4. You manage and maintain the VM and the software running on it.

The key point is that you are still responsible for managing the VM, even though it runs on Azure infrastructure.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+1



## 2. Moving from IaaS to PaaS

The instructor then introduces another cloud service model: PaaS – Platform as a Service.

The idea is simple: what if you do not want to manage virtual machines, operating systems or runtime environments?

Azure provides managed services where you can deploy your application without managing the underlying VM infrastructure.

| Responsibility          | IaaS      | PaaS                           |
| ----------------------- | --------- | ------------------------------ |
| Physical infrastructure | Microsoft | Microsoft                      |
| Operating system        | Customer  | Microsoft                      |
| Runtime / platform      | Customer  | Microsoft manages the platform |
| Application code        | Customer  | Customer                       |
| Application data        | Customer  | Customer                       |

IaaS vs PaaS – visual comparison

IaaS

Application

You manage

Runtime

You manage

OS / VM

You manage

Hardware

Azure manages

PaaS

Application

You manage

Runtime

Azure manages

OS / Platform

Azure manages

Hardware

Azure manages

Simplified responsibility comparison. Application configuration, security and data remain customer responsibilities in PaaS.

Remember: With IaaS, you manage the server. With PaaS, you focus more on the application while Azure manages the underlying platform.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+1



## 3. Azure App Service (Web Apps)

The instructor introduces Azure Web Apps, part of Azure App Service, as an example of PaaS.

Instead of creating a VM, installing an operating system, configuring a runtime and deploying your application manually, you provide your application to Azure App Service.

For example, suppose you have a Java-based application:

With an Azure VM (IaaS):

1. Create a VM.
2. Configure its operating system.
3. Install the Java runtime.
4. Configure the web server or application server.
5. Deploy your Java application.
6. Maintain the VM and its software.

With Azure App Service (PaaS):

1. Create an App Service web app.
2. Select the appropriate runtime stack, such as Java.
3. Deploy your application.
4. Azure manages the underlying hosting platform.

Application deployment comparison

IaaS – Azure VM

Application code

Runtime / Web server

Operating system

Azure infrastructure

PaaS – Azure App Service

Your application code

Azure-managed runtime and hosting platform

Azure infrastructure

Azure App Service supports application stacks such as Java, .NET, Node.js, Python and PHP, on Windows and Linux hosting options. It also supports custom containers.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+2



## 4. When should you choose IaaS or PaaS?

| Requirement                                                     | Suitable option          |
| --------------------------------------------------------------- | ------------------------ |
| Need control over the operating system                          | IaaS – Azure VM          |
| Need to install custom OS-level software                        | IaaS – Azure VM          |
| Want to deploy a supported web application without managing VMs | PaaS – Azure App Service |
| Want Azure to manage the hosting platform and runtime           | PaaS – Azure App Service |

## 5. Quick revision – interview questions

1\. What is IaaS?

Infrastructure as a Service. Azure provides the infrastructure, while the customer manages the VM operating system and applications.

2\. What is PaaS?

Platform as a Service. Azure manages the underlying platform, allowing customers to focus on deploying and managing their applications.

3\. What is Azure App Service?

A managed PaaS offering for hosting web applications, mobile backends and REST APIs.

4\. What is the main difference between Azure VM and Azure App Service?

With a VM, you manage the operating system and runtime. With App Service, Azure manages the underlying hosting platform.

5\. Can we deploy a Java application to Azure App Service?

Yes. Azure App Service supports Java application hosting.

## 6. Final mental model

IaaS – Azure Virtual Machines

Create and manage the VM, OS, runtime and application.

PaaS – Azure App Service

Deploy your application; Azure manages the hosting platform.

Key takeaway: Azure App Service reduces the infrastructure-management work required to host a web application. You concentrate on your application rather than managing the underlying VM.

4 November session notes complete.

Scope note: The transcript's earlier VM-series, VM Scale Sets, fault/update domains, Automation Account, ARM template and user-data discussions were responses to participant questions, so I have deliberately excluded them. These notes cover Rochak's independent lecture introducing PaaS and Azure App Service.

References

- [Original Azure class transcript – 4 November 2026](https://github.com/jeevanadv2000-pixel/python/blob/main/Azure%28Morning%29%20transcript_2026-11-04_09.30.50.txt?utm_source=chatgpt.com)
- [Microsoft Learn – Azure App Service overview](https://learn.microsoft.com/en-us/azure/app-service/overview?utm_source=chatgpt.com)
- [Microsoft Learn – Shared responsibility in Azure](https://learn.microsoft.com/en-us/azure/security/fundamentals/shared-responsibility?utm_source=chatgpt.com)