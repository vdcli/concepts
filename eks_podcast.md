# 🎙️ The Orchestration Station
## The EKS Podcast — Kubernetes on AWS Without the Chaos

> **Episode Runtime:** ~60 Minutes | **Format:** Conversational Two-Host
>
> **ALEX** — The developer: enthusiastic, asks great newbie questions, builds things hands-on.
> **PRIYA** — The architect: experienced, explains concepts with analogies, connects the dots.

---

## 📋 Speaker Guide

- **[ALEX]** — The developer voice. Asks the questions a curious engineer would ask.
- **[PRIYA]** — The architect voice. Explains with depth, analogies, and real-world context.
- `🎵` — Sound/music direction for the audio producer.
- `📝` — Production notes.

---

## Topics at a Glance

| Time | Topic | Key Concepts |
|------|-------|--------------|
| 0:00–7:00 | What Is EKS? | Kubernetes, managed control plane, EKS vs ECS |
| 7:00–16:00 | Kubernetes Primitives | Pods, Deployments, Services, ConfigMaps, Namespaces |
| 16:00–24:00 | Node Groups & Fargate | Managed node groups, Karpenter, Fargate profiles |
| 24:00–33:00 | Networking | VPC CNI, CoreDNS, Ingress, AWS Load Balancer Controller |
| 33:00–41:00 | IAM & Security | IRSA, OIDC, Pod identity, RBAC, network policies |
| 41:00–48:00 | Scaling | HPA, VPA, KEDA, Karpenter node autoscaling |
| 48:00–54:00 | Deployments & GitOps | Helm, Argo CD, rolling updates, canary |
| 54:00–58:00 | Observability | Fluent Bit, CloudWatch Container Insights, Prometheus |
| 58:00–62:00 | Best Practices & Project Ideas | Multi-tenant, cost, when to choose EKS |

---

# SEGMENT 1: Cold Open & What Is EKS? `(0:00 – 7:00)`

🎵 *Upbeat tech-themed intro music fades in, plays for 5 seconds, then fades under dialogue*

**[ALEX]:** Welcome to The Orchestration Station — where we tackle the most powerful, occasionally maddening, container orchestration technology in the world. I'm Alex.

**[PRIYA]:** And I'm Priya. Today: Amazon EKS — Elastic Kubernetes Service. If Kubernetes has ever felt like you're trying to fly a 747 just to deliver a pizza, this episode will help.

**[ALEX]:** Honest question — why Kubernetes? Why not just use ECS which we covered in a previous episode?

**[PRIYA]:** Totally valid question. Let me give you the honest answer. ECS is simpler, faster to get started with, and deeply integrated with AWS. Kubernetes — and EKS — is more complex but more powerful. You choose EKS when you need things ECS can't give you: portability across clouds, advanced scheduling, custom controllers, a massive open-source ecosystem, or when your organization already has Kubernetes expertise and tooling.

**[ALEX]:** So EKS is Kubernetes. But what does "managed" mean? What does AWS actually manage?

**[PRIYA]:** Great question. Kubernetes has two planes: the control plane and the data plane. The control plane is the brain — the API server, etcd (the configuration database), the scheduler, the controller manager. Running this reliably is genuinely hard. With EKS, AWS runs and manages the control plane entirely. It's multi-AZ, automatically patched, and 99.95% SLA backed.

**[ALEX]:** And the data plane is the nodes where my containers actually run?

**[PRIYA]:** Right. That's still yours to manage — unless you use Fargate. Think of it like a commercial kitchen. AWS provides the kitchen management system — the scheduling boards, the ordering system, the health and safety inspector. You still need to hire the chefs and buy the stoves — that's the data plane.

**[ALEX]:** Who uses EKS at scale?

**[PRIYA]:** Amazon itself uses Kubernetes internally. Spotify runs Kubernetes for their entire backend — millions of pods. Airbnb, Robinhood, Shopify — when you have hundreds of engineers and dozens of services, Kubernetes's primitives for multi-tenant cluster management and GitOps workflows become genuinely valuable.

🎵 *Short musical sting — segment transition*

---

# SEGMENT 2: Kubernetes Primitives — The Building Blocks `(7:00 – 16:00)`

**[PRIYA]:** You can't understand EKS without understanding Kubernetes primitives. Let's go through the key ones: Pods, Deployments, Services, ConfigMaps, and Namespaces.

**[ALEX]:** Start with Pods — the smallest unit.

**[PRIYA]:** A **Pod** is one or more containers that always run together on the same node and share a network namespace. Think of a Pod like an apartment — all the roommates share the same address and can talk to each other over localhost. The main container is the tenant, sidecar containers are like shared appliances.

