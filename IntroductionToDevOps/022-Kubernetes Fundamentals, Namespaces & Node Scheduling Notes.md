DevOps Session — Kubernetes Fundamentals, Namespaces & Node Scheduling Notes
Date: 8 October 2026
Instructor: Rochak Agrawal
Focus: Container orchestration, Kubernetes architecture revision, declarative configuration, API objects, namespaces, node management, scheduling, labels, taints and tolerations.

These notes condense the instructor's explanations, remove participant discussions and repetition, correct speech-to-text errors, and clarify a few technical points that are important in real DevOps environments.
1. Why do we need container orchestration?
Running a few containers on a single Docker host is straightforward. However, as applications grow, managing containers across multiple machines becomes difficult.

The instructor revisits the main challenges.
Challenge	Explanation
Single-host failure	If the only host fails, all containers running on it become unavailable.
Manual placement	Someone must decide which host should run each container.
Resource limitations	A container cannot be scheduled onto a host that lacks sufficient resources.
Scaling	Increasing or decreasing application instances becomes difficult to manage manually.
Service discovery	Containers need a stable way to find one another, even when their IP addresses change.
Load balancing	Traffic must be distributed across healthy application instances.
Application updates	Replacing all instances at once can cause downtime.
Configuration management	Application configuration and sensitive information should not be hardcoded into container images.
Persistent storage	Applications such as databases need storage that survives container replacement.

What does container orchestration provide?
Container orchestration is a system for managing containerized workloads across multiple nodes.

Its major capabilities include:
1. Scheduling: Selects suitable nodes based on available resources and scheduling rules.
2. Self-healing: Replaces failed Pods or containers to restore the desired state.
3. Scaling: Adds or removes application replicas manually or automatically.
4. Service discovery: Provides stable ways for applications to locate services.
5. Load balancing: Distributes traffic across available application instances.
6. Rolling updates: Gradually replaces old application versions with new versions.
7. Rollback: Allows a deployment to return to an earlier version.
8. Configuration and storage: Separates application configuration, secrets and persistent data from the container image.

GitHub+1
Example: Rolling update
Suppose an application is running three replicas using image version v1.

You want to upgrade it to v2.

Instead of stopping all three instances simultaneously, Kubernetes can replace them gradually.
Before update
v1
Replica 1

v1
Replica 2

v1
Replica 3


During update
v1

v2

v1


After update
v2
Replica 1

v2
Replica 2

v2
Replica 3



The goal is to keep the application available while replacing old replicas. Actual availability depends on the rollout strategy, readiness probes, replica count and application behavior.
Important distinction: Scaling
There are two common types:
Type	Meaning
Manual scaling	An administrator changes the number of replicas.
Automatic scaling	The system adjusts replicas based on configured metrics or policies.

Kubernetes supports automatic Pod scaling through the Horizontal Pod Autoscaler (HPA), provided the required metrics and configuration are available.
2. Container orchestration tools and ecosystem
The instructor compares several orchestration platforms.
Tool	Description	Main use
Kubernetes	Open-source container orchestration platform	Complex, scalable container environments
Docker Swarm	Docker's native orchestration solution	Simpler Docker-based clusters
HashiCorp Nomad	Workload scheduler supporting containers and other workloads	Flexible workload scheduling
Amazon ECS	AWS-native container orchestration service	Container workloads on AWS
Red Hat OpenShift	Enterprise Kubernetes platform	Kubernetes with integrated enterprise tooling and support
Apache Mesos	Distributed systems cluster manager	Primarily historical/legacy context

Kubernetes vs Docker Swarm
Feature	Docker Swarm	Kubernetes
Setup	Relatively simple	More components and configuration
Configuration	Uses service-oriented YAML	Uses Kubernetes API objects and declarative YAML
Scaling	Manual service scaling is available	Manual and automatic scaling
Networking	Overlay networking and routing mesh	Pluggable CNI networking and Services
Network policies	No equivalent native Kubernetes-style NetworkPolicy model	Supports NetworkPolicy through compatible networking plugins
Updates	Rolling updates supported	Rolling updates, rollout history and rollback
Storage	Docker volumes and plugins	Volumes, PersistentVolumes and CSI integration
Ecosystem	Smaller ecosystem	Large ecosystem
Cloud options	No equivalent first-party managed Swarm service from the major cloud providers	EKS, AKS and GKE managed Kubernetes services

Technical correction: Do not conclude that Docker Swarm has no rolling updates or rollback, or that AWS ECS has no autoscaling. Both Swarm and ECS support more capabilities than the simplified comparison suggests. ECS supports service autoscaling through AWS integrations. Kubernetes is especially attractive when its broader ecosystem, portability and advanced scheduling/networking capabilities are required. 

GitHub

Managed Kubernetes services
Cloud provider	Service
AWS	Amazon EKS
Microsoft Azure	Azure Kubernetes Service (AKS)
Google Cloud	Google Kubernetes Engine (GKE)

