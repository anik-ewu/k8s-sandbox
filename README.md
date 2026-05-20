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

---

### ✅ 2.2 — K8s Manifest Files

**Why:** Declaring desired state in YAML files means your infrastructure is version-controlled, repeatable, and reviewable — just like code.

**How:** Every YAML file has 3 parts: `metadata` (name/labels), `spec` (what you want), and K8s auto-fills `status` (what actually exists).

**When:** Always. Every resource in production should be defined in a YAML file, never created imperatively with `kubectl run`.

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
- Multiple resources in one file are separated by `---`

---

### ✅ 2.3 — Pod vs Deployment

**Why Deployment over Pod:**
A bare Pod has no supervisor. If it crashes, it's gone. A Deployment is a controller that constantly watches your Pods and fights to keep the desired number running.

**How:**
- Pod → just a container wrapper, no self-healing
- Deployment → wraps a ReplicaSet which manages N identical Pods

**When to use Pod:** Only for learning or one-off debugging. Never in production.
**When to use Deployment:** Every real stateless application.

| | Pod | Deployment |
|---|---|---|
| **If it crashes** | ❌ Gone forever | ✅ Auto-recreated |
| **Scaling** | ❌ Manual | ✅ Change `replicas: N` |
| **Rolling updates** | ❌ No | ✅ Zero downtime |
| **Use for** | Learning / one-off debug | All real applications |

> ⚠️ For databases — use **StatefulSet** (not Pod or Deployment). Covered in Stage 3.

---

### ✅ 2.4 — Secret vs ConfigMap

**Why:** Separating config from code is a 12-factor app best practice. You should not hardcode URLs or passwords in your container image.

**How:** Both are injected into Pods as environment variables or mounted as files. The difference is sensitivity — Secrets are base64 encoded (not encrypted by default).

**When:**
- ConfigMap → any non-sensitive config: URLs, hostnames, feature flags, port numbers
- Secret → any sensitive data: passwords, API tokens, TLS certificates

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

---

### ✅ 2.5 — Services

**Why:** Pods are ephemeral — they die and get new IPs. A Service gives you a stable IP and DNS name that never changes, even as Pods come and go.

**How:** A Service uses label selectors to find matching Pods and load balances traffic across them. K8s internal DNS resolves the Service name to its ClusterIP automatically.

**When:**
- ClusterIP → pod-to-pod traffic inside the cluster (default)
- NodePort → expose to your local machine for dev/testing
- LoadBalancer → expose to the public internet (cloud only)

| Service Type | Accessible From | Use Case |
|---|---|---|
| **ClusterIP** (default) | Inside cluster only | Pod-to-pod communication |
| **NodePort** | Your local machine (via Minikube) | Dev/testing access |
| **LoadBalancer** | Public internet | Cloud production deployments |

```
Browser
  │ NodePort :30081
  ▼
mongo-express (Deployment)
  │ ClusterIP: mongodb-service:27017
  ▼
mongodb-pod
```

---

### ✅ 2.6 — Namespaces

**Why:** Without namespaces, all teams and environments share one flat space. Resources clash, access can't be controlled, and it becomes impossible to manage at scale.

**How:** Namespaces are virtual clusters inside your physical cluster. Each namespace has its own isolated set of resources.

**When:**
- Small teams / single projects → `default` is fine
- Multiple teams, environments (dev/staging/prod), or microservices → use namespaces

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
   > ⚠️ ConfigMaps and Secrets **cannot** be shared across namespaces — each needs its own copy.

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

---

### ✅ 2.7 — Ingress

**Why:** NodePort exposes apps on ugly ports (e.g. `:30081`) with no domain name, no HTTPS, and one port per service. Ingress gives you a single clean entry point — domain-based routing with TLS support.

**How:**
Ingress has two parts:
1. **Ingress Resource** (your YAML) — defines the routing rules (which host/path → which service)
2. **Ingress Controller** (a running Pod) — reads those rules and handles the actual traffic. You must install this separately. In Minikube we use the nginx controller.

```
Internet → Ingress Controller (nginx) → reads Ingress rules → routes to Service → Pod
```

