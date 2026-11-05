# DevOps Session — Kubernetes Architecture Notes

Date: 6 October 2026\
Instructor: Rochak Agrawal\
Focus: Docker limitations, container orchestration, Kubernetes architecture, control plane and worker-node components, managed Kubernetes services, and deployment workflow.

These notes condense the instructor's explanations, remove participant discussions and repetition, and correct a few technical oversimplifications in the transcript.

## 1. Why do we need container orchestration?

Docker makes it easy to build and run containers. However, managing a large number of containers across multiple servers introduces operational challenges.

### Docker challenges discussed in the session

| Challenge                              | Explanation                                                                                                                      |
| -------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| Security risks                         | Containers share the host kernel. Misconfigured privileged containers can increase the risk of host compromise.                  |
| Docker socket exposure                 | Mounting `/var/run/docker.sock` into a container can give that container extensive control over the host's Docker daemon.        |
| Secrets in image layers                | Hardcoding passwords, API keys, or tokens in a Dockerfile can expose them through image layers and image inspection.             |
| Stateful data management               | Docker volumes are normally host-local; sharing persistent data across multiple hosts requires additional storage solutions.     |
| Permission mismatch                    | Host and container user IDs may differ, causing permission problems with bind mounts.                                            |
| Networking complexity                  | Managing communication between containers across multiple hosts requires additional networking and service-discovery mechanisms. |
| Firewall exposure                      | Publishing container ports can modify host firewall and packet-filtering rules, potentially exposing services unexpectedly.      |
| No built-in cluster-wide orchestration | Running containers manually does not provide a complete solution for scheduling and managing workloads across a cluster.         |
| Scaling and recovery                   | Automated scaling, replacement of failed workloads, and rescheduling require orchestration capabilities.                         |

Important: Docker Engine is absolutely usable in production. The limitation is that running containers alone does not provide all the cluster-wide orchestration capabilities needed by complex production environments.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



### Why container orchestration?

Container orchestration automates the deployment, scheduling, scaling, networking, and management of containers across multiple machines.

For example:

- An application has 500 container instances.
- They are distributed across 30 worker nodes.
- A node fails.
- The orchestration platform detects the failure and arranges replacement workloads on healthy nodes.
- If traffic increases, additional replicas can be created.

Without orchestration, managing these operations manually becomes difficult.

### Docker Swarm vs Kubernetes

| Docker Swarm                                       | Kubernetes                                                                     |
| -------------------------------------------------- | ------------------------------------------------------------------------------ |
| Docker's built-in orchestration solution           | Open-source container orchestration platform                                   |
| Relatively simple to set up and operate            | More extensive features and greater operational complexity                     |
| Suitable for simpler orchestration requirements    | Commonly used for complex, large-scale environments                            |
| Provides services, replicas and overlay networking | Provides advanced scheduling, controllers, networking and storage abstractions |

The choice depends on application complexity and operational requirements.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub



## 2. What is Kubernetes?

Kubernetes (K8s) is an open-source container orchestration platform that automates:

- Deployment of containerized applications
- Scaling of applications
- Scheduling workloads across nodes
- Self-healing and recovery
- Service discovery and load balancing
- Configuration and resource management

Kubernetes was originally developed at Google and is now an open-source project hosted by the Cloud Native Computing Foundation (CNCF).

K8s terminology: The number 8 represents the eight letters between K and S in Kubernetes.

### Key capabilities

| Capability            | Explanation                                                     | Example                                              |
| --------------------- | --------------------------------------------------------------- | ---------------------------------------------------- |
| Automated deployment  | Deploy workloads using declarative configuration                | Apply a YAML deployment file                         |
| Self-healing          | Replace failed workloads to restore the desired state           | Maintain 3 replicas when one Pod fails               |
| Scaling               | Increase or decrease the number of replicas                     | Scale from 3 to 5 Pods                               |
| Load balancing        | Distribute traffic across application instances                 | Send traffic to available application replicas       |
| Resource optimization | Schedule workloads based on available resources and constraints | Place a Pod on a node with sufficient CPU and memory |

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

### Declarative vs imperative configuration

