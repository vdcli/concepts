# 🎙️ Functions as a Feature
## The Lambda Podcast — Serverless Computing on AWS

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
| 0:00–7:00 | What Is Lambda? | Serverless, FaaS, cold starts, pricing model |
| 7:00–15:00 | Function Anatomy | Handler, event, context, runtime, layers |
| 15:00–24:00 | Triggers & Event Sources | API Gateway, SQS, SNS, S3, EventBridge, DynamoDB Streams |
| 24:00–32:00 | Execution Model | Cold starts, warm instances, concurrency, reserved concurrency |
| 32:00–40:00 | Performance & Memory | Memory as CPU proxy, power tuning, duration optimization |
| 40:00–47:00 | Error Handling & Retries | Retry behavior by trigger, DLQs, destinations, idempotency |
| 47:00–53:00 | Networking & Security | VPC Lambda, IAM execution role, environment variables, Secrets Manager |
| 53:00–57:30 | Observability | CloudWatch Logs, X-Ray, Lambda Insights, structured logging |
| 57:30–62:00 | Best Practices & Project Ideas | When not to use Lambda, SAM, CDK, project ideas |

---

# SEGMENT 1: Cold Open & What Is Lambda? `(0:00 – 7:00)`

🎵 *Upbeat tech-themed intro music fades in, plays for 5 seconds, then fades under dialogue*

**[ALEX]:** Welcome to Functions as a Feature — the podcast where we demystify serverless until it clicks. I'm Alex.

**[PRIYA]:** And I'm Priya. Today we're going deep on AWS Lambda — the service that, more than any other, defined what serverless computing means in practice.

**[ALEX]:** Serverless. I know it doesn't literally mean no servers. So what does it mean?

**[PRIYA]:** "Serverless" means you don't think about servers. You write a function — a piece of code — and AWS handles everything else. No EC2 instances to provision, no OS to patch, no capacity to plan, no load balancers to configure. You bring the code, AWS brings the infrastructure.

**[ALEX]:** So Lambda is Function as a Service.

**[PRIYA]:** Exactly — FaaS. Here's the mental model. Imagine you're a freelance translator. The old way: you rent an office 24/7 in case someone brings you a document to translate — that's always-on EC2. The Lambda way: you're on-call. Someone has a document? You show up, translate it, leave. You're billed only for the time you spent translating.

**[ALEX]:** And AWS handles finding you when needed.

**[PRIYA]:** Right. Lambda was launched in 2014 and it fundamentally changed how people think about backend architecture. The pitch: you write code that runs in response to events, scales to zero when idle, and scales to thousands of concurrent executions automatically.

**[ALEX]:** Scales to zero — that's huge. No idle costs.

**[PRIYA]:** That's the killer feature for many workloads. If your app handles 3 requests per hour at 2 AM, Lambda charges you for 3 invocations. An always-on t3.small costs you 24 hours of compute regardless.

**[ALEX]:** And the pricing model is...?

**[PRIYA]:** Two dimensions. First: number of requests — $0.20 per 1 million invocations. Second: duration — calculated as GB-seconds: the amount of memory your function uses, multiplied by how many seconds it runs. And AWS gives you 1 million free requests and 400,000 GB-seconds per month forever, as part of the free tier.

**[ALEX]:** So small workloads can literally be free.

**[PRIYA]:** Many hobby projects run entirely within the free tier. It's one of the best deals in cloud computing.

🎵 *Short musical sting — segment transition*

---

# SEGMENT 2: Function Anatomy — The Handler, Event, Context, Runtime `(7:00 – 15:00)`

**[PRIYA]:** Let's look at what a Lambda function actually looks like. At minimum, it's a handler function. Here's a Python example:

```python
import json

def handler(event, context):
    name = event.get('name', 'World')
    return {
        'statusCode': 200,
        'body': json.dumps({'message': f'Hello, {name}!'})
    }
```

**[ALEX]:** Three things: the handler function, `event`, and `context`. What's in each?

**[PRIYA]:** The `event` is the input data from whatever triggered the Lambda. It changes completely depending on the trigger. An API Gateway invocation puts the HTTP request in event. An S3 event puts bucket and object information. An SQS trigger puts the message batch. Your handler needs to know what trigger it's expecting.