**When:**
- Use **NodePort** for quick local dev/testing (simple, no setup)
- Use **Ingress** for any real external-facing app (clean URLs, HTTPS, multiple services)

| Scenario | Use |
|---|---|
| Internal pod-to-pod traffic | ClusterIP Service |
| Quick local dev/testing | NodePort Service |
| Production external HTTP/HTTPS | Ingress |
| Multiple services on one domain | Ingress (path/host routing) |

**Key concepts:**
- `host` — the domain name that triggers this rule (e.g. `dashboard.com`)
- `path` — the URL path to match (`/` = everything, `/api` = only that path)
- `pathType: Prefix` — matches `/api`, `/api/users`, `/api/anything`
- `pathType: Exact` — matches only the exact path `/api`
- Ingress must be in the **same namespace** as the Service it routes to

**Minikube setup commands:**
```bash
minikube addons enable ingress          # Install nginx Ingress Controller
kubectl apply -f dashboard-ingress.yaml # Apply your Ingress rules
kubectl get ingress -n kubernetes-dashboard  # Verify ADDRESS is assigned

# Mac only — Docker driver requires tunnel
minikube tunnel                         # Run in separate terminal, keep open
# /etc/hosts: 127.0.0.1    dashboard.com
```

**Files:**
- `phase-3-ingress/dashboard-ingress.yaml` — basic single-host Ingress for Kubernetes Dashboard

---

### 🔜 2.8 — Troubleshooting
- Check container logs: `kubectl logs <pod>`
- Check cluster status: `kubectl get nodes`, `kubectl get pods -A`
- Inspect events: `kubectl describe pod <pod>`
- Verify networking: `kubectl get svc`, `kubectl get endpoints`

### 🔜 2.9 — Common Misconfigurations & Bad Practices
- Hardcoding secrets in manifests
- No resource limits (CPU/memory) on containers
- No readiness/liveness probes → broken pods receive traffic
- Using `latest` image tag in production
- Running everything in the `default` namespace

---

## ✅ Stage 3: Deep Dive on Deployment Components (User Path)

---

### ✅ 3.1 — Helm

**Why:**
Deploying complex apps (e.g. Elasticsearch, MongoDB, Prometheus) requires 6–10+ YAML files wired together correctly. Writing all of this from scratch is slow and error-prone. Helm gives you pre-built, production-tested K8s manifests you can install with one command — like `npm install` but for Kubernetes.

Second use case: your own microservices. If you have 5 services with nearly identical YAML, Helm templating eliminates copy-paste duplication.

**How:**
Helm has three core concepts:

1. **Helm Chart** — a bundle of K8s YAML templates + a `values.yaml` for configuration
   ```
   my-chart/
   ├── Chart.yaml        # Chart name, version, description
   ├── values.yaml       # Default config — you override these
   └── templates/        # K8s YAML with {{ .Values.xxx }} placeholders
       ├── deployment.yaml
       ├── service.yaml
       └── secret.yaml
   ```

2. **Templating Engine** — placeholders in templates are filled with values at install time
   ```yaml
   # templates/deployment.yaml
   containers:
     - name: {{ .Values.appName }}
       image: {{ .Values.image }}
       ports:
         - containerPort: {{ .Values.port }}
   ```

3. **Release Management** — every `helm install` creates a named release with full revision history
   ```bash
   helm install my-mongo bitnami/mongodb     # Creates release "my-mongo"
   helm upgrade my-mongo bitnami/mongodb     # New revision
   helm rollback my-mongo 1                  # Revert to revision 1
   helm uninstall my-mongo                   # Remove everything cleanly
   helm list                                 # See all releases + revisions
   ```

**When:**
| Situation | Use Helm? |
|---|---|
| Deploying 3rd-party tools (MongoDB, Nginx, Prometheus) | ✅ Yes — use existing public charts |
| Own microservices with repeated YAML structure | ✅ Yes — use as templating engine |
| Simple one-off learning manifests | ❌ No — plain YAML is fine |
| Production with rollback requirements | ✅ Yes — release management |

