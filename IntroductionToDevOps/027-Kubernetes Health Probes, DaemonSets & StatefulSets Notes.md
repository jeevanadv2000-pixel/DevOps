# DevOps Session — Kubernetes Health Probes, DaemonSets & StatefulSets Notes

Date: 19 October 2026\
Instructor: Rochak Agrawal\
Focus: Liveness, Readiness and Startup Probes; Pod scheduling; DaemonSets; Stateless vs Stateful applications; StatefulSets.

These notes are condensed from the class transcript. Participant discussions and repetitions have been removed, speech-to-text errors corrected, and technical oversimplifications clarified.

## 1. Kubernetes Health Probes

In the previous session, we learned about Deployments, Pods and Services.

Now imagine that you deploy an application in Kubernetes.

The Pod status shows `Running`. Does that automatically mean your application is working correctly and ready to serve users?

No.

A Pod can be running while:

- The application is still initializing.
- The application is unable to process requests.
- The application has become unresponsive.
- A required dependency is temporarily unavailable.

Kubernetes provides health checks called Probes to monitor containers and make decisions about restarting them or sending traffic to them.

There are three types of probes:

1. Liveness Probe
2. Readiness Probe
3. Startup Probe

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 2. Liveness Probe

A Liveness Probe checks whether a container is still functioning.

Suppose:

- Your application starts successfully.
- After some time, it becomes unresponsive.
- The container is still running, but the application is stuck.

Kubernetes may not automatically identify this application-level problem simply by checking whether the container process exists.

A Liveness Probe helps Kubernetes detect this situation.

Purpose: Detect and restart unhealthy containers

### How does it work?

1. Kubernetes periodically checks the container.
2. If the probe succeeds, the container continues running.
3. If the probe fails the configured number of consecutive times, Kubernetes considers the container unhealthy.
4. Kubernetes restarts the container according to its restart policy.

Example: A Java application becomes stuck due to an internal deadlock. Its process is still running, but it cannot make progress. A suitable liveness endpoint can detect this condition and trigger a restart.

### Liveness Probe YAML

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment

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
          image: nginx

          livenessProbe:
            httpGet:
              path: /
              port: 80

            initialDelaySeconds: 10
            periodSeconds: 10
            timeoutSeconds: 2
            failureThreshold: 3
```

### Understanding the parameters

| Parameter             | Meaning                                                             |
| --------------------- | ------------------------------------------------------------------- |
| `httpGet`             | Performs an HTTP GET request                                        |
| `path`                | Endpoint to check                                                   |
| `port`                | Container port to access                                            |
| `initialDelaySeconds` | Delay before the first probe, when this setting applies             |
| `periodSeconds`       | Interval between probe checks                                       |
| `timeoutSeconds`      | Maximum time to wait for a response                                 |
| `failureThreshold`    | Consecutive failures required before the probe is considered failed |

In this example:

- Kubernetes starts checking after the initial delay.
- It checks every 10 seconds.
- It waits up to 2 seconds for a response.
- Three consecutive failures trigger a liveness failure and container restart.

Important: A failed liveness probe restarts the container, not necessarily the entire Pod. The Pod object can remain the same.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 3. Readiness Probe

A Readiness Probe checks whether a container is ready to accept application traffic.

Imagine:

- Your application container has started.
- The application is still loading configuration.
- It is connecting to a database.
- It is not yet ready to serve requests.

The container may be running, but sending traffic to it at this stage could cause failed requests.

A Readiness Probe prevents this.

Purpose: Control whether a Pod receives traffic

### How does it work?

1. Kubernetes checks whether the application is ready.
2. If the readiness probe succeeds, the Pod can become eligible to receive Service traffic.
3. If the readiness probe fails, the Pod is marked NotReady.
4. The Pod is removed from the eligible Service backend endpoints.
5. When the probe succeeds again, the Pod can become eligible to receive traffic.

Important: A readiness failure does not restart the container.

### Readiness Probe YAML

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment

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
          image: nginx

          readinessProbe:
            httpGet:
              path: /
              port: 80

            initialDelaySeconds: 5
            periodSeconds: 10
            timeoutSeconds: 2
            failureThreshold: 3
```

The definition looks almost identical to a Liveness Probe.

The difference is its purpose.

### Real-world example

Imagine an application with three replicas:

