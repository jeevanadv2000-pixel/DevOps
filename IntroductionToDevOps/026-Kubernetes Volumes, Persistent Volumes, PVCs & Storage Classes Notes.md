# DevOps Session — Kubernetes Volumes, Persistent Volumes, PVCs & Storage Classes Notes

Date: 15 October 2026\
Instructor: Rochak Agrawal\
Focus: Kubernetes storage, `emptyDir`, `hostPath`, Persistent Volumes (PV), Persistent Volume Claims (PVC), Storage Classes, static and dynamic provisioning.

These notes are condensed from the class transcript. Participant discussions and repetitions have been removed, speech-to-text errors corrected, and technical oversimplifications clarified.

## 1. Why do we need volumes in Kubernetes?

Let's first understand a problem with containers.

When a container runs, it has a writable filesystem layer. Any data written only to that layer is tied to the container's lifecycle.

For example:

1. An application runs inside a container.
2. The application writes logs or files.
3. The container crashes and is restarted.
4. The previous container's writable layer is discarded.
5. Data stored only in that layer is lost.

Kubernetes adds another lifecycle consideration: Pods are replaceable. If a Pod is deleted and a new Pod is created, the new Pod does not automatically receive the old Pod's local filesystem data.

Kubernetes Volumes solve this problem by providing storage that can be mounted into containers.

Depending on the volume type, data may survive a container restart, a Pod replacement, or even a node failure.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

### Container storage vs Kubernetes volume

| Storage type             | What happens to data?                                                        |
| ------------------------ | ---------------------------------------------------------------------------- |
| Container writable layer | Lost when that container is replaced                                         |
| `emptyDir`               | Survives container restarts, but deleted with the Pod                        |
| `hostPath`               | Stored on the node's filesystem                                              |
| Persistent Volume        | Independent of the Pod lifecycle; persistence depends on the storage backend |

## 2. Types of Kubernetes volumes

The session introduces these storage concepts:

1. `emptyDir`
2. `hostPath`
3. ConfigMap and Secret volumes
4. Persistent Volume (PV)
5. Persistent Volume Claim (PVC)
6. Storage Class

The first two are useful for understanding Pod-level and node-level storage. PV, PVC and Storage Classes are the main concepts for persistent application storage.

## 3. emptyDir Volume

An `emptyDir` volume is temporary storage created when a Pod is assigned to a node.

It exists for the entire lifetime of the Pod.

Survives container restarts

Deleted with the Pod

### 3.1 How does emptyDir work?

Suppose you have one Pod containing two containers:

- Container 1 writes application logs.
- Container 2 reads those logs and ships them to a logging system.

Both containers need access to the same files.

You can create an `emptyDir` volume and mount it into both containers.

## Shared emptyDir volume

One Kubernetes Pod

Writer container

Writes log.txt

Reader container

Reads log.txt

Shared emptyDir volume

Both containers mount the same volume

The writer creates a file in the shared volume. The reader can immediately access the same file through its own mount.

Important: Volumes are defined at the Pod level, but mounted into individual containers using `volumeMounts`.

### 3.2 emptyDir YAML example

```
apiVersion: v1
kind: Pod
metadata:
  name: shared-volume-pod
spec:
  containers:
    - name: writer
      image: busybox
      command:
        - sh
        - -c
        - |
          while true; do
            date >> /shared/log.txt
            sleep 5
          done
      volumeMounts:
        - name: shared-data
          mountPath: /shared

    - name: reader
      image: busybox
      command:
        - sh
        - -c
        - |
          while true; do
            cat /shared/log.txt
            sleep 5
          done
      volumeMounts:
        - name: shared-data
          mountPath: /shared

  volumes:
    - name: shared-data
      emptyDir: {}
```

Let's understand the important parts:

| Field          | Meaning                                  |
| -------------- | ---------------------------------------- |
| `volumes`      | Defines the volume at Pod level          |
| `name`         | Name of the volume                       |
| `emptyDir: {}` | Creates an empty temporary directory     |
| `volumeMounts` | Mounts the volume inside a container     |
| `mountPath`    | Directory where the volume is accessible |