**[ALEX]:** Pods are ephemeral though, right? They can die and get replaced.

**[PRIYA]:** Exactly. Never depend on a Pod's IP address or hostname. That's why we have **Deployments**. A Deployment declares "I want 3 replicas of this Pod spec." If a Pod dies, the Deployment controller creates a replacement. If you push a new image, the Deployment does a rolling update.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-api
  template:
    spec:
      containers:
      - name: api
        image: 123456789.dkr.ecr.us-east-1.amazonaws.com/my-api:v1.5.0
        resources:
          requests:
            cpu: "250m"
            memory: "256Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
```

**[ALEX]:** And Services? That's how you expose Pods?

**[PRIYA]:** A **Service** is a stable network endpoint for a set of Pods selected by labels. Think of it like a company's main phone number — behind that number are many employees (pods) who answer, and the phone system routes calls to whichever is available. The Service IP stays constant even as Pods come and go.

**[ALEX]:** Types of Services?

**[PRIYA]:** `ClusterIP` — internal only, accessible within the cluster. `NodePort` — exposes the service on a port on every node. `LoadBalancer` — provisions an AWS NLB or CLB. But for HTTP routing, you want **Ingress**.

**[ALEX]:** What's Ingress?

**[PRIYA]:** An Ingress is an HTTP routing layer. One ALB, many services. It routes `/api/users` to the users service, `/api/orders` to the orders service, based on path or hostname rules. Much more cost-efficient than one LoadBalancer Service per microservice.

**[ALEX]:** ConfigMaps and Secrets?

**[PRIYA]:** **ConfigMaps** store non-sensitive configuration — think app settings, feature flags, environment-specific config. **Secrets** store sensitive data — passwords, API keys, TLS certificates. They're injected into Pods as environment variables or mounted as files. On EKS, you encrypt Kubernetes Secrets with KMS using envelope encryption.

**[ALEX]:** And Namespaces?

**[PRIYA]:** Namespaces are virtual clusters within a cluster. Think of a large office building — HR, Engineering, and Finance all work in the same building but have their own floors with their own access controls. You can apply resource quotas, network policies, and RBAC permissions per namespace, enabling safe multi-tenant clusters.

🎵 *Transition music sting*

---

# SEGMENT 3: Node Groups & Fargate — Where Pods Actually Run `(16:00 – 24:00)`

**[ALEX]:** OK so Kubernetes schedules Pods. But on what machines?

**[PRIYA]:** That's the data plane. In EKS, you have three options. **Managed Node Groups** — ECS launches and manages EC2 Auto Scaling Groups on your behalf. You pick instance types, EKS handles the Kubernetes node lifecycle — draining nodes before termination, registering them with the cluster.

**[ALEX]:** Managed sounds better than unmanaged.

**[PRIYA]:** Almost always. Self-managed nodes are only worth it for very custom scenarios — custom AMIs, specific kernel modules, baremetal requirements. Managed node groups handle OS patching, node provisioning, and graceful drains automatically.

**[ALEX]:** What about Karpenter? I keep hearing that name.

**[PRIYA]:** Karpenter is an open-source cluster autoscaler built by AWS that's now the recommended approach. Traditional Cluster Autoscaler scales node groups — it's slow and coarse-grained. Karpenter watches for unschedulable pods and directly provisions the optimal EC2 instance type within seconds.

**[ALEX]:** Optimal how?

**[PRIYA]:** If you have a pod requesting 3.5 vCPU and 7GB memory, Karpenter finds the cheapest instance that fits — maybe an `m5.2xlarge` on Spot, or a smaller instance if one is available. It can consolidate idle nodes automatically, saving 40–60% on compute costs. Think of Karpenter as a smart hotel concierge — "you need a room for 3 people, we have exactly the right room type, here's the best price available right now."

**[ALEX]:** And EKS Fargate?

**[PRIYA]:** Fargate Profiles let certain Pods run on AWS-managed infrastructure. You define a profile with namespace and label selectors:

```yaml
# Pods in the 'batch' namespace with label 'compute: fargate' run on Fargate
fargateProfileName: batch-jobs
selectors:
  - namespace: batch
    labels:
      compute: fargate