| Pod   | Application status | Readiness |
| ----- | ------------------ | --------- |
| Pod 1 | Working            | Ready     |
| Pod 2 | Still initializing | NotReady  |
| Pod 3 | Working            | Ready     |

The Service sends traffic to eligible ready Pods, so Pod 2 is temporarily excluded.

Once Pod 2 becomes ready, it can rejoin the Service's backend endpoints.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 4. Liveness Probe vs Readiness Probe

This is one of the most important Kubernetes interview questions.

| Feature                          | Liveness Probe                              | Readiness Probe                                 |
| -------------------------------- | ------------------------------------------- | ----------------------------------------------- |
| Purpose                          | Checks whether the container is functioning | Checks whether it can accept traffic            |
| On failure                       | Restarts the container after threshold      | Marks container/Pod as not ready                |
| Removes Pod from Service traffic | Not its primary function                    | Yes                                             |
| Restarts container               | Yes, after probe failure                    | No                                              |
| Common use case                  | Detect application deadlocks                | Wait for initialization or dependency readiness |

### Simple real-life example

Imagine switching on your television.

- Liveness: Is the television switched on?
- Readiness: Can you actually watch Netflix?

Your television may be switched on, but without an internet connection, Netflix may not work.

Similarly:

- An application can be alive.
- But it may not be ready to serve requests.

### Can we use both probes together?

Yes.

For example:

- Liveness checks whether the application is stuck.
- Readiness checks whether it can serve requests.

If a temporary dependency issue occurs, the application might fail readiness without needing a restart.

If the application becomes permanently stuck, liveness can trigger a restart.

Important: Liveness and readiness are independent checks. Kubernetes does not require one to be configured before the other.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 5. Startup Probe

Now consider an application that takes a long time to start.

For example:

- A lightweight NGINX application might start in a few seconds.
- A large Java application may take several minutes.
- An application server with extensive initialization may take even longer.

Suppose you configure a Liveness Probe that starts checking after 5 seconds.

Your application needs 10 minutes to initialize.

The Liveness Probe may fail repeatedly before the application has finished starting, causing Kubernetes to restart the container unnecessarily.

Startup Probe solves this problem.

Purpose: Protect slow-starting applications

### How does it work?

1. The container starts.
2. Kubernetes runs the Startup Probe.
3. While the Startup Probe has not succeeded, Kubernetes does not run the configured Liveness and Readiness Probes for that container.
4. Once the Startup Probe succeeds, normal Liveness and Readiness checks begin.
5. If the Startup Probe fails the configured number of consecutive times, Kubernetes restarts the container.

### Startup Probe YAML

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-application

spec:
  replicas: 2

  selector:
    matchLabels:
      app: java-app

  template:
    metadata:
      labels:
        app: java-app

    spec:
      containers:
        - name: java-app
          image: my-java-app:1.0

          startupProbe:
            httpGet:
              path: /health
              port: 8080

            periodSeconds: 10
            failureThreshold: 30

          readinessProbe:
            httpGet:
              path: /ready
              port: 8080

            periodSeconds: 10

          livenessProbe:
            httpGet:
              path: /live
              port: 8080

            periodSeconds: 10