**Chart Repositories:**
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami   # Most popular repo
helm repo update                                            # Fetch latest charts
helm search repo mongodb                                    # Find a chart
helm show values bitnami/mongodb                            # Preview all config options
```
> 📦 Browse all charts at: https://artifacthub.io — like npmjs.com but for Helm

**Best Practices:**
- Always run `helm show values` before installing — understand what you're deploying
- Override values with `-f custom-values.yaml` or `--set key=value`, never edit Chart files directly
- Pin chart versions (`--version 13.6.0`) in production — never use `latest` blindly
- Use `helm diff` plugin to preview changes before `helm upgrade`
- Store your `values.yaml` overrides in git — treat them like code

**Useful commands:**
```bash
brew install helm                                    # Install Helm on Mac
helm version                                         # Verify install
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo <name>                              # Search available charts
helm install <release-name> <repo/chart>             # Install
helm install <release-name> <repo/chart> -f values.yaml  # Install with overrides
helm upgrade <release-name> <repo/chart>             # Upgrade
helm rollback <release-name> <revision>              # Roll back
helm uninstall <release-name>                        # Remove
helm list                                            # List all releases
helm status <release-name>                           # Inspect a release
```

**Files:**
- `phase-4-helm/mongo-values.yaml` — Custom values for bitnami/mongodb chart (install, upgrade, rollback practiced)

---

### ✅ 3.2 — Volumes & Persistent Storage (High-Level Overview)

**Why:**
Pods are stateless by default. Every time a Pod restarts, its filesystem is wiped. For databases or any app that writes data, you need storage that outlives the Pod lifecycle.

**How:**
Three-layer abstraction:
- **PersistentVolume (PV)** — the actual storage (disk, NFS, cloud block storage). Created by admin or dynamically.
- **PersistentVolumeClaim (PVC)** — a Pod's request for storage ("I need 10Gi"). K8s binds a matching PV to it.
- **StorageClass** — automates PV creation. Defines the type of storage (SSD, HDD, cloud disk).

```
Pod → PVC (claim) → PV (actual storage) → physical disk/cloud
```

**When:**
- Any database (MongoDB, MySQL, Postgres) → needs PVC
- File uploads, logs that must survive restarts → needs PVC
- Ephemeral scratch space (no persistence needed) → use `emptyDir` (no PVC)

**Best Practices:**
- Never use `hostPath` in production (ties your Pod to a specific node)
- Use `StorageClass` with dynamic provisioning — avoid manually creating PVs
- Set appropriate `storageClassName` per environment (fast SSD for prod, cheap HDD for dev)
- Set `ReclaimPolicy: Retain` for production data — prevents accidental deletion

---

### 🔜 3.3 — StatefulSet (for Databases)

**Why:**
Regular Deployments treat all Pods as identical and interchangeable. Databases are NOT identical — each replica has its own data, and the order of startup/shutdown matters. StatefulSet gives each Pod a stable, predictable identity.

**How:**
- Each Pod gets a **sticky identity**: `mongodb-0`, `mongodb-1`, `mongodb-2` (not random hash)
- Each Pod gets its **own PVC** — not shared storage
- Pods are **created in order** (0 → 1 → 2) and **deleted in reverse** (2 → 1 → 0)
- If `mongodb-1` dies, K8s recreates it with the **same name and same PVC**

**When:**
| App type | Use |
|---|---|
| Stateless (API, web app, worker) | Deployment |
| Stateful (MongoDB, MySQL, Redis, Kafka) | StatefulSet |
| Stateful with complex clustering | StatefulSet + Headless Service |

**Best Practices:**
- Always pair StatefulSet with a **Headless Service** (for stable DNS per Pod)
- For production databases, prefer **managed services** (AWS RDS, Atlas) over self-hosted StatefulSets — they are complex to operate
- Use StatefulSet for learning/dev; evaluate managed DB for production

---

### 🔜 3.4 — Deployment Strategies

**Why:**
How you roll out updates determines whether your users experience downtime. Different strategies balance speed, safety, and resource cost differently.

**How & When:**

| Strategy | How | When to use |
|---|---|---|
| **Rolling Update** (default) | Replace old Pods one at a time | Most apps — zero downtime, low risk |
| **Recreate** | Kill all old Pods, then create new | Dev/staging — accepts brief downtime |
| **Blue-Green** | Run v1 and v2 in parallel, switch Service | High-stakes releases — instant rollback |
| **Canary** | Send 5–10% traffic to v2, ramp up if stable | Testing in production safely |

```bash
kubectl rollout status deployment/my-app   # Watch a rolling update
kubectl rollout undo deployment/my-app     # Instant rollback to previous version
kubectl rollout history deployment/my-app  # See revision history
```

**Best Practices:**
- Always set `readinessProbe` — K8s only sends traffic to Pods that pass it
- Set `maxSurge` and `maxUnavailable` to control rolling update speed
- Use `kubectl rollout undo` immediately if you spot issues post-deploy
- Tag images with specific versions, not `latest` — makes rollbacks meaningful

---

### 🔜 3.5 — K8s Services Deep Dive

**Why:**
Beyond basic ClusterIP/NodePort, different service types and patterns solve specific networking problems — headless services for StatefulSets, LoadBalancer for cloud, multi-port for complex apps.

**How & When:**

| Type | How | When |
|---|---|---|
| **ClusterIP** | Virtual IP, load balances across Pods | Default — pod-to-pod inside cluster |
| **NodePort** | Exposes port on every node (30000–32767) | Local dev, simple external access |
| **LoadBalancer** | Cloud provider creates external LB | Production on AWS/GCP/Azure |
| **Headless** | No ClusterIP, DNS returns Pod IPs directly | StatefulSets — direct Pod addressing |

**Headless Service** (for StatefulSets):
```yaml
spec:
  clusterIP: None   # Makes it headless
