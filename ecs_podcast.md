# 🎙️ The Container Ship
## The ECS Podcast — Running Containers at Scale on AWS

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
| 0:00–7:00 | What Is ECS? | Containers, Docker, ECS vs EC2, control plane |
| 7:00–15:00 | Core Concepts | Clusters, Task Definitions, Tasks, Services |
| 15:00–24:00 | Launch Types | EC2 vs Fargate, when to use each, pricing model |
| 24:00–33:00 | Networking | VPCs, security groups, service discovery, load balancing |
| 33:00–41:00 | Deployments | Rolling updates, blue/green, CodeDeploy, health checks |
| 41:00–48:00 | Auto Scaling | Service Auto Scaling, capacity providers, target tracking |
| 48:00–54:00 | IAM & Security | Task roles, execution roles, secrets, least privilege |
| 54:00–58:00 | Observability | CloudWatch, Container Insights, logging drivers |
| 58:00–62:00 | Best Practices & Projects | Cost optimization, project ideas, ECS vs EKS decision |

---

# SEGMENT 1: Cold Open & What Is ECS? `(0:00 – 7:00)`

🎵 *Upbeat tech-themed intro music fades in, plays for 5 seconds, then fades under dialogue*

**[ALEX]:** Welcome to The Container Ship — where we take AWS services apart piece by piece until they actually make sense. I'm Alex.

**[PRIYA]:** And I'm Priya. Today we are doing a full deep dive into Amazon ECS — Elastic Container Service. If you've ever wondered how big companies run Docker containers at massive scale without losing their minds, this is the episode.

**[ALEX]:** I know Docker. I can write a Dockerfile, build an image, run a container locally. But then what? How do you go from "it works on my machine" to "it runs reliably in production"?

**[PRIYA]:** That is the exact problem ECS solves. Let me set up the mental model. Imagine you're a shipping company. You have cargo — that's your application. You've put it in a standardized shipping container — that's Docker. Now you need someone to manage the fleet of ships, figure out which ship carries which container, make sure containers are restarted if they fall overboard, and scale up when demand spikes.

**[ALEX]:** And ECS is the shipping company management system.

**[PRIYA]:** ECS is the port authority. It knows about every ship, every container slot, where everything is, and it orchestrates the whole operation. The technical term is container orchestration.

**[ALEX]:** So ECS is AWS's answer to Kubernetes?

**[PRIYA]:** Yes and no. They solve the same problem — running containers at scale. But ECS is AWS-native, deeply integrated with other AWS services, and has a significantly simpler learning curve. Kubernetes is more powerful and portable but the complexity is real. We'll do an EKS episode for that.

**[ALEX]:** So ECS is the right choice if you're already deep in the AWS ecosystem?

**[PRIYA]:** Exactly. If your company uses ALB, CloudWatch, IAM, Route 53 — ECS just plugs into all of it seamlessly. And AWS manages the control plane entirely. You never patch a Kubernetes API server at 2 AM.

**[ALEX]:** What kinds of workloads is ECS designed for?

**[PRIYA]:** Web APIs and microservices, background job workers, batch processing pipelines, ML inference endpoints, scheduled tasks. Basically any containerized workload. Netflix, Samsung, Duolingo — all use ECS for production workloads.

🎵 *Short musical sting — segment transition*

---

# SEGMENT 2: Core Concepts — The ECS Mental Model `(7:00 – 15:00)`

**[PRIYA]:** OK, there are four core concepts in ECS you need to understand before anything else clicks. Clusters, Task Definitions, Tasks, and Services.

**[ALEX]:** Let's go one by one.

**[PRIYA]:** A **Cluster** is the logical boundary for your ECS resources. Think of it as the shipping port itself — a named space where containers will eventually run. The cluster is also where you register compute capacity — either EC2 instances or Fargate capacity.

**[ALEX]:** So a cluster is just a grouping.

**[PRIYA]:** Right. One cluster per environment is a common pattern — `my-app-prod`, `my-app-staging`. Or one cluster per team. Next: **Task Definition**. This is a blueprint. It's a JSON document that describes how your container should run.

**[ALEX]:** Like a docker-compose file?

