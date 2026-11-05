# DevOps Session — Kubernetes Services & Ingress Notes

Date: 12 October 2026\
Instructor: Rochak Agrawal\
Focus: Kubernetes Services, Service Types, Labels and Selectors, DNS, Load Balancing, Ingress and Ingress Controllers.

These notes are condensed from the class transcript. Participant discussions and repetitions have been removed, speech-to-text errors corrected, and technical explanations clarified where necessary.

## 1. Why do we need Kubernetes Services?

In the previous session, we learned that Deployments create and manage Pods.

The problem is that Pod IP addresses are not permanent.

For example:

1. A Deployment creates three Pods.
2. Each Pod receives its own IP address.
3. One Pod crashes or is deleted.
4. Kubernetes creates a replacement Pod.
5. The replacement may receive a different IP address.

If another application directly communicates with the old Pod IP, communication can fail.

Kubernetes Service solves this problem.

A Service provides:

- A stable network endpoint (usually a stable ClusterIP and DNS name).
- A way to discover and access backend Pods.
- Traffic distribution across eligible backend Pods.

The Service continues to work even when Pods are replaced, provided the replacement Pods match its selector and become ready.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



### Example

Frontend application

Backend Service

Stable IP / DNS endpoint

Distributes traffic

Pod 1

10.0.1.5

Pod 2

10.0.1.6

Pod 3

10.0.1.7

If Pod 2 is replaced, the Service discovers the replacement instead of requiring the frontend to learn its new IP.

Remember: Applications should generally communicate with a Service rather than depending on individual Pod IP addresses.

## 2. How does a Service identify Pods?

Two important concepts:

- Labels: Metadata attached to Kubernetes objects, such as Pods.
- Selectors: Rules used by a Service to identify which Pods should receive traffic.

For example, a Pod can have this label:

```
metadata:
  labels:
    app: nginx
```

The Service can use this selector:

```
selector:
  app: nginx
```

The Service selects Pods whose labels match `app: nginx`.

### Why is this important?

Imagine you have:

- Three NGINX Pods.
- Two Jenkins Pods.

You can label them separately:

| Application | Pod label      |
| ----------- | -------------- |
| NGINX       | `app: nginx`   |
| Jenkins     | `app: jenkins` |

The NGINX Service selects only NGINX Pods, while the Jenkins Service selects only Jenkins Pods.

When a Deployment creates replacement Pods, its Pod template supplies the same labels. The Service can therefore discover the replacement Pods automatically.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



Important: If your Service selector is incorrect, it may select no Pods or the wrong Pods.

## 3. Understanding Service ports

Three important fields are used when defining a Service:

| Field        | Meaning                                              |
| ------------ | ---------------------------------------------------- |
| `port`       | Port exposed by the Service                          |
| `targetPort` | Port on which the application listens inside the Pod |
| `nodePort`   | Port exposed on each node for a NodePort Service     |

Example:

```
ports:
  - port: 80
    targetPort: 8080
```

The traffic flow is:

`Client → Service port 80 → Pod targetPort 8080`

If your application listens on port 8080 inside the Pod, the Service can expose it on port 80.

Important distinction:

- `containerPort` in a Pod specification documents the container's intended listening port; it does not publish that port externally.
- `targetPort` tells the Service which backend port to forward traffic to.

## 4. Three main Kubernetes Service types

This is one of the most frequently asked Kubernetes interview topics.

Kubernetes has three commonly used core Service types:

1. ClusterIP
2. NodePort
3. LoadBalancer

### 4.1 ClusterIP

Default Service type

ClusterIP exposes an application using an internal IP address accessible within the cluster.

Use case: Internal communication between applications.

Example:

- Frontend Pods need to communicate with backend Pods.
- Backend Pods need to communicate with database Pods.
- These applications communicate through ClusterIP Services.

YAML:

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

If you don't specify `type`, Kubernetes uses `ClusterIP` by default.

Key point: ClusterIP is not directly accessible from outside the cluster through its normal internal endpoint.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



### 4.2 NodePort

NodePort exposes a Service on a static port on every node in the cluster.

The default NodePort range is:

`30000–32767`

Example:

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

Suppose your cluster has three nodes:

| Node   | Node IP  |
| ------ | -------- |
| Node 1 | 10.0.0.1 |
| Node 2 | 10.0.0.2 |
| Node 3 | 10.0.0.3 |

