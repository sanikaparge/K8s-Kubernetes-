# K8s-Kubernetes-
Detailed Kubernetes notes covering architecture, Pods, Deployments, Services, networking, storage, autoscaling, security, troubleshooting, and essential kubectl commands with practical examples.

# Kubernetes Complete Notes
### Fundamentals • Architecture • Workloads • Networking • Storage • Troubleshooting

---

# 1. Introduction to Kubernetes

## What is Kubernetes?

Kubernetes (K8s) is an open-source container orchestration platform used to deploy, manage, scale, and monitor containerized applications.

It automates tasks such as:
- Deploying applications across multiple nodes.
- Restarting failed containers through workload controllers.
- Scaling applications based on demand.
- Distributing traffic across application replicas.
- Managing application configuration and storage.
- Performing rolling updates and rollbacks.

**Example:** If an application needs four running replicas, Kubernetes can maintain the desired number and create replacement Pods when managed Pods fail.

## Why do we need Kubernetes?

Docker can run containers, but managing hundreds of containers across multiple servers manually becomes difficult.

Kubernetes provides:
- Automated scheduling.
- Self-healing through workload controllers and health checks.
- Service discovery and load distribution.
- Horizontal scaling.
- Rolling updates and rollbacks.
- Configuration and storage management.

## What is container orchestration?

Container orchestration is the automated management of containers across a group of machines.

For example, Kubernetes can schedule application Pods across worker nodes, maintain their desired replica count, and expose them through Services.

## Kubernetes vs Docker

| Docker | Kubernetes |
|---|---|
| Builds and runs containers. | Orchestrates containerized workloads. |
| Uses Docker Engine to run containers. | Uses a container runtime on cluster nodes. |
| Docker Compose manages multi-container applications on a host. | Kubernetes manages workloads across a cluster. |
| Supports container networking and volumes. | Provides cluster networking, Services, and storage orchestration. |
| Does not provide Kubernetes-style cluster scheduling by itself. | Schedules Pods and manages workload availability. |

**Important:** Kubernetes does not require Docker Engine as its container runtime. Common runtimes include containerd and CRI-O.

# 2. Kubernetes Architecture

A Kubernetes cluster consists of a control plane and worker nodes.

## 2.1 Control Plane

The control plane manages the overall state of the cluster.

### API Server (`kube-apiserver`)

- Acts as the entry point for Kubernetes API requests.
- Validates and processes requests from `kubectl`, controllers, and other clients.
- Exposes the Kubernetes API.

Example:

```bash
kubectl get pods
```

`kubectl` communicates with the API server to retrieve Pod information.

### etcd

- A distributed key-value store.
- Stores Kubernetes cluster state and configuration data.
- Requires appropriate backup and access-control practices.

### Scheduler (`kube-scheduler`)

- Selects a suitable worker node for newly created, unscheduled Pods.
- Considers resource requests, scheduling constraints, affinity, taints, and tolerations.

The scheduler selects a node; it does not directly run the container.

### Controller Manager (`kube-controller-manager`)

Runs controllers that continuously compare actual cluster state with desired state.

Examples:
- Deployment and ReplicaSet-related reconciliation.
- Node lifecycle management.
- Job management.

### Cloud Controller Manager

Integrates Kubernetes with supported cloud-provider functionality, such as cloud load balancers, routes, and node information.

Its presence depends on the cluster environment.

## 2.2 Worker Node Components

### Kubelet

- Runs on each node and communicates with the API server.
- Ensures assigned Pods are running according to their specifications.
- Reports node and Pod status.

### Container Runtime

Runs containers and manages their lifecycle.

Examples:
- containerd
- CRI-O

### Kube-proxy

Implements Service networking rules on nodes in many Kubernetes installations.

Some cluster networking implementations replace or omit kube-proxy by implementing equivalent Service functionality themselves.

## 2.3 Architecture Flow

1. A user submits a manifest using `kubectl apply`.
2. The API server validates and processes the request.
3. The desired state is stored in the cluster's backing datastore.
4. Controllers reconcile the requested state.
5. The scheduler selects a node for an unscheduled Pod.
6. The kubelet on that node asks the container runtime to run the containers.
7. Kubernetes reports status through the API.

