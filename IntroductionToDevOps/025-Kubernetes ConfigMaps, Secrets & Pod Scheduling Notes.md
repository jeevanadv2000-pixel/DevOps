# DevOps Session — Kubernetes ConfigMaps, Secrets & Pod Scheduling Notes

Date: 13 October 2026\
Instructor: Rochak Agrawal\
Focus: ConfigMaps, Secrets, Node Selectors, Node Affinity, Taints and Tolerations.

These notes are condensed from the class transcript. Participant discussions and repetitions have been removed, speech-to-text errors corrected, and technical oversimplifications clarified.

## 1. ConfigMap — What is it?

Before understanding ConfigMaps, let's understand a common application deployment problem.

Imagine you have one application Docker image that needs to run in three environments:

- Development
- Testing
- Production

The application connects to a database using a database host variable.

| Environment | Database host         |
| ----------- | --------------------- |
| Development | `dev-db.example.com`  |
| Testing     | `test-db.example.com` |
| Production  | `prod-db.example.com` |

If you hardcode the database host inside the application image, you may need to build a different image for each environment.

That is not ideal.

The solution is to separate application configuration from the application image.

This is where ConfigMaps are useful.

A ConfigMap is a Kubernetes API object used to store non-sensitive configuration data as key-value pairs.

Examples:

- Database hostname
- Application environment
- Application port
- Feature flags
- Configuration files
- Application-specific settings

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

### How ConfigMap works

One Docker image

Same application image in every environment

Dev ConfigMap

dev-db.example.com

Test ConfigMap

test-db.example.com

Prod ConfigMap

prod-db.example.com

Each environment supplies its own configuration to the same application image.

Important: A ConfigMap only stores configuration data. It does not establish database connections or implement application logic. Your application must read the supplied configuration and use it.

### 1.1 Creating a ConfigMap

There are multiple ways to create a ConfigMap.

Method 1: Using literal values

```
kubectl create configmap app-config \
  --from-literal=DB_HOST=dev-db.example.com \
  --from-literal=APP_ENV=development
```

Here:

- `app-config` is the ConfigMap name.
- `DB_HOST` and `APP_ENV` are keys.
- The values after `=` are their corresponding values.

Method 2: From a single file

```
kubectl create configmap app-config \
  --from-file=application.properties
```

The filename becomes the ConfigMap key, and the file contents become its value.

Method 3: From multiple files

```
kubectl create configmap app-config \
  --from-file=application.properties \
  --from-file=database.properties
```

Method 4: From a directory

```
kubectl create configmap app-config \
  --from-file=./config/
```

Kubernetes packages eligible files from that directory as ConfigMap entries. Each filename becomes a key.

Method 5: Using YAML

Create `configmap.yaml`:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DB_HOST: dev-db.example.com
  APP_ENV: development
  APP_PORT: "8080"
```

Apply it:

```
kubectl apply -f configmap.yaml
```

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

### 1.2 Viewing a ConfigMap

```
kubectl get configmaps
```

Short form:

```
kubectl get cm
```

View a specific ConfigMap:

```
kubectl describe configmap app-config
```

View its YAML:

```
kubectl get configmap app-config -o yaml
```

Remember that ConfigMaps are namespace-scoped. If you do not specify a namespace, kubectl uses the current namespace.

### 1.3 How does a Pod consume a ConfigMap?

Creating a ConfigMap is only the first step. Your application must receive the values.

There are two common ways:

1. As environment variables.
2. As mounted files using a volume.

Method A: ConfigMap as environment variables

Example Pod YAML:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-config-pod
spec:
  containers:
    - name: nginx
      image: nginx
      env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: DB_HOST
```

Understand these fields:

| Field                          | Meaning                                                       |
| ------------------------------ | ------------------------------------------------------------- |
| `name`                         | Environment variable name inside the container                |
| `valueFrom`                    | Indicates that the value comes from another Kubernetes object |
| `configMapKeyRef`              | References a particular ConfigMap key                         |
| `name` under `configMapKeyRef` | ConfigMap resource name                                       |
| `key`                          | The key whose value should be injected                        |

You can also inject all ConfigMap keys as environment variables:

```
envFrom:
  - configMapRef:
      name: app-config
```

Method B: ConfigMap as a volume