```

Matching pods get scheduled on Fargate — no node management at all. Each pod gets its own isolated VM. Great for batch workloads and workloads with strict isolation requirements.

**[ALEX]:** What's the tradeoff with Fargate on EKS versus EC2 nodes?

**[PRIYA]:** Fargate pods start slower — 30 to 60 seconds versus a few seconds for a pod on an existing node. No DaemonSets on Fargate — things like node-level log collectors won't work. And Fargate pricing is higher per vCPU/memory than EC2 Spot. But zero node management. Most teams use EC2 managed node groups for steady-state workloads and Fargate for batch or burst.

🎵 *Transition music sting*

---

# SEGMENT 4: Networking — VPC CNI, Ingress & Service Mesh `(24:00 – 33:00)`

**[ALEX]:** Kubernetes networking is notoriously complex. What does EKS do with it?

**[PRIYA]:** EKS uses the **AWS VPC CNI plugin** — Container Network Interface. This is what makes EKS special compared to self-managed Kubernetes. Every pod gets a real VPC IP address from your subnet — not an overlay network IP.

**[ALEX]:** What's the difference?

**[PRIYA]:** With an overlay network, pods get private pod-space IPs that are tunneled through the node's real IP. Extra hops, extra latency, harder to debug. With VPC CNI, pods are first-class VPC citizens. Your pod IP `10.0.1.45` is directly routable within your VPC. RDS, ElastiCache, other EC2 instances — they all see your pod's real IP. Security group rules just work.

**[ALEX]:** The trade-off is that pods consume IP addresses from your VPC subnets.

**[PRIYA]:** Correct. This is important to plan. If you have 500 pods, that's 500 IP addresses. Plan your VPC subnets with this in mind — use larger CIDR blocks or enable VPC CNI custom networking to draw IPs from secondary CIDRs.

**[ALEX]:** Let's talk Ingress. You mentioned the AWS Load Balancer Controller.

**[PRIYA]:** The **AWS Load Balancer Controller** is a Kubernetes controller that watches for Ingress resources and provisions ALBs automatically. You annotate your Ingress with AWS-specific settings:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  rules:
  - host: api.myapp.com
    http:
      paths:
      - path: /users
        backend:
          service:
            name: users-service
            port:
              number: 8080
      - path: /orders
        backend:
          service:
            name: orders-service
            port:
              number: 8080
```

**[ALEX]:** One ALB, routing to multiple services based on path. Cost-efficient.

**[PRIYA]:** And the ALB is automatically updated when you change the Ingress spec. For service-to-service communication, you use Kubernetes DNS — `users-service.default.svc.cluster.local` resolves to the ClusterIP of the users service. CoreDNS handles this resolution.

**[ALEX]:** And for more advanced traffic management — canary deployments, circuit breaking?

**[PRIYA]:** That's where a service mesh comes in. AWS App Mesh with Envoy, or the popular open-source option Istio. They inject an Envoy sidecar into every pod and intercept all traffic, giving you observability, retries, circuit breakers, and fine-grained traffic splitting without touching your application code.

🎵 *Transition music sting*

---

# SEGMENT 5: IAM & Security — IRSA and RBAC `(33:00 – 41:00)`

**[ALEX]:** How does IAM work with Kubernetes? My pods need to access S3, DynamoDB...

**[PRIYA]:** This is the most important security topic in EKS. The solution is **IRSA — IAM Roles for Service Accounts**. Here's how it works. EKS clusters have an OIDC identity provider. Kubernetes service accounts can be annotated with an IAM role ARN. When a pod uses that service account, AWS STS issues temporary credentials for that IAM role.

**[ALEX]:** So each pod gets its own IAM role automatically.

**[PRIYA]:** Exactly. No node-level IAM roles that give all pods on a node the same permissions. No credentials in environment variables. True least privilege — this pod can only read S3 bucket `my-data`, that pod can only write to DynamoDB table `orders`.

```yaml
# Service Account with IRSA annotation
apiVersion: v1
kind: ServiceAccount
metadata:
  name: s3-reader
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789:role/s3-read-role

---
# Pod using that service account
spec:
  serviceAccountName: s3-reader
  containers:
  - name: app
    image: my-app:latest
```

**[ALEX]:** The AWS SDK automatically picks up the credentials?

**[PRIYA]:** Transparently. EKS injects a token via a projected volume, the AWS SDK finds it and exchanges it for temporary STS credentials. Your application code doesn't change at all. AWS is also rolling out **Pod Identity** — a simpler alternative to IRSA that doesn't require OIDC configuration.

**[ALEX]:** What about Kubernetes-level access control?

**[PRIYA]:** That's **RBAC — Role-Based Access Control**. In Kubernetes, you define Roles with rules about what API operations are allowed on which resources, then bind them to users or service accounts.

```yaml
# Role: read-only access to pods in the 'app' namespace
kind: Role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]

---
# Bind this role to the 'developer' group
kind: RoleBinding
subjects:
- kind: Group
  name: developer
roleRef:
  kind: Role
  name: pod-reader
```

**[ALEX]:** And AWS IAM users map to Kubernetes RBAC groups?

