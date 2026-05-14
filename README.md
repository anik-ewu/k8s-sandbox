# ☸️ Kubernetes Learning — User Path (CKAD)
> Following: **TechWorld with Nana — Kubernetes User Roadmap**
> Role: **Application Developer / K8s User** (not Admin/CKA)
> Goal: Deploy and run applications in a K8s cluster with high availability.

---

## 🗺️ The Roadmap — 4 Stages

```
Stage 1: Fundamental Concepts  →  Stage 2: Work with K8s  →  Stage 3: Deep Dive (User)  →  Stage 4: Advanced User Topics
```

---

## ✅ Stage 1: Fundamental Concepts

> *"Learn the WHY before the HOW."*

### ✅ 1.1 — Why Kubernetes?
- Docker Compose runs containers on **one machine**
- K8s manages containers across **hundreds of servers**
- Solves: auto-healing, scaling, zero-downtime deployments, load balancing
- Job market: K8s job searches grew **2125% in 4 years**

### ✅ 1.2 — Architecture (Control Plane vs Worker Nodes)
| Component | Role |
|---|---|
| **Control Plane** | The brain — schedules, manages desired state |
| **API Server** | Entry point — all kubectl commands go here |
| **etcd** | The cluster's database — stores all state |
| **Scheduler** | Assigns Pods to Nodes |
| **Controller Manager** | Runs controllers (e.g. Deployment controller) |
| **Worker Node** | Where your app Pods actually run |
| **kubelet** | Agent on each node — talks to control plane |
| **kube-proxy** | Handles networking rules on each node |

### ✅ 1.3 — Core K8s Objects (The 9 you MUST know)
| Object | What it does |
|---|---|
| **Pod** | Smallest unit — wraps your container |
| **Deployment** | Manages Pods — replicas, rolling updates, self-healing |
| **Service** | Stable network endpoint to reach Pods |
| **ConfigMap** | Non-sensitive environment config |
| **Secret** | Sensitive config (passwords, tokens) |
| **Ingress** | HTTP routing / external traffic management |
| **StatefulSet** | Like Deployment but for stateful apps (e.g. databases) |
| **Namespace** | Virtual clusters — isolate environments (dev/staging/prod) |
| **Volume** | Persistent storage for Pods |

### ✅ 1.4 — How K8s Works Behind the Scenes
- **Desired State vs Actual State** — you declare what you want, K8s fights to achieve it
- **Self-healing** — controllers continuously reconcile actual → desired state
- **ReplicaSet** — the controller that keeps N copies of your Pod alive

---

## ✅ Stage 2: Work with Kubernetes (Fundamentals)

### ✅ 2.1 — Access & Interact with K8s (kubectl)
- Cluster: `minikube` v1.38.1 | Kubernetes v1.35.1 | kubectl v1.36.0

**Essential kubectl commands:**
```bash
minikube start                        # Start local cluster
kubectl get nodes                     # Check cluster health
kubectl apply -f <file.yaml>          # Deploy any manifest
kubectl get pods                      # List pods
kubectl get pods -l app=taskmanager   # Filter by label
kubectl describe pod <pod-name>       # Full details + events
kubectl logs <pod-name>               # Container logs
kubectl delete pod <pod-name>         # Delete a pod
kubectl get all                       # See everything in namespace
```
📎 [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

### ✅ 2.2 — K8s Manifest Files
- YAML files that declare the **desired state** of your cluster
- Files live in `phase-2-pods-deployments/`
  - `01-pod.yaml` — Raw Pod (for learning only)
  - `02-deployment.yaml` — Deployment with `replicas: 3`, readiness/liveness probes

**Key insight from practice:**
- Deleted a Pod → K8s spawned a replacement in **seconds** (self-healing proven!)
- `CrashLoopBackOff` = app is crashing (expected — no MongoDB yet)

### 🔜 2.3 — Troubleshooting
- Check container logs: `kubectl logs <pod>`
- Check cluster status: `kubectl get nodes`, `kubectl get pods -A`
- Inspect events: `kubectl describe pod <pod>`
- Verify networking: `kubectl get svc`, `kubectl get endpoints`

### 🔜 2.4 — Common Misconfigurations & Bad Practices
- Hardcoding secrets in manifests
- No resource limits (CPU/memory) on containers
- No readiness/liveness probes → broken pods receive traffic
- Using `latest` image tag in production
- Running everything in the `default` namespace

### 🔜 2.5 — Helm Charts
- Package manager for K8s (like npm, but for K8s apps)
- Learn: What is Helm, what are Helm charts, when to use them
- Especially useful for deploying 3rd-party services (e.g. MongoDB, Nginx)

---

## 🔜 Stage 3: Deep Dive on Deployment Components (User Path)

### 🔜 3.1 — Deep Dive on Core Objects
- **Deployments** — rolling updates, rollbacks, replica management
- **ReplicaSets** — how Deployments use them under the hood
- **StatefulSets** — ordered pod creation, stable network IDs (for DBs)
- **Services** — ClusterIP, NodePort, LoadBalancer
- **Volumes** — emptyDir, hostPath, PersistentVolumeClaim

### 🔜 3.2 — Deep Dive on K8s Manifest Files
- Advanced configuration options for all core resources
- **Init Containers** — run before main container starts
- Configuring different Service types
- ConfigMaps and Secrets injection patterns

### 🔜 3.3 — Deployment Strategies
- **Rolling Updates** — default, zero downtime, gradually replace old pods
- **Rollbacks** — `kubectl rollout undo` to revert bad deployments
- **Blue-Green Deployment** — run v1 and v2 side-by-side, switch traffic
- **Canary Deployment** — send small % of traffic to new version first

---

## 🔜 Stage 4: Advanced User Topics

### 🔜 4.1 — Application Networking
- Connect microservices within the cluster
- **Ingress** — handle external HTTP/HTTPS traffic
- **Service Mesh** (e.g. Istio) — advanced traffic management within cluster
- **Message Broker** (e.g. RabbitMQ, Kafka) — async communication between services

### 🔜 4.2 — CI/CD Integration
- Automate deployments via pipelines (GitHub Actions, Jenkins)
- Deploy to both Self-Managed and Managed K8s (AWS EKS, GKE, AKS)

---

## 📦 Project: Task Manager API on K8s

We are deploying `sabbirhasananik/docker-sandbox:latest` — a Node.js Task Manager API.

| Phase | Status | Folder |
|---|---|---|
| Pod + Deployment | ✅ Done | `phase-2-pods-deployments/` |
| Services (ClusterIP, NodePort) | 🔜 Next | `phase-3-services/` |
| ConfigMaps + Secrets | 🔜 | `phase-4-config/` |
| Ingress | 🔜 | `phase-5-ingress/` |
| Persistent Volumes (MongoDB) | 🔜 | `phase-6-volumes/` |
| StatefulSet for MongoDB | 🔜 | `phase-6-volumes/` |
| Deployment Strategies | 🔜 | `phase-7-strategies/` |
| Helm Charts | 🔜 | `phase-8-helm/` |

---

## 🛠️ Local Setup
```bash
# Prerequisites
docker desktop   # Must be running
minikube start   # Start cluster
kubectl get nodes # Verify: STATUS=Ready
```