Both containers use the same volume name, `shared-data`, and mount it at `/shared`.

### 3.3 Deploy and verify

```
kubectl apply -f shared-volume.yaml

kubectl get pods
```

View the reader container's logs:

```
kubectl logs shared-volume-pod -c reader
```

Here, `-c` specifies which container's logs you want to see.

### 3.4 What happens when a container restarts?

Suppose the writer container crashes and Kubernetes restarts it.

The `emptyDir` volume remains because the Pod itself still exists. The writer can continue writing to the same directory after restarting.

However, if the entire Pod is deleted and recreated, a new `emptyDir` is created and the old data is lost.

| Event                                           | Is emptyDir data retained? |
| ----------------------------------------------- | -------------------------- |
| Container crashes and restarts                  | Yes                        |
| One container in a multi-container Pod restarts | Yes                        |
| Pod is deleted                                  | No                         |
| Pod is replaced by a new Pod                    | No                         |

This distinction is extremely important.

### 3.5 Common use cases

- Sharing files between containers in the same Pod.
- Sidecar log collection.
- Temporary file processing.
- Caching downloaded assets.
- Sharing files created by an init container with the main application container.

Example: An init container downloads required files into an `emptyDir`. The main application container later reads those files.

### 3.6 emptyDir size limit

You can optionally limit the storage consumed by an `emptyDir` volume.

Example:

```
volumes:
  - name: shared-data
    emptyDir:
      sizeLimit: 128Mi
```

This sets a size limit of 128 MiB.

You can also use memory-backed storage:

```
volumes:
  - name: shared-data
    emptyDir:
      medium: Memory
      sizeLimit: 128Mi
```

With `medium: Memory`, the volume is backed by tmpfs (memory), so its usage counts against the memory limit of the container that writes to it.

Remember: `emptyDir` is temporary Pod-scoped storage, not persistent application storage.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



## 4. hostPath Volume

Now let's understand another volume type.

A `hostPath` volume mounts a file or directory from the worker node's filesystem directly into a Pod.

For example:

- Node directory: `/data`
- Container directory: `/demo`

When the container writes a file inside `/demo`, that file is written to `/data` on the worker node.

## hostPath volume

Worker node filesystem

/data/test.txt

Pod container

/demo/test.txt

Both paths refer to the same underlying files through the mount.

### 4.1 hostPath YAML example

```
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-pod
spec:
  containers:
    - name: app
      image: busybox
      command:
        - sh
        - -c
        - "while true; do sleep 3600; done"
      volumeMounts:
        - name: host-volume
          mountPath: /demo

  volumes:
    - name: host-volume
      hostPath:
        path: /data
        type: DirectoryOrCreate
```

Here:

- `hostPath.path` is the path on the worker node.
- `mountPath` is the path inside the container.
- `DirectoryOrCreate` creates the directory on the node if it does not already exist.

### 4.2 hostPath types

| Type                | Meaning                                |
| ------------------- | -------------------------------------- |
| `Directory`         | Directory must already exist           |
| `DirectoryOrCreate` | Creates directory if it does not exist |
| `File`              | File must already exist                |
| `FileOrCreate`      | Creates file if it does not exist      |

### 4.3 What happens when a Pod is recreated?

Suppose your cluster has two nodes:

- Node 1
- Node 2

Your Pod runs on Node 1 and writes data to `/data`.

Later, the Pod is deleted and a replacement is scheduled on Node 2.

The new Pod mounts `/data` from Node 2, not Node 1.

Therefore, it may not find the data written on Node 1.

This is the major limitation of hostPath: storage is tied to a particular node.

### 4.4 Risks of hostPath

1\. Portability problem

A Pod can be rescheduled to another node. Data stored on the previous node is not automatically available on the new node.

2\. Security risk

A Pod with access to the node's filesystem may access sensitive host files if the mount is too permissive.

This can expose:

- Credentials
- Certificates
- Host configuration
- Other application data

### 4.5 When is hostPath useful?

Although it is generally avoided for ordinary application storage in production, there are specific use cases:

- Single-node development clusters.
- Logging agents that need to read host log files.
- Monitoring agents that need access to host information.
- Certain DaemonSet workloads.