**[PRIYA]:** Very similar analogy! It defines the Docker image to use, how much CPU and memory to allocate, what ports to expose, environment variables, which IAM role to use, logging configuration. Think of it like a recipe card — "to make this dish you need these ingredients in these quantities, cook at this temperature."

```json
{
  "family": "my-web-app",
  "containerDefinitions": [
    {
      "name": "web",
      "image": "123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:latest",
      "cpu": 256,
      "memory": 512,
      "portMappings": [{"containerPort": 8080}],
      "environment": [{"name": "ENV", "value": "production"}]
    }
  ]
}
```

**[ALEX]:** And a Task?

**[PRIYA]:** A **Task** is a running instance of a Task Definition. If the Task Definition is the recipe, the Task is the actual dish being cooked in the kitchen right now. One task runs one or more containers together — similar to a Kubernetes Pod.

**[ALEX]:** So if I run three copies of my API, that's three Tasks all from the same Task Definition.

**[PRIYA]:** Exactly. And finally — **Services**. A Service is what ensures a desired number of Tasks are always running. You tell it "I want 3 tasks of this task definition running at all times." The service controller constantly reconciles reality with that desired state.

**[ALEX]:** So if one task crashes, the service restarts it.

**[PRIYA]:** Immediately. Like a night watchman who keeps counting — "I need 3 guards posted. One fell asleep. I'll wake them up or find a replacement." The Service is always counting, always reconciling.

**[ALEX]:** And services also handle load balancing?

**[PRIYA]:** Services integrate directly with ALB — Application Load Balancer. When a new task starts, it's automatically registered as a target. When a task stops, it's deregistered. Your traffic is always routed to healthy, running tasks.

🎵 *Transition music sting*

---

# SEGMENT 3: Launch Types — EC2 vs Fargate `(15:00 – 24:00)`

**[ALEX]:** OK so I have tasks and services. But where do the containers actually run?

**[PRIYA]:** This is where the two launch types come in. With **EC2 launch type**, you provision and manage a fleet of EC2 instances that join your cluster. ECS places containers on those instances based on available CPU and memory. You're responsible for the instances — patching, scaling the fleet, choosing instance types.

**[ALEX]:** That sounds like a lot of work.

**[PRIYA]:** It is, but you have full control. Think of it like owning a fleet of trucks. You decide how many, what size, where they're parked. Maximum flexibility, maximum responsibility.

**[ALEX]:** And Fargate?

**[PRIYA]:** **Fargate** is serverless containers. You define the CPU and memory your task needs, and AWS runs it — you never see or touch an EC2 instance. Think of it like hiring a logistics company. You say "I need to ship this package, it weighs 2kg, here it is." They handle everything else — which truck, which driver, what route.

**[ALEX]:** So with Fargate I just pay for the resources my tasks use?

**[PRIYA]:** Exactly. You're billed per vCPU and GB of memory per second of task runtime. No idle EC2 costs. Perfect for workloads that are variable or if you're a small team that doesn't want to manage infrastructure.

**[ALEX]:** When would you choose EC2 over Fargate?

**[PRIYA]:** Great question. EC2 wins when you need GPU instances — for ML workloads. When you need specific instance types — like memory-optimized for in-memory databases. When you have steady, predictable workloads and can use Reserved Instances to save 60–70%. And when you need tasks to mount EBS volumes directly.

**[ALEX]:** So Fargate for variable or spiky workloads, EC2 for steady state at high scale.

**[PRIYA]:** That's the rule of thumb. Many teams start with Fargate for simplicity and migrate hot paths to EC2 later when they understand their traffic patterns. There's also Fargate Spot — tasks that can be interrupted but cost up to 70% less. Perfect for batch jobs and non-critical workers.

**[ALEX]:** What about the resource allocation for Fargate? Is it flexible?

**[PRIYA]:** Fargate has predefined CPU/memory combinations. The smallest is 0.25 vCPU and 512MB memory. You go up in increments — 0.5, 1, 2, 4, 8, 16 vCPU. The memory range grows with the CPU selection. Important: you can't choose 0.3 vCPU — it's fixed tiers.

🎵 *Transition music sting*

---