**[PRIYA]:** Via the `aws-auth` ConfigMap — or the newer Access Entries API. You map IAM roles or users to Kubernetes groups. An IAM role used by your developers gets mapped to the `developer` group, which has limited read-only RBAC access.

🎵 *Transition music sting*

---

# SEGMENT 6: Scaling — HPA, VPA & Karpenter `(41:00 – 48:00)`

**[ALEX]:** Let's talk scaling. In Kubernetes there are multiple types of autoscaling?

**[PRIYA]:** Three levels. **HPA** — Horizontal Pod Autoscaler — scales the number of pod replicas. **VPA** — Vertical Pod Autoscaler — adjusts pod CPU and memory requests. **Node autoscaling** — Karpenter — scales the number of nodes.

**[ALEX]:** Walk me through HPA.

**[PRIYA]:** HPA watches metrics and adjusts replicas to maintain a target. The classic is CPU:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-api
  minReplicas: 3
  maxReplicas: 50
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

**[ALEX]:** Same target-tracking concept as ECS.

**[PRIYA]:** Very similar. But Kubernetes HPA can scale on custom metrics too — requests per second from your application, queue depth from SQS, latency from Prometheus. This is where **KEDA** — Kubernetes Event-Driven Autoscaling — shines. It extends HPA to support dozens of event sources natively.

**[ALEX]:** So I could scale pods based on how many messages are in an SQS queue?

**[PRIYA]:** One YAML file:

```yaml
kind: ScaledObject
spec:
  scaleTargetRef:
    name: sqs-worker
  triggers:
  - type: aws-sqs-queue
    metadata:
      queueURL: https://sqs.us-east-1.amazonaws.com/123/my-queue
      queueLength: "5"    # Scale up when more than 5 messages per worker
```

Zero messages in the queue? Scale to zero. 1000 messages? Scale up aggressively. Extremely powerful for event-driven workloads.

**[ALEX]:** And VPA — vertical scaling?

**[PRIYA]:** VPA analyzes historical resource usage and recommends or automatically applies better CPU and memory requests. If your pod requests 512MB but always uses 200MB, VPA rightsizes it. Runs in recommendation-only mode safely — you review the suggestions before applying.

**[ALEX]:** And Karpenter ties it all together by ensuring there are nodes to schedule onto.

**[PRIYA]:** The complete picture: HPA says "I need 20 pods." Kubernetes tries to schedule them. 15 fit on existing nodes. 5 are pending. Karpenter sees 5 unschedulable pods, provisions the optimal instance in 30 seconds, and those pods start. Load drops, HPA scales to 8 pods. Karpenter consolidates to fewer nodes. The whole system breathes in and out automatically.

🎵 *Transition music sting*

---

# SEGMENT 7: Deployments & GitOps — Helm and Argo CD `(48:00 – 54:00)`

**[ALEX]:** How do you manage deploying dozens of Kubernetes YAML files across multiple environments?

**[PRIYA]:** **Helm** — the Kubernetes package manager. A Helm chart is a parameterized collection of Kubernetes manifests. Instead of maintaining separate YAMLs for dev, staging, and prod, you have one chart with `values.yaml` files per environment.

```bash
# Install a release from a chart with production values
helm upgrade --install my-api ./charts/my-api \
  --namespace production \
  --values ./values/production.yaml \
  --set image.tag=$GIT_SHA
```

**[ALEX]:** Like a templating system for Kubernetes.

**[PRIYA]:** Right. And the Kubernetes ecosystem publishes Helm charts for everything — `helm install prometheus prometheus-community/prometheus`. You can stand up an entire monitoring stack in minutes.

**[ALEX]:** What's GitOps and where does Argo CD fit in?

**[PRIYA]:** GitOps is a deployment philosophy: Git is the source of truth. Your cluster should always match what's in Git. **Argo CD** watches a Git repository and continuously syncs the cluster state to match it. You never `kubectl apply` manually in production — you push to Git, Argo CD detects the change and applies it.

```
Developer → git push → GitHub → Argo CD detects diff → applies to cluster
```

**[ALEX]:** So deployment is just a git commit.

**[PRIYA]:** And rollback is `git revert`. Every change is auditable — who changed what, when, why. Drift detection — if someone manually changes something in the cluster, Argo CD flags it and can auto-revert it. Think of Argo CD as a vigilant compliance officer who constantly compares the written policy (Git) with what's actually happening (cluster), and enforces alignment.

**[ALEX]:** What about canary deployments in Kubernetes?

**[PRIYA]:** Argo Rollouts — a companion to Argo CD — adds advanced deployment strategies. You deploy 5% of traffic to a new version, wait 10 minutes, check your error rate, then promote to 20%, 50%, 100%. Or automatically rollback if the error rate spikes above threshold. Fully automated progressive delivery.