You can access the application using:

```
http://10.0.0.1:30080
http://10.0.0.2:30080
http://10.0.0.3:30080
```

All three addresses can route traffic through the Service to eligible backend Pods, even if the Pods are running on only one of the nodes.

Why? NodePort exposes the Service on every node, not the individual Pod on a particular node.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



When is NodePort useful?

- Learning and testing.
- Development environments.
- Simple external access when a separate load balancer is not being used.

It is less commonly used as the direct production entry point because it exposes a fixed port on every node.

### 4.3 LoadBalancer

LoadBalancer exposes a Service externally using a cloud provider's load-balancing infrastructure.

For example, in Azure AKS, Kubernetes can request an Azure Load Balancer.

YAML:

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

What happens?

1. You create a LoadBalancer Service.
2. Kubernetes communicates with the cloud integration.
3. The cloud provider provisions a load balancer, subject to configuration and permissions.
4. An external IP or hostname is assigned.
5. External users access the application through that endpoint.
6. Traffic is forwarded to eligible backend Pods.

Use case: Exposing an application to external users.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

### Service types — quick comparison

| Feature           | ClusterIP              | NodePort                 | LoadBalancer                                   |
| ----------------- | ---------------------- | ------------------------ | ---------------------------------------------- |
| Internal access   | Yes                    | Yes                      | Yes                                            |
| External access   | Not directly           | Through node IP and port | Through external LB                            |
| Stable Service IP | Yes                    | Yes                      | Yes, ClusterIP underneath                      |
| Node port         | No                     | Yes                      | Usually allocated as part of NodePort behavior |
| Common use        | Internal communication | Testing / special cases  | External application access                    |

## 5. What happens when a request reaches a NodePort?

This was an important classroom discussion.

Suppose:

- Your cluster has three nodes.
- Your application Pod is running only on Node 3.
- You expose the application using NodePort `30080`.

You can still access it through Node 1's IP:

```
http://10.0.0.1:30080
```

You do not need the application Pod to be running on Node 1.

The Service provides a cluster-wide abstraction. Node networking rules forward the request to an eligible backend Pod.

In many Kubernetes networking implementations, `kube-proxy` configures the required network rules. Some clusters use alternative implementations such as eBPF instead.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



## 6. How does Kubernetes know when Pod IP addresses change?

This is another important concept.

Suppose:

- Your application has three replicas.
- Each Pod receives an IP address.
- One Pod is deleted.
- A new Pod is created with a different IP.

How does the Service find the replacement?

Through labels and selectors.

The Service does not depend on a hardcoded list of Pod IP addresses. It selects Pods using labels, and Kubernetes updates the Service's backend endpoint information as Pods are created, deleted, or become ready.

Modern Kubernetes uses EndpointSlices to represent these backend endpoints.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 7. Kubernetes DNS and Service discovery

Kubernetes provides DNS-based service discovery, typically through CoreDNS.

Instead of remembering a Service IP address, another application can communicate using its Service name.

For example, suppose you have:

- Service name: `backend-service`
- Namespace: `default`

Another Pod in the same namespace can access it using:

```
http://backend-service
```

From another namespace, use the fully qualified Service DNS name:

```
backend-service.default.svc.cluster.local
```

General format:

```
<service-name>.<namespace>.svc.cluster.local
```

This makes communication easier because applications don't need to keep track of changing Pod IP addresses or manually configure Service IPs.

Important: DNS resolves the Service name to its Service address. The Service then routes traffic to its eligible backend Pods.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



## 8. What is Ingress?

Now imagine you have several applications running inside Kubernetes:

- Website
- Image service
- Video service
- Payment service

You want users to access all these applications through one external entry point.

You could create separate LoadBalancer Services for each application, but that may create additional cost and management overhead.

This is where Ingress becomes useful.

Ingress provides HTTP/HTTPS routing rules to direct external traffic to internal Kubernetes Services.

### Example

Suppose your domain is:

`myapp.com`

You want:

| URL                  | Destination     |
| -------------------- | --------------- |
| `myapp.com/images`   | Image Service   |
| `myapp.com/videos`   | Video Service   |
| `myapp.com/payments` | Payment Service |

Ingress can route requests based on the URL path.

It can also support host-based routing:

| Hostname           | Destination   |
| ------------------ | ------------- |
| `images.myapp.com` | Image Service |
| `videos.myapp.com` | Video Service |