**[ALEX]:** And `context`?

**[PRIYA]:** Context is Lambda runtime information — the function name, memory limit, request ID, and crucially: `context.get_remaining_time_in_millis()`. You can use this to check if you're about to hit the timeout and gracefully exit rather than getting killed mid-operation.

**[ALEX]:** What about runtimes? Do I have to use Python?

**[PRIYA]:** Lambda supports Node.js, Python, Java, Go, Ruby, .NET, and any language via a custom runtime. The custom runtime is powerful — you can run Rust, PHP, even COBOL if that's your thing. AWS provides the runtime as a managed execution environment.

**[ALEX]:** And Layers?

**[PRIYA]:** **Layers** are reusable ZIP archives you attach to functions — typically containing shared libraries or dependencies. Think of Layers like a shared pantry. Instead of every chef (function) carrying their own flour and eggs, they all draw from the shared pantry. Your `pandas` and `numpy` libraries go in a Layer, all your data functions share it. Functions stay small, cold starts are faster.

```python
# With a layer providing 'requests' library
import requests

def handler(event, context):
    response = requests.get('https://api.example.com/data')
    return response.json()
```

**[ALEX]:** Deployment package size limits?

**[PRIYA]:** 50MB zipped for direct upload, 250MB unzipped. If your dependencies are larger — heavy ML libraries, for instance — use a container image. Lambda supports Docker images up to 10GB. Same handler interface, same event model, but packaged as a container.

🎵 *Transition music sting*

---

# SEGMENT 3: Triggers — The Event-Driven Universe `(15:00 – 24:00)`

**[ALEX]:** Lambda only does something when triggered. Let's go through the main triggers.

**[PRIYA]:** This is where Lambda's power really shows. The ecosystem of triggers is enormous. Let's hit the big ones.

**[ALEX]:** API Gateway — making Lambda a web server.

**[PRIYA]:** API Gateway + Lambda is the classic serverless API pattern. API Gateway handles HTTPS, routing, rate limiting, and auth. Every HTTP request becomes a Lambda invocation. Zero servers, infinite scale, you pay per request.

```python
def handler(event, context):
    path = event['path']              # e.g., '/users/123'
    method = event['httpMethod']       # 'GET', 'POST', etc.
    body = json.loads(event['body'] or '{}')
    headers = event['headers']
    
    # Your logic here
    return {
        'statusCode': 200,
        'headers': {'Content-Type': 'application/json'},
        'body': json.dumps({'userId': 123})
    }
```

**[ALEX]:** There's also Lambda Function URLs now — no API Gateway needed?

**[PRIYA]:** Right. Function URLs give you a dedicated HTTPS endpoint directly on the function. Simpler, cheaper, but fewer features — no path routing, no usage plans, no custom auth. Great for simple webhooks.

**[ALEX]:** S3 triggers.

**[PRIYA]:** One of the most common patterns. Upload a file to S3, Lambda fires. Resize an image, transcode a video, parse a CSV, trigger a workflow. The event tells you exactly which bucket and object triggered the function.

```python
def handler(event, context):
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']
        # Process the uploaded file
        process_file(bucket, key)
```

**[ALEX]:** SQS — making Lambda a queue consumer.

**[PRIYA]:** Lambda polls SQS on your behalf. Messages come in batches. Lambda scales concurrency to drain the queue. This is a brilliant pattern — SQS provides the buffer, Lambda provides elastic compute to drain it. If processing fails, messages go back to the queue for retry. Failed messages end up in a Dead Letter Queue after max retries.

**[ALEX]:** DynamoDB Streams?

**[PRIYA]:** Change data capture for DynamoDB. Every write, update, or delete to a DynamoDB table streams as an event to Lambda. Perfect for: updating a search index, invalidating caches, sending notifications when records change, auditing, syncing to another system.

**[ALEX]:** EventBridge.

**[PRIYA]:** EventBridge is AWS's event router — a central hub that receives events from AWS services and your own applications and routes them to targets including Lambda. The killer feature: EventBridge Scheduler. You create a cron rule, Lambda runs on schedule. Zero infrastructure.

```
# Run at 9 AM every weekday
cron(0 9 ? * MON-FRI *)
```

**[ALEX]:** Kinesis, SNS, Cognito, ALB — the list goes on.