🎵 *Transition music sting*

---

# SEGMENT 8: Observability — The Full Stack `(54:00 – 58:00)`

**[ALEX]:** How do you get visibility into an EKS cluster?

**[PRIYA]:** Three pillars: logs, metrics, and traces. For **logs**: Fluent Bit as a DaemonSet ships container logs to CloudWatch Logs or an OpenSearch cluster. CloudWatch Container Insights gives you cluster, node, pod, and container level metrics with pre-built dashboards.

**[ALEX]:** And for Prometheus-style metrics?

**[PRIYA]:** AWS Managed Prometheus — AMP — is a managed Prometheus-compatible service. Deploy the Prometheus Operator with a Helm chart, configure it to remote-write to AMP, and connect AWS Managed Grafana for dashboards. This is the standard cloud-native observability stack.

**[ALEX]:** What do you alert on?

**[PRIYA]:** Critical signals: pod crash loop back-off rate, pending pods older than 5 minutes, node not ready, high CPU throttling (means resource limits are too tight), PVC storage approaching full, HPA at max replicas (means you may need to raise the max).

**[ALEX]:** And tracing?

**[PRIYA]:** AWS X-Ray or OpenTelemetry. With a service mesh like Istio, you get distributed traces automatically without any SDK changes. Without a mesh, add the OTEL SDK to your application — it's a few lines of initialization code and the SDK auto-instruments HTTP and database calls.

🎵 *Closing music begins to fade in softly*

---

# SEGMENT 9: Wrap-Up, Best Practices & Project Ideas `(58:00 – 62:00)`

**[PRIYA]:** Best practices speed round — go.

**[ALEX]:** Hit me.

**[PRIYA]:** Always set resource requests and limits — unset limits let one pod consume an entire node. Use pod disruption budgets to ensure rolling upgrades don't take down all your pods simultaneously. Separate workloads into namespaces with resource quotas. Use multiple availability zones for your node groups. Upgrade EKS versions proactively — AWS supports each version for about 14 months.

**[ALEX]:** And the EKS vs ECS decision in one line?

**[PRIYA]:** ECS for simplicity and AWS-native teams. EKS for power users, multi-cloud, or organizations with existing Kubernetes investment.

**[ALEX]:** What should people build?

**[PRIYA]:** Four projects:

1. **Multi-Service App** — Deploy a frontend, API, and Postgres-backed service with Ingress routing, IRSA for S3 access, and Kubernetes Secrets encrypted with KMS.
2. **Event-Driven Worker** — SQS consumer scaled by KEDA — zero pods when the queue is empty, auto-scales under load.
3. **GitOps Pipeline** — Argo CD setup where pushing to a Git repo automatically deploys to your cluster with Helm charts.
4. **Canary Deployment** — Argo Rollouts progressive delivery with automatic rollback on error rate spike.

**[ALEX]:** That is a comprehensive education in EKS. We covered Kubernetes primitives, node groups, Karpenter, VPC CNI, IRSA, HPA and KEDA, GitOps with Argo CD, and observability.

**[PRIYA]:** All the manifests, Helm charts, and Argo CD configs from today are in the show notes. Kubernetes rewards patience — keep building.

**[ALEX]:** Until next time —

**[PRIYA]:** Stay orchestrated. ☸️

🎵 *Outro music swells and fades out over 5 seconds*

---

## 🎯 Key Analogies Reference Card

| Concept | Analogy |
|---------|---------|
| EKS Control Plane | AWS manages the kitchen management system — you manage the chefs |
| Pod | Apartment — roommates share the same address, talk over localhost |
| Deployment | Property manager — ensures desired number of occupied apartments at all times |
| Service | Company phone number — stable even as the employees behind it change |
| Ingress | Building reception — routes visitors to the right floor/department |
| Namespace | Floors in an office building — shared building, separate access controls |
| IRSA | Employee ID badge for each pod — specific access to specific systems only |
| RBAC | Office security policy — who can open which doors in the building |
| HPA | HVAC thermostat — maintain target temperature by adding/removing units |
| KEDA | Demand-driven staffing — hire workers as orders come in, let go when idle |
| Karpenter | Smart hotel concierge — find exactly the right room at the best price |
| Helm | Recipe book — parameterized instructions reused across environments |
| Argo CD | Compliance officer — constantly checks that reality matches policy (Git) |
| VPC CNI | Pods are residents, not guests — they get real VPC citizenship |

---

## 📚 Quick Reference