Instead of environment variables, you can mount the ConfigMap into the container as files.

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-config-pod
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
        - name: config-volume
          mountPath: /etc/app-config
  volumes:
    - name: config-volume
      configMap:
        name: app-config
```

The ConfigMap entries become files inside `/etc/app-config`.

For example:

```
/etc/app-config/DB_HOST
/etc/app-config/APP_ENV
/etc/app-config/APP_PORT
```

The application can read these files.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

### 1.4 What happens when a ConfigMap is updated?

This is an important interview question.

The behavior depends on how the Pod consumes the ConfigMap.

| Consumption method             | What happens after ConfigMap update?           |
| ------------------------------ | ---------------------------------------------- |
| Environment variable           | Existing container environment does not change |
| Mounted volume                 | Files are eventually updated by Kubernetes     |
| Mounted volume using `subPath` | Automatic updates are not propagated           |

If you use environment variables, you generally need to restart or recreate the Pods to load the new values.

For a Deployment, you can trigger a rollout:

```
kubectl rollout restart deployment nginx-deployment
```

For mounted ConfigMap volumes, updates are eventually reflected, but not instantaneously. Your application may also need to reread the file to use the new configuration.

Remember: Updating a ConfigMap does not automatically mean that your application starts using the new value.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 2. Kubernetes Secrets

A ConfigMap is intended for non-sensitive configuration.

But what if you need to store:

- Database passwords
- API keys
- Authentication tokens
- TLS certificates
- Private keys

You should use a Kubernetes Secret.

A Secret is a Kubernetes API object designed to hold sensitive information separately from application code and container images.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

### 2.1 ConfigMap vs Secret

| Feature             | ConfigMap                         | Secret                                      |
| ------------------- | --------------------------------- | ------------------------------------------- |
| Purpose             | Non-sensitive configuration       | Sensitive information                       |
| Typical data        | DB hostname, application settings | Passwords, tokens, keys                     |
| Data representation | Plain text                        | Base64 encoded in standard Secret manifests |
| Intended use        | Application configuration         | Credentials and confidential data           |
| Size limit          | 1 MiB                             | 1 MiB                                       |

Important security point

Base64 encoding is not encryption.

For example, a password such as:

```
demo@123
```

can be encoded into Base64:

```
echo -n 'demo@123' | base64
```

To decode it:

```
echo -n 'BASE64_VALUE' | base64 --decode
```

Anyone who can read the encoded value can decode it.

So, do not assume that a Kubernetes Secret is automatically secure just because its contents are Base64 encoded.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

### 2.2 Creating a Secret

Method 1: Using kubectl

```
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password='StrongPassword123'
```

Here:

- `generic` creates an Opaque Secret.
- `db-secret` is the Secret name.
- `username` and `password` are the keys.

Method 2: Using YAML

First, encode the values:

```
echo -n 'admin' | base64
echo -n 'StrongPassword123' | base64
```

Then create `secret.yaml`:

```
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=
  password: BASE64_ENCODED_PASSWORD
```

Apply:

```
kubectl apply -f secret.yaml
```

Alternative: Use `stringData` to provide plaintext values in a manifest; Kubernetes converts them into the Secret's stored data representation when processing the request.

```
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
stringData:
  username: admin
  password: StrongPassword123
```

Be careful: this YAML still contains plaintext credentials. Do not commit it to Git.

### 2.3 Secret types

Kubernetes supports different Secret types.

| Type                                  | Purpose                             |
| ------------------------------------- | ----------------------------------- |
| `Opaque`                              | General-purpose user-defined Secret |
| `kubernetes.io/service-account-token` | Service account token               |
| `kubernetes.io/tls`                   | TLS certificate and private key     |
| `kubernetes.io/dockerconfigjson`      | Docker registry authentication      |
| `kubernetes.io/basic-auth`            | Basic authentication credentials    |
| `kubernetes.io/ssh-auth`              | SSH authentication credentials      |

The most commonly used general-purpose type is `Opaque`.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

### 2.4 Using a Secret inside a Pod

A Secret can be consumed through environment variables or mounted as a volume, just like a ConfigMap.

Method A: Secret as an environment variable

```
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
    - name: app
      image: nginx
      env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
```

The application receives the password as the `DB_PASSWORD` environment variable.

Method B: Secret as a volume

```
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
    - name: app
      image: nginx
      volumeMounts:
        - name: secret-volume
          mountPath: /etc/secrets
          readOnly: true
  volumes:
    - name: secret-volume
      secret:
        secretName: db-secret