**[PRIYA]:** Lambda integrates with virtually every AWS service. That's part of what makes it so powerful — it's the universal glue of the AWS ecosystem.

🎵 *Transition music sting*

---

# SEGMENT 4: Execution Model — Cold Starts, Concurrency & Warm Instances `(24:00 – 32:00)`

**[ALEX]:** Let's talk cold starts. This is the famous pain point of Lambda.

**[PRIYA]:** A cold start happens when Lambda needs to spin up a new execution environment — download your code, initialize the runtime, run your initialization code outside the handler. This adds latency, typically 100ms to a few seconds depending on the runtime and package size.

**[ALEX]:** Why does it vary so much?

**[PRIYA]:** Several factors. JVM-based runtimes — Java, Kotlin, Scala — are the slowest because the JVM takes time to warm up. Python and Node.js are fast — usually under 200ms. Go is very fast. Package size matters — larger packages take longer to load. VPC Lambda adds more time because it needs to set up network interfaces.

**[ALEX]:** So what happens after the cold start?

**[PRIYA]:** Lambda keeps the execution environment warm for a period — typically 5 to 15 minutes after the last invocation. Subsequent requests hit a warm environment with no cold start. Like a restaurant: the first customer waits for the kitchen to open (cold start). Later customers find the kitchen already running (warm).

**[ALEX]:** How do you minimize cold start impact?

**[PRIYA]:** Several strategies. **Provisioned Concurrency** — you pay Lambda to keep N execution environments always warm. Zero cold starts, guaranteed. **ARM/Graviton2** — Lambda on ARM initializes faster and runs cheaper. **Minimize package size** — only bundle what you need. **Move initialization outside the handler** — SDK clients, database connections, config loading:

```python
import boto3

# This runs ONCE during cold start, reused across warm invocations
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('my-table')

def handler(event, context):
    # This runs on every invocation — already have warm DynamoDB client
    result = table.get_item(Key={'id': event['id']})
    return result['Item']
```

**[ALEX]:** That's smart — amortize initialization cost across invocations.

**[PRIYA]:** And you can even reuse database connections across invocations using RDS Proxy, which maintains a connection pool on your behalf — solving the classic Lambda + RDS problem of too many database connections.

**[ALEX]:** Concurrency — how does Lambda scale?

**[PRIYA]:** Each invocation of a Lambda function runs in its own isolated environment. If 1000 requests arrive simultaneously, Lambda spins up 1000 concurrent environments. By default, there's a burst limit of 3000 concurrent executions and a steady-state limit of 1000 per region (these are account-level, not per function).

**[ALEX]:** And Reserved Concurrency?

**[PRIYA]:** You can reserve a specific concurrency pool for a function — say 200. This guarantees the function can always scale to 200 but also limits it to 200. It prevents a runaway function from consuming all your account's concurrency and throttling other functions. Like reserving tables at a restaurant — you're guaranteed your table, and other diners can't take them.

🎵 *Transition music sting*

---

# SEGMENT 5: Performance & Memory — The Hidden Levers `(32:00 – 40:00)`

**[ALEX]:** Memory configuration. I've heard this is more nuanced than it seems.

**[PRIYA]:** This is one of the most underappreciated Lambda optimizations. Memory ranges from 128MB to 10GB. Here's the counterintuitive part: **memory controls CPU allocation**. Lambda doesn't expose CPU as a separate setting. More memory = more vCPU allocated proportionally.

**[ALEX]:** So if I double memory, I double CPU.

**[PRIYA]:** Roughly. And since you're billed on memory × duration, a function that runs in 100ms at 1024MB costs the same as one that runs in 200ms at 512MB — 102,400 GB-ms in both cases. If extra memory makes your function finish faster, the cost can be identical or even lower.

**[ALEX]:** So the sweet spot isn't always the minimum memory.

**[PRIYA]:** Exactly. The **AWS Lambda Power Tuning** open-source tool — runs as a Step Functions state machine — automatically tests your function at multiple memory configurations, measures duration and cost, and plots the results. Most teams find the optimal configuration is not where they started.

**[ALEX]:** What about the execution timeout?