This is more flexible than simply distributing incoming traffic across backends.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



### How Ingress works

External users

Ingress Controller / Gateway

HTTP routing, TLS, host and path rules

Image

Service

Video

Service

Payment

Service

Each Service forwards requests to its own matching backend Pods.

## 9. Ingress vs Ingress Controller

These two terms are often confused.

Ingress: A Kubernetes API resource containing routing rules.

Ingress Controller: The actual software that watches Ingress resources and implements those rules.

For example, defining an Ingress YAML file alone does not make traffic routing work. You need a compatible Ingress Controller running in the cluster.

Examples include:

- NGINX Ingress Controller
- Traefik
- Kong
- Cloud-provider-specific ingress solutions

The controller reads the Ingress configuration and configures the underlying proxy or gateway to route traffic accordingly.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



## 10. LoadBalancer Service vs Ingress

This is a very important interview question.

Both can expose applications externally, but they solve different problems.

| Feature            | LoadBalancer Service                         | Ingress                                             |
| ------------------ | -------------------------------------------- | --------------------------------------------------- |
| Main purpose       | External network access to a Service         | HTTP/HTTPS routing to Services                      |
| Routing            | Primarily network/transport-level forwarding | Application-level routing                           |
| Path-based routing | Not its primary function                     | Yes                                                 |
| Host-based routing | Not its primary function                     | Yes                                                 |
| TLS termination    | Depends on the external LB configuration     | Commonly supported through controller configuration |
| Backend            | One Service                                  | Multiple Services can be routed to                  |

### Example

LoadBalancer Service:

```
External user
     |
     v
Cloud Load Balancer
     |
     v
Application Service
     |
     v
Backend Pods
```

Ingress:

```
External user
     |
     v
Ingress Controller
     |
     |-- /images   --> Image Service
     |
     |-- /videos   --> Video Service
     |
     |-- /payments --> Payment Service
```

Remember:

- LoadBalancer Service: Exposes a Service externally.
- Ingress: Routes HTTP/HTTPS requests to different Services based on rules.

Ingress is not a replacement for ClusterIP. Internal Services are still commonly used as Ingress backends.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



Modern Kubernetes clarification: Ingress remains widely used, but Kubernetes Gateway API is a newer, more expressive option for managing advanced traffic routing. Also, Ingress is a separate Kubernetes API resource, not a fourth Service type.

## 11. Public cluster vs Private cluster

This topic was discussed again during the session.

The difference is primarily about access to the Kubernetes control plane API server.

| Public cluster                                                               | Private cluster                                                             |
| ---------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| API server has a publicly reachable endpoint, subject to access restrictions | API server is reachable through private networking                          |
| Administrators can connect from permitted external networks                  | Administrators generally need access through the configured private network |
| Requires careful endpoint access controls                                    | Reduces exposure of the control plane to the public internet                |

### Important interview point

A private Kubernetes cluster does not automatically mean that applications running inside it must be private.

For example:

- Kubernetes API server: Private.
- Application running inside the cluster: Publicly accessible through a properly configured LoadBalancer or Ingress.

These are separate networking decisions.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



## 12. Useful kubectl commands

| Command                              | Purpose                            |
| ------------------------------------ | ---------------------------------- |
| `kubectl get services`               | List Services                      |
| `kubectl get svc`                    | Short form                         |
| `kubectl get svc -o wide`            | Show additional Service details    |
| `kubectl describe svc nginx-service` | Inspect a Service                  |
| `kubectl get endpoints`              | View legacy Endpoints objects      |
| `kubectl get endpointslices`         | View EndpointSlices                |
| `kubectl get ingress`                | List Ingress resources             |
| `kubectl describe ingress`           | Inspect Ingress configuration      |
| `kubectl get pods --show-labels`     | View Pod labels                    |
| `kubectl get pods -l app=nginx`      | Filter Pods using a label selector |
| `kubectl get svc -A`                 | List Services across namespaces    |

## 13. Practical troubleshooting scenarios