For example, a node-level monitoring agent may need access to host filesystem or system information.

Important: A DaemonSet ensures a Pod runs on eligible nodes, but it does not automatically require `hostPath`. The volume is used when the agent specifically needs access to host files.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

### 4.6 emptyDir vs hostPath

| Feature                    | emptyDir                               | hostPath                                   |
| -------------------------- | -------------------------------------- | ------------------------------------------ |
| Storage location           | Node-local temporary storage or memory | Node's filesystem                          |
| Survives container restart | Yes                                    | Yes                                        |
| Survives Pod deletion      | No                                     | Usually, because data remains on node      |
| Shared between containers  | Yes                                    | Yes, when mounted into multiple containers |
| Available across nodes     | No                                     | No, not by itself                          |
| Main use                   | Temporary shared storage               | Host filesystem access                     |

Key difference:

- `emptyDir` belongs to the Pod lifecycle.
- `hostPath` refers to a location on the worker node.

## 5. Why do we need Persistent Volumes?

We have seen that:

- `emptyDir` data disappears when the Pod is deleted.
- `hostPath` data is tied to a particular worker node.

But production applications often need storage that survives Pod deletion and rescheduling.

Examples:

- Database data.
- Application uploads.
- Persistent logs.
- User-generated files.
- Application state.

For these use cases, Kubernetes provides Persistent Volumes.

A Persistent Volume (PV) represents a storage resource available to the cluster. It exists independently of the Pod lifecycle.

A Pod can be deleted and recreated, and a replacement Pod can reconnect to the same persistent storage, provided the storage backend and access mode support the operation.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



## 6. Understanding PV, PVC and Storage Class

There are three important concepts:

1. Persistent Volume (PV)
2. Persistent Volume Claim (PVC)
3. Storage Class (SC)

Let's understand them one by one.

### 6.1 Persistent Volume (PV)

A Persistent Volume is a storage resource provisioned in the Kubernetes cluster.

Think of it as a hard disk that Kubernetes can make available to applications.

The underlying storage can be:

- Cloud disks.
- Network file systems.
- Local storage.
- Other supported storage systems.

For example:

- A 5 GB disk.
- A 10 GB disk.
- A 50 GB disk.

The PV represents the storage resource, not the application that uses it.

### 6.2 Persistent Volume Claim (PVC)

A Persistent Volume Claim is a request for storage.

The application or user specifies what storage is required.

For example:

```
I need 5 GB of storage.
I need ReadWriteOnce access.
I want to use a particular Storage Class.
```

Kubernetes looks for a matching PV or provisions storage dynamically through a Storage Class.

When a suitable PV is found, the PVC and PV become bound to each other.

### 6.3 Storage Class

A Storage Class defines how storage should be dynamically provisioned.

For example, in Azure AKS, a Storage Class can specify an Azure Disk CSI provisioner and parameters such as disk type.

In AWS EKS, an EBS CSI provisioner can be used for supported block storage.

Storage Classes help avoid manually creating every PV before an application requests storage.

### 6.4 How do PV, PVC and Pod work together?

## PV and PVC relationship

Persistent Volume (PV)

Actual storage resource — for example, 10 GB disk

Kubernetes binds matching resources

Persistent Volume Claim (PVC)

Application's storage request — for example, 10 GB

Pod / Application

Uses the PVC to mount storage inside a container

The application does not normally refer directly to the PV. It uses a PVC.

The PVC acts as the connection between the application and the storage resource.

## 7. Namespace scope vs Cluster scope

This is an important Kubernetes concept covered during the session.

Most application resources are namespace-scoped.

Examples:

- Pods
- Deployments
- Services
- PVCs
- ConfigMaps
- Secrets

However, Persistent Volumes and Storage Classes are cluster-scoped resources.

| Resource      | Scope     |
| ------------- | --------- |
| Pod           | Namespace |
| Deployment    | Namespace |
| Service       | Namespace |
| ConfigMap     | Namespace |
| Secret        | Namespace |
| PVC           | Namespace |
| PV            | Cluster   |
| Storage Class | Cluster   |
| Node          | Cluster   |