# SEGMENT 4: Networking — VPCs, Load Balancers & Service Discovery `(24:00 – 33:00)`

**[ALEX]:** Let's talk networking. Containers running in a VPC — how does that work?

**[PRIYA]:** In Fargate, every task gets its own Elastic Network Interface — an ENI. That means every task gets its own private IP address within your VPC. This is the `awsvpc` network mode, and it's the default for Fargate.

**[ALEX]:** So my container is a first-class citizen in the VPC, just like an EC2 instance.

**[PRIYA]:** Exactly. Which means you can attach security groups directly to tasks. If your API task needs to talk to RDS, you create a security group rule: "allow inbound 5432 from the API task security group." No CIDR ranges, no IP management — just security group references.

**[ALEX]:** That's clean. How does traffic reach the tasks from the internet?

**[PRIYA]:** Through an Application Load Balancer. The ALB sits in public subnets, accepts internet traffic, and routes it to your tasks in private subnets. You create a Target Group, your ECS Service registers tasks into it automatically, and the ALB distributes traffic.

```
Internet → ALB (public subnet) → Target Group → ECS Tasks (private subnet)
```

**[ALEX]:** And health checks?

**[PRIYA]:** The ALB runs health checks against each task — typically hitting a `/health` endpoint. If a task fails health checks, the ALB stops sending it traffic. ECS also sees the failing task and replaces it. Two layers of protection.

**[ALEX]:** What about service-to-service communication? Like my API calling my auth service?

**[PRIYA]:** You have two good options. First: **AWS Cloud Map** — ECS Service Discovery. You register services with Cloud Map and they get DNS names like `auth.my-namespace.local`. Your API container resolves that DNS name and gets the IPs of running auth tasks. Round-robin load balancing at the DNS level.

**[ALEX]:** So `http://auth.my-namespace.local:8080` works from any container in the VPC.

**[PRIYA]:** Exactly. Second option is **ECS Service Connect** — a newer feature that builds on Cloud Map but adds traffic management, metrics, and connection pooling. It injects a sidecar proxy into each task transparently.

**[ALEX]:** Like a built-in service mesh?

**[PRIYA]:** A lightweight one, yes. For a full service mesh with circuit breaking and advanced traffic policies, you'd use App Mesh with Envoy proxies. But Service Connect handles 80% of use cases without that complexity.

🎵 *Transition music sting*

---

# SEGMENT 5: Deployments — Rolling Updates, Blue/Green & Health Checks `(33:00 – 41:00)`

**[ALEX]:** Let's talk about deployments. I have a new version of my image. How do I update my service with zero downtime?

**[PRIYA]:** ECS supports **rolling updates** by default. When you update a service — usually by pushing a new image tag and updating the task definition — ECS replaces tasks gradually. It starts new tasks with the new version, waits for them to be healthy, then stops old ones.

**[ALEX]:** How does it decide the pace?

**[PRIYA]:** Two parameters control this. **Minimum Healthy Percent** — the percentage of your desired tasks that must be running at any time during the update. Set to 100% and ECS must spin up new tasks before stopping old ones. **Maximum Percent** — how many extra tasks can run simultaneously. Set to 200% and ECS doubles your fleet during the transition.

**[ALEX]:** So with minimum 100% and maximum 200% on a service of 4 tasks, I'd temporarily have 8 tasks running during the update?

**[PRIYA]:** Exactly. That's the zero-downtime approach — you always have at least 4 healthy tasks serving traffic. A common production setting is minimum 50%, maximum 200%, which is more cost-efficient — you replace half the fleet at a time.

**[ALEX]:** What about blue/green deployments?

**[PRIYA]:** Blue/green with ECS uses CodeDeploy. You run two sets of tasks — the blue environment is currently live, the green environment has the new version. Traffic is shifted from blue to green, gradually or all at once, using weighted target groups on your ALB.

**[ALEX]:** And if something goes wrong with the green environment?

**[PRIYA]:** CodeDeploy automatically rolls back — it shifts traffic back to blue within seconds. This is the real power of blue/green: instant rollback. With rolling updates, rollback means re-deploying the old version, which takes minutes.

**[ALEX]:** How do I trigger a deployment in practice?