```bash
# ── CLUSTER SETUP ────────────────────────────────────
eksctl create cluster \
  --name my-prod-cluster \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type m5.xlarge \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 10

aws eks update-kubeconfig --name my-prod-cluster --region us-east-1

# ── KUBECTL BASICS ───────────────────────────────────
kubectl get pods -n my-namespace
kubectl get pods -A                        # All namespaces
kubectl describe pod <pod-name>
kubectl logs <pod-name> -f                 # Follow logs
kubectl exec -it <pod-name> -- /bin/sh    # Shell into pod
kubectl top pods                           # CPU/memory usage

# ── DEPLOYMENTS ──────────────────────────────────────
kubectl apply -f deployment.yaml
kubectl rollout status deployment/my-api
kubectl rollout history deployment/my-api
kubectl rollout undo deployment/my-api     # Rollback

# ── SCALING ──────────────────────────────────────────
kubectl scale deployment my-api --replicas=5
kubectl get hpa

# ── HELM ─────────────────────────────────────────────
helm repo add stable https://charts.helm.sh/stable
helm search repo nginx
helm install my-release ./my-chart --values values-prod.yaml
helm upgrade my-release ./my-chart --set image.tag=v2.0.0
helm list
helm rollback my-release 1

# ── NAMESPACES & RBAC ────────────────────────────────
kubectl create namespace staging
kubectl config set-context --current --namespace=staging
kubectl auth can-i list pods --namespace=production

# ── SECRETS & CONFIGMAPS ─────────────────────────────
kubectl create secret generic db-creds \
  --from-literal=password=supersecret
kubectl create configmap app-config \
  --from-file=config.properties

# ── NODE MANAGEMENT ──────────────────────────────────
kubectl get nodes -o wide
kubectl describe node <node-name>
kubectl cordon <node-name>    # Mark unschedulable
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

# ── IRSA SETUP ───────────────────────────────────────
eksctl create iamserviceaccount \
  --cluster my-prod-cluster \
  --namespace default \
  --name s3-reader-sa \
  --attach-policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess \
  --approve
```

---

*The Orchestration Station is produced independently. All trademarks belong to their respective owners.*

---

## 📖 Glossary — Plain English Definitions

Every term used in this episode, explained without jargon.

---

### Core Concepts

**Container**
A lightweight, self-contained package that holds your application code plus everything it needs to run (libraries, settings, runtime). Think of it like a lunchbox — everything needed for the meal is packed inside, and it works the same whether you open it at home or at the office.

**Kubernetes**
An open-source system that automatically manages where and how containers run across many machines. It handles starting containers, restarting them if they crash, and scaling them up or down based on demand. The name comes from the Greek word for "helmsman" or "pilot."

**EKS (Elastic Kubernetes Service)**
Amazon's managed version of Kubernetes. AWS runs the complex brain of Kubernetes for you, so you don't have to set it up or maintain it yourself.

**ECS (Elastic Container Service)**
Amazon's simpler, AWS-only alternative to Kubernetes for running containers. Easier to learn but less powerful and not portable to other clouds.

**Cluster**
A group of machines (nodes) that Kubernetes manages as one unit. Your applications run somewhere inside the cluster.

**Control Plane**
The "brain" of a Kubernetes cluster — the part that makes decisions about where to run things, tracks what's running, and responds to changes. With EKS, AWS manages this for you.

**Data Plane**
The "muscle" of a Kubernetes cluster — the actual machines (nodes) where your containers run. You manage this part (unless you use Fargate).

**etcd**
The database Kubernetes uses internally to store all its configuration and state. You never interact with it directly, but it's critical — if etcd goes down, the cluster can't make decisions.

---

### Kubernetes Building Blocks

**Pod**
The smallest unit in Kubernetes. A pod contains one or more containers that always run together on the same machine and share a network connection. Pods are temporary — they can be stopped and replaced at any time.

**Deployment**
A Kubernetes instruction that says "keep N copies of this pod running at all times." If a pod crashes, the Deployment automatically starts a replacement. If you update your app, it handles the rollout.

**Service**
A stable network address that points to a group of pods. Because pods come and go, a Service gives you a fixed address that always works, routing traffic to whichever pods are currently healthy.

**Ingress**
An HTTP traffic router that sits in front of multiple services. One Ingress can route `/api/users` to the users service and `/api/orders` to the orders service, saving you from needing a separate load balancer for every service.

**ConfigMap**
A Kubernetes object for storing non-sensitive configuration — things like app settings, environment names, or feature flags. Pods read these values without them being baked into the container image.

**Secret**
Like a ConfigMap, but for sensitive data — passwords, API keys, certificates. The data is stored encoded and can be encrypted at rest.

**Namespace**
A way to divide one Kubernetes cluster into separate sections. Teams or environments (dev, staging, prod) can each have their own namespace with their own access controls and resource limits.

