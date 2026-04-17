# 🎙️ Bucket List
## The S3 Podcast — Object Storage That Powers the Internet

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
| 0:00–7:00 | What Is S3? | Object storage, buckets, objects, keys, durability |
| 7:00–15:00 | Core Operations | PUT, GET, DELETE, multipart, presigned URLs |
| 15:00–23:00 | Storage Classes | Standard, IA, Glacier, Intelligent-Tiering, lifecycle policies |
| 23:00–31:00 | Security & Access Control | Bucket policies, ACLs, Block Public Access, presigned URLs, IAM |
| 31:00–39:00 | Static Website Hosting | CloudFront, OAC, HTTPS, caching strategies |
| 39:00–46:00 | Advanced Features | Versioning, replication, Object Lock, S3 Batch Operations |
| 46:00–53:00 | Event-Driven Patterns | S3 Event Notifications, EventBridge, Lambda triggers |
| 53:00–57:30 | Performance | Transfer Acceleration, multipart uploads, request rate scaling |
| 57:30–62:00 | Best Practices & Project Ideas | Cost optimization, naming, project ideas |

---

# SEGMENT 1: Cold Open & What Is S3? `(0:00 – 7:00)`

🎵 *Upbeat tech-themed intro music fades in, plays for 5 seconds, then fades under dialogue*

**[ALEX]:** Welcome to Bucket List — the podcast where we check off everything you need to know about AWS S3. I'm Alex.

**[PRIYA]:** And I'm Priya. Amazon S3 — Simple Storage Service. Launched in 2006, it was one of the very first AWS services. And it's still arguably the most important one.

**[ALEX]:** S3 stores files, right? Isn't that straightforward?

**[PRIYA]:** That's what makes it deceptively interesting. At its core, yes — it stores files. But "stores files at any scale, with eleven nines of durability, accessible from anywhere on the internet, for fractions of a cent per gigabyte" — that's a fundamentally different thing than your laptop's hard drive.

**[ALEX]:** Eleven nines — 99.999999999% durability. What does that actually mean?

**[PRIYA]:** It means that if you store 10 million objects in S3, on average you can expect to lose one object every 10,000 years. AWS achieves this by automatically storing your data across at least three Availability Zones — three separate physical data centers. If one burns down, your data is safe.

**[ALEX]:** How does S3 actually store data? It's not a traditional file system.

**[PRIYA]:** S3 is **object storage**, not block storage or file storage. Here's the distinction. Think of a traditional file system like a physical filing cabinet — hierarchical folders, file names, you navigate down a tree. Object storage is more like a post office with P.O. boxes. Every object has a unique key — its address — and you go directly to that address. There's no hierarchy, no folders. Just a flat namespace.

**[ALEX]:** But S3 shows me folders in the console.

**[PRIYA]:** That's a UI illusion. The key `photos/2024/january/vacation.jpg` is just a string. The slash characters look like a path but they're part of the key. The console pretends they're folders for usability. Internally: just one flat bucket of objects identified by keys.

**[ALEX]:** And the bucket — what is that?

**[PRIYA]:** A bucket is the top-level container. Think of it as the post office branch. Globally unique name — across all AWS accounts worldwide. You pick a region, create a bucket, and start storing objects. Bucket names become part of the URL: `my-bucket.s3.amazonaws.com`.

**[ALEX]:** Who uses S3?

**[PRIYA]:** Dropbox originally built their entire service on S3. Netflix stores all their video assets in S3. Airbnb, Reddit, Pinterest. The AWS data lake ecosystem — Athena, Glue, Redshift Spectrum — all treat S3 as the foundational data store. S3 stores more data than any other storage service in the world.

🎵 *Short musical sting — segment transition*

---

# SEGMENT 2: Core Operations — Working With Objects `(7:00 – 15:00)`

**[PRIYA]:** S3 operations are simple CRUD through an HTTP API — or through the AWS SDK and CLI. Let's walk through the essentials.

**[ALEX]:** The CLI basics first.

**[PRIYA]:** 

```bash
# Upload a file (PUT)
aws s3 cp photo.jpg s3://my-bucket/photos/photo.jpg

# Download a file (GET)
aws s3 cp s3://my-bucket/photos/photo.jpg ./downloaded.jpg

# List objects
aws s3 ls s3://my-bucket/photos/ --recursive

# Delete
aws s3 rm s3://my-bucket/photos/old-photo.jpg

# Sync a directory
aws s3 sync ./local-folder s3://my-bucket/folder/ --delete
```