In a managed Kubernetes service, the cloud provider manages the control-plane infrastructure. The customer still configures and manages workloads and many cluster-related resources.
What is OpenShift?
OpenShift is Red Hat's enterprise Kubernetes platform.

It builds on Kubernetes and provides additional enterprise capabilities, such as:
- Integrated developer and deployment tooling.
- Security-focused defaults and controls.
- Enterprise lifecycle management.
- Vendor support.
- Integrated platform services.
Interview point: OpenShift is not a separate replacement for Kubernetes. It is an enterprise Kubernetes distribution/platform.
3. Declarative vs imperative approach
This is one of the most important concepts in Kubernetes.
Imperative approach
You tell Kubernetes exactly which command to execute.

Example:
kubectl create deployment nginx --image=nginx:1.27
kubectl scale deployment nginx --replicas=3


You are explicitly telling Kubernetes what actions to perform.
Declarative approach
You describe the desired state in a YAML file.

Example:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.27


Apply it:
kubectl apply -f deployment.yaml


Kubernetes works to make the actual cluster state match the configuration.
Difference
Imperative	Declarative
Defines how to perform an action	Defines the desired outcome
Uses direct commands	Uses configuration files
Harder to review a sequence of manual changes	Easier to track configuration changes
Less convenient for Git-based configuration management	Works well with GitOps and version control

What is desired state?
Desired state means what you want Kubernetes to maintain.

Example:
spec:
  replicas: 3


This means you want three application replicas running.

Suppose one Pod fails:
State	Replica count
Desired state	3
Actual state	2
Difference	1

The relevant Kubernetes controllers detect the difference and work to create a replacement Pod.

This process is called reconciliation.
#chatgpt-mermaid-_r_3mq_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-_r_3mq_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-_r_3mq_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-_r_3mq_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-_r_3mq_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-_r_3mq_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-_r_3mq_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-_r_3mq_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-_r_3mq_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-_r_3mq_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-_r_3mq_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-_r_3mq_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-_r_3mq_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-_r_3mq_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-_r_3mq_ p{margin:0;}#chatgpt-mermaid-_r_3mq_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-_r_3mq_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-_r_3mq_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-_r_3mq_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-_r_3mq_ .label text,#chatgpt-mermaid-_r_3mq_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-_r_3mq_ .node rect,#chatgpt-mermaid-_r_3mq_ .node circle,#chatgpt-mermaid-_r_3mq_ .node ellipse,#chatgpt-mermaid-_r_3mq_ .node polygon,#chatgpt-mermaid-_r_3mq_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-_r_3mq_ .rough-node .label text,#chatgpt-mermaid-_r_3mq_ .node .label text,#chatgpt-mermaid-_r_3mq_ .image-shape .label,#chatgpt-mermaid-_r_3mq_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-_r_3mq_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-_r_3mq_ .rough-node .label,#chatgpt-mermaid-_r_3mq_ .node .label,#chatgpt-mermaid-_r_3mq_ .image-shape .label,#chatgpt-mermaid-_r_3mq_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-_r_3mq_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-_r_3mq_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-_r_3mq_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-_r_3mq_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-_r_3mq_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-_r_3mq_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-_r_3mq_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-_r_3mq_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-_r_3mq_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-_r_3mq_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-_r_3mq_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-_r_3mq_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-_r_3mq_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-_r_3mq_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-_r_3mq_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-_r_3mq_ .icon-shape,#chatgpt-mermaid-_r_3mq_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-_r_3mq_ .icon-shape p,#chatgpt-mermaid-_r_3mq_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-_r_3mq_ .icon-shape .label rect,#chatgpt-mermaid-_r_3mq_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-_r_3mq_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-_r_3mq_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-_r_3mq_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-_r_3mq_ [data-look="neo"].node rect,#chatgpt-mermaid-_r_3mq_ [data-look="neo"].cluster rect,#chatgpt-mermaid-_r_3mq_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-_r_3mq_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_3mq_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-_r_3mq_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-_r_3mq_-gradient);stroke-width:1px;}#chatgpt-mermaid-_r_3mq_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_3mq_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-_r_3mq_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-_r_3mq_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_3mq_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-_r_3mq_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-_r_3mq_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_3mq_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-_r_3mq_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_3mq_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-_r_3mq_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-_r_3mq_ .node tspan[font-weight="normal"],#chatgpt-mermaid-_r_3mq_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-_r_3mq_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-_r_3mq_ .node rect,#chatgpt-mermaid-_r_3mq_ .node circle,#chatgpt-mermaid-_r_3mq_ .node ellipse,#chatgpt-mermaid-_r_3mq_ .node polygon,#chatgpt-mermaid-_r_3mq_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-_r_3mq_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-_r_3mq_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-_r_3mq_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-_r_3mq_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-_r_3mq_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}Desired state: 3 replicasKubernetes observes actualstateAre 3 replicas running?Maintain current stateController creates replacementPodYesNo, only 2