# 3. Kubernetes Objects

Kubernetes objects represent the desired state of applications and cluster resources.

Common objects include:

- Pod
- Deployment
- ReplicaSet
- Service
- Namespace
- ConfigMap
- Secret
- StatefulSet
- DaemonSet
- PersistentVolume
- PersistentVolumeClaim
- Ingress
- Job and CronJob
- HorizontalPodAutoscaler

A typical YAML object contains:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx:stable
          ports:
            - containerPort: 80
```

### Important fields

- `apiVersion`: API group and version.
- `kind`: Resource type.
- `metadata`: Name, namespace, labels, and annotations.
- `spec`: Desired configuration.
- `status`: Observed state maintained by Kubernetes.

# 4. Pods

## What is a Pod?

A Pod is the smallest deployable unit in Kubernetes. It can contain one or more containers that share networking and may share storage volumes.

Containers in the same Pod share the Pod IP address and port space, so they must avoid conflicting port bindings.

## Why are Pods used?

- To run containerized application processes.
- To group tightly coupled containers.
- To provide a shared network identity.
- To attach shared volumes to related containers.

## Create a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx:stable
      ports:
        - containerPort: 80
```

Commands:

```bash
kubectl apply -f pod.yaml
kubectl get pods
kubectl describe pod nginx-pod
kubectl logs nginx-pod
kubectl delete pod nginx-pod
```

## Pod lifecycle

Common Pod phases:

- `Pending`: Pod has been accepted but is not yet fully running; it may be waiting for scheduling, image downloads, or setup.
- `Running`: At least one container is running or starting/restarting.
- `Succeeded`: All containers completed successfully.
- `Failed`: All containers have terminated and at least one failed.
- `Unknown`: Kubernetes cannot determine the Pod's state.

`CrashLoopBackOff` is a container waiting reason, not a Pod phase. It indicates repeated container failures with restart backoff.

## Multi-container Pods

Containers may share a Pod when they are tightly coupled, such as:
- An application container and a sidecar proxy.
- An application and a logging helper.
- An application and a configuration-reloading helper.

Containers that need independent scaling and deployment should generally use separate Pods.

# 5. Deployments and ReplicaSets

## What is a Deployment?

A Deployment manages stateless application releases using ReplicaSets and Pods.

Features:
- Desired replica management.
- Rolling updates.
- Rollbacks.
- Declarative application updates.
- Replacement of failed or deleted managed Pods.

## Deployment example

```yaml
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
          image: nginx:stable
          ports:
            - containerPort: 80
```

Commands:

```bash
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl get replicasets
kubectl get pods -o wide
kubectl scale deployment nginx-deployment --replicas=5
kubectl rollout status deployment/nginx-deployment
kubectl rollout history deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment
```

## What is a ReplicaSet?

A ReplicaSet maintains a specified number of matching Pod replicas.

If one managed Pod disappears, the ReplicaSet attempts to create a replacement.

In normal application management, create a Deployment instead of managing ReplicaSets directly.

## Deployment vs ReplicaSet

| Deployment | ReplicaSet |
|---|---|
| Manages application releases. | Maintains a desired number of Pods. |
| Supports rollout history and rollback workflows. | Does not independently provide Deployment-style rollout management. |
| Creates and manages ReplicaSets. | Manages matching Pods. |
| Recommended for most stateless workloads. | Usually managed indirectly through a Deployment. |

# 6. Kubernetes Services and Networking

## What is a Service?

A Service provides a stable network endpoint for a group of Pods selected by labels.

Pods can be replaced and their IP addresses can change. A Service provides a consistent way to reach eligible backend Pods.

## ClusterIP

The default Service type.

- Exposes the application inside the cluster.
- Useful for internal APIs, databases, and service-to-service communication.
- Is not ordinarily directly accessible from outside the cluster.

Example:

```yaml
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

## NodePort

Exposes a Service through a port on eligible cluster nodes.

- The default allocated range is typically `30000–32767`.
- Can make an application reachable through a node IP and allocated NodePort when network rules allow it.
- Often used for learning and specific networking setups.

## LoadBalancer

Requests an external load balancer through an appropriate cloud provider or other supported implementation.

- Commonly used for external application access in cloud environments.
- Availability and behavior depend on cluster infrastructure.
- May incur cloud-provider charges.

## Service comparison

| Type | Typical access |
|---|---|
| ClusterIP | Inside the cluster |
| NodePort | Through a node IP and allocated port |
| LoadBalancer | Through an external load-balancer endpoint, when provisioned |

## How does a ClusterIP application get accessed externally?

A ClusterIP is internal by default, but external access can be arranged through:
- An Ingress controller that routes traffic to the Service.
- A gateway or reverse proxy.
- `kubectl port-forward` for development and debugging.
- A suitable NodePort or LoadBalancer Service.

## Important networking concepts

### Pod IP

An IP assigned to a Pod by the cluster networking implementation.

### Service IP

A stable virtual IP assigned to a Service of a type that uses one.

### `port` vs `targetPort` vs `nodePort`

- `port`: Port exposed by the Service.
- `targetPort`: Port targeted on backend Pods.
- `nodePort`: Port exposed on cluster nodes for a NodePort Service.

### Network policies

NetworkPolicies can control traffic to and from Pods when supported and enforced by the cluster's network plugin.

# 7. Namespaces

A Namespace logically separates groups of Kubernetes resources within a cluster.

Common built-in namespaces include:
- `default`
- `kube-system`
- `kube-public`
- `kube-node-lease`

Commands:

```bash
kubectl get namespaces
kubectl create namespace development
kubectl get pods -n development
kubectl apply -f deployment.yaml -n development
kubectl config set-context --current --namespace=development
```

Namespaces are useful for organizing teams, environments, and resource policies. They are not a strong security boundary by themselves.

# 8. ConfigMaps and Secrets

## ConfigMap

Stores non-confidential configuration, such as application settings and feature flags.

Example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  LOG_LEVEL: info
```

## Secret

Stores sensitive configuration, such as passwords, tokens, and credentials.

Example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
stringData:
  username: appuser
  password: change-me
```

`stringData` accepts plain strings and Kubernetes converts them to Secret data on write. Replace placeholder credentials before using the example.

**Important security points:**
- Base64 encoding is not encryption.
- Restrict Secret access through RBAC.
- Configure encryption at rest where appropriate.
- Avoid committing real credentials to GitHub.
- Prefer an appropriate external secret-management solution when required.

## ConfigMap vs Secret

| ConfigMap | Secret |
|---|---|
| Non-sensitive configuration. | Sensitive configuration. |
| Values are stored as ordinary configuration data. | Data is encoded in the API representation and requires appropriate protection. |
| Used for settings and feature flags. | Used for passwords, tokens, and credentials. |

# 9. StatefulSet and DaemonSet

## StatefulSet

Manages workloads that require stable Pod identities and, commonly, persistent storage.

Examples:
- Database clusters.
- Distributed stateful applications.
- Systems that require stable network identities.

Features:
- Stable Pod names and identities.
- Ordered deployment and scaling behavior by default.
- Integration with volume claim templates.

## DaemonSet

Ensures a copy of a Pod runs on each eligible node, subject to scheduling rules.

Common use cases:
- Node-level log collection.
- Monitoring agents.
- Node networking components.

## Deployment vs StatefulSet vs DaemonSet

| Resource | Main purpose |
|---|---|
| Deployment | Manage stateless application replicas and releases. |
| StatefulSet | Manage workloads with stable identities and storage needs. |
| DaemonSet | Run a Pod on every eligible node. |

# 10. Kubernetes Storage

## PersistentVolume (PV)

A cluster storage resource backed by storage infrastructure.

## PersistentVolumeClaim (PVC)

A request for storage by a workload.

## StorageClass

Defines a storage class and provisioner configuration that can dynamically create storage when supported.

## How storage works

1. An administrator or provisioner makes storage available.
2. A workload requests storage through a PVC.
3. Kubernetes binds the claim to suitable storage or provisions storage dynamically.
4. A Pod mounts the claim.
5. Data persistence depends on the underlying storage and its lifecycle settings.

For AWS EKS, persistent storage may use Amazon EBS or Amazon EFS through suitable drivers and configuration.

- **EBS:** Commonly used for block storage attached to a node for a workload.
- **EFS:** Provides shared network file storage that can be mounted by multiple eligible clients.

# 11. Liveness, Readiness and Startup Probes

## Liveness probe

Checks whether a container should be restarted because it is unhealthy.

## Readiness probe

Checks whether a container is ready to receive traffic.

A failed readiness probe removes the Pod from the ready backend endpoints of Services that select it, as applicable.

## Startup probe

Gives a slow-starting application time to initialize before liveness or readiness checks begin.

| Probe | Purpose |
|---|---|
| Liveness | Detect an unhealthy container that may need restarting. |
| Readiness | Determine whether the application can receive traffic. |
| Startup | Determine whether application startup has completed. |

A poorly configured liveness probe can cause unnecessary restarts. Probe thresholds and timings should match the application's behavior.

# 12. Resource Requests and Limits

## Requests

Requests represent the resources Kubernetes uses for scheduling and resource-allocation decisions.

## Limits

Limits define resource constraints enforced according to the resource and container configuration.

Example:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "256Mi"
```