```

### Understanding the Startup Probe configuration

Here:

- `periodSeconds: 10` means the probe runs approximately every 10 seconds.
- `failureThreshold: 30` allows 30 consecutive failures before the startup probe is considered failed.

The approximate startup allowance is:

\\[ 30 \times 10 = 300 \text{ seconds} \\]

That is approximately 5 minutes.

This gives the application time to initialize.

Note: The exact timing can be affected by probe execution duration and Kubernetes scheduling. The product of these two values is a useful configuration estimate, not an exact deadline.

### When should we use a Startup Probe?

Use it when:

- Applications take a long time to initialize.
- Java or application-server workloads have slow startup.
- Database services require extensive initialization.
- You want to avoid premature restarts during startup.

For a simple application that starts quickly, a Startup Probe may not be necessary.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 6. Three probes — Complete comparison

| Feature        | Startup                           | Liveness                          | Readiness                              |
| -------------- | --------------------------------- | --------------------------------- | -------------------------------------- |
| Checks         | Successful initialization         | Continued health                  | Ability to serve traffic               |
| When used      | During startup                    | After startup succeeds            | After startup succeeds                 |
| Failure action | Restart container after threshold | Restart container after threshold | Stop routing traffic to it             |
| Main purpose   | Protect slow startup              | Recover from unhealthy state      | Prevent traffic to unready application |

### Visual understanding

## Container health-check lifecycle

Container starts

Startup Probe

Checks initialization

Startup succeeds

Liveness Probe

Restart if unhealthy

Readiness Probe

Control traffic eligibility

Liveness and readiness checks can run independently after startup succeeds.

### Important clarification

You do not have to configure all three probes.

You can configure:

- Only Liveness.
- Only Readiness.
- Liveness and Readiness together.
- Startup along with one or both of the other probes.
- None of them, if the workload does not require them.

The correct choice depends on the application's behavior.

## 7. Probe mechanisms

The instructor also revisited how probes are defined.

Kubernetes supports several probe mechanisms:

| Mechanism  | What it does                                                     |
| ---------- | ---------------------------------------------------------------- |
| HTTP GET   | Sends an HTTP request to an endpoint                             |
| TCP Socket | Checks whether a TCP connection can be established               |
| Exec       | Executes a command inside the container                          |
| gRPC       | Checks application health through the gRPC health-check protocol |

### Example: HTTP GET

```
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
```

### Example: TCP Socket

```
livenessProbe:
  tcpSocket:
    port: 8080
  periodSeconds: 10
```

### Example: Exec

```
livenessProbe:
  exec:
    command:
      - cat
      - /tmp/healthy
  periodSeconds: 10
```

The command must exit with status code `0` for the probe to succeed.

### Example: gRPC

```
readinessProbe:
  grpc:
    port: 50051
```

The application must implement the gRPC health-check protocol.

Practical tip: Use a dedicated health endpoint or command for each purpose. Avoid making health checks unnecessarily expensive.

## 8. Probes vs Monitoring Tools

During the class, an important question was raised: If Kubernetes already has health probes, why do we need monitoring tools?

The answer is that health checks and monitoring solve different problems.

| Kubernetes Probes                    | Monitoring tools                               |
| ------------------------------------ | ---------------------------------------------- |
| Check application health             | Collect detailed operational metrics           |
| Trigger restart or traffic exclusion | Help analyze system behavior                   |
| Basic health validation              | CPU, memory, request rate, error rate, latency |
| Part of Pod specification            | Requires monitoring infrastructure             |

Examples of monitoring tools:

- Prometheus — collects and stores metrics.
- Grafana — visualizes metrics and dashboards.

For example, a Readiness Probe can determine whether an application is ready to receive requests.

Prometheus and Grafana can help answer:

- How many HTTP 500 errors occurred?
- What is the application's response time?
- How much CPU is being consumed?
- How many requests are being processed?

Remember: Probes are not replacements for a full monitoring and observability solution.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub



# 9. How are Pods distributed across worker nodes?

Now the instructor moves to another important scheduling question.

Suppose you have:

- Three worker nodes.
- One Deployment.
- Three replicas.

Will Kubernetes always place one Pod on each node?

No.

Kubernetes Scheduler selects nodes based on scheduling constraints, available resources, policies and scoring.

It does not guarantee a round-robin distribution.

For example, all three replicas could run on Node 1 if the scheduling decisions and constraints allow it.

### Example

## Possible Deployment placement

Node 1

2 Pods

Pod A, Pod B

Node 2

1 Pod

Pod C

Node 3

0 Pods

No replica placed

This is one possible placement, not a guaranteed distribution.

Why might this happen?

The scheduler considers:

- CPU and memory requests.
- Node capacity and allocatable resources.
- Node selectors and affinity.
- Taints and tolerations.
- Topology spread constraints.
- Other scheduling policies and constraints.

### How do we distribute Pods across nodes?

If you want to encourage or require distribution, Kubernetes provides Pod Topology Spread Constraints and Pod Anti-Affinity.

For example, a topology spread constraint can distribute replicas across different nodes or availability zones.

This is a useful additional concept for real-world deployments.

# 10. DaemonSet

Now consider a different requirement.

Suppose you have a Kubernetes cluster with five worker nodes.

You want to run a monitoring agent on every node to collect:

- CPU usage.
- Memory usage.
- Disk information.
- Node-level logs.

Can you achieve this using a Deployment with five replicas?

Not reliably.

A Deployment with five replicas does not guarantee one replica per node. Multiple replicas may be placed on the same node.

There is another problem: if you add a sixth node, a Deployment does not automatically increase its replica count to six.

Kubernetes provides a special workload object called DaemonSet for this requirement.

A DaemonSet ensures that a Pod runs on every eligible node in the cluster.

Purpose: One Pod per eligible node

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 11. How does a DaemonSet work?

Imagine a hostel with several rooms.

You want to install one fire alarm in every room.

- Room 1 gets one fire alarm.
- Room 2 gets one fire alarm.
- Room 3 gets one fire alarm.

When a new room is added, a fire alarm should automatically be installed there.

A DaemonSet works similarly.

### DaemonSet behavior

1. You create a DaemonSet.
2. Kubernetes creates one matching Pod on every eligible node.
3. When a new eligible node is added, a Pod is created there.
4. When a node is removed, its DaemonSet Pod is removed.
5. Kubernetes continuously reconciles the desired state.

### Example

| Cluster event                | DaemonSet behavior |
| ---------------------------- | ------------------ |
| Cluster has 3 eligible nodes | 3 Pods             |
| Add a fourth eligible node   | 4 Pods             |
| Remove one node              | 3 Pods             |
| Add another eligible node    | 4 Pods             |

The actual number of DaemonSet Pods is based on eligible nodes, not a manually specified replica count.

### Common DaemonSet use cases

| Use case              | Example                               |
| --------------------- | ------------------------------------- |
| Log collection        | Fluent Bit or Fluentd                 |
| Monitoring            | Node-level monitoring agents          |
| Networking            | Certain CNI or node networking agents |
| Security              | Node-level security agents            |
| Kubernetes networking | kube-proxy in clusters that use it    |

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 12. DaemonSet YAML example

A DaemonSet looks similar to a Deployment, but the `kind` is different.

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-daemonset

spec:
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
          image: nginx
```