**[ALEX]:** And with the SDK in Python?

**[PRIYA]:**

```python
import boto3

s3 = boto3.client('s3')

# Upload
s3.upload_file('photo.jpg', 'my-bucket', 'photos/photo.jpg')

# Upload from memory (BytesIO)
from io import BytesIO
data = b'{"key": "value"}'
s3.put_object(Bucket='my-bucket', Key='data.json', Body=data)

# Download
s3.download_file('my-bucket', 'photos/photo.jpg', 'local.jpg')

# Read object content directly
response = s3.get_object(Bucket='my-bucket', Key='data.json')
content = response['Body'].read()

# Generate a presigned URL
url = s3.generate_presigned_url(
    'get_object',
    Params={'Bucket': 'my-bucket', 'Key': 'photos/photo.jpg'},
    ExpiresIn=3600  # URL valid for 1 hour
)
```

**[ALEX]:** Let's talk about presigned URLs — this is a pattern I see everywhere.

**[PRIYA]:** Presigned URLs are one of the most useful S3 features. You generate a URL server-side that includes temporary AWS credentials. You give that URL to a client — a browser, a mobile app — and they can upload or download directly to S3 without going through your server.

**[ALEX]:** So my server generates the URL, client uploads directly to S3?

**[PRIYA]:** Exactly. Think of it like a guest valet ticket. The hotel (your server) gives the guest a ticket with limited permissions. The guest hands it to the valet (S3) directly. The hotel never handles the car (file). This removes your servers from the data transfer path entirely — massively better performance for large files.

```
Browser → POST /upload-url → Your Server → generates presigned URL → Browser
Browser → PUT {presigned URL} → S3 directly (your server not involved)
```

**[ALEX]:** And for large files — multipart upload.

**[PRIYA]:** Multipart upload lets you split a file into parts — minimum 5MB each — and upload them in parallel. S3 assembles them into a single object. Benefits: parallel upload speeds, ability to pause and resume, individual part retry on failure. For files over 100MB, always use multipart. The SDK handles this automatically with `upload_file()` — it switches to multipart when the file exceeds the threshold.

🎵 *Transition music sting*

---

# SEGMENT 3: Storage Classes — Pay for What You Actually Need `(15:00 – 23:00)`

**[ALEX]:** Not all data is accessed equally. S3 has different storage classes for this?

**[PRIYA]:** Yes, and choosing the right class can cut your storage costs by 40–90%. Let me walk through them.

**[ALEX]:** Standard first.

**[PRIYA]:** **S3 Standard** — highest availability (99.99%), instant access, no minimum storage duration. The default. Use it for data you access frequently — active application data, current media, anything read more than once a month.

**[ALEX]:** Standard-IA?

**[PRIYA]:** **Standard-IA** — Infrequent Access. Same durability, same instant access, but 40% cheaper storage cost. The catch: you pay a per-retrieval fee and there's a 30-day minimum storage duration. If you store something for 1 day and delete it, you're billed for 30 days. Use it for backups, older data, disaster recovery files you rarely need.

**[ALEX]:** One Zone-IA?

**[PRIYA]:** Like Standard-IA but only stored in one Availability Zone. 20% cheaper than Standard-IA. If that AZ has a disaster, data is gone. Use only for data you can recreate — thumbnail images, transcoded video versions. Never for originals.

**[ALEX]:** Glacier options — for long-term archival.

**[PRIYA]:** Three Glacier tiers. **Glacier Instant Retrieval** — archive pricing but millisecond access. For data accessed maybe once a quarter. **Glacier Flexible Retrieval** — access takes minutes to hours, you choose the urgency. **Glacier Deep Archive** — the cheapest tier, $0.00099 per GB per month. Access takes 12 hours. For compliance archives you need to keep for 7 years but pray you never touch.

**[ALEX]:** And Intelligent-Tiering?

**[PRIYA]:** My personal recommendation for most teams. S3 Intelligent-Tiering monitors access patterns and automatically moves objects between Standard, Standard-IA, Archive Instant Access, and Archive tiers. You pay a small monitoring fee per object, but you never pay retrieval fees and you never have to classify your data manually.

**[ALEX]:** So it's like a self-organizing library.

**[PRIYA]:** Exactly — books that get checked out frequently stay near the front desk. Books nobody has touched in months move to the deep stacks. Automatically.

**[ALEX]:** Lifecycle policies — automating this.

**[PRIYA]:** Lifecycle policies automate class transitions based on object age. A common pattern:

```
Days 0–30:   Standard (active use)
Days 30–90:  Standard-IA (cooling off)
Days 90–365: Glacier Instant Retrieval (archival)
Days 365+:   Glacier Deep Archive (long-term compliance)
```

One lifecycle rule, set and forget. You're not paying Standard prices for 7-year-old log files.

🎵 *Transition music sting*

---

# SEGMENT 4: Security & Access Control `(23:00 – 31:00)`

**[ALEX]:** S3 security — there have been some famous data breach stories involving misconfigured S3 buckets.

**[PRIYA]:** Absolutely. The root cause of most S3 breaches is accidental public access. AWS introduced **Block Public Access** settings to address this. Four settings at the bucket — or account — level that prevent public ACLs and public bucket policies from taking effect. Enable all four at the account level and no bucket in your account can accidentally become public.

**[ALEX]:** What are the mechanisms for granting access?

**[PRIYA]:** Three layers. **IAM policies** — attached to users, roles, or groups. "This IAM role can `s3:GetObject` on this bucket." This is how your EC2 instances, Lambda functions, and ECS tasks access S3. **Bucket policies** — JSON policies attached to the bucket. "This bucket allows `s3:GetObject` from this specific IAM role." **ACLs** — legacy, object-level permissions. Avoid ACLs for new designs.

**[ALEX]:** When do you use bucket policies versus IAM policies?