**DaemonSet**
A Kubernetes object that ensures one copy of a pod runs on every node in the cluster. Used for things like log collectors or monitoring agents that need to run everywhere.

**Pod Disruption Budget (PDB)**
A rule that tells Kubernetes "never take down more than X pods of this type at the same time." Protects your app from going fully offline during upgrades or node replacements.

**Service Account**
A Kubernetes identity assigned to a pod. Used to control what the pod is allowed to do — similar to a user account, but for applications instead of people.

---

### Compute & Nodes

**Node**
A single machine (virtual or physical) in a Kubernetes cluster where pods actually run. A cluster typically has many nodes.

**Managed Node Group**
A set of EC2 machines that AWS creates and maintains for you as Kubernetes nodes. AWS handles OS updates, node registration, and graceful shutdowns.

**Fargate**
An AWS service where you run containers without managing any servers at all. You just define the pod, and AWS figures out where to run it. Each pod gets its own isolated compute environment.

**EC2 (Elastic Compute Cloud)**
Amazon's virtual machine service. Kubernetes nodes are typically EC2 instances.

**Spot Instance**
A spare EC2 machine rented at a big discount (often 60–90% cheaper) that AWS can reclaim with 2 minutes' notice. Good for workloads that can handle interruption.

**Karpenter**
An AWS-built tool that automatically adds or removes nodes from your cluster based on what pods need. Much faster and smarter than the older Cluster Autoscaler — it picks the exact right machine type for the workload.

---

### Networking

**VPC (Virtual Private Cloud)**
Your private, isolated section of the AWS network. All your AWS resources (EC2, RDS, EKS nodes, etc.) live inside a VPC.

**Subnet**
A smaller section of a VPC. Resources in a subnet share a range of IP addresses. Public subnets can reach the internet; private subnets cannot.

**CIDR**
A notation for describing a range of IP addresses (e.g., `10.0.0.0/16` means "all addresses from 10.0.0.0 to 10.0.255.255"). Used when planning how many IP addresses a network has available.

**CNI (Container Network Interface)**
A plugin system that controls how containers get network connections. It's the standard way Kubernetes assigns IP addresses to pods.

**VPC CNI**
The AWS-specific CNI plugin used by EKS. It gives each pod a real IP address from your VPC (instead of a fake internal address), so pods can talk directly to other AWS services like RDS or ElastiCache.

**CoreDNS**
The internal DNS server inside every Kubernetes cluster. It translates service names like `users-service.default.svc.cluster.local` into the IP address of the right service.

**ALB (Application Load Balancer)**
An AWS load balancer that works at the HTTP/HTTPS level. It can route requests to different targets based on URL path or hostname.

**NLB (Network Load Balancer)**
An AWS load balancer that works at the TCP/UDP level — faster and lower-latency than an ALB, but with no HTTP-level routing.

**Service Mesh**
A layer of infrastructure that controls all network traffic between your services. Tools like Istio inject a small proxy (Envoy) into every pod that handles retries, circuit breaking, encryption, and traffic splitting automatically.

**Envoy**
A high-performance network proxy used by service meshes (like Istio) to intercept and manage traffic between pods.

**Istio**
A popular open-source service mesh for Kubernetes. Adds traffic management, observability, and security between services without changing application code.

---

### Security & Identity

**IAM (Identity and Access Management)**
The AWS system for controlling who (or what) can access which AWS resources. Every API call to AWS is checked against IAM permissions.

**IRSA (IAM Roles for Service Accounts)**
A mechanism that lets individual Kubernetes pods assume specific AWS IAM roles. This means each pod gets only the AWS permissions it needs — not the permissions of the entire machine it runs on.

**OIDC (OpenID Connect)**
An identity protocol that lets one system verify who you are via a trusted third party. EKS uses OIDC so that Kubernetes service accounts can prove their identity to AWS and receive temporary IAM credentials.

**STS (Security Token Service)**
The AWS service that issues temporary, short-lived credentials. IRSA uses STS under the hood — instead of storing permanent AWS keys, pods get temporary credentials that expire automatically.

**Pod Identity**
A newer, simpler AWS alternative to IRSA for giving pods IAM roles. It achieves the same result but with less setup (no OIDC configuration required).

**RBAC (Role-Based Access Control)**
The Kubernetes system for controlling who can do what inside the cluster. You define Roles (a set of allowed actions) and bind them to users or service accounts.

**Role / ClusterRole**
A Kubernetes RBAC object listing permitted actions (e.g., "can read pods, cannot delete deployments"). A Role applies within one namespace; a ClusterRole applies across the entire cluster.

**KMS (Key Management Service)**
AWS's service for creating and managing encryption keys. Used in EKS to encrypt Kubernetes Secrets at rest.