Notice the differences:

| Deployment                                                 | DaemonSet                                        |
| ---------------------------------------------------------- | ------------------------------------------------ |
| `kind: Deployment`                                         | `kind: DaemonSet`                                |
| Specifies `replicas`                                       | No replicas field                                |
| Scheduler places Pods according to constraints and scoring | Controller targets one Pod on each eligible node |
| Used for application workloads                             | Used for node-level workloads                    |

Important: A DaemonSet does not mean every Pod is guaranteed to be Running. If a node lacks resources or has incompatible scheduling constraints, its DaemonSet Pod may remain Pending.

### Useful commands

```
kubectl apply -f daemonset.yaml

kubectl get daemonsets

kubectl get ds

kubectl describe daemonset nginx-daemonset

kubectl get pods -o wide
```

The `kubectl get ds` command displays DaemonSets.

### Understanding DaemonSet status

You may see columns such as:

| Column     | Meaning                                             |
| ---------- | --------------------------------------------------- |
| DESIRED    | Number of eligible nodes that should run the Pod    |
| CURRENT    | Number of nodes currently running a DaemonSet Pod   |
| READY      | Number of DaemonSet Pods that are Ready             |
| UP-TO-DATE | Number of Pods updated to the current specification |
| AVAILABLE  | Number of Pods considered available                 |

## 13. When should we NOT use a DaemonSet?

A DaemonSet is not suitable for every application.

Suppose you have an application that needs to scale based on traffic.

For example:

- Normal traffic: 3 replicas.
- High traffic: 10 replicas.
- Low traffic: 2 replicas.

A Deployment with Horizontal Pod Autoscaler is appropriate.

A DaemonSet is not intended for traffic-based replica scaling because it maintains one Pod per eligible node.

| Requirement                                           | Suitable Kubernetes object |
| ----------------------------------------------------- | -------------------------- |
| Scale application based on traffic                    | Deployment + HPA           |
| Run one log collector per node                        | DaemonSet                  |
| Run one monitoring agent per node                     | DaemonSet                  |
| Run multiple interchangeable web application replicas | Deployment                 |
| Run database replicas with stable identities          | StatefulSet                |

### Important clarification: DaemonSet and automatic Kubernetes clusters

During the instructor's Azure demonstration, the DaemonSet showed a desired count of zero.

