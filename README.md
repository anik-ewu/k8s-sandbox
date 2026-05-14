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
minikube stop                         # Pause cluster (resources preserved)
kubectl get nodes                     # Check cluster health
kubectl apply -f <file.yaml>          # Deploy any manifest
kubectl delete -f <file.yaml>         # Remove resources defined in file
kubectl get pods                      # List pods
kubectl get all                       # See everything in namespace
kubectl get pods -l app=taskmanager   # Filter by label
kubectl describe pod <pod-name>       # Full details + events
kubectl describe service <svc-name>   # Inspect a service + endpoints
kubectl logs <pod-name>               # Container logs
kubectl delete pod <pod-name>         # Delete a pod (bare pod stays dead)
kubectl delete deployment <name>      # Delete deployment + all its pods
```
📎 [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

### ✅ 2.2 — K8s Manifest Files
- YAML files that declare the **desired state** of your cluster
- Multiple resources can live in one file — separated by `---`
- Files live in `phase-2-pods-deployments/`

| File | Kind(s) | Purpose |
|---|---|---|
| `01-pod.yaml` | Pod | Bare Pod concept (learning only) |
| `02-deployment.yaml` | Deployment | 3 replicas, readiness/liveness probes |
| `03-mongodb-pod.yaml` | Pod + Service | MongoDB pod + internal ClusterIP service |
| `mongo-secret.yaml` | Secret | base64-encoded credentials |
| `mongo-configmap.yaml` | ConfigMap | Non-sensitive config (DB hostname) |
| `mongo-express.yaml` | Deployment + Service | Mongo Express UI + NodePort service |

**Key insights from practice:**
- Deleted a Pod → K8s spawned a replacement in **seconds** (self-healing proven!)
- `CrashLoopBackOff` = app is crashing on startup (often bad health probe config)
- **Never delete individual pods** managed by a Deployment — delete the Deployment instead
- **Secret must be applied before the Pod** that references it, or pod fails with `CreateContainerConfigError`

### ✅ 2.3 — Pod vs Deployment — When to Use Each

| | Pod | Deployment |
|---|---|---|
| **If it crashes** | ❌ Gone forever | ✅ Auto-recreated |
| **Scaling** | ❌ Manual | ✅ Change `replicas: N` |
| **Rolling updates** | ❌ No | ✅ Zero downtime |
| **Use for** | Learning / one-off debug | All real applications |

> ⚠️ For databases — use **StatefulSet** (not Pod or Deployment). Covered in Stage 3.

### ✅ 2.4 — Secret vs ConfigMap — What Goes Where

| | ConfigMap | Secret |
|---|---|---|
| **Data type** | Non-sensitive | Sensitive |
| **Examples** | URLs, hostnames, flags | Passwords, tokens, certs |
| **Stored as** | Plain text | base64 encoded |
| **Your case** | `mongodb-service` (hostname) | username & password |

```bash
# Generate base64 values for secrets
echo -n 'mypassword' | base64
```

> ⚠️ base64 is **encoding**, not encryption. Never commit secret files to git.

### ✅ 2.5 — Services — How Pods Talk to Each Other

| Service Type | Accessible From | Use Case |
|---|---|---|
| **ClusterIP** (default) | Inside cluster only | Pod-to-pod communication |
| **NodePort** | Your local machine (via Minikube) | Dev/testing access |
| **LoadBalancer** | Public internet | Cloud production deployments |

**Key insight:** Service names act as **internal DNS hostnames**.
`mongodb-service` → K8s resolves it → ClusterIP → Pod. No IPs needed.

```
Browser
  │ NodePort :30081
  ▼
mongo-express (Deployment)
  │ ClusterIP: mongodb-service:27017
  ▼
mongodb-pod
```

### ✅ 2.6 — Namespaces

Namespaces are **virtual clusters** inside your physical cluster. Use them to organise and isolate resources.

**4 Default Namespaces:**
| Namespace | Purpose |
|---|---|
| `default` | Where your resources live if no namespace specified |
| `kube-system` | K8s internal components — never touch! |
| `kube-public` | Publicly accessible cluster info |
| `kube-node-lease` | Heartbeat tracking for node availability |

**4 Reasons to Use Namespaces:**

1. **Structure Components** — group resources by team, app, or environment
   ```
   namespace: database     → MongoDB, Redis
   namespace: monitoring   → Prometheus, Grafana
   namespace: app-staging  → API (staging)
   namespace: app-prod     → API (production)
   ```

2. **Avoid Conflicts** — two teams can both have a resource named `my-app` without clashing
   ```bash
   kubectl get pod my-app -n team-a   # team-a's app
   kubectl get pod my-app -n team-b   # team-b's app — no conflict
   ```

3. **Share Services** — central tools like Prometheus can be shared across namespaces
   ```
   # Full DNS format to access service in another namespace:
   <service-name>.<namespace>.svc.cluster.local
   ```
   > ⚠️ ConfigMaps and Secrets **cannot** be shared across namespaces — each namespace needs its own copy.

4. **Access & Resource Limits** — RBAC per namespace + CPU/memory quotas per team
   ```yaml
   kind: ResourceQuota
   spec:
     hard:
       cpu: "2"       # Max 2 CPUs for this namespace
       memory: 2Gi    # Max 2GB RAM
       pods: "10"     # Max 10 pods
   ```

**Useful commands:**
```bash
kubectl get namespaces                        # List all namespaces
kubectl apply -f file.yaml -n my-namespace    # Deploy to specific namespace
kubectl get pods -n kube-system               # View system pods
```

### 🔜 2.7 — Troubleshooting
- Check container logs: `kubectl logs <pod>`
- Check cluster status: `kubectl get nodes`, `kubectl get pods -A`
- Inspect events: `kubectl describe pod <pod>`
- Verify networking: `kubectl get svc`, `kubectl get endpoints`

### 🔜 2.8 — Common Misconfigurations & Bad Practices
- Hardcoding secrets in manifests
- No resource limits (CPU/memory) on containers
- No readiness/liveness probes → broken pods receive traffic
- Using `latest` image tag in production
- Running everything in the `default` namespace

### 🔜 2.9 — Helm Charts
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
| Pod + Deployment basics | ✅ Done | `phase-2-pods-deployments/` |
| MongoDB + Mongo Express demo | ✅ Done | `phase-2-pods-deployments/` |
| Namespaces | ✅ Done | — |
| Ingress | 🔜 Next | `phase-3-ingress/` |
| Helm | 🔜 | `phase-4-helm/` |
| Persistent Volumes | 🔜 | `phase-5-volumes/` |
| StatefulSet for MongoDB | 🔜 | `phase-5-volumes/` |
| Deployment Strategies | 🔜 | `phase-6-strategies/` |

---

## 🛠️ Local Setup
```bash
# Prerequisites
docker desktop   # Must be running
minikube start   # Start cluster
kubectl get nodes # Verify: STATUS=Ready

# Apply the demo project (in order!)
kubectl apply -f mongo-secret.yaml       # Secret first!
kubectl apply -f mongo-configmap.yaml
kubectl apply -f 03-mongodb-pod.yaml
kubectl apply -f mongo-express.yaml

# Open Mongo Express in browser
minikube service mongo-express-service
```