What is configuration drift?
Configuration drift is the difference between the desired state and the actual state.

For example:
- Desired state: 3 Pods.
- Actual state: 2 Pods.
- Drift: 1 missing Pod.
If the relevant controller cannot operate, it cannot reconcile that difference. Further failures can increase the gap.

GitHub+1
4. Kubernetes objects and API resources
The instructor emphasizes an important concept:

In Kubernetes, almost everything you manage is represented as an API object.

Examples:
Kubernetes object	Purpose
Pod	Runs containers
Deployment	Manages application replicas and updates
ReplicaSet	Maintains a specified number of Pod replicas
Service	Provides a stable network endpoint for Pods
ConfigMap	Stores non-sensitive application configuration
Secret	Stores sensitive configuration data
PersistentVolume (PV)	Represents persistent storage in the cluster
PersistentVolumeClaim (PVC)	Requests storage for an application
ServiceAccount	Provides an identity for workloads
Namespace	Organizes namespaced resources
Node	Represents a cluster machine

You create and manage these objects through the Kubernetes API, usually using kubectl and YAML manifests.
How does kubectl work?
kubectl is the Kubernetes command-line tool.

It communicates with the Kubernetes API server. You do not normally connect directly to a worker node to create or manage application resources.

Examples:
kubectl get nodes
kubectl get pods
kubectl get deployments
kubectl apply -f deployment.yaml
kubectl delete -f deployment.yaml


The API server processes these requests and coordinates with the other control-plane components.
5. Kubernetes architecture — revision
A Kubernetes cluster consists of:
- Control plane — manages the cluster.
- Worker nodes — execute application workloads.
Control-plane components
Component	Responsibility
kube-apiserver	Entry point for API requests
etcd	Stores persistent cluster state
kube-scheduler	Selects worker nodes for unscheduled Pods
kube-controller-manager	Runs controllers that maintain desired state
cloud-controller-manager	Integrates with cloud infrastructure where applicable

Worker-node components
Component	Responsibility
kubelet	Manages Pods assigned to the node
kube-proxy	Implements Service networking rules, when used
Container runtime	Runs containers
Pods	Run application containers

What happens if a component fails?
Failed component	Possible impact
API server	New API requests and normal control-plane operations are disrupted
etcd	Control plane loses access to persistent cluster state
Scheduler	New unscheduled Pods remain Pending
Controller manager	Desired-state reconciliation is disrupted
kubelet	Node-level Pod management and status reporting are affected
Container runtime	Containers cannot be started or managed normally on that node

Important: Failure of a control-plane component does not necessarily mean that all running application containers immediately stop. Existing workloads may continue running while management and recovery capabilities are impaired. The exact behavior depends on which component failed and for how long.
What happens when etcd fails?
etcd is particularly important because it stores the cluster's persistent state.
- The API server cannot normally read or persist cluster state.
- Controllers cannot reliably reconcile desired state.
- New management operations are disrupted.
- Existing containers may continue running temporarily.
For disaster recovery, etcd backups are important in self-managed clusters. Managed Kubernetes providers generally manage the control-plane datastore and its infrastructure according to their service model.
Managed Kubernetes: who manages what?
              Cloud Provider
                    |
             Control Plane
          (Managed by provider)
                    |
             API Server Endpoint
                    |
          ---------------------
          |         |         |
       Worker 1  Worker 2  Worker 3
          |         |         |
        Pods      Pods      Pods


For EKS, AKS and GKE, the provider manages the managed control-plane infrastructure. You still need to manage workload deployment, node capacity and other customer responsibilities.

GitHub+1
6. Kubernetes namespaces
Important interview topic
A namespace is a logical grouping mechanism inside a Kubernetes cluster.

It allows teams to organize and separate resources without creating a separate physical cluster for every application or environment.
Why do we need namespaces?
Imagine an organization has three environments:
- Development
- Staging
- Production
Instead of creating three independent Kubernetes clusters, the organization can create three namespaces within one cluster.







medium.com