The reason was that the instructor was using an AKS Automatic cluster, where the node infrastructure is managed by Azure and the cluster restricts certain node-level operations.

This does not mean DaemonSets are unsupported in ordinary AKS clusters.

The broader lesson is that managed Kubernetes services can restrict workloads that require direct node-level access, depending on the service and its configuration.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



# 14. Stateless vs Stateful Applications

Before understanding StatefulSets, we need to understand the difference between stateless and stateful applications.

### Stateless application

A stateless application does not depend on session or application state stored locally on a particular application instance between requests.

Each request can generally be handled independently.

Example:

Imagine visiting a news website.

You open the website today and see the latest news. You return tomorrow and see updated news.

The application does not need to remember which particular application server handled your previous request.

Examples:

- Stateless REST APIs.
- Many microservices.
- Static web servers.

### Stateful application

A stateful application maintains information across interactions.

Example:

Imagine adding two products to your Amazon shopping cart.

You leave the website and return later. The products are still in your cart.

The application has retained your previous state.

Other examples:

- Databases.
- Online games with saved scores.
- Applications maintaining server-side sessions.
- Systems maintaining persistent transaction state.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

### Stateless vs Stateful comparison

| Feature                         | Stateless                                  | Stateful                                      |
| ------------------------------- | ------------------------------------------ | --------------------------------------------- |
| Remembers previous interactions | Does not depend on local previous state    | Maintains or depends on previous state        |
| Session information             | Often externalized or carried with request | May be stored on server or persistent storage |
| Scaling                         | Generally simpler                          | Requires coordination and data consistency    |
| Pod replacement                 | Usually straightforward                    | Must preserve identity and data               |
| Examples                        | REST APIs, web frontends                   | Databases, stateful services                  |

Important clarification: A stateless application can still use a database or cache. Stateless means the application instance does not depend on its own local memory or identity to retain client state. The state can be stored externally.

# 15. Deployment vs StatefulSet

In previous sessions, we learned that Deployments manage multiple Pod replicas.

Now let's understand what happens when we use a Deployment.

Suppose a Deployment has three replicas:

```
nginx-deployment
    |
    |-- nginx-abc123
    |-- nginx-def456
    |-- nginx-ghi789
```

If one Pod is deleted, the ReplicaSet creates a replacement Pod.

The replacement has a new Pod identity and generally a different generated name.

For a stateless web application, this is perfectly fine.

But what if you are running a database?

You may need:

- Stable Pod names.
- Stable network identities.
- Persistent storage associated with individual Pods.
- Predictable creation and deletion order.

This is where StatefulSet is useful.

A StatefulSet manages stateful applications and gives each Pod a stable, predictable identity.

Purpose: Stable identity and persistent storage association

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 16. What is a StatefulSet?

A StatefulSet is a Kubernetes workload API object used to manage stateful applications.

It provides:

1. Stable and predictable Pod names.
2. Stable network identities.
3. Persistent storage association.
4. Ordered Pod creation.
5. Ordered scaling.
6. Ordered rolling updates.

### Example

Suppose you create a StatefulSet with three replicas.

Kubernetes creates:

```
mysql-0
mysql-1
mysql-2
```

Unlike Deployment-generated Pod names, these identities are predictable.

If `mysql-1` is deleted, StatefulSet creates a replacement with the same ordinal identity:

```
mysql-1
```

The replacement is a new Pod instance, not the original Pod object. However, it retains the same StatefulSet identity and can reconnect to its associated persistent storage.

### Deployment vs StatefulSet — Pod identity

## Deployment

Three interchangeable replicas

Pod A

Generated name

Pod B

Generated name

Pod C

Generated name

If Pod B is deleted, a replacement with a new generated name is created.

## StatefulSet

Three stable identities

mysql-0

Ordinal 0

mysql-1

Ordinal 1

mysql-2

Ordinal 2

If mysql-1 is deleted, its replacement is again named mysql-1.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 17. StatefulSet and Persistent Volumes

A StatefulSet is especially useful when each Pod needs its own persistent storage.

For example:

| Pod       | Persistent storage |
| --------- | ------------------ |
| `mysql-0` | PVC for mysql-0    |
| `mysql-1` | PVC for mysql-1    |
| `mysql-2` | PVC for mysql-2    |

StatefulSets support this through `volumeClaimTemplates`.