**[PRIYA]:** You update the task definition — typically by pushing a new image to ECR and registering a new task definition revision. Then update the service with that new revision. The full CI/CD flow looks like:

```bash
# Build and push image
docker build -t my-app:$GIT_SHA .
docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:$GIT_SHA

# Register new task definition revision
aws ecs register-task-definition --family my-web-app --container-definitions ...

# Update service
aws ecs update-service \
  --cluster my-prod-cluster \
  --service my-web-service \
  --task-definition my-web-app:42
```

**[ALEX]:** And this integrates with GitHub Actions, CodePipeline, all of that?

**[PRIYA]:** Seamlessly. AWS has a GitHub Actions action — `aws-actions/amazon-ecs-deploy-task-definition` — that handles the whole flow. Push to main, action triggers, new image deployed.

🎵 *Transition music sting*

---

# SEGMENT 6: Auto Scaling — Handling Traffic Spikes `(41:00 – 48:00)`

**[ALEX]:** What happens when traffic spikes? How does ECS scale?

**[PRIYA]:** Two levels of scaling. First: **Service Auto Scaling** — scaling the number of tasks. Second: if you're on EC2 launch type, **Cluster Auto Scaling** — scaling the EC2 instances. Fargate removes that second concern entirely.

**[ALEX]:** Let's start with Service Auto Scaling.

**[PRIYA]:** You set a minimum and maximum number of tasks for your service. Then you define scaling policies. The most common is **Target Tracking** — you pick a metric and a target value, and ECS adjusts tasks to maintain that target.

**[ALEX]:** Like "keep CPU at 70%."

**[PRIYA]:** Exactly. If CPU creeps above 70%, ECS scales out — adds tasks. If it drops well below, ECS scales in. The math is automatic. Think of it like a thermostat — you set 70 degrees, the HVAC system adds or removes heat to maintain that temperature.

```bash
aws application-autoscaling put-scaling-policy \
  --policy-name cpu-target-tracking \
  --resource-id service/my-cluster/my-service \
  --scalable-dimension ecs:service:DesiredCount \
  --policy-type TargetTrackingScaling \
  --target-tracking-scaling-policy-configuration '{
    "TargetValue": 70.0,
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ECSServiceAverageCPUUtilization"
    },
    "ScaleOutCooldown": 60,
    "ScaleInCooldown": 300
  }'
```

**[ALEX]:** Why is the scale-in cooldown longer than scale-out?

**[PRIYA]:** Asymmetry is intentional. You want to scale out fast when traffic spikes — customer experience suffers immediately if you're under-provisioned. But you want to scale in slowly — traffic can spike again, and if you scale in too aggressively you'll be yo-yoing tasks up and down constantly.

**[ALEX]:** What about Capacity Providers?

**[PRIYA]:** Capacity Providers are a newer, more powerful abstraction. Instead of managing scaling manually, you define a Capacity Provider with target capacity — say 80% of your EC2 fleet should be utilized. ECS manages both task placement and EC2 fleet size together, automatically. It's like having a smart building manager who adjusts both office assignments AND building size simultaneously.

**[ALEX]:** And for Fargate you don't need Capacity Providers for compute scaling.

**[PRIYA]:** Right. Fargate scales instantly with your task count — there are no servers to provision. Your only limit is the AWS service quota for running tasks in your account, which you can request increases for.

🎵 *Transition music sting*

---

# SEGMENT 7: IAM & Security — Doing It Right `(48:00 – 54:00)`

**[ALEX]:** Let's talk security. My containers need to access DynamoDB, S3, Secrets Manager. How does that work without hardcoding credentials?

**[PRIYA]:** This is critical to get right. ECS has two distinct IAM roles and they're often confused. First: the **Task Execution Role** — this is the role that ECS itself uses to launch your task. It needs permissions to pull your image from ECR, write logs to CloudWatch, and fetch secrets from Secrets Manager or SSM Parameter Store.

**[ALEX]:** So the execution role is for ECS, not for my application code.

**[PRIYA]:** Exactly. Second: the **Task Role** — this is what your application code uses. When your Node.js or Python app calls `AWS.DynamoDB()`, it automatically gets credentials from the task role. No access keys, no secrets in environment variables.