Why is PV cluster-scoped?

Because a PV is a storage resource that can be claimed by a PVC in a namespace. It is not owned by one particular application namespace.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 8. PV and PVC binding

Let's understand how Kubernetes connects a PV to a PVC.

Suppose you have three Persistent Volumes:

| PV   | Capacity | Access mode   |
| ---- | -------- | ------------- |
| pv-1 | 5 GB     | ReadWriteOnce |
| pv-2 | 10 GB    | ReadWriteOnce |
| pv-3 | 20 GB    | ReadWriteOnce |

Your application requests 10 GB through a PVC.

Kubernetes looks for an available PV satisfying the claim's requirements.

If `pv-2` matches, Kubernetes binds the PVC to `pv-2`.

### 8.1 PV lifecycle states

| State     | Meaning                                                                 |
| --------- | ----------------------------------------------------------------------- |
| Available | PV is available to be claimed                                           |
| Bound     | PV is bound to a PVC                                                    |
| Released  | PVC has been deleted, but the PV is not yet available for another claim |
| Failed    | Automatic reclamation or another storage-related operation failed       |

### 8.2 PVC lifecycle states

| State   | Meaning                                                                 |
| ------- | ----------------------------------------------------------------------- |
| Pending | No suitable PV has been bound yet, or provisioning is still in progress |
| Bound   | PVC is successfully bound to a PV                                       |
| Lost    | The bound PV is no longer available                                     |

### 8.3 One-to-one binding relationship

A PV can be bound to only one PVC at a time.

A PVC can be bound to only one PV.

This is a one-to-one binding relationship.

However, be careful about one important distinction:

One-to-one PV/PVC binding does not mean only one Pod can use the PVC.

Multiple Pods can use the same PVC if the storage backend and access mode support the required mounting pattern.

For example, a PVC with `ReadWriteMany` can be used by multiple Pods across nodes when the storage backend supports it.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

### 8.4 What happens when a Pod is deleted?

Suppose:

- PV is bound to PVC.
- Pod uses that PVC.
- Pod is deleted.

The PV and PVC are not automatically deleted just because the Pod is deleted.

A replacement Pod can use the same PVC and reconnect to the storage, provided the storage can be mounted in its new location.

This is the main advantage of persistent storage.

## 9. Static Provisioning

In static provisioning, the administrator manually creates the PV before the application requests storage.

Example scenario:

1. Administrator creates a 5 GB PV.
2. Application requests 5 GB through a PVC.
3. Kubernetes finds the matching PV.
4. PVC becomes Bound.
5. Application mounts the PVC.

### 9.1 Creating a PV using YAML

Create `pv.yaml`:

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: demo-pv
spec:
  capacity:
    storage: 5Gi

  accessModes:
    - ReadWriteOnce

  persistentVolumeReclaimPolicy: Retain

  storageClassName: manual

  hostPath:
    path: /mnt/data
    type: DirectoryOrCreate
```

Apply:

```
kubectl apply -f pv.yaml
```

Verify:

```
kubectl get pv
```

Expected initial status:

```
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS
demo-pv   5Gi        RWO            Retain           Available
```

Important production clarification: This example uses `hostPath` only to demonstrate PV/PVC binding in a simple learning environment. It is not an appropriate general-purpose production storage design for a multi-node cluster.

### 9.2 Creating a PVC

Create `pvc.yaml`:

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-pvc
spec:
  accessModes:
    - ReadWriteOnce

  storageClassName: manual

  resources:
    requests:
      storage: 5Gi
```

Apply:

```
kubectl apply -f pvc.yaml
```

Check:

```
kubectl get pvc
kubectl get pv
```

The PVC should become `Bound` if a suitable PV is available.

### 9.3 What if the requested storage does not match?

Suppose:

- PV capacity: 5 GB.
- PVC request: 15 GB.

The PV cannot satisfy the request.

The PVC remains Pending until a suitable PV becomes available or dynamic provisioning succeeds.

```
kubectl describe pvc demo-pvc
```

This helps identify why the claim has not been bound.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 10. Using a PVC inside a Pod

After creating and binding the PVC, the application can use it.