#chatgpt-mermaid-_r_3ns_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-_r_3ns_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-_r_3ns_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-_r_3ns_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-_r_3ns_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-_r_3ns_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-_r_3ns_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-_r_3ns_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-_r_3ns_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-_r_3ns_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-_r_3ns_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-_r_3ns_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-_r_3ns_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-_r_3ns_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-_r_3ns_ p{margin:0;}#chatgpt-mermaid-_r_3ns_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-_r_3ns_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-_r_3ns_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-_r_3ns_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-_r_3ns_ .label text,#chatgpt-mermaid-_r_3ns_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-_r_3ns_ .node rect,#chatgpt-mermaid-_r_3ns_ .node circle,#chatgpt-mermaid-_r_3ns_ .node ellipse,#chatgpt-mermaid-_r_3ns_ .node polygon,#chatgpt-mermaid-_r_3ns_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-_r_3ns_ .rough-node .label text,#chatgpt-mermaid-_r_3ns_ .node .label text,#chatgpt-mermaid-_r_3ns_ .image-shape .label,#chatgpt-mermaid-_r_3ns_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-_r_3ns_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-_r_3ns_ .rough-node .label,#chatgpt-mermaid-_r_3ns_ .node .label,#chatgpt-mermaid-_r_3ns_ .image-shape .label,#chatgpt-mermaid-_r_3ns_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-_r_3ns_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-_r_3ns_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-_r_3ns_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-_r_3ns_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-_r_3ns_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-_r_3ns_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-_r_3ns_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-_r_3ns_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-_r_3ns_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-_r_3ns_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-_r_3ns_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-_r_3ns_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-_r_3ns_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-_r_3ns_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-_r_3ns_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-_r_3ns_ .icon-shape,#chatgpt-mermaid-_r_3ns_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-_r_3ns_ .icon-shape p,#chatgpt-mermaid-_r_3ns_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-_r_3ns_ .icon-shape .label rect,#chatgpt-mermaid-_r_3ns_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-_r_3ns_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-_r_3ns_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-_r_3ns_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-_r_3ns_ [data-look="neo"].node rect,#chatgpt-mermaid-_r_3ns_ [data-look="neo"].cluster rect,#chatgpt-mermaid-_r_3ns_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-_r_3ns_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_3ns_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-_r_3ns_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-_r_3ns_-gradient);stroke-width:1px;}#chatgpt-mermaid-_r_3ns_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_3ns_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-_r_3ns_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-_r_3ns_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_3ns_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-_r_3ns_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-_r_3ns_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_3ns_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-_r_3ns_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_3ns_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-_r_3ns_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-_r_3ns_ .node tspan[font-weight="normal"],#chatgpt-mermaid-_r_3ns_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-_r_3ns_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-_r_3ns_ .node rect,#chatgpt-mermaid-_r_3ns_ .node circle,#chatgpt-mermaid-_r_3ns_ .node ellipse,#chatgpt-mermaid-_r_3ns_ .node polygon,#chatgpt-mermaid-_r_3ns_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-_r_3ns_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-_r_3ns_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-_r_3ns_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-_r_3ns_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-_r_3ns_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}Kubernetes ClusterNamespace: devNamespace: stagingNamespace: productionApplication resourcesApplication resourcesApplication resources





Benefits of namespaces
1. Logical separation of applications and environments.
2. Easier resource organization.
3. Access control using Role-Based Access Control (RBAC).
4. Resource management through ResourceQuotas and LimitRanges.
5. Ability to use the same resource name in different namespaces.
6. Network segmentation when used with NetworkPolicies.
Can the same application name exist in multiple namespaces?
Yes.

For example:
dev namespace
    └── Deployment: web

staging namespace
    └── Deployment: web

production namespace
    └── Deployment: web


These are different Kubernetes objects because they belong to different namespaces.
Are namespaces security boundaries?
This is a particularly important technical distinction.

Namespaces provide logical and organizational separation, but they do not automatically provide network isolation.

By default, Pods in different namespaces can generally communicate if the cluster networking allows it.

To restrict communication, you can use:
- NetworkPolicies.
- RBAC.
- Service accounts.
- Appropriate security controls.
For example, you might want:
- Application A to communicate with Application B.
- Application B to communicate with Application C.
- Application A to be blocked from communicating directly with Application C.
NetworkPolicies can express these communication rules, provided the cluster's CNI plugin supports enforcement.
Four default Kubernetes namespaces
A typical Kubernetes cluster starts with four namespaces:
Namespace	Purpose
default	Used when a namespaced resource is created without specifying a namespace
kube-system	Contains Kubernetes system components
kube-public	Publicly readable namespace intended for cluster-wide information
kube-node-lease	Contains Lease objects used for node heartbeats and health tracking

Best practice: Avoid deploying business applications into kube-system, kube-public or kube-node-lease. Use dedicated application namespaces instead.
Namespace commands
List namespaces:
kubectl get namespaces


Short form:
kubectl get ns


Create a namespace:
kubectl create namespace dev


Get Pods in a particular namespace:
kubectl get pods -n dev


Deploy an application into a namespace:
kubectl create deployment nginx \
  --image=nginx:1.27 \
  -n dev


Delete a namespace:
kubectl delete namespace dev



Caution
 Deleting a namespace also deletes the namespaced resources inside it. Check its contents before deleting it.
How to define a namespace in YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: dev
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.27


Cross-namespace communication
Applications in different namespaces can communicate when network policies and other security controls permit it.

A Kubernetes Service can be addressed using its DNS name.