**[ALEX]:** How does the app get those credentials?

**[PRIYA]:** ECS injects temporary credentials into the container via a metadata endpoint — a local HTTP endpoint only accessible from inside the container. The AWS SDK automatically queries this endpoint. It's transparent to your code.

```json
// Task Definition with both roles
{
  "taskRoleArn": "arn:aws:iam::123456789:role/my-app-task-role",
  "executionRoleArn": "arn:aws:iam::123456789:role/ecs-task-execution-role",
  ...
}
```

**[ALEX]:** And secrets? Database passwords, API keys?

**[PRIYA]:** Never put secrets in environment variables as plain text. Store them in AWS Secrets Manager or SSM Parameter Store. Then reference them in your task definition:

```json
"secrets": [
  {
    "name": "DB_PASSWORD",
    "valueFrom": "arn:aws:secretsmanager:us-east-1:123456789:secret:prod/db-password"
  }
]
```

ECS fetches the secret at task launch time and injects it as an environment variable. Your app sees it as a normal env var but it was never in your code or config files.

**[ALEX]:** And secrets are encrypted at rest in Secrets Manager.

**[PRIYA]:** With KMS encryption. And you can rotate them automatically — Secrets Manager can rotate database passwords on a schedule. For network security, always run tasks in private subnets. Your tasks should never have a direct route to the internet — use NAT Gateway or VPC Endpoints for outbound calls to AWS services.

🎵 *Transition music sting*

---

# SEGMENT 8: Observability — Logs, Metrics & Container Insights `(54:00 – 58:00)`

**[ALEX]:** How do I know what's happening inside my containers?

**[PRIYA]:** ECS integrates with CloudWatch for everything. For logs, configure the `awslogs` log driver in your task definition:

```json
"logConfiguration": {
  "logDriver": "awslogs",
  "options": {
    "awslogs-group": "/ecs/my-web-app",
    "awslogs-region": "us-east-1",
    "awslogs-stream-prefix": "ecs"
  }
}
```

Every line your container writes to stdout or stderr goes to CloudWatch Logs automatically.

**[ALEX]:** And for metrics?

**[PRIYA]:** Enable **Container Insights** on your cluster. This deploys a CloudWatch agent as a daemon task that collects CPU, memory, network, and disk metrics at the cluster, service, and task level. You get pre-built dashboards and you can set alarms on service CPU exceeding 90%, for example.

**[ALEX]:** What about distributed tracing?

**[PRIYA]:** AWS X-Ray. Run the X-Ray daemon as a sidecar container in your task definition. Your app sends trace data to the sidecar, which forwards it to X-Ray. You get request traces across services — see exactly where latency is accumulating.

**[ALEX]:** The sidecar pattern — running helper containers alongside your main app container in the same task.

**[PRIYA]:** Very common in ECS. Log routers like FireLens with Fluent Bit, service mesh proxies, secrets managers, monitoring agents — all run as sidecars. Each container in a task shares the same network namespace and can communicate over localhost.

🎵 *Closing music begins to fade in softly*

---

# SEGMENT 9: Wrap-Up, Best Practices & What to Build `(58:00 – 62:00)`

**[PRIYA]:** Speed round of best practices before we close out.

**[ALEX]:** Go for it.

**[PRIYA]:** Tag your images with Git SHAs, never with `latest` in production. Always set CPU and memory limits — a runaway task can starve others. Use Fargate Spot for batch workloads to cut costs dramatically. Set proper health check grace periods — especially for apps with slow startup times. Use ECR lifecycle policies to automatically clean up old image versions.

**[ALEX]:** And the ECS vs EKS question that I know everyone asks.

**[PRIYA]:** Choose ECS when you want simplicity and AWS integration with less operational overhead. Choose EKS when you need Kubernetes-specific features — custom schedulers, complex networking policies, portability across clouds, or a large existing Kubernetes investment.

**[ALEX]:** What should listeners build to get hands-on experience?

**[PRIYA]:** Four projects in order of complexity:

1. **Containerized REST API** — Deploy a Node.js or FastAPI app on Fargate with ALB, CloudWatch logging, and a Secrets Manager-backed database connection.
2. **Background Job Worker** — ECS service that polls SQS and processes messages, with auto scaling based on queue depth.
3. **Scheduled Batch Job** — EventBridge rule triggers an ECS task on a cron schedule to run a data processing job.
4. **Multi-Service Application** — Frontend, API, and worker services with Service Connect, ALB routing, and blue/green deployments via CodePipeline.

**[ALEX]:** That covers the full gamut. Alright — that's a wrap on The Container Ship's ECS deep dive. Clusters, task definitions, Fargate vs EC2, networking, deployments, scaling, IAM, and observability.

**[PRIYA]:** All the infrastructure code and example task definitions from today are in the show notes. Go ship something.

**[ALEX]:** Until next time —

**[PRIYA]:** Stay containerized. 🐳

🎵 *Outro music swells and fades out over 5 seconds*

---

## 🎯 Key Analogies Reference Card

| Concept | Analogy |
|---------|---------|
| ECS | Port authority managing a fleet of container ships |
| Cluster | The shipping port — named space where containers operate |
| Task Definition | Recipe card — ingredients, quantities, cooking temperature |
| Task | The actual dish being cooked right now |
| Service | Night watchman counting guards — always reconciling to desired count |
| Fargate | Hiring a logistics company — you hand off the box, they handle everything |
| EC2 Launch Type | Owning a fleet of trucks — full control, full responsibility |
| Rolling Update | Replacing crew members one at a time while the ship keeps sailing |
| Blue/Green Deploy | Two identical docks — switch all ships from dock A to dock B instantly |
| Target Tracking Scaling | Thermostat — you set the target, HVAC maintains it automatically |
| Task Role | Employee badge — what YOUR code is authorized to access |
| Execution Role | Manager badge — what ECS is authorized to do on your behalf |
| Service Connect | Built-in concierge that knows where every service lives |

---

## 📚 Quick CLI Reference

```bash
# ── CLUSTER ──────────────────────────────────────────
aws ecs create-cluster --cluster-name my-prod-cluster
aws ecs list-clusters
aws ecs describe-clusters --clusters my-prod-cluster

# ── TASK DEFINITIONS ─────────────────────────────────
aws ecs register-task-definition --cli-input-json file://task-def.json
aws ecs list-task-definitions --family-prefix my-web-app
aws ecs describe-task-definition --task-definition my-web-app:42

# ── SERVICES ─────────────────────────────────────────
aws ecs create-service \
  --cluster my-prod-cluster \
  --service-name my-web-service \
  --task-definition my-web-app:42 \
  --desired-count 3 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-abc],securityGroups=[sg-xyz],assignPublicIp=DISABLED}"

aws ecs update-service \
  --cluster my-prod-cluster \
  --service my-web-service \
  --desired-count 5

aws ecs describe-services \
  --cluster my-prod-cluster \
  --services my-web-service

# ── TASKS ────────────────────────────────────────────
aws ecs list-tasks --cluster my-prod-cluster --service-name my-web-service
aws ecs describe-tasks --cluster my-prod-cluster --tasks <task-arn>
aws ecs run-task \
  --cluster my-prod-cluster \
  --task-definition my-batch-job:5 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-abc],assignPublicIp=DISABLED}"

# ── EXEC INTO CONTAINER (debugging) ─────────────────
aws ecs execute-command \
  --cluster my-prod-cluster \
  --task <task-arn> \
  --container web \
  --interactive \
  --command "/bin/sh"

# ── DEPLOYMENTS ──────────────────────────────────────
aws ecs describe-services \
  --cluster my-prod-cluster \
  --services my-web-service \
  --query 'services[0].deployments'

# ── ECR ──────────────────────────────────────────────
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin 123456789.dkr.ecr.us-east-1.amazonaws.com

docker build -t my-app:latest .
docker tag my-app:latest 123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:$GIT_SHA
docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:$GIT_SHA

# ── LOGS ─────────────────────────────────────────────
aws logs tail /ecs/my-web-app --follow
aws logs filter-log-events \
  --log-group-name /ecs/my-web-app \
  --filter-pattern "ERROR"
```

---

*The Container Ship is produced independently. All trademarks belong to their respective owners.*
