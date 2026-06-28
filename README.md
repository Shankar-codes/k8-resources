# ☸️ k8-resources

A hands-on, progressively structured collection of Kubernetes manifest examples covering every core concept — from Namespaces and Pods all the way through StatefulSets with persistent volumes. Built as part of a **DevOps learning curriculum** by [Shankar Thimmappa](https://github.com/Shankar-codes), using the **Roboshop** e-commerce project as the running example.

> 📄 A companion reference document `K8S-Services-explained.docx` is included for a deeper conceptual explanation of Kubernetes Services.

---

## 📚 What's Inside

The repository is organized as **21 numbered YAML files**, each building on the previous one. They are designed to be studied and applied in order.

```
k8-resources/
├── volumes/                        # Volume & PersistentVolume examples
├── 01-namespace.yaml               # Namespace
├── 02-pods.yaml                    # Basic Pod
├── 03-multicontainer.yaml          # Multi-container Pod (sidecar pattern)
├── 04-labels.yaml                  # Labels
├── 05-errors.yaml                  # Intentional error scenarios
├── 06-annotations.yaml             # Annotations
├── 07-resource.yaml                # Resource Requests & Limits
├── 08-environment.yaml             # Environment Variables (inline)
├── 09-configmap.yaml               # ConfigMap definition
├── 10-pod-config.yaml              # Pod consuming a ConfigMap
├── 11-secrets.yaml                 # Secret definition
├── 12-pod-secret.yaml              # Pod consuming a Secret
├── 13-service.yaml                 # ClusterIP Service
├── 14-pod-service.yaml             # Pod targeted by a ClusterIP Service
├── 15-service-node-port.yaml       # NodePort Service
├── 16-pod-node-port.yaml           # Pod targeted by a NodePort Service
├── 17-service-load-balancer.yaml   # LoadBalancer Service
├── 18-pod-load-balancer.yaml       # Pod targeted by a LoadBalancer Service
├── 19-replicaset.yaml              # ReplicaSet
├── 20-deployment.yaml              # Deployment + Headless Service
├── 21-statefulset.yaml             # StatefulSet + Headless + Normal Service
└── K8S-Services-explained.docx     # Reference doc: K8s Services deep-dive
```

---

## 🗺️ Learning Path

The files follow a deliberate progression through Kubernetes concepts:

```
Namespace → Pod → Multi-container → Labels & Annotations
    → Resources → Environment → ConfigMap → Secrets
        → Services (ClusterIP → NodePort → LoadBalancer)
            → ReplicaSet → Deployment → StatefulSet
```

---

## 📋 File-by-File Reference

### 01 — Namespace
Creates the `roboshop` namespace used by all subsequent resources.

```yaml
kind: Namespace
metadata:
  name: roboshop
  labels:
    environment: dev
    project: roboshop
```

### 02 — Basic Pod
A single nginx Pod inside the `roboshop` namespace with a label for service selection.

```yaml
kind: Pod
metadata:
  name: nginx
  namespace: roboshop
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

### 03 — Multi-container Pod
Demonstrates the **sidecar pattern** — two containers sharing the same Pod lifecycle, network, and volumes.

### 04 — Labels
Explores label selectors, their role in routing Services to the right Pods, and how to query with `kubectl get pods -l`.

### 05 — Errors (Intentional)
Deliberately misconfigured manifests to practice reading and debugging Kubernetes error states (`CrashLoopBackOff`, `ImagePullBackOff`, `ErrImagePull`).

### 06 — Annotations
Shows how non-identifying metadata (build info, contact, tool configs) is attached to resources without affecting scheduling.

### 07 — Resource Requests & Limits
Demonstrates setting CPU and memory requests/limits to enable the scheduler to make informed placement decisions and prevent noisy-neighbour issues.

### 08 — Environment Variables (Inline)
Injecting configuration directly into a container via `env:` key-value pairs in the Pod spec.

### 09 — ConfigMap
Defines a standalone `ConfigMap` (`ecommerce-pod-config`) holding non-sensitive configuration:

```yaml
kind: ConfigMap
metadata:
  name: ecommerce-pod-config
data:
  course: DevOps
  duration: 3 months
  trainer: Shankar Thimmappa
```

### 10 — Pod + ConfigMap
Shows how to mount the ConfigMap above as environment variables or a volume inside a Pod, decoupling config from image.

### 11 — Secrets
Creates a Kubernetes `Secret` to hold base64-encoded sensitive data (credentials, tokens).

### 12 — Pod + Secret
Mounts the Secret defined in `11-secrets.yaml` into a Pod, either as environment variables or a volume-mounted file.

### 13 — ClusterIP Service (Default)
The most common Service type — exposes Pods internally within the cluster only.

```yaml
kind: Service
metadata:
  name: nginx
spec:
  selector:
    project: ellamma-ecommerce
    component: frontend
    environment: dev
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

### 14 — Pod + ClusterIP Service
The Pod whose labels are matched by the ClusterIP Service in `13-service.yaml`.

### 15 — NodePort Service
Exposes a Service on a static port on each Node, making it reachable from outside the cluster (useful for dev/testing).

### 16 — Pod + NodePort Service
The corresponding Pod targeted by the NodePort Service.

### 17 — LoadBalancer Service
Provisions a cloud provider Load Balancer (e.g., AWS ELB) in front of the cluster — the standard production external-access pattern.

### 18 — Pod + LoadBalancer Service
The corresponding Pod targeted by the LoadBalancer Service.

### 19 — ReplicaSet
Ensures a specified number of identical Pod replicas are always running. Foundation for self-healing workloads.

### 20 — Deployment + Headless Service
A full `Deployment` with 3 nginx replicas demonstrating rolling updates, revision history, and a **Headless Service** (`clusterIP: None`) for direct Pod DNS addressing.

```yaml
kind: Deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      purpose: deployment
      project: headless-service-demo
---
kind: Service
spec:
  clusterIP: None   # Headless — no load-balancing, direct Pod DNS
```

### 21 — StatefulSet + Headless + Normal Service
The most advanced example. Combines:
- A **Headless Service** (`nginx-headless`, `clusterIP: None`) for stable Pod DNS (`nginx-0.nginx-headless`, `nginx-1.nginx-headless`, …)
- A **normal ClusterIP Service** (`nginx`) for general load-balanced access
- A **StatefulSet** with 3 replicas, each with its own `PersistentVolumeClaim` via `volumeClaimTemplates` (using the `ellamma-ebs` StorageClass)

```yaml
kind: StatefulSet
spec:
  replicas: 3
  serviceName: "nginx-headless"
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: "ellamma-ebs"
      resources:
        requests:
          storage: 1Gi
```

### volumes/
Additional examples covering:
- `emptyDir` — ephemeral shared volume within a Pod
- `hostPath` — mounts a node directory into a container
- `PersistentVolume` (PV) and `PersistentVolumeClaim` (PVC) — decoupled storage lifecycle
- `StorageClass` — dynamic provisioning

---

## ⚡ Quick Start

### Prerequisites

- A running Kubernetes cluster (local: [minikube](https://minikube.sigs.k8s.io/), [kind](https://kind.sigs.k8s.io/); cloud: EKS, GKE, AKS)
- `kubectl` configured and pointing to your cluster
- For StatefulSet examples (`21-statefulset.yaml`): the `ellamma-ebs` StorageClass must exist (see [k8-ecommerce-database](https://github.com/Shankar-codes/k8-ecommerce-database))

### Apply a Single Manifest

```bash
kubectl apply -f 01-namespace.yaml
kubectl apply -f 02-pods.yaml
```

### Apply All Manifests in Order

```bash
kubectl apply -f 01-namespace.yaml
for i in $(seq -f "%02g" 2 21); do
  kubectl apply -f ${i}-*.yaml
done
```

### Verify Resources

```bash
# All resources in the roboshop namespace
kubectl get all -n roboshop

# Describe a specific Pod
kubectl describe pod nginx -n roboshop

# Follow Pod logs
kubectl logs -f nginx -n roboshop

# Check PVCs (for StatefulSet)
kubectl get pvc -n roboshop
```

### Clean Up

```bash
kubectl delete namespace roboshop
```

---

## 🔑 Key Concepts Covered

| Concept | Files |
|---|---|
| Namespace isolation | `01` |
| Pod lifecycle | `02`, `03`, `05` |
| Metadata (Labels & Annotations) | `04`, `06` |
| Resource management | `07` |
| Configuration injection | `08`, `09`, `10` |
| Secrets management | `11`, `12` |
| Service types | `13–18` |
| Self-healing replicas | `19` |
| Deployments & rolling updates | `20` |
| StatefulSets & persistent storage | `21`, `volumes/` |
| Headless Services | `20`, `21` |

---

## 🛠️ Tech Stack

| Tool | Purpose |
|---|---|
| Kubernetes | Container orchestration |
| kubectl | CLI for cluster interaction |
| nginx | Demo application image |
| AWS EBS (CSI) | Persistent storage for StatefulSet example |

---

## 📖 Additional Reference

`K8S-Services-explained.docx` — included in the repo root — provides a detailed written explanation of the three Service types (ClusterIP, NodePort, LoadBalancer) with diagrams and use-case guidance.

---

## 🤝 Contributing

Pull requests and issues are welcome. If you spot an error or want to add a new concept (e.g., Ingress, HPA, NetworkPolicy), feel free to open a PR.

---

## 👤 Author

**Shankar Thimmappa** — DevOps Trainer  
GitHub: [@Shankar-codes](https://github.com/Shankar-codes)