Each Pod receives its own PVC based on the template.

If `mysql-1` is deleted and recreated, its replacement can use the PVC associated with `mysql-1`.

This is why StatefulSets are useful for databases and other stateful systems.

Important: A StatefulSet does not automatically make a database highly available, replicate database contents, elect a primary or guarantee data consistency. Those functions must be implemented by the database or its operator. StatefulSet provides the stable identities and storage association that help run such systems.

## 18. StatefulSet and Ordered Scaling

One important difference between Deployment and StatefulSet is the default ordering behavior.

Suppose you have three StatefulSet Pods:

```
mysql-0
mysql-1
mysql-2
```

When scaling up from zero, Kubernetes normally creates them in order:

```
mysql-0 → mysql-1 → mysql-2
```

When scaling down from three to one, Kubernetes normally terminates them in reverse order:

```
mysql-2 → mysql-1 → mysql-0
```

So if you scale from three replicas to one, the remaining Pod is `mysql-0`.

This is useful for database clusters where Pod identities may represent different roles or positions.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

### Important clarification

StatefulSet ordering is not the same as database primary/replica management.

For example, StatefulSet does not automatically decide that `mysql-0` must be the database primary. The database configuration or clustering software must establish that role.

Also, ordered creation and deletion behavior can be configured through StatefulSet management policies.

## 19. StatefulSet YAML example

A simplified StatefulSet example:

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web

spec:
  serviceName: web-headless
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
          image: nginx
          ports:
            - containerPort: 80
```

The Pods will be named:

```
web-0
web-1
web-2
```

### Why is serviceName required?

A StatefulSet uses a governing Service to provide stable network identities for its Pods.

This is commonly a headless Service.

Example:

```
apiVersion: v1
kind: Service
metadata:
  name: web-headless

spec:
  clusterIP: None

  selector:
    app: nginx

  ports:
    - port: 80
      targetPort: 80
```

The headless Service does not provide a normal virtual ClusterIP. Instead, it enables DNS records that can identify individual StatefulSet Pods.

For example, a Pod DNS name can look like:

```
web-0.web-headless.default.svc.cluster.local
```

The exact namespace and Service name depend on your configuration.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

### 19.1 StatefulSet with persistent storage

To give each Pod its own PVC, add `volumeClaimTemplates`:

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql

spec:
  serviceName: mysql-headless
  replicas: 3

  selector:
    matchLabels:
      app: mysql

  template:
    metadata:
      labels:
        app: mysql

    spec:
      containers:
        - name: mysql
          image: mysql:8.0

          volumeMounts:
            - name: mysql-data
              mountPath: /var/lib/mysql

  volumeClaimTemplates:
    - metadata:
        name: mysql-data

      spec:
        accessModes:
          - ReadWriteOnce

        resources:
          requests:
            storage: 10Gi
```

This is a simplified storage example; a suitable Storage Class and MySQL configuration are also required.

The PVCs are created with names such as:

```
mysql-data-mysql-0
mysql-data-mysql-1
mysql-data-mysql-2
```

Each Pod has its own claim.

Remember: Deleting a StatefulSet Pod does not normally delete its PVC. Deleting the StatefulSet also does not automatically delete the PVCs created from its volume claim templates.

# 20. Deployment vs DaemonSet vs StatefulSet

This is a very important interview comparison.

| Feature            | Deployment                  | DaemonSet                        | StatefulSet                                 |
| ------------------ | --------------------------- | -------------------------------- | ------------------------------------------- |
| Main purpose       | Manage application replicas | Run a Pod on every eligible node | Manage stateful application replicas        |
| Replica count      | Explicit                    | Based on eligible nodes          | Explicit                                    |
| Pod identity       | Interchangeable replicas    | One per eligible node            | Stable ordinal identity                     |
| Pod names          | Generated names             | Generated names                  | Predictable names                           |
| Persistent storage | Can use PVCs                | Can use volumes/PVCs             | Supports per-Pod PVC templates              |
| Ordered scaling    | Not generally guaranteed    | Based on eligible nodes          | Ordered by default                          |
| Typical use        | Web applications, APIs      | Logging and monitoring agents    | Databases and distributed stateful services |

### Easy way to remember

- Deployment: I need a certain number of application replicas.
- DaemonSet: I need one Pod on every eligible node.
- StatefulSet: I need Pods with stable identities and storage associations.

