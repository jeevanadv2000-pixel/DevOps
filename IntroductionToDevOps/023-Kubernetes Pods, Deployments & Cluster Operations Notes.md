# DevOps Session — Kubernetes Pods, Deployments & Cluster Operations Notes

Date: 9 October 2026\
Instructor: Rochak Agrawal\
Focus: Creating a managed Kubernetes cluster, connecting with `kubectl`, public vs private clusters, Pods and their lifecycle, multi-container Pod patterns, Deployments, ReplicaSets, scaling and self-healing.

These notes condense the instructor's explanations and hands-on demonstration, remove participant discussions and repetition, and correct technical oversimplifications where necessary.

## 1. Creating a managed Kubernetes cluster

The session moves from Kubernetes theory to practical implementation using Azure Kubernetes Service (AKS).

The instructor explains that the same Kubernetes commands can be used with AKS, EKS and GKE. The cloud-provider-specific steps are mainly involved in creating and connecting to the cluster.

### What happens when you create a managed cluster?

A managed Kubernetes cluster generally consists of:

1. Control plane: Managed by the cloud provider.
2. Node pools: Groups of worker nodes where application workloads run.

[Thomas Van Laere | Kubernetes in a Microsoft World - Part 2](https://images.openai.com/static-rsc-4/s2wfqwWbvl5OFC8dwVshO9a6YoDd-toAZgBo-4A_lfjw7BaHyg-0edaMcrRjaSzdY7JmrtEnZLNMLTH9zQnxPGtgJlLXX-q8DFlq-Tf4hl-DxafJW72-UWlURgcBx7TaYYkTFliLb6jCyBVU9FadficeTyRdmZ2zEJkDdzb2qNU?purpose=inline)

[thomasvanlaere.com](https://thomasvanlaere.com/posts/2019/10/kubernetes-in-a-microsoft-world-part-2/?utm_source=chatgpt.com)

### What is a node pool?

A node pool is a group of worker nodes with a common configuration.

For example:

| Node pool        | Purpose                                   |
| ---------------- | ----------------------------------------- |
| System node pool | Runs critical Kubernetes system workloads |
| User node pool   | Runs application workloads                |

You can create multiple node pools with different configurations.

Example:

```
AKS Cluster
    |
    |-- Managed Control Plane
    |
    |-- System Node Pool
    |      |-- Worker Node 1
    |      |-- Worker Node 2
    |
    |-- User Node Pool
           |-- Worker Node 3
           |-- Worker Node 4
```

Why use multiple node pools?

- Separate system workloads from application workloads.
- Use different VM sizes for different workloads.
- Use GPU-enabled nodes for specialized applications.
- Apply different scaling configurations.
- Isolate workloads based on operational requirements.

### System vs user node pools

| System node pool                                      | User node pool                            |
| ----------------------------------------------------- | ----------------------------------------- |
| Intended for critical Kubernetes system Pods          | Intended primarily for application Pods   |
| Runs infrastructure-related workloads                 | Runs business application workloads       |
| Should have sufficient capacity for system components | Can be scaled based on application demand |

A system node pool does not automatically guarantee that only system Pods run there. Scheduling controls, labels and taints help enforce workload placement.

### Node pool scaling

The instructor demonstrates configuring a minimum and maximum number of nodes.

For example:

```
Minimum nodes = 2
Maximum nodes = 5
```

The node pool can start with two nodes and scale up to five when autoscaling conditions require it.

You can also manually change the number of nodes.

Important distinction:

- Node scaling increases or decreases the number of worker machines.
- Pod scaling increases or decreases the number of application instances.

These are two different operations.

### Kubernetes version and upgrades

Kubernetes releases new versions periodically. Managed Kubernetes providers support specific version ranges and upgrade schedules.

The instructor discusses the `N-2` version concept, meaning that a provider may support a limited number of minor versions behind the latest version.

Technical clarification: Do not treat N-2 as a universal Kubernetes rule. Supported versions, release channels, patch updates and upgrade deadlines vary by cloud provider and service policy.

In production, teams should:

- Monitor Kubernetes version support.
- Plan control-plane and node-pool upgrades.
- Test application compatibility.
- Perform upgrades in a controlled manner.
- Follow the provider's supported upgrade sequence.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+2

## 2. Connecting to a Kubernetes cluster

Once the cluster is created, you need a way to communicate with its API server.

The instructor demonstrates connecting to AKS using Azure CLI.

### What is kubectl?

`kubectl` is the Kubernetes command-line tool used to communicate with the Kubernetes API server.

It is used to:

- Create resources.
- Deploy applications.
- View Pods and nodes.
- Scale deployments.
- Inspect resource details.
- Delete Kubernetes objects.

The important point is that `kubectl` is not specific to AKS. It works with any Kubernetes cluster when configured with the appropriate credentials and endpoint.

### What is kubeconfig?

When you connect to a managed Kubernetes cluster, the provider supplies the necessary connection and authentication configuration.

This information is stored in a file called:

```
~/.kube/config
```

The kubeconfig file typically contains:

| Field           | Purpose                                                 |
| --------------- | ------------------------------------------------------- |
| Clusters        | Cluster API server endpoint and certificate information |
| Users           | Authentication configuration                            |
| Contexts        | Associates a cluster, user and optional namespace       |
| Current context | Identifies the context used by default                  |

Example structure:

```
apiVersion: v1
kind: Config

clusters:
  - name: my-cluster
    cluster:
      server: https://kubernetes-api.example.com

users:
  - name: my-user
    user:
      token: REPLACE_WITH_AUTH_TOKEN

contexts:
  - name: my-context
    context:
      cluster: my-cluster
      user: my-user
      namespace: default

current-context: my-context
```

Illustrative structure only. Real managed-cluster configurations may use cloud authentication plugins or certificates rather than a static token.

### AKS connection example

```
az login
```

Get the cluster credentials:

```
az aks get-credentials \
  --resource-group my-resource-group \
  --name my-aks-cluster
```

Check the current context:

```
kubectl config current-context
```

Check cluster connectivity:

```
kubectl cluster-info
```

List nodes:

```
kubectl get nodes
```

### What happens when you execute kubectl?

\#chatgpt-mermaid-\_r_41g\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_41g\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_41g\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_41g\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_41g\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_41g\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_41g\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_41g\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_41g\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_41g\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_41g\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_41g\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_41g\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_41g\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_41g\_ p{margin:0;}#chatgpt-mermaid-\_r_41g\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_41g\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_41g\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_41g\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_41g\_ .label text,#chatgpt-mermaid-\_r_41g\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_41g\_ .node rect,#chatgpt-mermaid-\_r_41g\_ .node circle,#chatgpt-mermaid-\_r_41g\_ .node ellipse,#chatgpt-mermaid-\_r_41g\_ .node polygon,#chatgpt-mermaid-\_r_41g\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_41g\_ .rough-node .label text,#chatgpt-mermaid-\_r_41g\_ .node .label text,#chatgpt-mermaid-\_r_41g\_ .image-shape .label,#chatgpt-mermaid-\_r_41g\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_41g\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_41g\_ .rough-node .label,#chatgpt-mermaid-\_r_41g\_ .node .label,#chatgpt-mermaid-\_r_41g\_ .image-shape .label,#chatgpt-mermaid-\_r_41g\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_41g\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_41g\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_41g\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_41g\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_41g\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_41g\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_41g\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_41g\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_41g\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_41g\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_41g\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_41g\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_41g\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_41g\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_41g\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_41g\_ .icon-shape,#chatgpt-mermaid-\_r_41g\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_41g\_ .icon-shape p,#chatgpt-mermaid-\_r_41g\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_41g\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_41g\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_41g\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_41g\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_41g\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_41g\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_41g\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_41g\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_41g\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_41g\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_41g\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_41g\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_41g\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_41g\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_41g\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_41g\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_41g\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_41g\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_41g\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_41g\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_41g\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_41g\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_41g\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_41g\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_41g\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_41g\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_41g\_ .node rect,#chatgpt-mermaid-\_r_41g\_ .node circle,#chatgpt-mermaid-\_r_41g\_ .node ellipse,#chatgpt-mermaid-\_r_41g\_ .node polygon,#chatgpt-mermaid-\_r_41g\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_41g\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_41g\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_41g\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_41g\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_41g\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}User runs kubectl commandkubectl reads kubeconfigConnects to Kubernetes APIServerAPI Server authenticates andauthorizes requestProcesses Kubernetes APIrequestControl plane coordinates theoperationWorker nodes executeworkload changes

Interview point: Users normally communicate with the API server, not directly with individual worker nodes. The control plane coordinates the required operations with the worker nodes.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 3. Public vs private Kubernetes clusters

Very important interview topic

The instructor emphasizes that people often confuse public/private clusters with public/private applications.

These are different concepts.

### Public Kubernetes cluster

A public cluster generally means the Kubernetes API server endpoint is accessible through a public network endpoint, subject to authentication, authorization and any network access restrictions.

For example:

- An administrator can connect from an external laptop.
- `kubectl` can communicate with the API server over the internet.
- The API server still requires valid authentication and authorization.

### Private Kubernetes cluster

A private cluster generally means the Kubernetes API server endpoint is private, accessible through the appropriate private network connectivity.

For example:

- The API server is reachable through a private IP or private network path.
- An administrator may need VPN, a bastion host, peering or another private connection.
- A laptop outside that network cannot directly reach the private API endpoint.

### Comparison

| Public cluster                                    | Private cluster                                     |
| ------------------------------------------------- | --------------------------------------------------- |
| API server has a public network endpoint          | API server uses private network access              |
| Can be accessed externally if network rules allow | Requires private network connectivity               |
| Requires authentication and authorization         | Also requires authentication and authorization      |
| Public endpoint exposure must be controlled       | Reduces direct public exposure of the control plane |

### Important interview clarification

A private Kubernetes cluster does not mean that all applications running inside it must be private.

For example, imagine a private AKS cluster:

- The Kubernetes API server is private.
- The application runs on worker nodes.
- An external load balancer or ingress gateway exposes the application to the internet.

Therefore, the application can be publicly accessible even though the Kubernetes API server is private.

Private Kubernetes cluster with a public application

Administrator

VPN / private connectivity

Private Kubernetes API server

Public users

Internet access

Public Load Balancer / Ingress

Application Pods on worker nodes

Remember: Public/private cluster classification is primarily about the API server's network accessibility, not whether the application is internet-facing.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 4. Basic kubectl commands

The instructor demonstrates common commands after connecting to the cluster.

### 4.1 Get commands

List Pods:

```
kubectl get pods
```

List Pods in a specific namespace:

```
kubectl get pods -n kube-system
```

List all namespaces:

```
kubectl get namespaces
```

Short form:

```
kubectl get ns
```

List nodes:

```
kubectl get nodes
```

List deployments:

```
kubectl get deployments
```

### 4.2 View additional details

The `-o wide` option displays additional information.

```
kubectl get pods -o wide
```

This can show:

- Pod IP address.
- Node on which the Pod is running.
- Additional status information.

For detailed resource information:

```
kubectl describe pod nginx
```

### 4.3 View logs

```
kubectl logs nginx
```

For a multi-container Pod:

```
kubectl logs nginx -c app-container
```

### 4.4 View cluster information

```
kubectl cluster-info
```

### Common kubectl commands

| Command            | Purpose                               |
| ------------------ | ------------------------------------- |
| `kubectl get`      | List resources                        |
| `kubectl describe` | Display detailed resource information |
| `kubectl create`   | Create a resource                     |
| `kubectl apply`    | Apply a configuration                 |
| `kubectl edit`     | Edit a live resource                  |
| `kubectl delete`   | Delete a resource                     |
| `kubectl logs`     | View container logs                   |
| `kubectl exec`     | Execute a command inside a container  |
| `kubectl scale`    | Change replica count                  |
| `kubectl explain`  | View resource field documentation     |

Use help whenever you are unsure about a command:

```
kubectl get --help
kubectl explain pod
kubectl explain deployment.spec
```

### Namespace behavior

If you run:

```
kubectl get pods
```

Kubernetes displays Pods in the current namespace configured in your context. If no namespace is configured, this is normally `default`.

To list Pods across all namespaces:

```
kubectl get pods -A
```

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 5. Understanding Kubernetes Pods

A Pod is the smallest deployable unit in Kubernetes.

A Pod can contain:

- One container (the most common pattern).
- Multiple containers that need to work closely together.

Kubernetes schedules and manages Pods, rather than scheduling individual containers independently.

### Container vs Pod

| Container                                                | Pod                                                        |
| -------------------------------------------------------- | ---------------------------------------------------------- |
| Runs an application or process                           | Kubernetes deployable unit                                 |
| Managed by a container runtime                           | Managed through Kubernetes                                 |
| Has its own container filesystem and process environment | Provides a shared execution environment for its containers |
| Does not independently provide Kubernetes scheduling     | Scheduled onto a worker node                               |

### What do containers inside a Pod share?

Containers inside the same Pod share:

- Network namespace and Pod IP address.
- Port space.
- Access to explicitly shared volumes.
- The same node placement and Pod lifecycle.

They can communicate using `localhost`.

Important: Containers in a Pod do not automatically share their filesystems. They can share files through a common mounted volume.

### Example architecture

```
Kubernetes Cluster
       |
   Worker Node
       |
      Pod
       |
       |-- Application Container
       |
       |-- Logging Sidecar Container
```

Both containers run on the same node, share the Pod network namespace and can communicate over localhost.

### Why use multiple containers in one Pod?

Suppose you have a Java application that generates logs.

You can run:

- Main container: Java application.
- Sidecar container: Log collection agent.

The logging agent reads logs from a shared volume and forwards them to a logging platform such as Splunk or an ELK-based system.

This avoids putting logging-agent functionality directly into the application code.

### Pod resource sharing

Although containers share the Pod's network namespace, CPU and memory resource requests and limits are configured per container (with additional Pod-level resource features available in Kubernetes).

## 6. Creating your first Pod

The instructor demonstrates creating an NGINX Pod using a YAML manifest.

### Step 1 — Create a Pod YAML file

File: `pod.yaml`

```
apiVersion: v1
kind: Pod

metadata:
  name: nginx

spec:
  containers:
    - name: nginx
      image: nginx:1.27
      ports:
        - containerPort: 80
```

### Understanding the YAML

| Field             | Explanation                                       |
| ----------------- | ------------------------------------------------- |
| `apiVersion: v1`  | Uses the core Kubernetes API                      |
| `kind: Pod`       | Creates a Pod                                     |
| `metadata.name`   | Names the Pod `nginx`                             |
| `spec.containers` | Defines the containers inside the Pod             |
| `name`            | Container name                                    |
| `image`           | Container image to run                            |
| `containerPort`   | Documents the container's intended listening port |

Important: `containerPort: 80` does not publish the port externally. A Kubernetes Service or another networking mechanism is needed for application access.

### Step 2 — Create the Pod

```
kubectl apply -f pod.yaml
```

Expected output:

```
pod/nginx created
```

### Step 3 — Verify the Pod

```
kubectl get pods
```

Example output:

```
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          30s
```

### Understanding READY

`1/1` means:

- Total containers in the Pod: 1.
- Ready containers: 1.

If you see `2/3`, it means two of the three containers are ready.

### Step 4 — Find the Pod IP and node

```
kubectl get pods -o wide
```

This shows the Pod's IP address and the worker node hosting it.

### Step 5 — View detailed information

```
kubectl describe pod nginx
```

This displays:

- Pod name and namespace.
- Labels.
- Pod IP.
- Node assignment.
- Container image.
- Container state.
- Events.
- Volumes.
- Scheduling information.

### Step 6 — View logs

```
kubectl logs nginx
```

### Step 7 — Delete the Pod

```
kubectl delete pod nginx
```

Verify:

```
kubectl get pods
```

The Pod will disappear because it was created directly and has no controller responsible for recreating it.

This leads to the next important concept: Deployments.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 7. Pod lifecycle and phases

Important troubleshooting topic

A Pod moves through different lifecycle phases.

The instructor emphasizes that understanding these phases helps identify why an application is not running.

There are five official Pod phases:

1. Pending
2. Running
3. Succeeded
4. Failed
5. Unknown

### 7.1 Pending

The Pod has been accepted by Kubernetes, but one or more steps required for it to become fully operational are incomplete.

Possible reasons:

- No suitable worker node is available.
- Insufficient CPU or memory.
- Node selector does not match any available node.
- Taints are not tolerated.
- Container image is still being pulled.
- Persistent storage is not yet available.

Example:

Suppose all worker nodes have been cordoned.

You create a new Pod.

The scheduler cannot find an eligible node, so the Pod remains Pending.

### 7.2 Running

The Pod has been bound to a node, and its containers have been created. At least one container is running or starting.

This is the normal phase for long-running applications such as websites and APIs.

### 7.3 Succeeded

All containers in the Pod have terminated successfully.

This is common for batch-processing tasks and Jobs.

Example:

- Compress 100 images.
- Process employee salaries.
- Generate a report.
- Complete a scheduled data-processing task.

Once the task is complete, the container exits successfully.

### 7.4 Failed

All containers in the Pod have terminated, and at least one container terminated unsuccessfully.

Example:

- Application exits with a non-zero exit code.
- A batch-processing task fails.
- A required script cannot complete.

### 7.5 Unknown

Kubernetes cannot determine the current state of the Pod, commonly because it cannot communicate with the node.

### Pod phase summary

| Phase     | Meaning                                                               | Example                    |
| --------- | --------------------------------------------------------------------- | -------------------------- |
| Pending   | Waiting for scheduling or startup requirements                        | No suitable node           |
| Running   | Bound to a node; containers created, at least one running or starting | NGINX application          |
| Succeeded | All containers completed successfully                                 | Batch job                  |
| Failed    | All containers terminated, with at least one failure                  | Failed processing task     |
| Unknown   | State cannot be determined                                            | Node communication failure |

Important clarification

`CrashLoopBackOff` is not a Pod phase. It is a container waiting reason that can appear while the Pod phase is Running.

Similarly, `ImagePullBackOff` is a container waiting reason, not a Pod phase.

### How to troubleshoot a Pending Pod

Start with:

```
kubectl get pods
```

Then:

```
kubectl describe pod <pod-name>
```

Check the Events section at the bottom.

Common messages include:

```
Insufficient memory
Insufficient cpu
FailedScheduling
node(s) were unschedulable
```

Then inspect nodes:

```
kubectl get nodes
kubectl describe node <node-name>
```

Check resource usage if Metrics Server is available:

```
kubectl top nodes
kubectl top pods
```

Troubleshooting principle: Do not assume every Pending Pod indicates a node failure. It may be caused by resource requests, scheduling constraints, image pulling or storage provisioning.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 8. Multi-container Pod design patterns

The instructor introduces four patterns:

1. Sidecar
2. Init container
3. Ambassador
4. Adapter

These patterns describe how multiple containers or supporting processes can work with an application.

### 8.1 Sidecar pattern

A sidecar is a supporting container that runs alongside the main application container in the same Pod.

Its purpose is to enhance or extend the main application's functionality without changing the application's code.

Common examples:

- Log collection.
- Monitoring.
- Proxy functionality.
- Certificate renewal.
- Data synchronization.

[Kubernetes Sidecar Pattern Explained with Real Examples (Multi-Container Pod YAML) | GoLinuxCloud](https://images.openai.com/static-rsc-4/35ztiATu6ktsMpxq275sYut4J8SNgy0JVi8b8ZlULWwhCJuTZa7KGl0t-iAMxRGvxECU31TYqN1AW4ztcQ-w3Qkjq7bWD7piq6-GZ9rKca1CyUVM5ImUFVaLCel4JDvJ2NlqpYmMrzEGrvhTz7E4pMxgW6zPstgZLnDY1837_kU?purpose=inline)

[golinuxcloud.com](https://www.golinuxcloud.com/kubernetes-sidecar-example/?utm_source=chatgpt.com)

Example:

```
Pod
 |
 |-- Application Container
 |       |
 |       |-- Generates application logs
 |
 |-- Logging Sidecar
         |
         |-- Reads shared log files
         |-- Sends logs to Splunk / ELK
```

A Pod may contain multiple sidecars if there are multiple supporting responsibilities.

Important: A sidecar is not simply any additional container in a Pod. It has a supporting role alongside the main application.

### 8.2 Init container pattern

An init container runs before the application containers start.

It performs prerequisite tasks that must complete successfully before the main application starts.

Examples:

- Checking database connectivity.
- Downloading configuration files.
- Preparing directories.
- Performing initialization scripts.
- Waiting for required dependencies.
- Running setup tasks or migrations when appropriate.

[Init Containers: The Setup Crew of Kubernetes](https://images.openai.com/static-rsc-4/wxWoEnW622wYM6CzthXJoEJf4JoCJ25haoJm2AkzAFz2j3bhCXcR8j60gVa3M_0mqiiYLFkjbtGIyRcwmmKE4RZEpFbtTAjNBkR5Yx4z4X4iR9zhLsFVr3ISlAXBmO2F5ASZ_ROKhwb3HVuN3GpYASq47Wsd8tDBwIWCTsn5OrE?purpose=inline)

[kubenatives.com](https://www.kubenatives.com/p/init-containers-in-kubernetes-the?utm_source=chatgpt.com)

Example workflow:

```
Pod starts
    |
    v
Init Container
    |
    |-- Check database connectivity
    |-- Validate configuration
    |-- Prepare required files
    |
    v
Init container exits successfully
    |
    v
Main Application Container starts
```

If the init container fails, the application containers do not start. Kubernetes normally retries the init container according to the Pod's restart policy.

### Example init container YAML

```
apiVersion: v1
kind: Pod
metadata:
  name: app-with-init
spec:
  initContainers:
    - name: init-check
      image: busybox:1.36
      command:
        - sh
        - -c
        - echo "Preparing application"

  containers:
    - name: nginx
      image: nginx:1.27
      ports:
        - containerPort: 80
```

Here:

- `initContainers` contains the prerequisite container.
- `containers` contains the main application container.
- The main container starts after the init container completes successfully.

### Sidecar vs init container

| Sidecar                                               | Init container                                           |
| ----------------------------------------------------- | -------------------------------------------------------- |
| Supports the main application while it runs           | Prepares requirements before the main application starts |
| Runs alongside application containers                 | Runs before application containers                       |
| Used for logging, monitoring and proxying             | Used for initialization and prerequisite checks          |
| Normally continues running while the application runs | Completes its task and exits                             |

Kubernetes nuance

Modern Kubernetes also supports native sidecar containers, represented as restartable init containers. These can start before application containers and continue running alongside them. The session's basic sidecar-vs-init comparison describes the traditional patterns.

### What happens if an init container fails?

This is an important interview question.

Suppose an init container checks database connectivity before starting a Java application.

It may fail because of:

| Failure reason       | Explanation                                      |
| -------------------- | ------------------------------------------------ |
| Database unavailable | The dependency is not running                    |
| Network issue        | The Pod cannot reach the database                |
| DNS issue            | The database hostname cannot be resolved         |
| Permission issue     | Credentials or database privileges are incorrect |
| Configuration issue  | Wrong hostname, port or environment variables    |
| Image issue          | Incorrect image name or registry access problem  |
| Script failure       | Initialization command exits unsuccessfully      |

Result: The application containers do not start until the init containers complete successfully.

Useful troubleshooting commands:

```
kubectl describe pod <pod-name>
```

```
kubectl logs <pod-name> -c <init-container-name>
```

If the Pod has multiple init containers, inspect the one that failed.

### 8.3 Ambassador pattern

The ambassador pattern uses a proxy container to handle communication between the main application and external services.

Example:

```
Pod
 |
 |-- Application Container
 |
 |-- Ambassador / Proxy Container
          |
          |-- Handles external communication
          |-- Routes requests
```

The application communicates through the proxy rather than directly managing all external communication details.

The instructor compares this with reverse-proxy behavior.

Practical note: This is a less commonly used standalone Pod pattern today. Kubernetes Services, Ingress controllers, Gateway API and service-mesh solutions often provide more appropriate traffic-management mechanisms.

### 8.4 Adapter pattern

The adapter pattern transforms or normalizes information so that another component can consume it.

Example:

- An application produces logs in one format.
- The monitoring system expects another format.
- An adapter transforms the data into the expected format.

```
Application Output
        |
        v
   Adapter
        |
        |-- Transform data
        |-- Normalize format
        v
 Monitoring System
```

The adapter pattern is more of an application-design pattern than a specific Kubernetes resource type.

### Four patterns — quick comparison

| Pattern        | Main purpose                           | Example                     |
| -------------- | -------------------------------------- | --------------------------- |
| Sidecar        | Supports the application alongside it  | Log collector               |
| Init container | Completes prerequisites before startup | Database connectivity check |
| Ambassador     | Handles communication through a proxy  | Local proxy                 |
| Adapter        | Transforms or normalizes data          | Log format converter        |

Interview priority: Focus especially on sidecar and init containers, their lifecycle differences, and failure scenarios.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+2

## 9. Why should we use Deployments instead of creating Pods directly?

This is one of the most important concepts in this session.

The instructor demonstrates that when you create a Pod directly and delete it, Kubernetes does not automatically create a replacement.

Example:

```
kubectl apply -f pod.yaml
kubectl get pods
```

Delete the Pod:

```
kubectl delete pod nginx
```

Check again:

```
kubectl get pods
```

The Pod is gone.

Why? A standalone Pod has no Deployment or ReplicaSet controller maintaining its desired replica count.

To solve this, Kubernetes provides the Deployment object.

### What is a Deployment?

A Deployment manages application releases and the desired number of Pod replicas.

It provides:

- Replica management.
- Self-healing through its controllers.
- Scaling.
- Rolling updates.
- Rollback support.

Instead of creating individual Pods manually, you create a Deployment that manages them.

### Deployment architecture

[SDC#25 - Practical Intro to Kubernetes - by Saurabh Dashora](https://images.openai.com/static-rsc-4/ZJDy4orbB-kG1IMFAtmdJlgjU6eUOiWxKa256mgXZkV3wvxLKSxh8zxdqMVIeitPv9wiwrK5vDo3lM1bIxw8TB3DBZdEbRT06VegSxaTPjUNnyaJgoSwAxXAINwyn2vPxzfTToUFTTVMtWdeFwefT3Dn6k2TiCoqzydLMXZsWKk?purpose=inline)

[newsletter.systemdesigncodex.com](https://newsletter.systemdesigncodex.com/p/practical-intro-to-kubernetes?utm_source=chatgpt.com)

```
Kubernetes Cluster
       |
       v
   Deployment
       |
       v
   ReplicaSet
       |
       |-- Pod 1
       |-- Pod 2
       |-- Pod 3
```

The Deployment manages the ReplicaSet, and the ReplicaSet maintains the required number of Pods.

### Why is ReplicaSet needed?

A ReplicaSet ensures that the desired number of Pod replicas is maintained.

For example:

```
replicas: 3
```

This means that three Pods should be running.

If one Pod is deleted, the ReplicaSet detects the difference and creates a replacement.

## 10. Creating a Kubernetes Deployment

The instructor demonstrates creating an NGINX Deployment with three replicas.

### Step 1 — Create deployment.yaml

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
          image: nginx:1.27
          ports:
            - containerPort: 80
```

### Understand the important fields

| Field                  | Explanation                                |
| ---------------------- | ------------------------------------------ |
| `apiVersion: apps/v1`  | Deployment API version                     |
| `kind: Deployment`     | Creates a Deployment                       |
| `metadata.name`        | Deployment name                            |
| `replicas: 3`          | Desired number of Pods                     |
| `selector.matchLabels` | Identifies Pods managed by this Deployment |
| `template`             | Defines the Pod template                   |
| `containers`           | Defines application containers             |
| `image`                | Container image                            |

Important: The labels in `template.metadata.labels` must match the Deployment selector. Otherwise, the Deployment cannot correctly identify its Pods.

### Step 2 — Apply the Deployment

```
kubectl apply -f deployment.yaml
```

### Step 3 — Verify the Deployment

```
kubectl get deployments
```

Example output:

```
NAME               READY   UP-TO-DATE   AVAILABLE
nginx-deployment   3/3     3            3
```

### Step 4 — Check the ReplicaSet

```
kubectl get replicasets
```

### Step 5 — Check the Pods

```
kubectl get pods -o wide
```

You should see three NGINX Pods.

## 11. Testing self-healing

The instructor demonstrates the most important advantage of a Deployment.

Suppose three Pods are running.

Delete one Pod:

```
kubectl delete pod <pod-name>
```

Now check:

```
kubectl get pods
```

A replacement Pod should be created automatically.

### What happens internally?

\#chatgpt-mermaid-\_r_4bn\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_4bn\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_4bn\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_4bn\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_4bn\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_4bn\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_4bn\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_4bn\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_4bn\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_4bn\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_4bn\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_4bn\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_4bn\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_4bn\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_4bn\_ p{margin:0;}#chatgpt-mermaid-\_r_4bn\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_4bn\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_4bn\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_4bn\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_4bn\_ .label text,#chatgpt-mermaid-\_r_4bn\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_4bn\_ .node rect,#chatgpt-mermaid-\_r_4bn\_ .node circle,#chatgpt-mermaid-\_r_4bn\_ .node ellipse,#chatgpt-mermaid-\_r_4bn\_ .node polygon,#chatgpt-mermaid-\_r_4bn\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_4bn\_ .rough-node .label text,#chatgpt-mermaid-\_r_4bn\_ .node .label text,#chatgpt-mermaid-\_r_4bn\_ .image-shape .label,#chatgpt-mermaid-\_r_4bn\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_4bn\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_4bn\_ .rough-node .label,#chatgpt-mermaid-\_r_4bn\_ .node .label,#chatgpt-mermaid-\_r_4bn\_ .image-shape .label,#chatgpt-mermaid-\_r_4bn\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_4bn\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_4bn\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_4bn\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_4bn\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_4bn\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_4bn\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_4bn\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_4bn\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_4bn\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_4bn\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_4bn\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_4bn\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_4bn\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_4bn\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_4bn\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_4bn\_ .icon-shape,#chatgpt-mermaid-\_r_4bn\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_4bn\_ .icon-shape p,#chatgpt-mermaid-\_r_4bn\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_4bn\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_4bn\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_4bn\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_4bn\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_4bn\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_4bn\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_4bn\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_4bn\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_4bn\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_4bn\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_4bn\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_4bn\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_4bn\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_4bn\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_4bn\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_4bn\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_4bn\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_4bn\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_4bn\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_4bn\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_4bn\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_4bn\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_4bn\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_4bn\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_4bn\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_4bn\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_4bn\_ .node rect,#chatgpt-mermaid-\_r_4bn\_ .node circle,#chatgpt-mermaid-\_r_4bn\_ .node ellipse,#chatgpt-mermaid-\_r_4bn\_ .node polygon,#chatgpt-mermaid-\_r_4bn\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_4bn\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_4bn\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_4bn\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_4bn\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_4bn\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}Deployment desires 3 replicasReplicaSet maintains 3 PodsOne Pod is deletedReplicaSet detects only 2 PodsReplicaSet creates replacementPodDesired replica count restoredto 3

This is called self-healing or automatic reconciliation.

The Deployment does not necessarily restart the deleted Pod itself. Its ReplicaSet creates a replacement Pod, which may have a different name and IP address.

## 12. Manual scaling of Deployments

The instructor demonstrates increasing and decreasing the number of replicas.

Suppose you currently have three Pods.

To increase the number to ten:

```
kubectl scale deployment nginx-deployment --replicas=10
```

Verify:

```
kubectl get deployments
kubectl get pods
```

Now you should have ten replicas, assuming sufficient cluster capacity.

To reduce the replicas to five:

```
kubectl scale deployment nginx-deployment --replicas=5
```

### Scaling diagram

Manual scaling

Before

3 replicas

Pod 1

Pod 2

Pod 3

After scaling

5 replicas

Pod 1

Pod 2

Pod 3

Pod 4

Pod 5

### Manual vs automatic scaling

| Manual scaling                      | Automatic scaling                                       |
| ----------------------------------- | ------------------------------------------------------- |
| Administrator changes replica count | Kubernetes adjusts replicas based on configured metrics |
| Uses `kubectl scale`                | Uses Horizontal Pod Autoscaler                          |
| Requires manual intervention        | Responds automatically to metric changes                |

### What is HPA?

HPA stands for Horizontal Pod Autoscaler.

It automatically adjusts the number of Pod replicas based on metrics such as CPU utilization or memory utilization, when supported and configured.

Example:

```
kubectl autoscale deployment nginx-deployment \
  --cpu-percent=70 \
  --min=2 \
  --max=10
```

This creates an HPA targeting the Deployment.

It requires appropriate resource requests and an available metrics source, commonly Metrics Server for basic resource metrics.

Important distinction:

- Deployment: Maintains and manages the application rollout and replica configuration.
- ReplicaSet: Maintains the desired number of Pods.
- HPA: Adjusts the desired replica count based on metrics.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 13. Updating a Deployment

A Deployment also manages application updates.

For example, you are running:

```
image: nginx:1.27
```

You want to upgrade to:

```
image: nginx:1.28
```

Update the image:

```
kubectl set image deployment/nginx-deployment \
  nginx=nginx:1.28
```

Check rollout status:

```
kubectl rollout status deployment/nginx-deployment
```

View rollout history:

```
kubectl rollout history deployment/nginx-deployment
```

Rollback to the previous revision:

```
kubectl rollout undo deployment/nginx-deployment
```

### What is a rolling update?

A rolling update gradually replaces Pods running the old image with Pods running the new image.

This helps reduce downtime when the application is configured correctly.

DevOps relevance: In a CI/CD pipeline, Jenkins can build and push a new image, then update the Kubernetes Deployment to use that image.

Example workflow:

\#chatgpt-mermaid-\_r_4e1\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_4e1\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_4e1\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_4e1\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_4e1\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_4e1\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_4e1\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_4e1\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_4e1\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_4e1\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_4e1\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_4e1\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_4e1\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_4e1\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_4e1\_ p{margin:0;}#chatgpt-mermaid-\_r_4e1\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_4e1\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_4e1\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_4e1\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_4e1\_ .label text,#chatgpt-mermaid-\_r_4e1\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_4e1\_ .node rect,#chatgpt-mermaid-\_r_4e1\_ .node circle,#chatgpt-mermaid-\_r_4e1\_ .node ellipse,#chatgpt-mermaid-\_r_4e1\_ .node polygon,#chatgpt-mermaid-\_r_4e1\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_4e1\_ .rough-node .label text,#chatgpt-mermaid-\_r_4e1\_ .node .label text,#chatgpt-mermaid-\_r_4e1\_ .image-shape .label,#chatgpt-mermaid-\_r_4e1\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_4e1\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_4e1\_ .rough-node .label,#chatgpt-mermaid-\_r_4e1\_ .node .label,#chatgpt-mermaid-\_r_4e1\_ .image-shape .label,#chatgpt-mermaid-\_r_4e1\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_4e1\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_4e1\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_4e1\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_4e1\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_4e1\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_4e1\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_4e1\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_4e1\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_4e1\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_4e1\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_4e1\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_4e1\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_4e1\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_4e1\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_4e1\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_4e1\_ .icon-shape,#chatgpt-mermaid-\_r_4e1\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_4e1\_ .icon-shape p,#chatgpt-mermaid-\_r_4e1\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_4e1\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_4e1\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_4e1\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_4e1\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_4e1\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_4e1\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_4e1\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_4e1\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_4e1\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_4e1\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_4e1\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_4e1\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_4e1\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_4e1\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_4e1\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_4e1\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_4e1\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_4e1\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_4e1\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_4e1\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_4e1\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_4e1\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_4e1\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_4e1\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_4e1\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_4e1\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_4e1\_ .node rect,#chatgpt-mermaid-\_r_4e1\_ .node circle,#chatgpt-mermaid-\_r_4e1\_ .node ellipse,#chatgpt-mermaid-\_r_4e1\_ .node polygon,#chatgpt-mermaid-\_r_4e1\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_4e1\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_4e1\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_4e1\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_4e1\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_4e1\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}Developer pushes code to GitJenkins pipelineBuild applicationBuild Docker imagePush image to registryUpdate KubernetesDeploymentRolling updateNew application versionrunning

## 14. Deployment vs ReplicaSet vs Pod

This distinction is frequently asked in Kubernetes interviews.

| Feature                 | Pod                                | ReplicaSet                                     | Deployment              |
| ----------------------- | ---------------------------------- | ---------------------------------------------- | ----------------------- |
| Runs containers         | Yes                                | Through Pods                                   | Through managed Pods    |
| Maintains replica count | No                                 | Yes                                            | Through its ReplicaSet  |
| Self-healing            | No independent controller          | Yes                                            | Yes, through ReplicaSet |
| Scaling                 | Not applicable as a standalone Pod | Can maintain changed replica count             | Yes                     |
| Rolling updates         | No                                 | No rollout management comparable to Deployment | Yes                     |
| Rollback                | No                                 | No deployment revision management              | Yes                     |

Remember the hierarchy:

```
Deployment
    |
    v
ReplicaSet
    |
    v
Pods
    |
    v
Containers
```

In normal application deployments, you work with the Deployment rather than creating ReplicaSets or Pods individually.

## 15. What is still missing? Kubernetes Services

At the end of the session, the instructor identifies one remaining challenge.

Suppose you have five replicas of an NGINX application.

How does a user access those Pods?

How does Kubernetes distribute incoming traffic across them?

A Deployment manages Pods, but it does not by itself provide a stable network endpoint for clients.

That is where Kubernetes Services come in.

The instructor introduces three Service types for the next session:

| Service type | Purpose                                                              |
| ------------ | -------------------------------------------------------------------- |
| ClusterIP    | Exposes an application inside the cluster                            |
| NodePort     | Exposes a Service through a port on each node                        |
| LoadBalancer | Exposes a Service through an external load balancer, where supported |

This will cover service discovery and load balancing.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 16. Interview revision — important questions

### Q1. What is a Kubernetes Pod?

A Pod is the smallest deployable unit in Kubernetes. It contains one or more containers that share a network namespace and can share storage volumes.

### Q2. Why should we use Deployments instead of standalone Pods?

Standalone Pods do not have a controller maintaining their desired replica count. Deployments manage ReplicaSets, which maintain the required number of Pods and help provide self-healing.

### Q3. What happens if you delete a Pod managed by a Deployment?

The ReplicaSet detects that the actual replica count is below the desired count and creates a replacement Pod.

### Q4. What is a ReplicaSet?

A ReplicaSet ensures that a specified number of Pod replicas are running at a given time.

### Q5. What is the difference between Deployment and ReplicaSet?

A Deployment manages ReplicaSets and provides rollout, update and rollback functionality. A ReplicaSet maintains the desired number of Pods.

### Q6. What is the difference between manual and automatic scaling?

Manual scaling requires an administrator to change the replica count. Automatic scaling uses HPA to adjust replicas based on configured metrics.

### Q7. What is a sidecar container?

A sidecar is a supporting container running alongside the main application container in the same Pod, commonly used for logging, monitoring or proxy functionality.

### Q8. What is an init container?

An init container executes prerequisite tasks before application containers start. It must complete successfully before the main application containers can start.

### Q9. What happens when an init container fails?

The application containers do not start. Kubernetes retries the init container according to the Pod's restart policy until it succeeds or the Pod is otherwise terminated.

### Q10. What is the difference between public and private Kubernetes clusters?

The main difference is the network accessibility of the Kubernetes API server endpoint. A private cluster can still host applications exposed publicly through an appropriate ingress or load-balancing configuration.

### Q11. What is kubeconfig?

It is a configuration file that contains cluster connection details, authentication information and contexts used by `kubectl`.

### Q12. What is a node pool?

A node pool is a group of worker nodes with a common configuration. It allows teams to manage, scale and organize worker-node capacity.

### Q13. What is a rolling update?

A rolling update gradually replaces old application Pods with new ones while attempting to maintain application availability.

### Q14. What is HPA?

Horizontal Pod Autoscaler automatically changes the number of Pod replicas based on configured metrics and scaling policies.

## 17. Quick revision sheet

Cluster and connectivity

| Term            | Meaning                                       |
| --------------- | --------------------------------------------- |
| AKS             | Azure Kubernetes Service                      |
| Node pool       | Group of worker nodes                         |
| System pool     | Intended for Kubernetes system workloads      |
| User pool       | Intended primarily for application workloads  |
| kubectl         | Kubernetes CLI                                |
| kubeconfig      | Cluster credentials and context configuration |
| Private cluster | Private API server access                     |
| Public cluster  | Publicly reachable API server endpoint        |

Pods and controllers

| Term           | Meaning                                    |
| -------------- | ------------------------------------------ |
| Pod            | Smallest deployable unit                   |
| Sidecar        | Supporting container alongside application |
| Init container | Runs prerequisites before application      |
| Deployment     | Manages application rollouts               |
| ReplicaSet     | Maintains Pod replica count                |
| HPA            | Automatically scales Pod replicas          |
| Rolling update | Gradually replaces application versions    |

### Commands to remember

| Command                                                  | Purpose                     |
| -------------------------------------------------------- | --------------------------- |
| `kubectl cluster-info`                                   | Cluster information         |
| `kubectl get pods`                                       | List Pods                   |
| `kubectl get pods -A`                                    | List Pods across namespaces |
| `kubectl get pods -o wide`                               | Show additional Pod details |
| `kubectl describe pod nginx`                             | Detailed Pod information    |
| `kubectl logs nginx`                                     | View Pod logs               |
| `kubectl apply -f pod.yaml`                              | Create/update Pod           |
| `kubectl delete pod nginx`                               | Delete Pod                  |
| `kubectl get deployments`                                | List Deployments            |
| `kubectl get replicasets`                                | List ReplicaSets            |
| `kubectl scale deployment nginx-deployment --replicas=5` | Manually scale Deployment   |
| `kubectl rollout status deployment/nginx-deployment`     | Check rollout               |
| `kubectl rollout undo deployment/nginx-deployment`       | Roll back Deployment        |

### Final mental model

```
Kubernetes Cluster
       |
       |-- Control Plane
       |      |-- API Server
       |      |-- Scheduler
       |      |-- Controllers
       |      |-- etcd
       |
       |-- Worker Nodes
              |
              |-- Node Pools
              |
              |-- Deployments
                     |
                     |-- ReplicaSets
                            |
                            |-- Pods
                                   |
                                   |-- Application Container
                                   |-- Sidecar Container
                                   |-- Shared Volumes
```

The five concepts I recommend prioritizing:

1. Public vs private Kubernetes cluster — particularly API server accessibility.
2. Pod lifecycle — Pending, Running, Succeeded, Failed and Unknown.
3. Sidecar vs init container — know the difference in execution lifecycle.
4. Deployment vs ReplicaSet vs Pod — understand the complete hierarchy.
5. Self-healing, manual scaling and HPA — be able to explain with commands and examples.

Next session: Kubernetes Services — ClusterIP, NodePort and LoadBalancer, followed by service discovery and traffic distribution.

Source transcript: [DevOps Session — 9 October 2026](https://github.com/jeevanadv2000-pixel/python/blob/main/DevOps%20Session\(Morning\)%20transcript_2026-10-09_09.22.44.txt?utm_source=chatgpt.com).