**[PRIYA]:** Use IAM policies when you're granting your AWS principals access to S3. Use bucket policies when you need cross-account access — "allow account B to read from this bucket" — or when you need to enforce conditions like "only allow access from this VPC" or "require HTTPS."

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": ["arn:aws:s3:::my-bucket/*"],
      "Condition": {
        "Bool": {"aws:SecureTransport": "false"}
      }
    }
  ]
}
```

**[ALEX]:** That denies any non-HTTPS request.

**[PRIYA]:** Enforce encryption in transit. Always. Also enforce encryption at rest — enable S3 default encryption with SSE-S3 or SSE-KMS. SSE-S3 uses AWS-managed keys, free, no configuration needed. SSE-KMS uses your own KMS keys — adds cost but gives you key management control, rotation, and audit trails.

**[ALEX]:** VPC access to S3 — can you keep traffic off the internet?

**[PRIYA]:** Via **S3 Gateway Endpoints** — free VPC endpoints that route S3 traffic through the AWS backbone, never touching the internet. Attach one to your route table and all S3 calls from your VPC stay private. For more granular control, **S3 Interface Endpoints** — powered by PrivateLink — give you private IP addresses for S3 within your VPC.

🎵 *Transition music sting*

---

# SEGMENT 5: Static Website Hosting & CloudFront `(31:00 – 39:00)`

**[ALEX]:** S3 for hosting static websites — let's talk about that pattern.

**[PRIYA]:** Enabling static website hosting on an S3 bucket turns it into an HTTP server — it serves your HTML, CSS, JS files directly. You get a website endpoint: `my-bucket.s3-website-us-east-1.amazonaws.com`. No servers, no EC2, no load balancers.

**[ALEX]:** But you put CloudFront in front of it in production.

**[PRIYA]:** Always. CloudFront is AWS's CDN — Content Delivery Network. It caches your static files at 450+ edge locations worldwide. A user in Tokyo downloads your site from a Tokyo edge, not your S3 bucket in Virginia. Dramatically lower latency.

**[ALEX]:** And CloudFront adds HTTPS.

**[PRIYA]:** S3 website hosting only supports HTTP. CloudFront adds HTTPS with free SSL certificates from ACM — AWS Certificate Manager. Your custom domain, HTTPS, global CDN — it's the complete package.

**[ALEX]:** The OAC — Origin Access Control — is the modern way to lock down the S3 bucket?

**[PRIYA]:** Right. **Origin Access Control** replaces the older OAI. With OAC, your S3 bucket is private — no public access. Only CloudFront can access it, and only via a bucket policy that allows the specific CloudFront distribution. Users can't bypass CloudFront to hit S3 directly.

```json
// Bucket policy allowing only CloudFront to access
{
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"Service": "cloudfront.amazonaws.com"},
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::my-site-bucket/*",
    "Condition": {
      "StringEquals": {
        "AWS:SourceArn": "arn:aws:cloudfront::123456789:distribution/DIST_ID"
      }
    }
  }]
}
```

**[ALEX]:** What about SPA deployments — React, Vue apps?

**[PRIYA]:** Two things to configure. First: custom error page — redirect 404 errors to `index.html` so React Router handles the routing. Second: cache invalidation on deploy — `aws cloudfront create-invalidation --paths '/*'` — to ensure users get the new version immediately after deployment.

**[ALEX]:** And the deployment pipeline?

**[PRIYA]:**

```bash
# Build and deploy
npm run build
aws s3 sync ./dist s3://my-site-bucket --delete
aws cloudfront create-invalidation \
  --distribution-id EDFDVBD6EXAMPLE \
  --paths '/*'
```

Simple, cheap, globally fast. This is how most static sites and SPAs should be deployed.

🎵 *Transition music sting*

---

# SEGMENT 6: Advanced Features — Versioning, Replication & Object Lock `(39:00 – 46:00)`

**[ALEX]:** Versioning. What does it do?

**[PRIYA]:** With versioning enabled, S3 keeps every version of every object — every overwrite, every delete. Delete a file? S3 creates a delete marker but keeps the previous version. Overwrite a file? The old version is preserved.

**[ALEX]:** Like a time machine for your bucket.

**[PRIYA]:** Exactly. If someone accidentally deletes your production database backup, you can restore the previous version. If your code writes corrupted data to S3, you roll back to yesterday's version.

**[ALEX]:** And it's not free — you pay for each version.

**[PRIYA]:** Right. Combine versioning with lifecycle policies to manage cost — "keep the last 5 versions, delete older versions after 90 days."

**[ALEX]:** Cross-Region Replication.

**[PRIYA]:** **CRR** automatically copies objects from a source bucket to a destination bucket in another region. Asynchronous — typically replicates within seconds to minutes. Use cases: disaster recovery in a separate geographic region, reducing latency by putting data closer to users, meeting data residency requirements.

**[ALEX]:** And Same-Region Replication — SRR?

**[PRIYA]:** For copying between buckets in the same region. Common pattern: replicate production data to a separate account for analytics teams — they get the data without having access to the production account.

**[ALEX]:** Object Lock — this sounds serious.

**[PRIYA]:** It is. Object Lock implements **WORM** — Write Once, Read Many. Once locked, an object cannot be deleted or overwritten for a specified period — even by the root account, even by AWS support. Two modes:

**Compliance mode**: The retention period cannot be shortened by anyone. Ever. Used for regulatory requirements — financial records, healthcare data, legal holds.

**Governance mode**: Accounts with special IAM permissions can override, but regular users can't. More flexible for general data protection.

**[ALEX]:** S3 Batch Operations?

**[PRIYA]:** For operating on millions of objects at once. Imagine you need to re-encrypt 500 million objects with a new KMS key, or add a tag to all objects from 2022, or restore a million Glacier objects. Batch Operations creates a job from an inventory report and runs your operation in parallel at AWS scale. What would take you weeks of scripting runs as a managed job in hours.

🎵 *Transition music sting*

---

# SEGMENT 7: Event-Driven Patterns — S3 + Lambda + EventBridge `(46:00 – 53:00)`

**[ALEX]:** S3 events. Every upload can trigger something.

**[PRIYA]:** This is where S3 becomes a cornerstone of event-driven architectures. S3 can emit events for: object creation (PUT, POST, COPY, multipart complete), object deletion, object restoration from Glacier, replication failures.

**[ALEX]:** Two ways to receive them — S3 Event Notifications and EventBridge?

**[PRIYA]:** Right. **S3 Event Notifications** are the original, simpler approach. Configure directly on the bucket — events go to SQS, SNS, or Lambda. Fast, simple, but limited filtering — only by prefix and suffix.

```json
// S3 Notification Configuration
{
  "LambdaFunctionConfigurations": [{
    "LambdaFunctionArn": "arn:aws:lambda:...:my-processor",
    "Events": ["s3:ObjectCreated:*"],
    "Filter": {
      "Key": {
        "FilterRules": [
          {"Name": "prefix", "Value": "uploads/"},
          {"Name": "suffix", "Value": ".jpg"}
        ]
      }
    }
  }]
}
```

**[ALEX]:** And EventBridge for S3?

**[PRIYA]:** Enable EventBridge notifications on the bucket and every S3 event flows into your default event bus. From there you can route with 28+ content-based filtering fields — not just prefix/suffix but object size, storage class, who made the request. Fan out to multiple targets. Archive and replay events. Much more powerful than direct notifications.

**[ALEX]:** Walk me through a real pipeline.

**[PRIYA]:** Classic image processing pipeline:

```
User uploads → S3 (uploads/raw/)
    → S3 Event → Lambda
        → Resize to 3 sizes
        → Store in S3 (uploads/processed/)
        → Write metadata to DynamoDB
        → Publish completion to SNS
            → Email notification
            → SQS queue for downstream workers
```

Every component is event-driven, serverless, scales automatically. Zero idle cost, infinite scale.

**[ALEX]:** And S3 Select — querying data in-place?

**[PRIYA]:** S3 Select lets you run SQL against CSV, JSON, or Parquet files stored in S3 — without downloading the whole file. If you have a 10GB CSV and need 100KB of rows matching a condition, S3 Select filters server-side. Combined with Athena — which runs SQL across S3 files at petabyte scale — S3 becomes a data warehouse without any database infrastructure.

🎵 *Transition music sting*

---

# SEGMENT 8: Performance — Moving Data Fast `(53:00 – 57:30)`

**[ALEX]:** Performance. S3 is fast by default?

**[PRIYA]:** For most use cases, yes. S3 scales request rates automatically — each prefix in a bucket supports 3,500 PUT/POST/DELETE and 5,500 GET/HEAD requests per second. If you need more, add randomness to key prefixes.

**[ALEX]:** What do you mean by randomness in key prefixes?

**[PRIYA]:** Historically, S3 partitioned by key prefix. Uploading thousands of files all starting with `2024-01-15-` put them on the same partition. Adding a random hash prefix — `a3f7/2024-01-15-filename.jpg` — distributed them. AWS now handles this automatically, so it's less of a concern, but it's still a pattern you'll see in older architectures.

**[ALEX]:** Transfer Acceleration.

**[PRIYA]:** S3 Transfer Acceleration uses CloudFront's edge network for uploads. Instead of uploading directly to S3 in us-east-1, your client uploads to the nearest CloudFront edge, and the data travels over AWS's optimized backbone to S3. Meaningful gains for users far from your S3 region — often 50–500% faster across continents.

**[ALEX]:** Multipart upload for large files — you mentioned this.

**[PRIYA]:** For files over 100MB: split into parts, upload in parallel, S3 assembles. The practical rules: part size between 5MB and 5GB, maximum 10,000 parts, maximum object size 5TB. One important gotcha: abort incomplete multipart uploads or they silently accumulate and you pay for the storage. Set a lifecycle rule to abort incomplete multiparts after N days.

```bash
# Lifecycle rule to clean up incomplete multipart uploads
aws s3api put-bucket-lifecycle-configuration \
  --bucket my-bucket \
  --lifecycle-configuration '{
    "Rules": [{
      "ID": "abort-incomplete-multipart",
      "Status": "Enabled",
      "AbortIncompleteMultipartUpload": {"DaysAfterInitiation": 7}
    }]
  }'
```

🎵 *Closing music begins to fade in softly*

---

# SEGMENT 9: Wrap-Up, Best Practices & Project Ideas `(57:30 – 62:00)`

**[PRIYA]:** Speed round of best practices.

**[ALEX]:** Go.

**[PRIYA]:** Enable Block Public Access at the account level — first thing, always. Enable versioning on critical buckets. Set lifecycle policies to manage cost. Enable server-side encryption — SSE-S3 at minimum. Enforce HTTPS with a bucket policy. Use presigned URLs for user uploads and downloads — never proxy files through your server. Enable S3 server access logging for audit trails. Use Intelligent-Tiering when you're not sure about access patterns.

**[ALEX]:** Naming tip for buckets?

**[PRIYA]:** Globally unique, lowercase, no underscores, include your account ID or org name to avoid conflicts. Never put sensitive information in bucket names or object keys — they appear in logs and URLs.

**[ALEX]:** What should people build?

**[PRIYA]:** Four projects:

1. **File Upload Service** — Presigned URL generation API with Lambda, metadata stored in DynamoDB, S3 event triggers thumbnail generation, CloudFront for fast delivery.
2. **Static Website** — React or plain HTML site hosted on S3 + CloudFront with custom domain, HTTPS, and a GitHub Actions deploy pipeline.
3. **Data Lake Pipeline** — CSV files uploaded to S3 trigger Glue crawlers that update the schema catalog, queryable with Athena, visualized in QuickSight.
4. **Backup System** — Application backups with versioning, lifecycle policies, Cross-Region Replication to a second region, Object Lock in Governance mode for ransomware protection.

**[ALEX]:** That covers everything — from object storage fundamentals to event-driven architectures and data lakes. We talked durability, storage classes, security, CloudFront hosting, versioning, replication, Object Lock, and performance.

**[PRIYA]:** All the CLI commands, bucket policy templates, and lifecycle configuration examples are in the show notes. Your data is safe in S3.

**[ALEX]:** Until next time —

**[PRIYA]:** Stay durable. 🪣

🎵 *Outro music swells and fades out over 5 seconds*

---

## 🎯 Key Analogies Reference Card

| Concept | Analogy |
|---------|---------|
| S3 vs file system | Post office P.O. boxes vs filing cabinet — flat namespace, direct addressing by key |
| Eleven nines durability | 10 million objects → expect to lose one every 10,000 years |
| Bucket | Post office branch — globally unique location, objects addressed within it |
| Storage classes | Library stacks — frequently checked out books near front desk, rare ones in deep stacks |
| Intelligent-Tiering | Self-organizing library — moves books automatically based on checkout frequency |
| Presigned URL | Guest valet ticket — limited permission handed directly to the valet, hotel not involved |
| Block Public Access | Building-wide no-smoking sign — overrides individual room signs |
| Versioning | Time machine for your bucket — every change preserved, recoverable |
| Object Lock (Compliance) | Safe deposit box — even the bank can't open it early |
| S3 Event Notifications | Security camera trigger — upload happens, alarm fires, something responds |
| CloudFront + S3 | Library branches worldwide — local copy in every city for fast access |
| Transfer Acceleration | Express highway — files travel AWS backbone instead of public internet |

---

## 📚 Quick Reference

```bash
# ── BUCKET MANAGEMENT ────────────────────────────────
aws s3 mb s3://my-unique-bucket --region us-east-1
aws s3 rb s3://my-bucket --force    # Force delete with contents
aws s3 ls                            # List all buckets
aws s3 ls s3://my-bucket/ --recursive --human-readable

# ── UPLOAD & DOWNLOAD ────────────────────────────────
aws s3 cp file.txt s3://my-bucket/prefix/file.txt
aws s3 cp s3://my-bucket/prefix/file.txt ./local.txt
aws s3 sync ./local-dir s3://my-bucket/dir/ --delete
aws s3 sync s3://my-bucket/dir/ ./local-dir

# ── LARGE FILE MULTIPART ─────────────────────────────
aws s3 cp large-file.zip s3://my-bucket/ \
  --expected-size 5368709120 \    # 5GB in bytes
  --multipart-threshold 64MB \
  --multipart-chunksize 64MB

# ── PRESIGNED URL ────────────────────────────────────
aws s3 presign s3://my-bucket/file.txt --expires-in 3600

# ── SECURITY ─────────────────────────────────────────
# Block all public access (run once per account)
aws s3api put-public-access-block \
  --bucket my-bucket \
  --public-access-block-configuration \
    "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"

# Enable default encryption
aws s3api put-bucket-encryption \
  --bucket my-bucket \
  --server-side-encryption-configuration '{
    "Rules": [{"ApplyServerSideEncryptionByDefault": {"SSEAlgorithm": "AES256"}}]
  }'

# ── VERSIONING ───────────────────────────────────────
aws s3api put-bucket-versioning \
  --bucket my-bucket \
  --versioning-configuration Status=Enabled

aws s3api list-object-versions --bucket my-bucket --prefix photos/

# ── LIFECYCLE ────────────────────────────────────────
aws s3api put-bucket-lifecycle-configuration \
  --bucket my-bucket \
  --lifecycle-configuration file://lifecycle.json

# ── REPLICATION ──────────────────────────────────────
aws s3api put-bucket-replication \
  --bucket source-bucket \
  --replication-configuration file://replication.json

# ── STATIC WEBSITE ───────────────────────────────────
aws s3 website s3://my-bucket/ \
  --index-document index.html \
  --error-document error.html

# CloudFront invalidation after deploy
aws cloudfront create-invalidation \
  --distribution-id EDFDVBD6EXAMPLE \
  --paths '/*'

# ── S3 SELECT ────────────────────────────────────────
aws s3api select-object-content \
  --bucket my-bucket \
  --key data.csv \
  --expression "SELECT * FROM S3Object WHERE age > 30" \
  --expression-type SQL \
  --input-serialization '{"CSV": {"FileHeaderInfo": "Use"}}' \
  --output-serialization '{"CSV": {}}' \
  /dev/stdout
```

---

*Bucket List is produced independently. All trademarks belong to their respective owners.*