```

The Secret keys become files inside `/etc/secrets`.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

### 2.5 Best practices for Kubernetes Secrets

This is particularly important in real DevOps projects.

1. Never hardcode passwords or API keys inside Docker images.
2. Never commit plaintext Secret manifests to Git.
3. Configure encryption at rest for Kubernetes Secrets.
4. Use RBAC to restrict who can read, update and delete Secrets.
5. Use external secret-management solutions where appropriate.
6. Avoid exposing Secrets through logs, command output or environment dumps.

Examples of external secret-management approaches:

- HashiCorp Vault
- Azure Key Vault
- AWS Secrets Manager
- External Secrets Operator
- Sealed Secrets

Important distinction: Kubernetes Secret objects are not necessarily encrypted at rest by default. Configure encryption at rest for the cluster's backing data store. Also, anyone with sufficient permissions to read a Secret can obtain its contents.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 3. Pod Scheduling — Why do we need it?

Now let's move to the next topic.

Suppose your Kubernetes cluster has multiple worker nodes.

You deploy ten Pods. Kubernetes scheduler decides which nodes should run them.

But imagine:

- Node 1 has high CPU and memory capacity.
- Node 2 has lower capacity.
- Node 3 is dedicated to database workloads.

You may want:

- Database Pods to run on Node 3.
- Application Pods to run on Node 1 or Node 2.
- Certain workloads to avoid Node 3 completely.

How do we control this?

Kubernetes provides three important scheduling mechanisms:

1. Node Selector
2. Node Affinity
3. Taints and Tolerations

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 4. Node Selector

A Node Selector is the simplest way to control where a Pod gets scheduled.

It uses labels attached to worker nodes.

### 4.1 What are node labels?

Labels are key-value pairs attached to Kubernetes objects.

Example:

```
environment=production
region=eastus
workload=database
```

You can view the nodes and their labels:

```
kubectl get nodes --show-labels
```

### 4.2 Adding a label to a node

```
kubectl label nodes node01 workload=database
```

This adds the label:

```
workload=database
```

to `node01`.

To view the node labels:

```
kubectl get nodes --show-labels
```

To remove a label:

```
kubectl label nodes node01 workload-
```

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

### 4.3 Using Node Selector in a Pod

Suppose you want a Pod to run only on a node with:

```
workload=database
```

Pod YAML:

```
apiVersion: v1
kind: Pod
metadata:
  name: database-pod
spec:
  nodeSelector:
    workload: database
  containers:
    - name: nginx
      image: nginx
```

The scheduler looks for nodes that have the matching label.

If no eligible node exists, the Pod remains Pending.

### 4.4 Node Selector in a Deployment

You can also define the Node Selector inside a Deployment.

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
      nodeSelector:
        workload: database
      containers:
        - name: nginx
          image: nginx
```

All Pods created by this Deployment must satisfy the node selector.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

### 4.5 Important limitation of Node Selector

Suppose:

- Node 1 has `workload=database`.
- Your database application uses a Node Selector.
- It gets scheduled on Node 1.

Does that mean other applications cannot run on Node 1?

No.

Node Selector only tells the selected application where it is allowed to run. It does not prevent other Pods from being scheduled on the same node.

This is the limitation that leads us to Taints and Tolerations.

## 5. Taints and Tolerations

Let's understand the problem.

Imagine you have a dedicated database node.

You want:

- Database Pods to run on that node.
- Other application Pods to avoid that node.

A Node Selector alone cannot guarantee this.

For this, Kubernetes provides Taints and Tolerations.

### 5.1 What is a Taint?

A Taint is applied to a node to repel Pods that do not have a matching toleration.

Think of it as putting a restriction on a node.

Example:

```
kubectl taint nodes node01 workload=database:NoSchedule
```

This means that Pods without a matching toleration cannot newly schedule onto `node01`.

### 5.2 What is a Toleration?

A Toleration is configured on a Pod to allow it to be scheduled onto a node with a matching taint.

Think of it as permission to enter a restricted area.

For example:

- Taint: Node is restricted.
- Toleration: Pod is permitted to tolerate that restriction.

Important: A toleration does not force a Pod onto that node. It only removes that particular taint as a scheduling barrier. You can combine it with a Node Selector or Node Affinity to select the intended node.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