The Pod specification needs two things:

1. `volumes`: Defines the PVC-backed volume.
2. `volumeMounts`: Defines where that volume should be mounted inside the container.

Example:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pv-pod
spec:
  containers:
    - name: nginx
      image: nginx

      volumeMounts:
        - name: app-storage
          mountPath: /usr/share/nginx/html

  volumes:
    - name: app-storage
      persistentVolumeClaim:
        claimName: demo-pvc
```

Understand the relationship:

| Field                   | Meaning                                    |
| ----------------------- | ------------------------------------------ |
| `volumes`               | Defines the Pod's volume                   |
| `name`                  | Name of the volume                         |
| `persistentVolumeClaim` | Indicates that the volume comes from a PVC |
| `claimName`             | Name of the PVC                            |
| `volumeMounts`          | Mounts the volume into a container         |
| `mountPath`             | Directory inside the container             |

Apply:

```
kubectl apply -f nginx-pv-pod.yaml
```

Verify:

```
kubectl get pods
kubectl describe pod nginx-pv-pod
```

### Testing persistence

Enter the Pod:

```
kubectl exec -it nginx-pv-pod -- bash
```

Create a file:

```
echo "Persistent Kubernetes storage" > /usr/share/nginx/html/test.txt
```

Exit:

```
exit
```

Delete the Pod:

```
kubectl delete pod nginx-pv-pod
```

Recreate it:

```
kubectl apply -f nginx-pv-pod.yaml
```

Check the file again:

```
kubectl exec -it nginx-pv-pod -- cat /usr/share/nginx/html/test.txt
```

If the PV backend retains the data and the new Pod successfully mounts the same PVC, the file should still exist.

That is the main benefit of PV and PVC.

## 11. PV Access Modes

Access modes define how storage can be mounted by nodes or workloads.

There are three traditional access modes covered in the session, plus an important additional mode.

| Access mode | Full form        | Meaning                              |
| ----------- | ---------------- | ------------------------------------ |
| RWO         | ReadWriteOnce    | Read-write mounting by a single node |
| ROX         | ReadOnlyMany     | Read-only mounting by many nodes     |
| RWX         | ReadWriteMany    | Read-write mounting by many nodes    |
| RWOP        | ReadWriteOncePod | Read-write mounting by a single Pod  |

### 11.1 ReadWriteOnce (RWO)

The volume can be mounted read-write by one node at a time.

Important: This does not necessarily mean only one Pod can use it. Multiple Pods on the same node may be able to use an RWO volume, depending on the driver and workload configuration.

### 11.2 ReadOnlyMany (ROX)

The volume can be mounted read-only by multiple nodes simultaneously.

Use case:

\- Shared static configuration or reference data that multiple applications need to read.

### 11.3 ReadWriteMany (RWX)

The volume can be mounted read-write by multiple nodes simultaneously.

Use case:

- Shared application uploads.
- Shared files.
- Multiple application replicas accessing common storage.

The storage backend must support this mode. Not every cloud disk supports RWX.

### 11.4 ReadWriteOncePod (RWOP)

The volume can be mounted read-write by only one Pod cluster-wide.

This is useful when a workload needs exclusive access at the Pod level.

Interview clarification

RWO means single-node mounting, not necessarily single-Pod mounting. RWOP is the access mode specifically intended for single-Pod read-write access.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 12. Reclaim Policies

A reclaim policy determines what happens to the underlying storage resource after the PVC is deleted.

The two main policies are:

1. Retain
2. Delete

### 12.1 Retain

When a PVC is deleted:

- The PV moves to Released.
- The underlying data is preserved.
- The PV is not automatically made available for another claim.
- An administrator must manually reclaim or prepare it for reuse.

This is useful when you want to protect important data.

### 12.2 Delete

When the PVC is deleted:

- Kubernetes removes the PV.
- The associated storage asset may also be deleted by the provisioner.

This behavior depends on the storage plugin and backend.

### Comparison

| Policy | PVC deleted                            | Storage data    |
| ------ | -------------------------------------- | --------------- |
| Retain | PV becomes Released                    | Preserved       |
| Delete | PV and provisioned storage are deleted | Usually deleted |

Important: The reclaim policy is triggered by deletion of the PVC, not simply by deleting a Pod.

Also, the older `Recycle` reclaim policy is deprecated and should not be treated as a modern general-purpose option.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 13. The problem with Static Provisioning

Let's understand why Storage Classes are useful.

Suppose you have a manually created PV with 100 GB of capacity.

Your application creates a PVC requesting only 2 GB.

Kubernetes binds the entire PV to that PVC.

The remaining 98 GB cannot be claimed by another PVC through that same PV.

Why?

Because PV and PVC have a one-to-one binding relationship.

This can result in inefficient storage allocation.

Another challenge is that administrators must manually create and manage PVs before applications request them.

This is where dynamic provisioning becomes useful.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 14. Storage Classes

A Storage Class provides a way to dynamically provision storage.

Think of it as a storage template.

It defines:

- Which storage provisioner should be used.
- Which type of storage should be created.
- Storage parameters.
- Reclaim policy.
- Volume expansion settings.
- Volume binding behavior.

For example:

In Azure AKS:

- Azure Disk CSI driver can provision block storage.
- Azure Files CSI driver can provision shared file storage.

In AWS EKS:

- EBS CSI driver can provision EBS volumes.
- EFS CSI driver can provision shared file storage.

### 14.1 Static vs Dynamic Provisioning

## Static provisioning

Administrator creates PV manually

Application creates PVC

Kubernetes binds matching PV and PVC

## Dynamic provisioning

Administrator defines Storage Class

Application creates PVC referencing Storage Class

Provisioner creates storage and Kubernetes binds the PVC

### 14.2 Creating a Storage Class

The instructor demonstrated Azure AKS.

Here is a simplified Azure Disk CSI example:

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-premium-custom

provisioner: disk.csi.azure.com

parameters:
  skuName: Premium_LRS

reclaimPolicy: Delete

allowVolumeExpansion: true

volumeBindingMode: WaitForFirstConsumer
```