**[PRIYA]:** Maximum is 15 minutes. Set your timeout to slightly more than your p99 execution time — this prevents hung functions from running forever and running up your bill, while giving legitimate slow executions room to complete. A Lambda you expect to finish in 3 seconds should have a 30-second timeout — not 15 minutes.

**[ALEX]:** What workloads are actually a bad fit for Lambda?

**[PRIYA]:** Great question. Long-running processes that exceed 15 minutes — use Fargate or EC2. Workloads that require persistent connections like WebSockets — use API Gateway WebSocket APIs or ECS. Very high steady-state request volumes where always-on EC2 is cheaper. Anything needing more than 10GB of memory or local disk beyond 512MB ephemeral storage.

**[ALEX]:** The 512MB ephemeral storage — that's /tmp?

**[PRIYA]:** Yes, `/tmp` is your ephemeral scratch space. Readable and writable, but gone when the environment recycles. For larger scratch space, you can increase it to 10GB now. For persistent files, use S3 or EFS — Lambda supports mounting EFS volumes for shared, persistent file access.

🎵 *Transition music sting*

---

# SEGMENT 6: Error Handling, Retries & Idempotency `(40:00 – 47:00)`

**[ALEX]:** What happens when a Lambda function fails?

**[PRIYA]:** It depends entirely on the trigger type. **Synchronous invocations** — API Gateway, Function URLs — Lambda returns the error to the caller. No automatic retry. The client is responsible for retrying.

**[ALEX]:** And asynchronous triggers?

**[PRIYA]:** **Asynchronous invocations** — SNS, S3, EventBridge — Lambda automatically retries twice with delays between attempts. If it still fails after 3 attempts, the event is discarded unless you configure a **Dead Letter Queue** or a **Lambda Destination**.

**[ALEX]:** What's a Destination versus a DLQ?

**[PRIYA]:** A DLQ is just an SQS queue or SNS topic that receives failed events. Lambda Destinations are more powerful — you configure separate targets for success AND failure. Failed events go to an SQS DLQ for reprocessing. Successful events go to an SNS topic to trigger downstream processing. More flexibility.

**[ALEX]:** What about SQS triggers? Those retry differently.

**[PRIYA]:** SQS-triggered Lambda uses the queue's own retry logic. Failed messages return to the queue and get retried up to the queue's `maxReceiveCount`. After that, they go to the SQS Dead Letter Queue. The key is to configure a DLQ on the SQS queue itself — not on the Lambda function.

**[ALEX]:** Idempotency. If something retries, you might process the same message twice.

**[PRIYA]:** This is critical to design for. Your Lambda function must be **idempotent** — processing the same event multiple times has the same effect as processing it once. Think of an idempotent operation like a light switch versus a volume knob. "Set volume to 50" is idempotent — do it 5 times, you end up at 50. "Turn volume up 10" is not — 5 times and you're at max.

**[ALEX]:** How do you implement idempotency?

**[PRIYA]:** Store a unique identifier for each event — the SQS message ID, the S3 event version ID, or a client-provided request ID — in DynamoDB with a conditional write. If you've already processed that ID, skip and return success. AWS also has the **Lambda Powertools** library which includes an idempotency decorator:

```python
from aws_lambda_powertools.utilities.idempotency import idempotent

@idempotent(config=IdempotencyConfig(event_key_jmespath="body"))
def handler(event, context):
    # This block is guaranteed to run only once per unique event body
    process_order(json.loads(event['body']))
    return {'statusCode': 200}
```

🎵 *Transition music sting*

---

# SEGMENT 7: Networking & Security `(47:00 – 53:00)`

**[ALEX]:** By default Lambda isn't in your VPC?

**[PRIYA]:** Correct. By default, Lambda runs in AWS-managed infrastructure with access to the internet and all AWS services via their public endpoints. This is fine for most workloads.

**[ALEX]:** But if I need to access RDS or ElastiCache in a private subnet?

**[PRIYA]:** You attach the Lambda to your VPC. You specify VPC, subnets, and security groups. Lambda creates ENIs in your specified subnets and traffic routes through your VPC.

**[ALEX]:** But then it loses internet access.

**[PRIYA]:** Right — your private subnet doesn't have an internet gateway. You need a NAT Gateway for outbound internet access, or VPC Interface Endpoints for AWS service calls without going through the internet. VPC Lambda historically added cold start latency, but AWS now pre-creates ENIs, so the impact is minimal.