For example:
nginx-service.dev.svc.cluster.local


This represents:
Service name: nginx-service
Namespace:    dev
Cluster:      svc.cluster.local



GitHub+1
7. Anatomy of a Kubernetes YAML file
The instructor explains that Kubernetes manifests follow a common structure.

There are four main fields:
1. apiVersion
2. kind
3. metadata
4. spec
Example:
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx
  namespace: dev
  labels:
    app: nginx
    environment: development

spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80


7.1 apiVersion
Defines the API group and version used for the Kubernetes object.

Examples:
Object	Common apiVersion
Pod	v1
Service	v1
Deployment	apps/v1
ReplicaSet	apps/v1
ConfigMap	v1
Secret	v1
PersistentVolume	v1

The API version depends on the object type and the supported API version in the cluster. It is not always apps/v1.
7.2 kind
Defines the type of Kubernetes object you want to create.

Examples:
kind: Pod


kind: Deployment


kind: Service


7.3 metadata
Metadata is information about the Kubernetes object.

Common fields:
Field	Purpose
name	Name of the object
namespace	Namespace to which it belongs
labels	Key-value tags used to organize and select resources
annotations	Additional non-identifying metadata

Example:
metadata:
  name: payment-service
  namespace: production
  labels:
    app: payment
    environment: production


What are labels?

Labels are key-value pairs attached to Kubernetes objects.

They help organize and identify resources.

For example:
labels:
  app: payment
  environment: production
  team: billing


Other Kubernetes resources, such as Services and Deployments, can use label selectors to identify matching Pods.
7.4 spec
spec means specification.

It defines the desired configuration of the object.

The contents of spec depend on the object type.

For example:
Object	What its spec defines
Pod	Containers, images, volumes and runtime settings
Deployment	Replicas, selectors and Pod template
Service	Type, ports and selectors
PersistentVolume	Storage capacity and storage configuration

This is where you define the desired state.
Remember the structure
Kubernetes YAML
      |
      |-- apiVersion  → API version
      |
      |-- kind        → Object type
      |
      |-- metadata    → Object information
      |
      |-- spec        → Desired configuration


Can multiple Kubernetes objects be written in one YAML file?
Yes.

You can write multiple Kubernetes objects in one YAML file by separating them with ---.

Example:
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service


You can also maintain separate files for each object.
kubernetes/
├── namespace.yaml
├── deployment.yaml
├── service.yaml
└── configmap.yaml


For readability and maintainability, separate files are often useful, especially as projects grow.

Apply a file:
kubectl apply -f deployment.yaml


Apply all YAML files in a directory:
kubectl apply -f kubernetes/


DevOps relevance: Store Kubernetes YAML manifests in Git. This allows teams to review, track, approve and roll back configuration changes.

GitHub+1
8. Kubernetes Nodes
A Node is a physical or virtual machine that runs workloads in a Kubernetes cluster.

Examples:
- AWS EC2 instance.
- Azure Virtual Machine.
- Google Compute Engine VM.
- Physical Linux server.
A worker node normally runs:
- kubelet
- Container runtime
- kube-proxy, where used
Node is a cluster-scoped object
This is an important distinction.

Nodes belong to the cluster itself, not to a namespace.
Resource	Scope
Node	Cluster-scoped
Pod	Namespaced
Deployment	Namespaced
Service	Namespaced
ConfigMap	Namespaced
Namespace	Cluster-scoped

A Pod from any namespace can be scheduled onto an eligible worker node, subject to scheduling constraints.
Node capacity vs allocatable resources
The instructor explains two important terms.

Capacity: Total resources available on a node.

Allocatable: Resources available for Kubernetes workloads after reserving resources for the operating system and system services.

Example:
Resource	Memory
Total node capacity	8 GB
Reserved for OS and system services	2 GB
Allocatable	6 GB

If each Pod requests 1 GB of memory, approximately six such requests could fit based on memory alone.

Actual scheduling also depends on CPU requests, existing allocations, other constraints and resource reservations.
Node status
A node must be healthy and schedulable for normal workload placement.

Common node conditions/status concepts include:
- Ready
- MemoryPressure
- DiskPressure
- PIDPressure
- NetworkUnavailable
When a node becomes unhealthy, Kubernetes may stop scheduling new Pods there. Depending on the failure and configured eviction behavior, workloads may eventually be recreated on other healthy nodes.
Node commands
List nodes:
kubectl get nodes


Detailed information:
kubectl describe node <node-name>


View node resource information:
kubectl top nodes


The kubectl top command requires the Metrics API, commonly provided by Metrics Server.
9. Pod scheduling — how Kubernetes chooses a node
When a Deployment creates Pods, the scheduler selects suitable worker nodes.