### 5.3 Three Taint effects

This is one of the most important Kubernetes interview questions.

| Taint effect       | Meaning                                                                                 |
| ------------------ | --------------------------------------------------------------------------------------- |
| `NoSchedule`       | New Pods without matching toleration cannot be scheduled on the node                    |
| `PreferNoSchedule` | Scheduler tries to avoid the node, but may schedule Pods there                          |
| `NoExecute`        | New non-tolerating Pods are not scheduled, and existing non-tolerating Pods are evicted |

Let's understand each one.

A. NoSchedule

```
kubectl taint nodes node01 workload=database:NoSchedule
```

- New Pods without matching toleration cannot be scheduled on the node.
- Existing Pods continue running.
- Pods with matching tolerations are allowed past this taint restriction.

B. PreferNoSchedule

```
kubectl taint nodes node01 workload=database:PreferNoSchedule
```

- Kubernetes tries to avoid scheduling non-tolerating Pods on this node.
- It is a soft restriction.
- It may still schedule Pods there if necessary.

C. NoExecute

```
kubectl taint nodes node01 workload=database:NoExecute
```

- New Pods without matching toleration cannot schedule there.
- Existing Pods without matching toleration are evicted.
- Pods with matching tolerations can remain or be scheduled, subject to other scheduling requirements.

A toleration can specify `tolerationSeconds` with `NoExecute` to allow a Pod to remain for a specified duration before eviction.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

### 5.4 Taint and Toleration YAML example

Suppose the node has this taint:

```
workload=database:NoSchedule
```

The Pod needs this toleration:

```
apiVersion: v1
kind: Pod
metadata:
  name: database-pod
spec:
  tolerations:
    - key: "workload"
      operator: "Equal"
      value: "database"
      effect: "NoSchedule"
  containers:
    - name: nginx
      image: nginx
```

Here:

| Field      | Meaning                        |
| ---------- | ------------------------------ |
| `key`      | Taint key                      |
| `operator` | How the key/value should match |
| `value`    | Taint value                    |
| `effect`   | Taint effect to tolerate       |

### 5.5 Combining Node Selector + Taint + Toleration

This is the most important practical concept from the session.

Suppose you want to dedicate Node 1 to a database application.

You need two things:

1. Prevent other Pods from scheduling on Node 1.
2. Make sure your database Pods are scheduled on Node 1.

Use the combination:

- Taint the node to repel unwanted Pods.
- Add a toleration to the database Pods.
- Add a Node Selector to direct database Pods to that node.

## Dedicated database node

Node 1

Tainted: NoSchedule

workload=database

Database Pod

Node Selector: workload=database

Toleration: workload=database:NoSchedule

Allowed

Other application Pods

No matching toleration

Blocked by taint

Example commands:

```
# Add label to the dedicated node
kubectl label nodes node01 workload=database

# Apply taint to the node
kubectl taint nodes node01 workload=database:NoSchedule
```

Then configure the Deployment:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: database-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: database
  template:
    metadata:
      labels:
        app: database
    spec:
      nodeSelector:
        workload: database
      tolerations:
        - key: "workload"
          operator: "Equal"
          value: "database"
          effect: "NoSchedule"
      containers:
        - name: database
          image: nginx
```

This example uses NGINX only to demonstrate scheduling; it is not a production database deployment.

Remember the difference:

- Node Selector: Where should my Pod go?
- Taint: Which Pods should be kept away from this node?
- Toleration: Which taints can my Pod tolerate?

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

### 5.6 Removing a Taint

To remove the taint:

```
kubectl taint nodes node01 workload=database:NoSchedule-
```

The trailing `-` removes the taint.

To remove a label:

```
kubectl label nodes node01 workload-
```

These commands do different things: one removes a taint, and the other removes a label.

## 6. Node Affinity

Node Selector works well for simple scheduling requirements.

But what if you want more flexible scheduling?

For example:

- Prefer nodes in a particular region.
- Require nodes with a particular environment label.
- Schedule on a preferred node if possible, but allow other nodes if necessary.

This is where Node Affinity comes in.

Node Affinity controls Pod placement based on node labels, with more expressive matching rules than Node Selector.

### 6.1 Two main types

| Node Affinity type                                | Meaning          |
| ------------------------------------------------- | ---------------- |
| `requiredDuringSchedulingIgnoredDuringExecution`  | Hard requirement |
| `preferredDuringSchedulingIgnoredDuringExecution` | Soft preference  |

A. Required During Scheduling

The Pod must find a node matching the specified condition.

If no eligible node exists, the Pod remains Pending.

Example:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: environment
                operator: In
                values:
                  - production
  containers:
    - name: nginx
      image: nginx
```