- `100m` CPU means one-tenth of a CPU core.
- CPU limits can throttle CPU usage.
- Exceeding a memory limit can result in a container being terminated, commonly with an `OOMKilled` indication.

# 13. Horizontal Pod Autoscaler (HPA)

HPA adjusts the number of replicas in a scalable workload based on observed metrics.

It can use CPU utilization, memory utilization, or supported custom/external metrics, depending on configuration.

Example:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2
  maxReplicas: 8
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

For CPU utilization-based scaling, relevant CPU requests and a working metrics pipeline are important.

Commands:

```bash
kubectl get hpa
kubectl describe hpa nginx-hpa
kubectl top pods
kubectl top nodes
```

# 14. Ingress and Ingress Controller

## Ingress

An Ingress resource defines rules for HTTP/HTTPS routing to Services.

It can route traffic based on:
- Hostname.
- URL path.
- TLS configuration.

## Ingress Controller

An Ingress resource alone does not provide traffic routing. A compatible Ingress controller must be installed and configured.

Examples include NGINX Ingress Controller and cloud-specific controllers.

In AWS EKS, AWS Load Balancer Controller can provision and manage AWS load-balancing resources for supported Kubernetes resources and configurations.

# 15. Rolling Updates and Rollbacks

## Rolling update

Gradually replaces old application Pods with new Pods to reduce disruption during an update.

## Rollback

Returns a Deployment to a previous revision when an update needs to be reversed and the required revision is available.