The scheduler considers multiple factors:
1. Resource requests and available resources.
2. Node selectors.
3. Node affinity and anti-affinity.
4. Taints and tolerations.
5. Other scheduling constraints.
Example
Suppose a cluster has five worker nodes.
Node	Available memory	Pod needs 1 GB?
Node 1	4 GB	Eligible
Node 2	2 GB	Eligible
Node 3	256 MB	Insufficient
Node 4	8 GB	Eligible
Node 5	3 GB	Eligible

The scheduler filters out nodes that cannot satisfy the Pod's requirements and evaluates the remaining nodes.

Important: The scheduler does not simply select the node with the most free memory. It uses a scheduling process involving filtering and scoring.
10. Labels and node selectors
A label is a key-value pair assigned to a Kubernetes resource.

Labels can be assigned to:
- Nodes
- Pods
- Deployments
- Other Kubernetes objects
Why are node labels useful?
Suppose you have different types of worker nodes:
- General-purpose nodes.
- High-memory nodes.
- GPU-enabled nodes.
You can label the nodes according to their purpose and schedule specific workloads accordingly.
Step 1 — Add a label to a node
kubectl label nodes worker-1 workload=high-memory


Step 2 — Verify the label
kubectl get nodes --show-labels


Step 3 — Use nodeSelector in a Pod specification
apiVersion: v1
kind: Pod
metadata:
  name: memory-intensive-app
spec:
  nodeSelector:
    workload: high-memory
  containers:
    - name: nginx
      image: nginx:1.27


This Pod can only be scheduled onto nodes matching the specified label, provided other scheduling requirements are also satisfied.
Real-world example
Imagine your organization has:
Node group	Label
General workloads	workload=general
High-memory workloads	workload=high-memory
GPU workloads	workload=gpu

You can direct an application to the appropriate node group instead of allowing it to run on any available worker.
11. Taints and tolerations
Important interview topic
Taints and tolerations control which Pods are allowed to be scheduled onto particular nodes.

The instructor explains taints using an office example:

Imagine an office with several rooms. Employees can enter normal rooms, but a restricted room requires special permission.

A node taint acts like a restriction on that node.
Taint
A taint is applied to a node to repel Pods that do not tolerate it.

Example:
kubectl taint nodes worker-1 dedicated=database:NoSchedule


This means Pods without a matching toleration will not normally be scheduled onto worker-1.
Toleration
A toleration is added to a Pod specification to allow it to be scheduled onto a node with a matching taint.

Example:
apiVersion: v1
kind: Pod
metadata:
  name: database-pod
spec:
  tolerations:
    - key: dedicated
      operator: Equal
      value: database
      effect: NoSchedule
  containers:
    - name: database
      image: nginx:1.27


The Pod tolerates the node's taint.
Important distinction
A toleration does not guarantee that a Pod will be scheduled on that node. It only removes the scheduling restriction caused by that matching taint. Other scheduling rules still apply.
Taint effects
Effect	Meaning
NoSchedule	New Pods without a matching toleration are not scheduled on the node
PreferNoSchedule	Kubernetes tries to avoid scheduling Pods without a matching toleration
NoExecute	Pods without a matching toleration are evicted, and new Pods without one are not scheduled

Labels vs taints
Labels	Taints
Identify and categorize nodes	Restrict which Pods can run on nodes
Used with node selectors and affinity	Used with Pod tolerations
Example: workload=gpu	Example: dedicated=gpu:NoSchedule
Helps attract suitable workloads	Helps repel unsuitable workloads

Simple mental model
- Label: I want this Pod to go to a particular type of node.
- Taint: I don't want Pods without permission to come to this node.
- Toleration: This Pod is allowed to tolerate that restriction.
Related commands
Add a taint:
kubectl taint nodes worker-1 dedicated=database:NoSchedule


Remove a taint:
kubectl taint nodes worker-1 dedicated=database:NoSchedule-


View node taints:
kubectl describe node worker-1



GitHub
12. Cordon vs drain
The instructor ends the session by explaining how to prepare a node for maintenance.

Imagine you need to patch or reboot a worker node.

You should prevent new Pods from being scheduled onto that node and safely move eligible existing workloads elsewhere.
Cordon
Cordon marks a node as unschedulable.
- New Pods will not normally be scheduled onto that node.
- Existing Pods continue running.
- It is useful when preparing a node for maintenance.
Command:
kubectl cordon worker-1


Drain
Drain prepares a node for maintenance by evicting eligible Pods.
- Prevents new Pods from being scheduled onto the node.
- Evicts eligible existing Pods.
- Controllers can recreate those Pods on other suitable nodes.
- Certain Pods may require special handling.
Command:
kubectl drain worker-1 --ignore-daemonsets


The --ignore-daemonsets option is commonly needed because DaemonSet-managed Pods are not normally evicted by drain.
Uncordon
After maintenance, make the node schedulable again.
kubectl uncordon worker-1