**[ALEX]:** IAM for Lambda — the execution role.

**[PRIYA]:** Your Lambda function assumes an **Execution Role** for every invocation. This role defines what AWS resources the function can access. Least privilege applies: if your function only reads from one DynamoDB table, give it only `dynamodb:GetItem` on that specific table ARN. Never use `*` resource or `*` action in production Lambda execution roles.

**[ALEX]:** Secrets and environment variables.

**[PRIYA]:** Environment variables are the simplest way to pass configuration. For sensitive values, store in Secrets Manager or SSM Parameter Store and fetch at runtime. **Lambda Powertools Parameters** utility caches these with configurable TTLs so you're not calling Secrets Manager on every invocation — which adds latency and costs money.

```python
from aws_lambda_powertools.utilities import parameters

def handler(event, context):
    # Cached for 5 minutes by default
    db_password = parameters.get_secret("prod/db/password")
    # Use db_password to connect
```

🎵 *Transition music sting*

---

# SEGMENT 8: Observability — Logs, Traces & Lambda Insights `(53:00 – 57:30)`

**[ALEX]:** How do you debug a Lambda function in production?

**[PRIYA]:** Three tools: CloudWatch Logs, X-Ray tracing, and Lambda Insights. CloudWatch Logs captures everything your function writes to stdout/stderr. Every invocation is a log stream. Use structured logging — JSON — so you can filter and query with CloudWatch Logs Insights.

```python
import json
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def handler(event, context):
    logger.info(json.dumps({
        "action": "process_order",
        "orderId": event['orderId'],
        "requestId": context.aws_request_id
    }))
```

**[ALEX]:** And X-Ray?

**[PRIYA]:** Enable active tracing on the function — one toggle in the console or a line in your config. X-Ray automatically captures: invocation time, any downstream AWS SDK calls, HTTP requests, SQL queries. You see exactly where time is spent across your entire request path through multiple Lambda functions.

**[ALEX]:** Lambda Insights is separate from Container Insights?

**[PRIYA]:** Yes — Lambda Insights is a CloudWatch feature that collects system-level metrics: cold start duration, memory utilization, init duration. It identifies functions with high cold start rates or memory waste. Enable it as a Lambda Layer — one line of config.

**[ALEX]:** Lambda Powertools for Python — you've mentioned it twice now.

**[PRIYA]:** Worth calling out explicitly. AWS Lambda Powertools is an open-source library from AWS — available for Python, Java, TypeScript, and .NET — that provides structured logging, tracing, idempotency, parameter caching, feature flags, batch processing utilities, and more. It's the fastest way to production-ready Lambda functions.

🎵 *Closing music begins to fade in softly*

---

# SEGMENT 9: Wrap-Up, Best Practices & Project Ideas `(57:30 – 62:00)`

**[PRIYA]:** Best practices to take away.

**[ALEX]:** Go.

**[PRIYA]:** Keep functions small and focused — one function, one purpose. Initialization code outside the handler. Use Provisioned Concurrency for latency-sensitive APIs. Always configure DLQs for async invocations. Design for idempotency from the start. Use Lambda Powertools. Right-size memory with the Power Tuning tool.

**[ALEX]:** And for infrastructure-as-code?

**[PRIYA]:** AWS SAM — Serverless Application Model — is YAML-based, simple, great for Lambda-first projects. AWS CDK is code-based — TypeScript, Python, Java — better for complex infrastructure that spans Lambda and non-Lambda resources. Serverless Framework is a community option with a huge plugin ecosystem.

**[ALEX]:** What should people build to practice?

**[PRIYA]:** Four projects:

1. **Image Processing Pipeline** — S3 upload triggers Lambda that resizes images to three sizes and stores them back in S3, with SQS + DLQ for retry handling.
2. **Serverless REST API** — API Gateway + Lambda + DynamoDB with CRUD operations, IAM execution role, Secrets Manager for any third-party API keys.
3. **Scheduled Data Pipeline** — EventBridge cron triggers Lambda to fetch data from an external API, transform it, and load it into DynamoDB or S3.
4. **Event-Driven Notifications** — DynamoDB Streams trigger Lambda to send personalized emails via SES when specific records change, with idempotency to prevent duplicate sends.