Apply:

```
kubectl apply -f storageclass.yaml
```

Verify:

```
kubectl get storageclass
```

Short form:

```
kubectl get sc
```

Important: This example depends on the Azure Disk CSI driver being installed and configured. Storage provisioner names and parameters differ across cloud providers.

### 14.3 Creating a PVC using a Storage Class

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
    - ReadWriteOnce

  storageClassName: managed-premium-custom

  resources:
    requests:
      storage: 5Gi
```

Apply:

```
kubectl apply -f dynamic-pvc.yaml
```

What happens?

1. PVC is created.
2. Kubernetes identifies the referenced Storage Class.
3. The CSI provisioner is asked to create the storage.
4. The cloud provider provisions the disk.
5. Kubernetes creates or registers the corresponding PV.
6. The PVC becomes Bound once provisioning and binding complete.

The exact timing depends on the provisioner's configuration and binding mode.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

### 14.4 What is WaitForFirstConsumer?

The instructor's example uses `WaitForFirstConsumer`.

This is an important Storage Class setting.

Two common values are:

| Value                  | Meaning                                                                                                      |
| ---------------------- | ------------------------------------------------------------------------------------------------------------ |
| `Immediate`            | Provisioning/binding starts as soon as the PVC is created                                                    |
| `WaitForFirstConsumer` | Provisioning/binding waits until a Pod using the PVC is created and scheduling constraints can be considered |

Why is this useful?

Suppose a cloud disk is available only in a particular availability zone.

If Kubernetes creates the disk before knowing where the Pod will run, the disk and Pod could end up in incompatible zones.

`WaitForFirstConsumer` allows scheduling constraints to be considered before provisioning the volume.

### 14.5 Dynamic provisioning and PVC Pending

With `WaitForFirstConsumer`, you may see a PVC remain Pending after creation.

That does not necessarily indicate an error.

The PVC may be waiting for a Pod to consume it so Kubernetes can make a suitable scheduling and provisioning decision.

Check:

```
kubectl get pvc
kubectl describe pvc dynamic-pvc
```

Then create a Pod or Deployment that uses the PVC.

Check again:

```
kubectl get pvc
kubectl get pv
kubectl get pods -o wide
```

## 15. Volume Expansion

What happens if your application initially needs 5 GB but later requires 20 GB?

A Storage Class can allow volume expansion.

Example:

```
allowVolumeExpansion: true
```

If the storage backend supports expansion, you can request a larger PVC capacity.

For example:

```
kubectl edit pvc dynamic-pvc
```

Change:

```
resources:
  requests:
    storage: 20Gi
