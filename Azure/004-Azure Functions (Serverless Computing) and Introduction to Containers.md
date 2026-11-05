Azure Functions, Containers, ACI & ACR Notes
Session Overview

This session covered:

Azure Functions (Serverless Computing)
Event-Driven Architecture
Triggers and Bindings
Function App
Scale-to-Zero Concept
Containers vs Virtual Machines
Docker Basics
Docker Images & Containers
Azure Container Instances (ACI)
Azure Container Registry (ACR)
DIY Containers vs Managed Containers

Source: Azure(Morning) transcript_2026-11-05_09.29.19.txt

Compute Services in Azure

Azure provides three major ways to run applications:

flowchart LR

VM[Virtual Machine - IaaS]
APP[Azure App Service - PaaS]
FUNC[Azure Functions - Serverless]

Service	Infrastructure Visibility	Infrastructure ManagementVirtual Machine	Yes	Customer
App Service	Yes	Azure
Azure Functions	No	Azure
Azure Functions

Azure Functions is a:

Serverless Compute Service


It allows execution of code without managing servers.

Key idea:

Focus on code,
not infrastructure.

What Does Serverless Mean?

❌ No servers exist

✅ Servers exist but are hidden from users

flowchart LR

Developer --> FunctionCode
FunctionCode --> Azure
Azure --> HiddenServers


Users only provide code.

Azure manages:

Infrastructure
Scaling
Availability
Runtime
Event-Driven Architecture

Azure Functions are designed primarily for:

Event Driven Architecture


Application execution starts only when an event occurs.

flowchart LR

Event --> Function --> Action

Example 1: File Upload

Scenario:

Upload File
↓
Trigger Function
↓
Compress File

flowchart LR

StorageUpload --> AzureFunction
AzureFunction --> CompressFile

Example 2: VM Stopped

Scenario:

VM Stops
↓
Azure Function Triggered
↓
Restart VM

flowchart LR

VMStopped --> Function
Function --> RestartVM

Triggers

Triggers define:

WHEN a function runs


Trigger = Input Event

flowchart LR

Trigger --> Function

Common Trigger Types
Trigger	ExampleHTTP	URL Request
Timer	Daily 9 PM Run
Storage Event	File Upload
Queue Message	Message Arrives
Bindings

Bindings define:

WHERE function sends output


Binding = Output Destination

flowchart LR

Function --> Binding

Trigger vs Binding
flowchart LR

Trigger --> Function --> Binding


Example:

File Uploaded
↓
Trigger
↓
Compress File
↓
Store in Database
↓
Binding

Azure Functions Pricing Advantage
Scale to Zero

Most important interview point.

Virtual Machine:

VM running = Billing continues


Even if:

No users
No requests

Billing remains active.

Function App:

No Trigger
↓
No Execution
↓
No Cost

Example

Function:

Trigger on File Upload


If no file uploaded entire month:

Execution Count = 0
Cost = 0

Function App Creation

Supported Languages:

RuntimePython
Java
Go
PowerShell
Custom Runtime
Function App Architecture
flowchart TD

FunctionApp
 --> Trigger

Trigger
 --> FunctionCode

FunctionCode
 --> Binding

Azure Functions Networking

Supports:

Public Access
Private Access
VNet Integration
flowchart LR

FunctionApp --> VNet
VNet --> Database
VNet --> VM

Virtual Machines vs Containers

Next Topic Introduced:

Containers


Before understanding Azure Container Services, first understand:

Virtual Machine vs Container

Virtual Machine Architecture
flowchart TD

PhysicalServer
 --> Hypervisor

Hypervisor
 --> VM1
Hypervisor
 --> VM2
Hypervisor
 --> VM3


Each VM contains:

Full OS
Runtime
Applications
Why Containers Were Introduced
Problem 1: Heavy Virtual Machines

To run a Python app:

Needed:

Application
Python


But VM also contains:

Chrome
Zoom
Extra Packages
Unused Components


VMs become large and resource-intensive.

Problem 2: Environment Differences

Different systems:

OS	Package ManagerUbuntu	apt
Linux	yum
Windows	Different entirely

Result:

Works on my machine
Fails on yours

Containers

Containers solve:

✅ Dependency issues

✅ Environment inconsistencies

✅ Heavy deployments

CD Example

Trainer compared containers to CDs.

Movie stored in CD


Any player can run it:

Sony
Samsung
Toshiba

Movie remains identical.

Similarly:

Application + Dependencies
=
Container Image

Container Terminology
Image

Packaged application.

Contains:

Application Code
Runtime
Libraries
Dependencies
flowchart TD

Image
 --> Code
 --> Runtime
 --> Libraries

Container

Running instance of an image.

Image = Blueprint
Container = Running Copy

Virtualization vs Containerization
Virtualization

Hardware-Level

flowchart LR

Server --> VM1
Server --> VM2
Server --> VM3


Each VM has separate OS.

Containerization