**[ALEX]:** That covers the full range of Lambda use cases. We talked about the pricing model, cold starts, concurrency, memory tuning, triggers, error handling, idempotency, VPC, IAM, and observability.

**[PRIYA]:** All the example code, SAM templates, and Lambda Powertools setup from today are in the show notes. Go build something that costs you $0.00 per month.

**[ALEX]:** Until next time —

**[PRIYA]:** Stay serverless. λ

🎵 *Outro music swells and fades out over 5 seconds*

---

## 🎯 Key Analogies Reference Card

| Concept | Analogy |
|---------|---------|
| Serverless / Lambda | Freelance translator — show up when there's work, leave when done, billed by the job |
| Cold Start | First customer of the day — kitchen needs to open before food is served |
| Warm Instance | Kitchen already running — subsequent customers get fast service |
| Provisioned Concurrency | Keeping the kitchen staffed 24/7 — no first-customer delay ever |
| Lambda Layers | Shared pantry — every chef draws from common ingredients, no duplication |
| Reserved Concurrency | Reserved restaurant tables — guaranteed capacity, can't exceed it |
| Idempotency | Light switch vs volume knob — "set to 50" is safe to repeat, "add 10" is not |
| Execution Role | Employee security badge — defines exactly which doors Lambda can open |
| SQS + Lambda | Buffer + elastic drain — queue absorbs spikes, Lambda scales to clear it |
| DLQ | Hospital emergency room — failed messages get special attention, not discarded |
| Memory = CPU | More memory = more vCPU allocated — tune both with one slider |
| VPC Lambda | Adding Lambda to the private network — loses internet, gains private resource access |

---

## 📚 Quick Reference

```bash
# ── DEPLOY WITH SAM ──────────────────────────────────
# template.yaml minimal example:
# AWSTemplateFormatVersion: '2010-09-09'
# Transform: AWS::Serverless-2016-10-31
# Resources:
#   MyFunction:
#     Type: AWS::Serverless::Function
#     Properties:
#       Handler: app.handler
#       Runtime: python3.12
#       MemorySize: 512
#       Timeout: 30
#       Events:
#         Api:
#           Type: Api
#           Properties:
#             Path: /hello
#             Method: get

sam build
sam deploy --guided

# ── DEPLOY WITH CLI ──────────────────────────────────
zip function.zip app.py
aws lambda create-function \
  --function-name my-function \
  --runtime python3.12 \
  --role arn:aws:iam::123456789:role/lambda-exec-role \
  --handler app.handler \
  --zip-file fileb://function.zip \
  --memory-size 512 \
  --timeout 30

aws lambda update-function-code \
  --function-name my-function \
  --zip-file fileb://function.zip

# ── INVOKE & TEST ────────────────────────────────────
aws lambda invoke \
  --function-name my-function \
  --payload '{"name": "Alex"}' \
  --cli-binary-format raw-in-base64-out \
  response.json
cat response.json

# ── CONCURRENCY ──────────────────────────────────────
aws lambda put-function-concurrency \
  --function-name my-function \
  --reserved-concurrent-executions 200

aws lambda put-provisioned-concurrency-config \
  --function-name my-function \
  --qualifier my-alias \
  --provisioned-concurrent-executions 50

# ── LOGS ─────────────────────────────────────────────
aws logs tail /aws/lambda/my-function --follow
aws logs filter-log-events \
  --log-group-name /aws/lambda/my-function \
  --filter-pattern "ERROR"

# ── ENVIRONMENT VARIABLES ────────────────────────────
aws lambda update-function-configuration \
  --function-name my-function \
  --environment "Variables={ENV=production,TABLE_NAME=orders}"

# ── ALIASES & VERSIONING ─────────────────────────────
aws lambda publish-version --function-name my-function
aws lambda create-alias \
  --function-name my-function \
  --name production \
  --function-version 42

# ── EVENT SOURCE MAPPING (SQS) ───────────────────────
aws lambda create-event-source-mapping \
  --function-name my-function \
  --event-source-arn arn:aws:sqs:us-east-1:123456789:my-queue \
  --batch-size 10 \
  --maximum-batching-window-in-seconds 5
```

---

*Functions as a Feature is produced independently. All trademarks belong to their respective owners.*