```
DNS then resolves `mongodb-0.mongodb-service` directly to that Pod's IP — needed so replicas can find each other.

**Best Practices:**
- Never use NodePort in production — use Ingress + ClusterIP instead
- Use LoadBalancer only on cloud — on bare metal use MetalLB or Ingress
- Headless Service is required for StatefulSet Pods to discover each other

---

## 🔜 Stage 4: Advanced User Topics

### 🔜 4.1 — Application Networking
- **Ingress** advanced — multi-path, TLS, default backend (planned for real-world project)
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
| Ingress — dashboard demo (basic) | ✅ Done | `phase-3-ingress/` |
| Ingress — real-world (multi-path, TLS, error page) | 🔜 Later | `phase-3-ingress/` |
| Helm | ✅ Done | `phase-4-helm/` |
| Persistent Volumes (overview) | ✅ Done | — |
| StatefulSet for MongoDB | 🔜 | `phase-5-volumes/` |
| Deployment Strategies | 🔜 | `phase-6-strategies/` |

### 📌 Real-World Ingress (Planned)

When we build the full real-world project, the Ingress will include:

- Multiple path routing — e.g. `/` → frontend, `/analytics` → analytics service
- Default backend — custom error page when no rule matches
- TLS/HTTPS setup — terminate SSL at Ingress with a certificate Secret

```yaml
# Planned structure for real-world Ingress
spec:
  tls:
    - hosts:
        - myapp.com
      secretName: myapp-tls-secret       # TLS cert stored as a K8s Secret
  rules:
    - host: myapp.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
          - path: /analytics
            pathType: Prefix
            backend:
              service:
                name: analytics-service
                port:
                  number: 80
  defaultBackend:                        # Catch-all for unmatched routes
    service:
      name: error-page-service
      port:
        number: 80
```

---

## 🛠️ Local Setup
```bash
# Prerequisites — Docker Desktop must be running
minikube start
kubectl get nodes   # Verify: STATUS=Ready

# Apply the demo project (order matters!)
kubectl apply -f mongo-secret.yaml       # Secret first!
kubectl apply -f mongo-configmap.yaml
kubectl apply -f 03-mongodb-pod.yaml
kubectl apply -f mongo-express.yaml

# Open Mongo Express in browser
minikube service mongo-express-service

# Ingress on Mac (Docker driver) — tunnel required!
minikube tunnel   # Run in a separate terminal and keep it open
# /etc/hosts entry: 127.0.0.1    dashboard.com
```