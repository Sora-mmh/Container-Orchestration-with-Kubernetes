# Container-Orchestration-with-Kubernetes

The listed projects in this module cover:

### **1. YAML-Driven Deployments**
- **Project**: Deployed **MongoDB + Mongo Express** stack using:
  - **Secret** for DB credentials  
  - **ConfigMap** for DB URL  
  - **Internal & External Services** for connectivity

---

### **2. Namespaces & Networking**
- Used **Namespaces** to logically isolate environments.  
- Exposed services via **ClusterIP, NodePort** and **Ingress**.  
- **Ingress Controller** (NGINX) deployed via Helm; routed traffic from browser.

---

### **3. Storage & Stateful Workloads**
- **Volumes** (emptyDir, hostPath, cloud block storage) for data persistence.  
- **StatefulSet** deployed **replicated MongoDB** with stable network IDs & persistent volumes.  
- Configured **ConfigMap & Secret volumes** to inject config files and secrets.

---

### **4. Helm & Package Management**
- **Helm Demo**:  
  - Deployed **stateful MongoDB replica set** with a single command.  
  - Installed **NGINX Ingress** and configured **Ingress rules**.  
- Created **custom Helm Charts** for microservices with `values.yaml` overrides.

---

### **5. Private Registry & Security**
- **Demo Project**: Deployed images from **AWS ECR** (private registry).  
  - Created **imagePullSecrets** for registry authentication.  

---

### **6. Microservices Deployments**
- **Online-Shop Microservices** deployed across 11 services:
  - **Liveness & Readiness probes** for health checks.  
  - **Resource requests & limits** for QoS.  
  - **Multiple replicas** & **non-root containers** for resilience.  
- **Production-grade refactor**:
  - Added **image version tags**, removed NodePorts, used Ingress.

---

### **7. One-Click Deployments with Helmfile**
- Packaged every microservice into **individual Helm Charts**.  
- **Helmfile** orchestrated multi-chart deployment (`helmfile sync`).  
- Achieved **single-command environment rollouts** (dev / staging / prod).