## 21. Common real-world scenarios

| Scenario                                          | Kubernetes resource | Why?                                    |
| ------------------------------------------------- | ------------------- | --------------------------------------- |
| Deploy a web application with 3 replicas          | Deployment          | Manages interchangeable replicas        |
| Automatically restart an unhealthy container      | Liveness Probe      | Detects application failure             |
| Prevent traffic during application initialization | Readiness Probe     | Controls traffic eligibility            |
| Protect a slow-starting Java application          | Startup Probe       | Allows initialization time              |
| Install a log collector on every node             | DaemonSet           | One Pod per eligible node               |
| Install monitoring agents on every node           | DaemonSet           | Node-level monitoring                   |
| Deploy MySQL with stable Pod identities           | StatefulSet         | Stable identity and storage             |
| Run Redis replicas with persistent identity       | StatefulSet         | Stable identity and storage association |
| Scale API replicas based on CPU usage             | Deployment + HPA    | Dynamic replica scaling                 |

## 22. Useful kubectl commands

| Command                                       | Purpose                            |
| --------------------------------------------- | ---------------------------------- |
| `kubectl get pods`                            | List Pods                          |
| `kubectl get pods -o wide`                    | View node placement                |
| `kubectl describe pod <pod-name>`             | Inspect Pod details and events     |
| `kubectl logs <pod-name>`                     | View container logs                |
| `kubectl logs <pod-name> -c <container-name>` | View a particular container's logs |
| `kubectl get deployments`                     | List Deployments                   |
| `kubectl get daemonsets`                      | List DaemonSets                    |
| `kubectl get ds`                              | Short form                         |
| `kubectl describe ds <name>`                  | Inspect a DaemonSet                |
| `kubectl get statefulsets`                    | List StatefulSets                  |
| `kubectl get sts`                             | Short form                         |
| `kubectl describe sts <name>`                 | Inspect a StatefulSet              |
| `kubectl get pvc`                             | List Persistent Volume Claims      |
| `kubectl get pv`                              | List Persistent Volumes            |
| `kubectl get services`                        | List Services                      |

## 23. Troubleshooting scenarios

| Problem                                         | What to check                                                                  |
| ----------------------------------------------- | ------------------------------------------------------------------------------ |
| Container repeatedly restarts                   | Liveness or Startup Probe failures, application logs and events                |
| Pod is Running but not receiving traffic        | Readiness Probe and Service endpoints                                          |
| Startup Probe keeps failing                     | Application startup duration, endpoint and failure threshold                   |
| DaemonSet shows desired count zero              | Eligible nodes, node selectors, node affinity and managed-cluster restrictions |
| DaemonSet Pod remains Pending                   | Node capacity, taints, tolerations and scheduling events                       |
| StatefulSet Pod is not created                  | Earlier ordinal Pod readiness, storage and scheduling                          |
| StatefulSet Pod is recreated with the same name | Expected stable identity behavior                                              |
| StatefulSet Pod cannot mount its PVC            | Storage backend, PVC binding, CSI driver and node attachment                   |
| Application traffic is unevenly distributed     | Check Service backends, readiness and Pod scheduling constraints               |

Useful commands:

```
kubectl describe pod <pod-name>

kubectl get pods -o wide

kubectl get events --sort-by=.metadata.creationTimestamp

kubectl get ds

kubectl describe ds <daemonset-name>

kubectl get sts

kubectl describe sts <statefulset-name>

kubectl get pvc
```

## 24. Interview questions and answers

Q1. What are the three types of Kubernetes probes?

Liveness, Readiness and Startup.

Q2. What is the difference between Liveness and Readiness?

Liveness checks whether a container is functioning and can trigger a restart. Readiness checks whether it can accept traffic and removes it from eligible Service backends when it fails.

Q3. Does a failed Readiness Probe restart a container?

No. It marks the Pod as not ready, so it stops receiving normal Service traffic.

Q4. Why do we need a Startup Probe?

To protect applications that take a long time to initialize from premature Liveness Probe failures and restarts.

Q5. What happens after a Startup Probe succeeds?

The configured Liveness and Readiness Probes can begin running.

Q6. Can we configure all three probes in the same container?

Yes. They can be configured together or independently.