```

Whether the filesystem also needs to be expanded depends on the volume driver and filesystem.

Important: Increasing a PVC request does not guarantee that every storage backend supports online expansion. Check the CSI driver and Storage Class capabilities.

## 16. Useful kubectl commands

| Command                                      | Purpose                          |
| -------------------------------------------- | -------------------------------- |
| `kubectl get pv`                             | List Persistent Volumes          |
| `kubectl get pvc`                            | List Persistent Volume Claims    |
| `kubectl describe pv demo-pv`                | Inspect a PV                     |
| `kubectl describe pvc demo-pvc`              | Inspect a PVC                    |
| `kubectl get sc`                             | List Storage Classes             |
| `kubectl describe sc managed-premium-custom` | Inspect a Storage Class          |
| `kubectl get pv -o wide`                     | View additional PV information   |
| `kubectl get pvc -A`                         | List PVCs across namespaces      |
| `kubectl get pods -o wide`                   | See Pod placement                |
| `kubectl describe pod nginx-pv-pod`          | Troubleshoot Pod volume mounting |
| `kubectl logs pod-name`                      | View application logs            |
| `kubectl exec -it pod-name -- sh`            | Enter a container                |

## 17. Troubleshooting scenarios

| Scenario                                                  | Possible reason                                                        |
| --------------------------------------------------------- | ---------------------------------------------------------------------- |
| PVC remains Pending                                       | No matching PV or dynamic provisioning is still in progress            |
| PVC requests 10 GB but PV has 5 GB                        | PV capacity is insufficient                                            |
| PV remains Released after PVC deletion                    | Reclaim policy is Retain                                               |
| Pod cannot mount a volume                                 | Storage driver, access mode, permissions or attachment issue           |
| Pod moves to another node and cannot access hostPath data | Data exists on the previous node                                       |
| emptyDir data disappears                                  | Entire Pod was deleted and replaced                                    |
| PVC stays Pending with WaitForFirstConsumer               | No consuming Pod has been scheduled yet                                |
| PVC expansion fails                                       | Storage backend or driver does not support the requested expansion     |
| Multiple Pods cannot share a disk                         | Access mode or backend does not support the requested mounting pattern |

Useful commands:

```
kubectl get pv
kubectl get pvc
kubectl get sc

kubectl describe pvc <pvc-name>
kubectl describe pv <pv-name>

