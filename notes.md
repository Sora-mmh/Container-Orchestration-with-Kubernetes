# Kubernetes Container Orchestration 

## 1. Introduction to Kubernetes

- Kubernetes (K8s or Kube) is an open-source platform originally developed by Google to automate deployment, scaling, management, and orchestration of containerized applications.

### Core Features:
- High availability (no downtime).
- Automatic scaling.
- Disaster recovery (backup and restore).
- Self-healing.

***

## 2. Kubernetes Components Overview

### Pod
- Smallest deployable unit in K8s.
- One or more containers grouped together.
- Each Pod has a unique internal IP.
- Pods are ephemeral; replaced Pods get new IPs.

### Service
- Provides a stable, permanent IP abstraction to access Pods.
- Life cycle decoupled from Pods.
- Supports load balancing across replicated Pods.
- Service types:
  - Internal Service (default)
  - External Service (accessible from browser)

### Ingress
- Entry point to the K8s cluster for HTTP(s) traffic.
- Routes incoming requests to Services.
- Addressed by domain names (e.g. https://my-app-domain).

### ConfigMap and Secret
- ConfigMap stores non-confidential key-value configuration data.
- Secret stores sensitive data (e.g., credentials).
- Both can be consumed by Pods as environment variables, CLI args, or config files via volumes.

### Volumes
- Provide persistent or shared storage to Pods.
- Persist data beyond container restarts.
- Types include local and external storage.

### Deployment
- Blueprint for Pods with specified replica count.
- Manages rollout and scaling.
- Pods created by deployment are load balanced by Service.

### StatefulSet
- Manages stateful applications.
- Guarantees stable network IDs and persistent storage.
- Pods are not interchangeable; update and scaling are sequential.

***

## 3. Kubernetes Architecture

- **Cluster**: collection of nodes.
- **Node Types**:
  - Worker Nodes: run containerized applications (Pods).
  - Control Planes: manage cluster state and scheduling, replicated for HA.

### Worker Node Components:
- Container Runtime (e.g., containerd, CRI-O, Docker) to run containers.
- Kubelet: ensures container lifecycle on Nodes.
- Kube-Proxy: network proxy routing pod traffic on Node.

### Control Plane Components:
- API Server: cluster gateway and authentication.
- Scheduler: assigns Pods to Nodes based on criteria.
- Controller Manager: monitors cluster health, re-schedules pods.
- etcd: distributed key-value store for cluster data and state.

***

## 4. Minikube – Local Kubernetes Cluster

- Minikube runs a single-node local K8s cluster, ideal for development/testing.
- Can run on Docker or VM (e.g., Hyperkit on macOS).
- Command examples:
  ```bash
  minikube start --driver=docker
  kubectl get nodes
  ```
- Kubectl config is automatically created at `~/.kube/config`.

***

## 5. Kubectl – CLI for Kubernetes

- Manage cluster resources declaratively and imperatively.
- Basic commands:
  - `kubectl get all`: list all components.
  - `kubectl create|edit|delete {component}`: manage components.
  - `kubectl logs {pod}`: view logs.
  - `kubectl exec -it {pod} -- bash`: enter container shell.
  - `kubectl apply -f config.yaml`: create/update from config.
  - `kubectl describe {resource}`: detailed info.
- Supports help, options, and metrics (`kubectl top`).

***

## 6. YAML Configuration Files

- Configuration files are declarative and specify desired state.
- Typical structure:
  - `metadata`: name, labels.
  - `spec`: resource-specific configuration.
  - `status`: added by K8s, actual state info.
- Example: Deployment describing Pods, labels, replicas.
- Labels and selectors connect deployments, pods, and services.
- Services specify `port` (service port) and `targetPort` (Pod container port).

***

## 7. Deploying MongoDB and UI

- Components:
  - MongoDB Deployment and Service.
  - MongoExpress Deployment and Service.
  - ConfigMap and Secret store DB connection info and credentials.
- Secrets hold base64-encoded sensitive data, mounted into Pods.
- Create secrets, configmaps, then deployments in order.
- Use `kubectl apply` and check Pod readiness.
- Expose external service with LoadBalancer or NodePort.
- Use `minikube service` command to access services locally on Minikube.

***

## 8. Namespaces for Resource Organization

- Namespaces partition cluster resources for isolation and management.
- Default namespaces: `default`, `kube-system`, `kube-public`, `kube-node-lease`.
- Create namespaces with `kubectl create namespace `.
- Use namespaces to logically group resources, isolate teams, define environments.
- Namespace-specific resource scoping; cannot cross-reference secrets/configmaps across namespaces.
- Services can be referenced with full names including namespace (e.g., `service.namespace`).

***

## 9. Services - Connecting Pods and Exposure

- Services provide stable IPs and load-balance Pods.
- Types:
  - **ClusterIP**: internal access (default).
  - **NodePort**: exposes service on each Node’s IP + port.
  - **LoadBalancer**: uses cloud provider to expose service externally via load balancer.
- Headless services provide direct Pod IPs (no cluster IP) useful for stateful apps.
- Each service uses selectors matching Pod labels to route traffic appropriately.

***

## 10. Ingress – External Access Routing

- Provides HTTP(s) routing using hostnames and paths (friendly URLs).
- Requires an Ingress Controller (e.g., Nginx Ingress Controller) that manages routing rules.
- Ingress resources define host rules, backend services, TLS.
- Use `minikube addons enable ingress` to install Ingress controller locally.
- Map Ingress hostname to cluster IP in `/etc/hosts` (or use tunneling).
- Define multiple paths and hosts for complex routing.
- TLS termination supported via secrets with certificates.

***

## 11. Persistent Storage with Volumes

- Pods lose local data upon restart; volumes enable persistence.
- PersistentVolume (PV): storage provisioned cluster-wide.
- PersistentVolumeClaim (PVC): user request for storage.
- StorageClass dynamically provisions PVs based on declarations.
- Pods mount PVCs to access storage.
- Support diverse backends: local disks, NFS, cloud block storage.
- PVC must reside in the same namespace as Pod; PV is cluster-scoped.

***

## 12. ConfigMap & Secret Volume Types

- Pass config or secrets as files or env vars to Pods.
- Example manifests include mounting ConfigMaps and Secrets as Volumes.
- Use for application configuration, certificates, credentials.

***

## 13. StatefulSet – Stateful Application Deployment

- For non-interchangeable Pods needing stable identities and persistent storage.
- Uses ordinal index for Pod names.
- Ensures ordered startup, deletion, scaling.
- Suitable for databases and other stateful services.
- Requires individual PersistentVolumes for each Pod.
- Adds complexity; developer responsible for backup/sync logic.

***

## 14. Managed Kubernetes Services

- Self-managed vs. Managed K8s clusters.
- Managed services handle control plane, storage, networking.
- Examples:
  - AWS Elastic Kubernetes Service (EKS)
  - Azure Kubernetes Service (AKS)
  - Google Kubernetes Engine (GKE)
  - Linode Kubernetes Engine (LKE)
- Managed service advantages: simplifies administration, increased reliability.
- Potential Vendor lock-in mitigated with Infrastructure as Code tools.

***

## 15. Helm – Kubernetes Package Manager

- Similar to yum/apt but for K8s resources.
- Manages packaging, templating, deploying complex applications.
- Helm Charts bundle YAML manifests, templates, metadata.
- Supports versioning, upgrades, rollbacks.
- Chart structure includes `Chart.yaml`, `values.yaml`, `templates/`, etc.
- Values allow parameterized deployments for environments and customizations.
- Helm version 3 removes Tiller for security; client-only with in-cluster state.

***

## 16. Helm Demo – Managed K8s Cluster Deployment

- Example deploying MongoDB StatefulSet using Bitnami Helm charts.
- Add Helm repos (`helm repo add`), search, install charts.
- Use `kubectl get` commands to follow deployment and state.
- Access internal services (Grafana, Prometheus) via port-forwarding or Ingress.
- Monitor components and edit alerting/rules as required.

***

## 17. Secure your Kubernetes Cluster with RBAC

- Authentication methods: static tokens, certs, third-party (LDAP).
- Authorization modes: ABAC, RBAC, Webhook.
- **RBAC** defines who can access what:
  - Roles and RoleBindings scoped to namespaces.
  - ClusterRoles and ClusterRoleBindings cluster-wide.
- ServiceAccounts represent non-human actors.
- Use `kubectl auth can-i  --as ` to check permissions.

***

## 18. Microservices in Kubernetes

- Microservices architecture involves multiple independent services, each responsible for a single business domain.
- Enables separate development, deployment, scaling.
- Communication via HTTP APIs, message queues, or service mesh proxy sidecars.
- Deploying microservices in K8s entails deploying:
  - Multiple Deployments and Services.
  - Shared ConfigMaps and Secrets.
- Helm Charts simplify deploying multiple related microservices with parameterizable templates.

***

## 19. Microservices Deployment Demo

- Microservices example repository deployed on a cluster.
- Service discovery with internal DNS and environment variables.
- Services exposed internally via `ClusterIP`.
- Frontend service exposed externally via `NodePort`.
- Redis caching with ephemeral storage.
- Use namespaces for logical grouping.
- Use Helm Charts and Helmfile tool for easier management and deployment.

***

## 20. Production & Security Best Practices for Kubernetes

- Pin image versions for reproducibility.
- Configure *liveness* and *readiness* probes to manage container availability.
- Set resource requests and limits to prevent resource starvation or overuse.
- Prefer `LoadBalancer` or Ingress over `NodePort` in production.
- Deploy multiple replicas per service to ensure availability.
- Avoid running containers as root user.
- Label resources for easy management.
- Keep K8s components updated.

***