Cordon vs drain comparison
Feature	Cordon	Drain
Prevents new scheduling	Yes	Yes
Existing Pods continue running	Yes	Eligible Pods are evicted
Moves workloads away	No	Controllers can recreate eligible workloads elsewhere
Common use	Mark node unschedulable	Prepare node for maintenance

Recommended maintenance workflow
#chatgpt-mermaid-_r_3ts_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-_r_3ts_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-_r_3ts_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-_r_3ts_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-_r_3ts_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-_r_3ts_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-_r_3ts_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-_r_3ts_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-_r_3ts_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-_r_3ts_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-_r_3ts_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-_r_3ts_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-_r_3ts_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-_r_3ts_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-_r_3ts_ p{margin:0;}#chatgpt-mermaid-_r_3ts_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-_r_3ts_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-_r_3ts_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-_r_3ts_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-_r_3ts_ .label text,#chatgpt-mermaid-_r_3ts_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-_r_3ts_ .node rect,#chatgpt-mermaid-_r_3ts_ .node circle,#chatgpt-mermaid-_r_3ts_ .node ellipse,#chatgpt-mermaid-_r_3ts_ .node polygon,#chatgpt-mermaid-_r_3ts_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-_r_3ts_ .rough-node .label text,#chatgpt-mermaid-_r_3ts_ .node .label text,#chatgpt-mermaid-_r_3ts_ .image-shape .label,#chatgpt-mermaid-_r_3ts_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-_r_3ts_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-_r_3ts_ .rough-node .label,#chatgpt-mermaid-_r_3ts_ .node .label,#chatgpt-mermaid-_r_3ts_ .image-shape .label,#chatgpt-mermaid-_r_3ts_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-_r_3ts_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-_r_3ts_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-_r_3ts_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-_r_3ts_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-_r_3ts_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-_r_3ts_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-_r_3ts_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-_r_3ts_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-_r_3ts_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-_r_3ts_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-_r_3ts_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-_r_3ts_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-_r_3ts_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-_r_3ts_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-_r_3ts_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-_r_3ts_ .icon-shape,#chatgpt-mermaid-_r_3ts_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-_r_3ts_ .icon-shape p,#chatgpt-mermaid-_r_3ts_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-_r_3ts_ .icon-shape .label rect,#chatgpt-mermaid-_r_3ts_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-_r_3ts_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-_r_3ts_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-_r_3ts_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-_r_3ts_ [data-look="neo"].node rect,#chatgpt-mermaid-_r_3ts_ [data-look="neo"].cluster rect,#chatgpt-mermaid-_r_3ts_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-_r_3ts_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_3ts_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-_r_3ts_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-_r_3ts_-gradient);stroke-width:1px;}#chatgpt-mermaid-_r_3ts_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_3ts_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-_r_3ts_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-_r_3ts_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_3ts_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-_r_3ts_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-_r_3ts_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_3ts_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-_r_3ts_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-_r_3ts_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-_r_3ts_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-_r_3ts_ .node tspan[font-weight="normal"],#chatgpt-mermaid-_r_3ts_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-_r_3ts_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-_r_3ts_ .node rect,#chatgpt-mermaid-_r_3ts_ .node circle,#chatgpt-mermaid-_r_3ts_ .node ellipse,#chatgpt-mermaid-_r_3ts_ .node polygon,#chatgpt-mermaid-_r_3ts_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-_r_3ts_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-_r_3ts_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-_r_3ts_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-_r_3ts_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-_r_3ts_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}Identify worker nodeCordon nodeDrain nodePerform patching /maintenance / rebootVerify node healthUncordon nodeNode available for newworkloads





Production caution: Before draining, check PodDisruptionBudgets, available replicas, local storage, DaemonSets and available capacity on other nodes. A drain can be blocked or disrupt workloads if these conditions are not handled.

GitHub
13. Application configuration: ConfigMaps vs Secrets
The session also revisits how Kubernetes separates configuration from container images.

Imagine your application needs:
- Database hostname.
- Database port.
- Database username.
- Database password.
You should not hardcode these values into the Dockerfile or container image.

Kubernetes provides two important objects.
ConfigMap
Used for non-sensitive application configuration.

Example:
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: dev
data:
  DB_HOST: mysql-service
  DB_PORT: "3306"


Secret
Used for sensitive information such as passwords and tokens.

Example:
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
  namespace: dev
type: Opaque
stringData:
  DB_PASSWORD: change-me


Security note
Kubernetes Secrets are not automatically encrypted just because they are called Secrets. Base64 encoding is not encryption. Configure encryption at rest, restrict access using RBAC, and follow secure secret-management practices.
ConfigMap vs Secret
ConfigMap	Secret
Non-sensitive configuration	Sensitive configuration
Database hostname	Database password
Application mode	API token
Feature flags	Credentials

Both can be exposed to containers through environment variables or mounted files.