Q7. How frequently does Kubernetes execute a probe?

It depends on the `periodSeconds` configuration, subject to Kubernetes probe timing behavior.

Q8. Are Kubernetes probes a replacement for Prometheus and Grafana?

No. Probes provide health-check decisions. Monitoring systems collect and visualize detailed operational metrics.

Q9. Does Kubernetes distribute Deployment Pods equally across nodes?

No. The scheduler considers resource availability and scheduling constraints. Equal distribution is not guaranteed.

Q10. What is a DaemonSet?

A Kubernetes workload object that ensures a Pod runs on every eligible node.

Q11. Why do we use DaemonSets?

For node-level workloads such as logging agents, monitoring agents, networking and security agents.

Q12. What happens when a new node is added to the cluster?

A DaemonSet creates a Pod on the new node if it is eligible.

Q13. Does a DaemonSet require a replica count?

No. The number of Pods is determined by the eligible nodes.

Q14. Can we use a DaemonSet for traffic-based autoscaling?

It is not intended for that. Use a Deployment with HPA for traffic-based replica scaling.

Q15. What is the difference between Stateful and Stateless applications?

Stateless applications do not depend on local previous-request state. Stateful applications maintain or depend on persistent state across interactions.

Q16. What is a StatefulSet?

A Kubernetes workload object that manages Pods with stable identities, predictable names, ordered operations and persistent storage association.

Q17. What happens if a StatefulSet Pod is deleted?

Kubernetes recreates the Pod with the same ordinal identity and can reconnect it to its associated PVC.

Q18. What is the difference between Deployment and StatefulSet?

Deployment manages interchangeable replicas. StatefulSet manages Pods that need stable identities and storage associations.

Q19. Why do StatefulSets use headless Services?

To provide stable DNS identities for individual StatefulSet Pods.

Q20. Does StatefulSet automatically configure database replication?

No. It provides workload identity, ordering and storage association. Database replication and primary/replica roles require application-specific configuration.

## 25. Quick revision

| Topic             | Remember this                                          |
| ----------------- | ------------------------------------------------------ |
| Liveness Probe    | Is the application functioning?                        |
| Readiness Probe   | Can the application accept traffic?                    |
| Startup Probe     | Has the application finished initializing?             |
| Liveness failure  | Container restart                                      |
| Readiness failure | Stop routing traffic to the Pod                        |
| Startup failure   | Container restart after threshold                      |
| Probe period      | How frequently checks run                              |
| Deployment        | Manages interchangeable replicas                       |
| Scheduler         | Chooses eligible nodes for Pods                        |
| DaemonSet         | One Pod per eligible node                              |
| StatefulSet       | Stable Pod identity and persistent storage association |
| Stateless         | Does not depend on local previous-request state        |
| Stateful          | Maintains or depends on persistent state               |
| Headless Service  | Enables stable Pod DNS identities                      |
| PVC               | Persistent storage associated with a StatefulSet Pod   |

## 26. Final mental model

## Kubernetes workload management

Kubernetes application

Deployment

Application replicas

DaemonSet

Node-level agents

StatefulSet

Stateful workloads

Container health management

Liveness

Restart

Readiness

Traffic control

Startup

Initialization

## 27. Topics covered in this session

- Kubernetes Liveness Probe.
- Kubernetes Readiness Probe.
- Kubernetes Startup Probe.
- HTTP, TCP, Exec and gRPC probe mechanisms.
- Probe timing and failure thresholds.
- Probes vs monitoring tools.
- How Kubernetes schedules Deployment replicas.
- DaemonSets and their real-world use cases.
- DaemonSet vs Deployment.
- AKS Automatic cluster scheduling restrictions.
- Stateless vs Stateful applications.
- StatefulSets and stable Pod identities.
- Persistent storage association using PVCs.
- Ordered StatefulSet scaling and updates.
- Deployment vs DaemonSet vs StatefulSet.

Next topic: The session introduced StatefulSets and completed the discussion with their identity and ordering behavior. The next class should continue the Kubernetes workload and resource-management topics.

Original transcript: [DevOps Session — 19 October 2026 (GitHub)](https://raw.githubusercontent.com/jeevanadv2000-pixel/python/main/DevOps%20Session%28Morning%29%20transcript_2026-10-19_09.10.36.txt?utm_source=chatgpt.com)