| Problem                           | What to check                                                             |
| --------------------------------- | ------------------------------------------------------------------------- |
| Service has no backend Pods       | Check Service selector and Pod labels                                     |
| Application is not reachable      | Check Service port, targetPort and application listening port             |
| Service exists but requests fail  | Check EndpointSlices and Pod readiness                                    |
| NodePort not reachable externally | Check node networking, firewall/security rules and NodePort configuration |
| LoadBalancer has no external IP   | Check cloud integration, permissions and provisioning events              |
| Ingress not routing traffic       | Check Ingress Controller, rules, backend Service and controller logs      |
| DNS name does not resolve         | Check CoreDNS, namespace and Service DNS name                             |

Useful troubleshooting commands:

```
kubectl get svc
kubectl describe svc nginx-service

kubectl get pods --show-labels
kubectl get endpointslices

kubectl get ingress
kubectl describe ingress nginx-ingress

kubectl get pods -n kube-system
```

## 14. Interview questions and answers

Q1. Why do we need a Kubernetes Service?

Pods are temporary and their IP addresses can change. A Service provides a stable endpoint and discovers the backend Pods using selectors.

Q2. What are the three main Service types?

ClusterIP, NodePort and LoadBalancer.

Q3. What is the default Service type?

ClusterIP.

Q4. What is the difference between `port` and `targetPort`?

`port` is the port exposed by the Service. `targetPort` is the port on which the backend application listens.

Q5. Can we access a NodePort Service using any node's IP?

Yes. NodePort exposes the Service on every node at the configured port, and traffic can be forwarded to an eligible backend Pod on another node.

Q6. What is the default NodePort range?

30000–32767.

Q7. How does a Service identify its backend Pods?

Using labels and selectors.

Q8. What happens when a backend Pod is deleted?

Its endpoint is removed when it is no longer eligible. When a replacement Pod becomes ready and matches the selector, its endpoint is added.

Q9. What is Ingress?

Ingress is a Kubernetes API resource that defines HTTP/HTTPS routing rules for external traffic to Services.

Q10. What is an Ingress Controller?

It is the software that watches Ingress resources and implements their routing rules.

Q11. Can Ingress work without an Ingress Controller?

No. An Ingress resource needs a compatible controller to implement its rules.

Q12. What is the difference between Ingress and LoadBalancer?

LoadBalancer exposes a Service externally. Ingress provides application-level HTTP/HTTPS routing, such as path-based and host-based routing.

Q13. Can a private Kubernetes cluster host a publicly accessible application?

Yes. The API server can remain private while an application is exposed publicly through an appropriately configured external entry point.

## 15. Quick revision

| Concept            | Remember this                                                   |
| ------------------ | --------------------------------------------------------------- |
| Pod IP             | Can change                                                      |
| Service            | Stable endpoint for accessing Pods                              |
| Label              | Identifies an object                                            |
| Selector           | Finds matching objects                                          |
| ClusterIP          | Internal communication                                          |
| NodePort           | Exposes Service on every node's IP and a static port            |
| LoadBalancer       | Exposes Service through external load-balancing infrastructure  |
| Ingress            | HTTP/HTTPS routing rules                                        |
| Ingress Controller | Implements Ingress rules                                        |
| Kubernetes DNS     | Enables access through Service names                            |
| Private cluster    | Restricts API server access, not necessarily application access |

## 16. Final mental model

Think about the request flow in this order:

1. Deployment creates and manages Pods.
2. Labels identify the Pods.
3. Service selects the matching Pods and provides a stable endpoint.
4. ClusterIP, NodePort or LoadBalancer determines how the Service is exposed.
5. Ingress provides HTTP/HTTPS routing rules for external requests.
6. Ingress Controller implements those routing rules.

## Kubernetes networking flow

External client

Ingress / LoadBalancer

Kubernetes Service

Backend Pods

## 17. Topics covered in this session

- Kubernetes Services and why they are required.
- Labels and selectors.
- Service ports and target ports.
- ClusterIP, NodePort and LoadBalancer.
- NodePort traffic routing.
- Service discovery and Kubernetes DNS.
- Ingress and Ingress Controllers.
- LoadBalancer vs Ingress.
- Public vs private Kubernetes clusters.

Next session: ConfigMaps, Secrets, Persistent Volumes and Persistent Volume Claims (PVCs), as mentioned by the instructor at the end of the class.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



Original transcript: [DevOps Session — 12 October 2026 (GitHub)](https://raw.githubusercontent.com/jeevanadv2000-pixel/python/main/DevOps%20Session%28Morning%29%20transcript_2026-10-12_09.22.54.txt?utm_source=chatgpt.com)