This Pod can only be scheduled on nodes whose `environment` label has the value `production`.

B. Preferred During Scheduling

The scheduler tries to place the Pod on the preferred node, but it can use another eligible node if necessary.

Example:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 80
          preference:
            matchExpressions:
              - key: environment
                operator: In
                values:
                  - production
  containers:
    - name: nginx
      image: nginx
```

The scheduler gives preference to matching nodes. The `weight` influences the scheduling score; it does not guarantee placement.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

### 6.2 Node Selector vs Node Affinity

| Feature                    | Node Selector                  | Node Affinity                       |
| -------------------------- | ------------------------------ | ----------------------------------- |
| Purpose                    | Restrict Pod to matching nodes | Control placement using node labels |
| Matching                   | Exact key-value match          | Supports operators and expressions  |
| Hard scheduling rule       | Yes                            | Yes                                 |
| Soft scheduling preference | No                             | Yes                                 |
| Flexibility                | Basic                          | More flexible                       |

Interview point: Node Selector is a simple hard requirement. Node Affinity supports both hard requirements and soft preferences.

## 7. Important comparison — Node Selector vs Taints vs Affinity

| Concept       | Applied to | Main purpose                                       |
| ------------- | ---------- | -------------------------------------------------- |
| Node Selector | Pod        | Select nodes using exact labels                    |
| Node Affinity | Pod        | Express node placement requirements or preferences |
| Taint         | Node       | Repel Pods that do not tolerate it                 |
| Toleration    | Pod        | Allow Pod to tolerate a node's taint               |

A useful way to remember:

- Selector/Affinity = Attraction: Where do I want to run?
- Taint = Repulsion: Which Pods should stay away?
- Toleration = Permission: Which restrictions can I tolerate?

## 8. Useful kubectl commands

| Command                                                    | Purpose                 |
| ---------------------------------------------------------- | ----------------------- |
| `kubectl get nodes`                                        | List nodes              |
| `kubectl get nodes --show-labels`                          | View node labels        |
| `kubectl describe node node01`                             | Inspect a node          |
| `kubectl label nodes node01 workload=database`             | Add a label             |
| `kubectl label nodes node01 workload-`                     | Remove a label          |
| `kubectl taint nodes node01 workload=database:NoSchedule`  | Add a taint             |
| `kubectl taint nodes node01 workload=database:NoSchedule-` | Remove a taint          |
| `kubectl get pods -o wide`                                 | View Pod placement      |
| `kubectl describe pod database-pod`                        | Troubleshoot scheduling |
| `kubectl get configmaps`                                   | List ConfigMaps         |
| `kubectl get secrets`                                      | List Secrets            |
| `kubectl get configmap app-config -o yaml`                 | View ConfigMap YAML     |
| `kubectl describe secret db-secret`                        | Inspect Secret metadata |

## 9. Troubleshooting scenarios

| Scenario                                       | Possible reason                                                     |
| ---------------------------------------------- | ------------------------------------------------------------------- |
| Pod stays Pending after adding Node Selector   | No node has the required label                                      |
| Pod cannot schedule on a tainted node          | Missing matching toleration                                         |
| Other Pods are running on a dedicated node     | Node Selector alone does not repel other Pods                       |
| Existing Pods are evicted after adding a taint | The taint uses `NoExecute`                                          |
| ConfigMap value does not change in application | Environment variable needs Pod restart                              |
| Mounted ConfigMap file is not updating         | Check volume mounting, `subPath`, and update propagation            |
| Secret value looks unreadable                  | It is Base64 encoded                                                |
| Secret is exposed in Git                       | The YAML file itself contains credentials; Base64 is not encryption |

## 10. Interview questions and answers

Q1. What is a ConfigMap?

A Kubernetes object used to store non-sensitive configuration data as key-value pairs, separately from application code and container images.

Q2. Why do we use ConfigMaps?

To deploy the same container image across multiple environments while supplying different environment-specific configuration.

Q3. What are the ways to use a ConfigMap in a Pod?

As environment variables or as a mounted volume.

Q4. What happens when a ConfigMap is updated?

Environment variables in existing containers do not change. Mounted volume files are eventually updated, except when mounted using `subPath`.

Q5. What is a Secret?

A Kubernetes object used to store sensitive information such as passwords, tokens and certificates.

Q6. Is Base64 encryption?

No. It is an encoding format and can be decoded without a secret key.

Q7. Are Kubernetes Secrets encrypted at rest by default?

Not necessarily. Encryption at rest must be configured for the cluster.

Q8. Can we store Secret YAML files in Git?

Avoid committing plaintext credentials or ordinary Base64-encoded Secret manifests. Use secure secret-management workflows.

Q9. What is Node Selector?

A simple way to schedule Pods onto nodes with matching labels.

Q10. What happens if no node matches the Node Selector?

The Pod remains Pending because the scheduler cannot find an eligible node.

Q11. What is the difference between Node Selector and Node Affinity?

Node Selector supports simple exact matches. Node Affinity supports more flexible expressions and hard or soft scheduling preferences.

Q12. What is a Taint?

A restriction applied to a node to repel Pods that do not have a matching toleration.

Q13. What is a Toleration?

A Pod-level setting that allows a Pod to tolerate a matching node taint.

Q14. What are the three Taint effects?

`NoSchedule`, `PreferNoSchedule` and `NoExecute`.

Q15. What is the difference between NoSchedule and NoExecute?

NoSchedule prevents new non-tolerating Pods from scheduling but does not evict existing Pods. NoExecute also evicts existing Pods that do not tolerate the taint.

Q16. Does a toleration guarantee that a Pod will run on a tainted node?

No. It only allows the Pod to tolerate that taint. Other scheduling constraints still apply.

Q17. How do you dedicate a node to a particular application?

Apply a taint to the node, add a matching toleration to the application Pod, and use a Node Selector or Node Affinity to direct the application to that node.

## 11. Quick revision

| Topic                          | One-line memory                                         |
| ------------------------------ | ------------------------------------------------------- |
| ConfigMap                      | Stores non-sensitive configuration                      |
| Secret                         | Stores sensitive configuration                          |
| Base64                         | Encoding, not encryption                                |
| ConfigMap environment variable | Requires Pod restart to load updated value              |
| ConfigMap volume               | Files eventually update                                 |
| Node Selector                  | Selects nodes using labels                              |
| Node Affinity                  | Flexible node selection and preference                  |
| Taint                          | Repels Pods                                             |
| Toleration                     | Allows Pods to tolerate taints                          |
| NoSchedule                     | Blocks new non-tolerating Pods                          |
| PreferNoSchedule               | Soft restriction                                        |
| NoExecute                      | Blocks new Pods and evicts existing non-tolerating Pods |

## 12. Final mental model

## Kubernetes configuration and scheduling

Application container image

Same image across environments

ConfigMap

Non-sensitive settings

Secret

Passwords and credentials

Kubernetes Pod

Receives configuration and runs the application

Kubernetes Scheduler

Uses Node Selector / Node Affinity / Taints and Tolerations

Worker Node

Runs the scheduled Pod

## 13. Topics covered in this session

- Why ConfigMaps are needed.
- Creating ConfigMaps using literals, files, directories and YAML.
- Injecting ConfigMap values into Pods.
- Mounting ConfigMaps as volumes.
- Kubernetes Secrets and Secret types.
- Base64 encoding vs encryption.
- Secret security best practices.
- Node Selector and node labels.
- Taints and Tolerations.
- NoSchedule, PreferNoSchedule and NoExecute.
- Node Affinity and its hard/soft scheduling rules.
- Combining Node Selector, Taints and Tolerations.

Next session: Persistent Volumes (PV), Persistent Volume Claims (PVC) and Storage Classes. The instructor also reminded students to delete their AKS clusters when they are not using them to avoid unnecessary cloud charges.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub



Original transcript: [DevOps Session — 13 October 2026 (GitHub)](https://raw.githubusercontent.com/jeevanadv2000-pixel/python/main/DevOps%20Session%28Morning%29%20transcript_2026-10-13_09.27.32.txt?utm_source=chatgpt.com)