Imperative: Tell Kubernetes exactly which action to perform.

```
kubectl create deployment nginx --image=nginx
kubectl scale deployment nginx --replicas=3
```

Declarative: Define the desired state in a configuration file and let Kubernetes reconcile the actual state with it.

```
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
          image: nginx:latest
          ports:
            - containerPort: 80
```

```
kubectl apply -f deployment.yaml
```

Why declarative configuration matters: You describe what you want, and Kubernetes continuously works to achieve and maintain that state.

## 3. Kubernetes cluster architecture

A Kubernetes cluster is a logical grouping of machines (physical servers or virtual machines) that work together to run containerized applications.

A cluster has two major parts:

1. Control plane — manages and coordinates the cluster.
2. Worker nodes — run the application workloads.

[Medium](https://images.openai.com/static-rsc-4/2vDFHqyK4Ad75qM6I0kvMzisyI8KXCe4CDNovVnj7h7ApF4WO5TFPnuIl19pZQdgtU9Ov4Lft6WFBmPw5P4WIKIoAr96je-FqmzhU0VBEJVQcofJwR9XP_0M3LDrRNiAuBIPZ2-stdAm_H46XxRLyIrAi4y0B_tMvgSs4tCe5f0?purpose=inline)

[medium.com](https://medium.com/%40vijaykumarbangaru45/medium-article-day-3-8989319f5f83?utm_source=chatgpt.com)

### High-level architecture

\#chatgpt-mermaid-\_r_3fq\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_3fq\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_3fq\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_3fq\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_3fq\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_3fq\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_3fq\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_3fq\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_3fq\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_3fq\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_3fq\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_3fq\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_3fq\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_3fq\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_3fq\_ p{margin:0;}#chatgpt-mermaid-\_r_3fq\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_3fq\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_3fq\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_3fq\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_3fq\_ .label text,#chatgpt-mermaid-\_r_3fq\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_3fq\_ .node rect,#chatgpt-mermaid-\_r_3fq\_ .node circle,#chatgpt-mermaid-\_r_3fq\_ .node ellipse,#chatgpt-mermaid-\_r_3fq\_ .node polygon,#chatgpt-mermaid-\_r_3fq\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_3fq\_ .rough-node .label text,#chatgpt-mermaid-\_r_3fq\_ .node .label text,#chatgpt-mermaid-\_r_3fq\_ .image-shape .label,#chatgpt-mermaid-\_r_3fq\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_3fq\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_3fq\_ .rough-node .label,#chatgpt-mermaid-\_r_3fq\_ .node .label,#chatgpt-mermaid-\_r_3fq\_ .image-shape .label,#chatgpt-mermaid-\_r_3fq\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_3fq\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_3fq\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_3fq\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_3fq\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_3fq\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_3fq\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_3fq\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_3fq\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_3fq\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_3fq\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_3fq\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_3fq\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_3fq\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_3fq\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_3fq\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_3fq\_ .icon-shape,#chatgpt-mermaid-\_r_3fq\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_3fq\_ .icon-shape p,#chatgpt-mermaid-\_r_3fq\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_3fq\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_3fq\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_3fq\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_3fq\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_3fq\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_3fq\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_3fq\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_3fq\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_3fq\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_3fq\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_3fq\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_3fq\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_3fq\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_3fq\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_3fq\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_3fq\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_3fq\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_3fq\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_3fq\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_3fq\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_3fq\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_3fq\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_3fq\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_3fq\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_3fq\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_3fq\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_3fq\_ .node rect,#chatgpt-mermaid-\_r_3fq\_ .node circle,#chatgpt-mermaid-\_r_3fq\_ .node ellipse,#chatgpt-mermaid-\_r_3fq\_ .node polygon,#chatgpt-mermaid-\_r_3fq\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_3fq\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_3fq\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_3fq\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_3fq\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_3fq\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}Kubernetes ClusterWorker Node 2Worker Node 1Control Planekubeletkube-proxyPods / ContainersContainer Runtimekubeletkube-proxyPods / ContainersContainer RuntimeKube API ServeretcdSchedulerController ManagerCloud Controller Manager(optional)User / kubectl

The control plane makes cluster-wide decisions, while worker nodes execute the workloads.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+2



### 3.1 Control plane

The control plane is the management layer of Kubernetes.

Its responsibilities include:

- Managing worker nodes and workloads.
- Accepting and validating requests.
- Scheduling Pods onto appropriate nodes.
- Maintaining the desired cluster state.
- Managing cluster configuration and state.

Think of it as the control room of a building: it coordinates what happens throughout the building without performing every activity itself.

A cluster can have multiple worker nodes. Production clusters can also have multiple control-plane instances for high availability.

### 3.2 Worker node

A worker node is a physical machine or virtual machine where application workloads run.

Each worker node runs the components required to manage Pods and communicate with the control plane.

A useful distinction:

- Control plane: decides and manages.
- Worker node: executes the workload.

### Cluster vs server

A cluster is a logical grouping, not necessarily one physical server.

For example, a Kubernetes cluster might contain:

- One control-plane VM.
- Three worker-node VMs.

Or it might contain multiple control-plane machines and many worker nodes.

Each independent Kubernetes cluster has its own control plane. One cluster's control plane does not automatically manage another cluster's workloads.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub



## 4. Worker-node components

The session introduces four important worker-node components:

1. Pod
2. kubelet
3. kube-proxy
4. Container runtime

### 4.1 Pod

A Pod is the smallest deployable unit in Kubernetes.

- A Pod can contain one container or multiple containers.
- Containers inside the same Pod share the Pod's network namespace and can communicate using `localhost`.
- Containers in a Pod can also share storage volumes.
- Pods are scheduled onto worker nodes.

Example:

```
Kubernetes Cluster
   |
   └── Worker Node
         |
         └── Pod
              |
              ├── Application Container
              └── Sidecar Container
```

Docker vs Kubernetes terminology

| Docker                              | Kubernetes                                            |
| ----------------------------------- | ----------------------------------------------------- |
| Container is the basic runtime unit | Pod is the smallest deployable unit                   |
| A container runs from an image      | A Pod runs one or more containers created from images |

A Pod is not simply another name for a container. It is a higher-level abstraction that can group multiple containers.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub



### 4.2 kubelet

kubelet is an agent that runs on every worker node.

Responsibilities:

- Communicates with the Kubernetes API server.
- Receives information about the Pods that should run on its node.
- Ensures the required containers are running and healthy.
- Reports node and Pod status to the control plane.

A useful comparison from the session is the Jenkins agent:

- Jenkins controller manages build agents.
- Kubernetes control plane manages worker nodes through kubelet.

If kubelet stops working, the control plane loses normal kubelet-based management and status communication with that node. Already-running containers may continue running, but new instructions and status updates can be affected.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub



### 4.3 kube-proxy

kube-proxy implements Kubernetes Service networking rules on nodes.

Its responsibilities include:

- Maintaining network rules for Kubernetes Services.
- Helping route traffic to the appropriate backend Pods.
- Supporting communication through Service virtual IPs.

For example, an application might have three backend Pods distributed across different worker nodes. A Kubernetes Service provides a stable endpoint for accessing those Pods.

Important correction

kube-proxy does not create the entire Kubernetes Pod network. The actual Pod-to-Pod networking is generally provided by a Container Network Interface (CNI) plugin, such as Calico or Flannel. kube-proxy handles Service traffic rules where that component is used.

### 4.4 Container runtime

A container runtime is responsible for running containers on worker nodes.

Examples:

- containerd
- CRI-O

Kubernetes communicates with runtimes through the Container Runtime Interface (CRI).

Important correction

Docker Engine is not required on every Kubernetes worker node. Kubernetes historically supported Docker through dockershim, but dockershim was removed in Kubernetes 1.24. Modern clusters commonly use containerd or CRI-O directly.

| Term          | Meaning                                                 |
| ------------- | ------------------------------------------------------- |
| CRI           | Container Runtime Interface — the integration API       |
| containerd    | Container runtime                                       |
| CRI-O         | Container runtime designed for Kubernetes               |
| Docker Engine | Container platform/engine; not a Kubernetes requirement |

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

### Worker-node component summary

| Component         | Main responsibility                                           |
| ----------------- | ------------------------------------------------------------- |
| Pod               | Runs one or more application containers                       |
| kubelet           | Communicates with the API server and manages Pods on the node |
| kube-proxy        | Implements Service networking rules                           |
| Container runtime | Executes containers                                           |

## 5. Control-plane components

The instructor explains five control-plane components:

1. kube-apiserver
2. etcd
3. kube-scheduler
4. kube-controller-manager
5. cloud-controller-manager (optional, depending on the environment)

### 5.1 kube-apiserver

kube-apiserver is the entry point to the Kubernetes control plane.

Whenever you execute a Kubernetes operation, the request goes through the API server.

Examples:

- Create a Pod.
- Deploy an application.
- Scale an application.
- Delete a deployment.
- Add or manage cluster resources.

Responsibilities:

- Exposes the Kubernetes API.
- Authenticates and authorizes requests.
- Performs admission control and validation.
- Acts as the central communication interface for control-plane components.
- Reads and persists cluster state through etcd.

Example:

```
kubectl get nodes
kubectl get pods
kubectl apply -f deployment.yaml
```

These commands communicate with the Kubernetes API server.

Think of the API server as a gatekeeper: it checks requests and makes the Kubernetes API available to users and other components.

What happens if the API server goes down?

- New API requests cannot be processed.
- Existing workloads may continue running.
- Normal management operations, such as scheduling and reconciliation, are disrupted.
- Failed Pods may not be replaced until control-plane functionality is restored.

The API server is therefore a critical component. Production environments use highly available control planes to reduce this risk.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub



### 5.2 etcd

etcd is a strongly consistent, distributed key-value store used by Kubernetes to store cluster state.

It stores information such as:

- Cluster configuration.
- Node information.
- Pod and Deployment state.
- Kubernetes object metadata.
- Service and endpoint information.
- Other Kubernetes API objects and their state.

Example of a key-value pair:

```
Key:   cluster/node/worker-1
Value: Worker node configuration
```

Why is etcd important?

- It stores the persistent state of the Kubernetes cluster.
- The API server uses it to retrieve and update cluster data.
- Losing access to etcd affects the control plane's ability to manage cluster state.

Backup recommendation: Regularly back up etcd and test the restore procedure. It is one of the most important parts of Kubernetes disaster recovery.

etcd vs ConfigMap

| etcd                                            | ConfigMap                        |
| ----------------------------------------------- | -------------------------------- |
| Stores Kubernetes cluster state                 | Stores application configuration |
| Used internally by the Kubernetes control plane | Used by applications and Pods    |
| A backing data store                            | A Kubernetes API object          |

A ConfigMap is itself stored in etcd as part of Kubernetes state; it is not a separate replacement database.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub



### 5.3 kube-scheduler

kube-scheduler decides which worker node should run a newly created, unscheduled Pod.

Suppose a cluster has five worker nodes:

- Node 1: 2 GB available memory.
- Node 2: 4 GB available memory.
- Node 3: 8 GB available memory.
- Node 4: 12 GB available memory.
- Node 5: 16 GB available memory.

A Pod requests 6 GB of memory.

The scheduler evaluates the available nodes and excludes those that cannot satisfy the resource requirements.

It considers factors such as:

- CPU and memory requests.
- Node selectors and node affinity.
- Taints and tolerations.
- Hardware and software constraints.
- Other scheduling policies.

It then selects a suitable node and records the scheduling decision through the API server.

Important: The scheduler selects a node; kubelet on that node then works with the container runtime to start the Pod's containers.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



### 5.4 kube-controller-manager

kube-controller-manager runs controllers that continuously monitor cluster state and work to maintain the desired state.

A controller compares:

- Desired state — what the user has declared.
- Actual state — what is currently running.

Example:

You define a Deployment with five replicas.

```
spec:
  replicas: 5
```

Suppose two Pods fail or are deleted.

| State                      | Pod count                 |
| -------------------------- | ------------------------- |
| Desired state              | 5                         |
| Actual state after failure | 3                         |
| Required action            | Create 2 replacement Pods |

The relevant controllers detect the difference and take action to restore the desired state.

Examples of controllers:

| Controller                        | Responsibility                                      |
| --------------------------------- | --------------------------------------------------- |
| Node controller                   | Monitors node health and responds to node failures  |
| Job controller                    | Manages Jobs and their completion                   |
| Deployment/ReplicaSet controllers | Maintain the desired number of application replicas |
| Service controller                | Helps manage Service-related resources              |

The controller manager runs multiple controller processes; they are not separate control-plane machines.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



### 5.5 cloud-controller-manager

The cloud-controller-manager integrates Kubernetes with an external cloud provider.

It allows Kubernetes to interact with cloud infrastructure through provider APIs.

Examples of cloud-related operations include:

- Managing cloud load balancers.
- Managing routes.
- Integrating node information with cloud infrastructure.
- Managing cloud-backed resources.

It is relevant when Kubernetes is integrated with a cloud provider. Its exact responsibilities depend on the provider and cluster setup.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub



### Control-plane component summary

| Component                | Main responsibility                        | Easy way to remember    |
| ------------------------ | ------------------------------------------ | ----------------------- |
| kube-apiserver           | Accepts and validates API requests         | Entry gate              |
| etcd                     | Stores persistent cluster state            | Cluster database        |
| kube-scheduler           | Selects a node for a new Pod               | Placement decision      |
| kube-controller-manager  | Reconciles actual state with desired state | Maintains desired state |
| cloud-controller-manager | Integrates with cloud infrastructure       | Cloud integration       |

## 6. How Kubernetes components work together

This is one of the most important interview topics: explain the complete workflow rather than only defining individual components.

### Example: Deploying an NGINX application

Step 1 — Build the Docker image

Create the image using a Dockerfile.

```
docker build -t my-nginx:1.0 .
```

Push the image to a container registry accessible to the cluster.

Step 2 — Create a Kubernetes YAML file

Reference the image in a Pod or Deployment manifest.

Example `deployment.yaml`:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
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
          image: my-nginx:1.0
          ports:
            - containerPort: 80
```

Step 3 — Submit the YAML file

```
kubectl apply -f deployment.yaml
```

Step 4 — API server receives the request

- Receives the request from `kubectl`.
- Authenticates and authorizes the user.
- Validates the configuration.
- Persists the accepted desired state through etcd.

Step 5 — Controllers and scheduler act

- Deployment and ReplicaSet controllers ensure the required Pods are created.
- The scheduler selects suitable worker nodes for unscheduled Pods.

Step 6 — kubelet starts the Pods

- kubelet on each selected worker node observes the assigned Pod.
- It asks the container runtime to pull the image if necessary and start the containers.
- Pod status is reported back to the API server.

Step 7 — Continuous monitoring

- Controllers keep checking the desired state.
- If a Pod fails, Kubernetes works to restore the required replica count.

### Deployment workflow diagram

\#chatgpt-mermaid-\_r_3iu\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_3iu\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_3iu\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_3iu\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_3iu\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_3iu\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_3iu\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_3iu\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_3iu\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_3iu\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_3iu\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_3iu\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_3iu\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_3iu\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_3iu\_ p{margin:0;}#chatgpt-mermaid-\_r_3iu\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_3iu\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_3iu\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_3iu\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_3iu\_ .label text,#chatgpt-mermaid-\_r_3iu\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_3iu\_ .node rect,#chatgpt-mermaid-\_r_3iu\_ .node circle,#chatgpt-mermaid-\_r_3iu\_ .node ellipse,#chatgpt-mermaid-\_r_3iu\_ .node polygon,#chatgpt-mermaid-\_r_3iu\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_3iu\_ .rough-node .label text,#chatgpt-mermaid-\_r_3iu\_ .node .label text,#chatgpt-mermaid-\_r_3iu\_ .image-shape .label,#chatgpt-mermaid-\_r_3iu\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_3iu\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_3iu\_ .rough-node .label,#chatgpt-mermaid-\_r_3iu\_ .node .label,#chatgpt-mermaid-\_r_3iu\_ .image-shape .label,#chatgpt-mermaid-\_r_3iu\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_3iu\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_3iu\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_3iu\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_3iu\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_3iu\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_3iu\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_3iu\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_3iu\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_3iu\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_3iu\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_3iu\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_3iu\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_3iu\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_3iu\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_3iu\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_3iu\_ .icon-shape,#chatgpt-mermaid-\_r_3iu\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_3iu\_ .icon-shape p,#chatgpt-mermaid-\_r_3iu\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_3iu\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_3iu\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_3iu\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_3iu\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_3iu\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_3iu\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_3iu\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_3iu\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_3iu\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_3iu\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_3iu\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_3iu\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_3iu\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_3iu\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_3iu\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_3iu\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_3iu\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_3iu\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_3iu\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_3iu\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_3iu\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_3iu\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_3iu\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_3iu\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_3iu\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_3iu\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_3iu\_ .node rect,#chatgpt-mermaid-\_r_3iu\_ .node circle,#chatgpt-mermaid-\_r_3iu\_ .node ellipse,#chatgpt-mermaid-\_r_3iu\_ .node polygon,#chatgpt-mermaid-\_r_3iu\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_3iu\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_3iu\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_3iu\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_3iu\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_3iu\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}Developer builds Docker imagePush image to registryWrite Kubernetes YAMLkubectl applykube-apiserverAuthentication, authorizationand validationPersist desired state in etcdDeployment / ReplicaSetcontrollersCreate Podskube-scheduler selects workernodeskubelet on selected nodesContainer runtime pulls imageand starts containersPods runningControllers continuouslyreconcile state

Important distinction: The scheduler does not create Pods. Controllers create the required Pod objects, and the scheduler assigns unscheduled Pods to nodes. The API server is the communication interface between these components.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+2



## 7. Dockerfile vs Kubernetes YAML

One important question raised during the session was whether Kubernetes replaces the Dockerfile.

No. They serve different purposes.

| Dockerfile                                                 | Kubernetes YAML                                                            |
| ---------------------------------------------------------- | -------------------------------------------------------------------------- |
| Defines how to build a container image                     | Defines how Kubernetes should run and manage workloads                     |
| Used during image creation                                 | Used during deployment and management                                      |
| Contains instructions such as `FROM`, `COPY`, `RUN`, `CMD` | Contains Kubernetes objects such as Deployment, Pod, Service and ConfigMap |
| Produces a container image                                 | References an existing image                                               |

Typical workflow:

```
Application Source Code
        |
        v
    Dockerfile
        |
        v
   Docker Image
        |
        v
   Image Registry
        |
        v
 Kubernetes YAML
        |
        v
   Kubernetes
        |
        v
       Pods
```

Example files in a project:

```
my-application/
├── Dockerfile
├── deployment.yaml
├── service.yaml
└── configmap.yaml
```

You can use Docker-compatible images built with Docker, BuildKit, Kaniko, Buildah or other image-building tools. Kubernetes does not require Docker Engine to build or run them.

## 8. Managed Kubernetes services in cloud platforms

Managing Kubernetes yourself involves installing, configuring, maintaining and upgrading its components.

Cloud providers offer managed Kubernetes services to reduce this operational burden.

| Cloud platform  | Managed Kubernetes service |
| --------------- | -------------------------- |
| AWS             | Amazon EKS                 |
| Microsoft Azure | Azure AKS                  |
| Google Cloud    | Google GKE                 |

### DIY vs managed Kubernetes

| DIY Kubernetes                                 | Managed Kubernetes                                        |
| ---------------------------------------------- | --------------------------------------------------------- |
| You install and configure the control plane    | Cloud provider manages the control plane                  |
| You manage etcd and control-plane availability | Provider manages the managed control-plane infrastructure |
| You plan and execute upgrades                  | Provider offers managed upgrade capabilities              |
| You handle more infrastructure maintenance     | Less control-plane maintenance                            |
| More operational responsibility                | Reduced operational responsibility                        |

What does the customer still manage?

Even when using EKS, AKS or GKE, teams remain responsible for many things, including:

- Application deployments and YAML manifests.
- Container images and registries.
- Application configuration and secrets.
- Workload resource requests and limits.
- Access policies and workload security.
- Networking and storage configuration according to the service model.
- Worker nodes when using self-managed node groups.

The exact division of responsibilities differs between providers and deployment options.

### Important terminology

- EKS: Amazon Elastic Kubernetes Service.
- AKS: Azure Kubernetes Service.
- GKE: Google Kubernetes Engine.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 9. Important interview questions

### Q1. What is Kubernetes?

Kubernetes is an open-source container orchestration platform used to automate the deployment, scaling and management of containerized applications across multiple machines.

### Q2. What is a Kubernetes cluster?

A Kubernetes cluster is a logical grouping of control-plane components and worker nodes that work together to run and manage containerized workloads.

### Q3. What is the difference between a control plane and a worker node?

The control plane manages the cluster, handles API requests, schedules workloads and maintains the desired state. Worker nodes run the actual application Pods.

### Q4. What is a Pod?

A Pod is the smallest deployable unit in Kubernetes. It can contain one or more containers that share networking and can share storage volumes.

### Q5. What is the difference between kubelet and kube-proxy?

- kubelet communicates with the API server and manages Pods on a worker node.
- kube-proxy implements Service networking rules and helps route traffic to backend Pods.

### Q6. What is etcd?

etcd is a strongly consistent distributed key-value store that Kubernetes uses to persist cluster state and configuration.

### Q7. Why is etcd backup important?

Because etcd contains the persistent state of the Kubernetes cluster. A valid backup is important for disaster recovery.

### Q8. What is the role of the scheduler?

The scheduler selects an appropriate worker node for a newly created Pod based on resource availability, scheduling constraints, affinity and other policies.

### Q9. What is the role of the controller manager?

It runs controllers that continuously compare actual cluster state with desired state and take corrective action when they differ.

### Q10. What happens if the API server goes down?

New API requests and normal control-plane management operations are disrupted. Existing workloads may continue running, but Kubernetes cannot normally process new changes or reconcile failures until API access is restored.

### Q11. What is the difference between Dockerfile and Kubernetes YAML?

A Dockerfile builds a container image. Kubernetes YAML defines how Kubernetes should deploy and manage that image.

### Q12. What are EKS, AKS and GKE?

They are managed Kubernetes services provided by AWS, Azure and Google Cloud respectively.

## 10. Quick revision — Kubernetes architecture

Control plane

| Component                | Remember                |
| ------------------------ | ----------------------- |
| kube-apiserver           | Entry point             |
| etcd                     | Stores cluster state    |
| kube-scheduler           | Selects worker node     |
| kube-controller-manager  | Maintains desired state |
| cloud-controller-manager | Cloud integration       |

Worker node

| Component         | Remember                 |
| ----------------- | ------------------------ |
| Pod               | Runs containers          |
| kubelet           | Node agent               |
| kube-proxy        | Service networking rules |
| Container runtime | Executes containers      |

### Five concepts to remember

1. Pod: Smallest deployable unit in Kubernetes.
2. kube-apiserver: Entry point for Kubernetes API requests.
3. etcd: Stores persistent cluster state.
4. Scheduler: Decides where a Pod should run.
5. Controller manager: Maintains the desired state.

### Final mental model

```
API Server  = Receives and validates requests
etcd        = Stores cluster state
Scheduler   = Selects a worker node
Controllers = Maintain desired state
kubelet     = Manages Pods on worker nodes
Runtime     = Runs containers
kube-proxy  = Implements Service networking rules
```

Recommended reading: [Kubernetes official documentation — Cluster Architecture](https://kubernetes.io/docs/concepts/architecture/?utm_source=chatgpt.com).

Source transcript: [DevOps Session — 6 October 2026](https://github.com/jeevanadv2000-pixel/python/blob/main/DevOps%20Session\(Morning\)%20transcript_2026-10-06_09.22.39.txt?utm_source=chatgpt.com).

My recommendation for revision: Spend extra time explaining the difference between kubelet, kube-proxy, scheduler and controller manager. Interviewers often ask how these components work together, not just what each one means.