Important correction: The application/container references the values at runtime; you do not need to rebuild the image just because the database hostname or password changes. Some configuration changes require a Pod restart or rollout to take effect, depending on how the application consumes the values.

GitHub+1
14. Interview revision — important questions
Q1. Why do we need Kubernetes?
Kubernetes automates container deployment, scheduling, scaling, self-healing, networking and application lifecycle management across multiple nodes.
Q2. What is the difference between imperative and declarative deployment?
Imperative deployment uses commands to perform specific actions. Declarative deployment defines the desired state in YAML, and Kubernetes reconciles the actual state with it.
Q3. What is desired state?
Desired state is the configuration or outcome that we want Kubernetes to maintain, such as three replicas of an application.
Q4. What is configuration drift?
It is the difference between the desired state and the actual state. Kubernetes controllers work to correct this difference.
Q5. What is a namespace?
A namespace is a logical grouping mechanism used to organize and isolate resources within one Kubernetes cluster.
Q6. Are namespaces security boundaries?
No. Namespaces provide logical organization, but do not automatically block network traffic. RBAC and NetworkPolicies provide additional access and traffic controls.
Q7. Can two namespaces communicate?
Yes. Pods in different namespaces can communicate when network connectivity and applicable policies permit it.
Q8. What is the difference between node capacity and allocatable?
Capacity is the total resource capacity of a node. Allocatable is the amount available for Kubernetes workloads after system reservations.
Q9. How does Kubernetes decide where to schedule a Pod?
The scheduler filters and scores nodes based on resource requirements, node selectors, affinity, taints, tolerations and other scheduling constraints.
Q10. What is the difference between labels and taints?
Labels help identify and select nodes. Taints repel Pods that do not have matching tolerations.
Q11. What is a toleration?
A toleration allows a Pod to be scheduled onto a node with a matching taint. It does not guarantee placement there.
Q12. What is the difference between cordon and drain?
Cordon prevents new Pods from being scheduled on a node but leaves existing Pods running. Drain also evicts eligible existing Pods to prepare the node for maintenance.
Q13. What happens if the scheduler stops working?
New unscheduled Pods generally remain in Pending state, while existing Pods can continue running.
Q14. Why should Kubernetes YAML files be stored in Git?
Git provides version history, collaboration, code review, auditability and the ability to restore previous configurations.
Q15. What is the difference between ConfigMap and Secret?
ConfigMap stores non-sensitive configuration, while Secret is intended for sensitive information such as credentials and tokens.

15. Quick revision sheet
Kubernetes concepts
Term	One-line meaning
Orchestration	Manages containers across nodes
Desired state	What you want
Actual state	What is currently running
Reconciliation	Bringing actual state towards desired state
Namespace	Logical resource grouping
Node	Machine in the cluster
Label	Identifies/selects resources
Taint	Restricts Pods from a node
Toleration	Allows a Pod to tolerate a taint
Cordon	Stops new scheduling
Drain	Evicts eligible Pods for maintenance
ConfigMap	Non-sensitive configuration
Secret	Sensitive configuration
kubectl	Kubernetes command-line tool
YAML	Declarative resource configuration


Commands to remember
Command	Purpose
kubectl get nodes	List nodes
kubectl describe node NODE	Inspect a node
kubectl get ns	List namespaces
kubectl create ns dev	Create namespace
kubectl get pods -n dev	List Pods in namespace
kubectl apply -f file.yaml	Apply manifest
kubectl label nodes NODE key=value	Add node label
kubectl taint nodes NODE key=value:NoSchedule	Add node taint
kubectl cordon NODE	Mark node unschedulable
kubectl drain NODE --ignore-daemonsets	Drain eligible Pods
kubectl uncordon NODE	Make node schedulable

Final mental model
Kubernetes
    |
    ├── Cluster
    |     |
    |     ├── Control Plane
    |     |      ├── API Server
    |     |      ├── etcd
    |     |      ├── Scheduler
    |     |      └── Controllers
    |     |
    |     └── Worker Nodes
    |            ├── kubelet
    |            ├── Container Runtime
    |            └── Pods
    |
    ├── Namespaces
    |      ├── dev
    |      ├── staging
    |      └── production
    |
    └── Scheduling
           ├── Resource requirements
           ├── Labels / Node selectors
           ├── Taints / Tolerations
           └── Affinity rules


Most important topics to understand before the hands-on sessions:
1. Kubernetes architecture and component responsibilities.
2. Declarative YAML and desired-state reconciliation.
3. Namespaces and their limitations as isolation boundaries.
4. Node capacity vs allocatable resources.
5. Labels, taints and tolerations.
6. Cordon vs drain.
Next session: The instructor closes by saying the conceptual foundation is complete and the next class will move into hands-on Kubernetes work, beginning with Pods.

Source transcript: DevOps Session — 8 October 2026.

You can use these notes as your GitHub revision document for the Kubernetes fundamentals section.