kubectl describe pod <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp
```

## 18. Interview questions and answers

Q1. Why do we need Kubernetes volumes?

Container writable storage is tied to the container lifecycle. Volumes provide storage that can survive certain container or Pod lifecycle events, depending on the volume type.

Q2. What is emptyDir?

A temporary volume created when a Pod is assigned to a node. It survives container restarts but is deleted when the Pod is removed.

Q3. Can two containers in the same Pod share an emptyDir volume?

Yes. Define the volume once at Pod level and mount it into both containers.

Q4. What is the difference between emptyDir and hostPath?

emptyDir is temporary Pod-scoped storage. hostPath mounts a file or directory from the worker node's filesystem.

Q5. What is the disadvantage of hostPath?

It is node-specific and creates portability and security concerns, especially in multi-node and multi-tenant clusters.

Q6. What is a Persistent Volume?

A cluster-scoped storage resource that exists independently of the Pod lifecycle.

Q7. What is a Persistent Volume Claim?

A namespace-scoped request for storage that Kubernetes binds to a matching PV or uses to request dynamically provisioned storage.

Q8. What is a Storage Class?

A cluster-scoped resource that defines how storage should be dynamically provisioned.

Q9. What is the difference between PV and PVC?

PV represents the storage resource. PVC represents an application's request for storage.

Q10. Can a Pod directly use a PV?

Normally, a Pod references a PVC, and the PVC is bound to the PV.

Q11. What happens when a Pod using a PVC is deleted?

The Pod is deleted, but the PVC and PV normally remain. A replacement Pod can use the same PVC.

Q12. What is the relationship between PV and PVC?

A PV is bound to one PVC at a time, and a PVC is bound to one PV.

Q13. Can multiple Pods use the same PVC?

Yes, depending on the access mode, storage driver and backend capabilities.

Q14. What is the difference between RWO and RWOP?

RWO permits read-write mounting from one node. RWOP restricts read-write mounting to one Pod.

Q15. What is a reclaim policy?

It determines what happens to the PV and underlying storage when the PVC is deleted.

Q16. What is the difference between Retain and Delete?

Retain preserves the storage for manual recovery or reuse. Delete allows the provisioner to delete the PV and underlying storage.

Q17. What is static provisioning?

An administrator creates PVs manually before applications request them.

Q18. What is dynamic provisioning?

Kubernetes provisions storage automatically through a Storage Class when an application creates a matching PVC.

Q19. Why do we use Storage Classes?

To define storage provisioning rules and avoid manually creating every PV.

Q20. What is WaitForFirstConsumer?

A volume binding mode that delays provisioning/binding until a Pod using the PVC is scheduled, allowing scheduling constraints to be considered.

## 19. Quick revision

| Concept              | Remember this                                                 |
| -------------------- | ------------------------------------------------------------- |
| Volume               | Storage mounted into containers                               |
| emptyDir             | Temporary storage tied to Pod lifecycle                       |
| hostPath             | Mounts node filesystem into Pod                               |
| PV                   | Cluster-level storage resource                                |
| PVC                  | Application's storage request                                 |
| Storage Class        | Defines dynamic provisioning                                  |
| Available            | PV is not yet claimed                                         |
| Bound                | PV is connected to a PVC                                      |
| Released             | PVC deleted, PV not automatically reusable                    |
| RWO                  | Read-write from one node                                      |
| ROX                  | Read-only from multiple nodes                                 |
| RWX                  | Read-write from multiple nodes                                |
| RWOP                 | Read-write from one Pod                                       |
| Retain               | Preserve storage after PVC deletion                           |
| Delete               | Delete provisioned storage when reclaimed                     |
| Static provisioning  | PV created manually                                           |
| Dynamic provisioning | Storage created through Storage Class                         |
| WaitForFirstConsumer | Wait for consuming Pod scheduling before binding/provisioning |

## 20. Final mental model

The most important thing to understand from this session is how Kubernetes separates application workloads from storage.

## Kubernetes storage architecture

Application / Pod

Needs persistent storage

Persistent Volume Claim (PVC)

Requests storage capacity and access mode

Persistent Volume (PV)

Represents the actual storage resource

Storage backend

Azure Disk / AWS EBS / NFS / other supported storage

Where does Storage Class fit?

Storage Class

Defines how the storage backend should provision storage dynamically for a PVC

Think of it like this:

- PV = Actual hard disk.
- PVC = Request for a hard disk.
- Storage Class = Instructions for creating the hard disk.
- Pod = Application that uses the hard disk.

## 21. Topics covered in this session

- Why Kubernetes needs volumes.
- `emptyDir` and its lifecycle.
- Sharing storage between containers.
- `hostPath` and its limitations.
- Persistent Volumes.
- Persistent Volume Claims.
- Namespace-scoped vs cluster-scoped resources.
- PV and PVC binding.
- PV and PVC lifecycle states.
- PV access modes.
- Reclaim policies.
- Static provisioning.
- Storage Classes and dynamic provisioning.
- Azure Disk CSI example.
- Volume expansion and WaitForFirstConsumer.

Next session: Horizontal Pod Autoscaler (HPA), CPU and memory resource requests and limits, and controlling application resource usage.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub



Original transcript: [DevOps Session — 15 October 2026 (GitHub)](https://raw.githubusercontent.com/jeevanadv2000-pixel/python/main/DevOps%20Session%28Morning%29%20transcript_2026-10-15_09.20.38.txt?utm_source=chatgpt.com)