**aws-auth ConfigMap**
A Kubernetes configuration file that maps AWS IAM users and roles to Kubernetes RBAC groups. This is how AWS credentials translate into Kubernetes permissions.

---

### Scaling

**HPA (Horizontal Pod Autoscaler)**
A Kubernetes feature that automatically increases or decreases the number of pod replicas based on metrics like CPU usage or requests per second.

**VPA (Vertical Pod Autoscaler)**
A Kubernetes feature that automatically adjusts how much CPU and memory a pod requests, based on observed usage. Instead of more pods, it makes each pod the right size.

**KEDA (Kubernetes Event-Driven Autoscaling)**
An extension to HPA that can scale pods based on external event sources — like the number of messages in an SQS queue, a Kafka topic, or a database row count. Can scale all the way to zero.

**Cluster Autoscaler**
The older tool for automatically adding/removing nodes. Still works, but Karpenter is now preferred for EKS because it's faster and smarter about instance selection.

**SQS (Simple Queue Service)**
AWS's managed message queue. One service puts messages in, another service reads and processes them. Often used to decouple parts of an application.

---

### Deployments & GitOps

**YAML**
A human-readable text format used to write Kubernetes configuration files. Uses indentation (spaces) to show structure. All Kubernetes objects — pods, deployments, services — are defined in YAML.

**Helm**
A package manager for Kubernetes. A Helm "chart" is a reusable, parameterized bundle of Kubernetes YAML files. You can install complex applications (like Prometheus) with a single command, and manage different configurations (dev vs prod) with separate values files.

**GitOps**
A way of managing deployments where Git is the single source of truth. Changes to the cluster are made by committing to a Git repository, not by running commands directly. This gives you a full audit trail and easy rollbacks.

**Argo CD**
A GitOps tool that watches a Git repository and automatically keeps your Kubernetes cluster in sync with it. If the cluster drifts from what's in Git, Argo CD flags it or fixes it automatically.

**Argo Rollouts**
A companion to Argo CD that adds advanced deployment strategies like canary releases and blue-green deployments to Kubernetes.

**Rolling Update**
A deployment strategy where new pods replace old ones gradually — a few at a time — so the application stays available throughout the update.

**Canary Deployment**
A deployment strategy where a small percentage of traffic (e.g., 5%) is sent to the new version first. If no problems appear, traffic is gradually shifted over. If errors spike, the rollout is stopped or reversed.

**kubectl**
The command-line tool for talking to a Kubernetes cluster. You use it to apply configs, check pod status, read logs, and manage resources.

**eksctl**
A command-line tool specifically for creating and managing EKS clusters. Makes cluster setup much simpler than doing it manually through AWS.

---

### Observability

**Fluent Bit**
A lightweight log collector. Runs as a DaemonSet on every node, collects container logs, and ships them to a destination like CloudWatch or OpenSearch.

**CloudWatch**
AWS's monitoring and logging service. Stores logs, metrics, and alarms. CloudWatch Container Insights provides pre-built dashboards for EKS cluster health.

**Prometheus**
An open-source monitoring system that collects and stores metrics from your applications and infrastructure. Very commonly used with Kubernetes.

**Grafana**
An open-source dashboard tool for visualizing metrics. Usually connected to Prometheus to create charts and alerts.

**AMP (Amazon Managed Prometheus)**
AWS's fully managed, Prometheus-compatible metrics service. You get Prometheus without running the Prometheus server yourself.

**OpenTelemetry (OTEL)**
An open standard for collecting logs, metrics, and traces from applications. Supported by most observability tools — vendor-neutral.

**Distributed Tracing**
Tracking a single request as it travels through multiple services in your application. Lets you see exactly where time is spent and where errors occur.

**AWS X-Ray**
Amazon's distributed tracing service. Instruments your application to show how requests flow through your services.

**Crash Loop Backoff**
A Kubernetes state where a pod keeps crashing and restarting repeatedly. Kubernetes slows down the restarts over time (backs off) rather than hammering the system. Usually means there's a bug or misconfiguration in the app.

---

### AWS Services Referenced

**S3 (Simple Storage Service)**
AWS's object storage service. Used to store files, images, backups, and datasets.

**DynamoDB**
AWS's fully managed NoSQL database. Fast, scalable, and serverless.

**RDS (Relational Database Service)**
AWS's managed relational database service (supports MySQL, PostgreSQL, etc.).

**ElastiCache**
AWS's managed in-memory caching service (supports Redis and Memcached).

**ECR (Elastic Container Registry)**
AWS's private container image registry. You push your Docker images here, and Kubernetes pulls them from here when starting pods.