Commands:

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.27
kubectl rollout status deployment/nginx-deployment
kubectl rollout history deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment
```

Always use valid image tags for your intended application version. A successful rollout does not by itself prove that the application works correctly.

# 16. Jobs and CronJobs

## Job

Creates Pods to run a task to completion.

Examples:
- Database migration.
- Batch processing.
- One-time maintenance task.

## CronJob

Creates Jobs according to a recurring schedule.

Examples:
- Periodic cleanup.
- Scheduled reports.
- Recurring database backups.

A Job is intended for finite work, unlike a typical continuously running application Deployment.

# 17. Scheduling and Node Management

## Labels and selectors

Labels identify resources, and selectors match resources based on their labels.

## Node selector

Restricts Pod scheduling to nodes with matching labels.

## Node affinity

Defines more expressive node placement preferences or requirements.

## Taints and tolerations

- A taint on a node can repel Pods that do not tolerate it.
- A toleration allows a Pod to be considered for a tainted node; it does not guarantee scheduling there.

## Pod affinity and anti-affinity

Influence Pod placement based on the labels and locations of other Pods.

These features are useful for spreading replicas or placing tightly coupled workloads together.

# 18. Kubernetes Security

Important security practices:

- Use RBAC to control API permissions.
- Follow the principle of least privilege.
- Protect Secrets and external credentials.
- Use trusted container images and scan them.
- Configure network restrictions where required.
- Run containers as non-root when possible.
- Avoid unnecessary Linux capabilities.
- Set appropriate resource requests and limits.
- Keep Kubernetes components and images updated.
- Audit access and monitor cluster events.

## RBAC resources

- Role: Permissions within a namespace.
- RoleBinding: Grants a Role or ClusterRole to subjects within a namespace.
- ClusterRole: Defines cluster-scoped permissions or reusable permissions.
- ClusterRoleBinding: Grants a ClusterRole at the cluster level.

# 19. Kubernetes Troubleshooting

## Common Pod errors

### ImagePullBackOff

Possible causes:
- Incorrect image name or tag.
- Private registry authentication failure.
- Registry connectivity problems.
- Missing image-pull credentials.

Commands:

```bash
kubectl describe pod <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp
```

### CrashLoopBackOff

Possible causes:
- Application startup failure.
- Incorrect configuration.
- Missing environment variables.
- Dependency connection errors.
- Incorrect startup command.

Commands:

```bash
kubectl logs <pod-name>
kubectl logs <pod-name> --previous
kubectl describe pod <pod-name>
```

### Pending Pod

Possible causes:
- Insufficient CPU or memory.
- Node selector or affinity mismatch.
- Untolerated taints.
- Unavailable persistent storage.
- Scheduling constraints.

Commands:

```bash
kubectl describe pod <pod-name>
kubectl get nodes
kubectl get events
```

### Service not working

Check:
1. The Service selector matches Pod labels.
2. Backend Pods are ready.
3. Service ports and target ports are correct.
4. EndpointSlices contain the expected backends.
5. Network policies, DNS, and routing are configured correctly.

Commands:

```bash
kubectl get svc
kubectl get endpointslices
kubectl describe svc <service-name>
kubectl get pods --show-labels
```

### Pod is OOMKilled

Possible causes:
- Memory usage exceeds the container's memory limit.
- Memory leak.
- Unexpected workload increase.
- Memory limit set too low.

Commands:

```bash
kubectl describe pod <pod-name>
kubectl top pod <pod-name>
kubectl logs <pod-name> --previous
```

# 20. Essential kubectl Commands

## Cluster information

```bash
kubectl cluster-info
kubectl get nodes -o wide
kubectl get namespaces
kubectl version
```

## Resource management

```bash
kubectl get all
kubectl get pods -A
kubectl describe pod <pod-name>
kubectl apply -f <file.yaml>
kubectl delete -f <file.yaml>
```

## Debugging

```bash
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>
kubectl exec -it <pod-name> -- sh
kubectl get events
kubectl top pods
kubectl top nodes
```

## Deployment operations

```bash
kubectl get deployments
kubectl scale deployment <deployment-name> --replicas=4
kubectl rollout status deployment/<deployment-name>
kubectl rollout history deployment/<deployment-name>
kubectl rollout undo deployment/<deployment-name>
```

## YAML validation and inspection

```bash
kubectl apply --dry-run=client -f deployment.yaml
kubectl diff -f deployment.yaml
kubectl explain deployment.spec
kubectl explain pod.spec.containers
```

# 21. Important Interview Revision Points

1. A Pod is the smallest deployable unit in Kubernetes.
2. A Deployment manages ReplicaSets and supports application rollouts.
3. A ReplicaSet maintains a desired number of matching Pods.
4. A Service provides a stable endpoint for selected Pods.
5. ClusterIP is the default Service type.
6. Ingress requires a compatible controller to implement its routing rules.
7. ConfigMaps store non-sensitive configuration; Secrets are intended for sensitive data.
8. PVCs request storage; PVs represent storage resources.
9. Liveness probes help detect containers that need restarting.
10. Readiness probes determine whether applications should receive traffic.
11. HPA scales workloads based on supported metrics.
12. Kubernetes uses desired-state reconciliation to maintain resources.
13. The scheduler selects nodes for unscheduled Pods.
14. The kubelet manages Pods assigned to its node.
15. `kubectl describe`, `kubectl logs`, and Kubernetes events are essential troubleshooting tools.