Operating-System-Level

flowchart LR

HostOS --> Container1
HostOS --> Container2
HostOS --> Container3


Containers share Host OS.

Container Components

Container contains only:

Application
Runtime
Dependencies


No full operating system.

This makes containers:

✅ Lightweight

✅ Fast

✅ Portable

Docker

Most popular containerization platform.

Used to:

Create Images
Run Containers
Manage Containers
DIY Container Approach

Process:

flowchart TD

VM
 --> DockerInstall
 --> PullImage
 --> RunContainer


Steps:

Create VM
Install Docker
Pull Image
Run Container
Limitations of DIY

You manage:

VM
Docker
Patching
Monitoring
Availability

If VM fails:

Container Fails

Docker Commands
List Images
docker images

List Running Containers
docker ps

Pull Image
docker pull nginx

Run Container
docker run -d --name mynginx -p 80:80 nginx

Port Mapping

Example:

-p 80:80


Meaning:

VM Port 80
→
Container Port 80

flowchart LR

Internet --> VM80
VM80 --> Container80

Azure Container Instances (ACI)

ACI is Azure's managed container service.

Instead of:

Create VM
Install Docker
Manage Everything


You simply provide:

Container Image


Azure runs it.

ACI Architecture
flowchart LR

ContainerImage --> AzureContainerInstance
AzureContainerInstance --> RunningContainer

Benefits of ACI

✅ No VM Management

✅ No Docker Installation

✅ Built-In Monitoring

✅ Start/Stop Easily

✅ Serverless Experience

✅ Fast Deployment

ACI Networking Modes
Mode	DescriptionPublic	Public IP
Private	VNet Access
None	Fully Isolated
When Use "None"?

Batch Jobs

Background Jobs

Internal Processing

No user access required.

ACI Features

Built-in:

Metrics
Logs
Monitoring
Start / Stop
Restart
Container Image Sources

ACI supports:

flowchart LR

DockerHub --> ACI
PrivateRegistry --> ACI
ACR --> ACI

Docker Hub

Public Registry

Website:

hub.docker.com


Contains images for:

NGINX
Ubuntu
Python
Java
MySQL
PostgreSQL

and thousands more.

Azure Container Registry (ACR)

ACR is:

Fully Managed Private Docker Registry


Used to:

Store Images
Manage Images
Secure Images
Why ACR?

Instead of storing images on public Docker Hub:

Store internally and securely

ACR Benefits

✅ Private Registry

✅ Geo Replication

✅ Security Controls

✅ Azure Integration

✅ Vulnerability Scanning

✅ Private Networking

✅ Customer Managed Keys

ACR Pricing Tiers
Tier	PurposeBasic	Learning & Small Workloads
Standard	Higher Throughput
Premium	Geo Replication + Advanced Security
Public vs Private Registry
flowchart LR

DockerHub --> Public

ACR --> Private


Docker Hub:

Public Platform


ACR:

Enterprise Private Repository

ACR Security

Premium tier supports:

Private Endpoints
Geo Replication
Customer Managed Keys
Defender Integration

ACR can integrate with:

Microsoft Defender for Cloud


Benefits:

Vulnerability Scanning
Security Recommendations
Image Security Assessment
End-to-End Container Workflow
flowchart LR

Developer
 --> DockerImage

DockerImage
 --> ACR

ACR
 --> ACI

ACI
 --> RunningApplication

Interview Questions
Question	AnswerWhat is Azure Function?	Serverless compute service
What is Event-Driven Architecture?	Code runs when event occurs
What are Triggers?	Inputs that invoke functions
What are Bindings?	Outputs from functions
What is Scale-to-Zero?	No execution = no cost
Function equivalent in AWS?	Lambda
What is Container Image?	Packaged application with dependencies
What is Container?	Running instance of image
Difference between VM and Container?	VM has OS, Container shares host OS
What is Docker?	Containerization tool
What is ACI?	Managed Azure container runtime
What is ACR?	Private container registry
Docker Hub vs ACR?	Public vs Private Registry
Why use ACR?	Security, integration, private storage
Final Revision Mind Map
mindmap
  root((Azure Compute))
    Virtual Machines
      Full OS
      Customer Managed

    App Service
      PaaS
      Web Apps

    Azure Functions
      Serverless
      Triggers
      Bindings
      Scale to Zero

    Containers
      Image
      Container
      Docker

    ACI
      Managed Containers
      Monitoring
      Public or Private

    ACR
      Private Registry
      Geo Replication
      Security
      Defender Integration

Key Exam Takeaways
Azure Functions = AWS Lambda
Trigger = Function Input Event
Binding = Function Output
Scale-to-Zero is a major serverless advantage
Image ≠ Container (Container is running instance of image)
Docker creates and runs containers
ACI is managed container hosting
ACR is Azure's private Docker registry
Containers share host OS, VMs have separate OS
ACR + ACI is a common Azure container deployment pattern