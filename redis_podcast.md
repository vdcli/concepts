# 🎙️ Cache Me If You Can
## The Redis Podcast — Building Real-World Applications

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
| 0:00–7:00 | What Is Redis? | In-memory, key-value, speed comparison, origin story |
| 7:00–14:00 | Installation & Mental Model | Redis CLI, SET/GET, TTL, key naming |
| 14:00–27:00 | Data Structures | Strings, Hashes, Lists, Sets, Sorted Sets, HyperLogLog |
| 27:00–35:00 | Caching Patterns | Cache-Aside, Write-Through, Write-Behind, eviction, stampede |
| 35:00–41:00 | Session Management | Stateless HTTP, external sessions, logout, TTL extension |
| 41:00–47:00 | Rate Limiting | Fixed Window, Sliding Window, atomic INCR, penalty boxes |
| 47:00–53:00 | Pub/Sub & Streams | Channels, fire-and-forget, persistent streams, consumer groups |
| 53:00–57:30 | Transactions & Lua | MULTI/EXEC, WATCH, optimistic locking, server-side scripts |
| 57:30–62:00 | Persistence & Scaling | RDB, AOF, Sentinel, Redis Cluster, hash slots |
| 62:00–65:00 | Best Practices & Projects | Monitoring, naming, TTLs, four project ideas |

---

# SEGMENT 1: Cold Open & What Is Redis? `(0:00 – 7:00)`

🎵 *Upbeat tech-themed intro music fades in, plays for 5 seconds, then fades under dialogue*

**[ALEX]:** Welcome to Cache Me If You Can — the podcast where we take backend technology, rip it apart, and put it back together so it actually makes sense. I'm Alex.

**[PRIYA]:** And I'm Priya. And today we are doing a deep dive — and I mean a proper deep dive — into Redis.

**[ALEX]:** Redis. I've seen that word everywhere. In job descriptions, in architecture diagrams, in that one Stack Overflow answer that saved my life at 2 AM.

**[PRIYA]:** Ha! We've all been there. So let's start at the very beginning. Redis stands for Remote Dictionary Server. But that name doesn't really do it justice.

**[ALEX]:** Remote Dictionary Server sounds like something that stores words.

**[PRIYA]:** Right? Here's a better way to think about it. Imagine your regular database — PostgreSQL, MySQL, whatever you use — is a massive filing cabinet in a storeroom. It's organized, it stores everything, but every time you want a document you have to walk to the storeroom, find the drawer, flip through the folders...

**[ALEX]:** That takes time.

**[PRIYA]:** Exactly. Now imagine you have a personal assistant sitting right next to you at your desk with a small notepad. If you've asked for something recently, they just hand it to you instantly from their notepad — no trip to the storeroom.

**[ALEX]:** Oh! Redis is the personal assistant.

**[PRIYA]:** Redis is the personal assistant. It lives in RAM — in memory — which is orders of magnitude faster than reading from disk. We're talking microseconds versus milliseconds.

**[ALEX]:** How much faster are we actually talking here?

**[PRIYA]:** A typical Redis read is under a millisecond. A database query hitting disk can be 10, 50, even 500 milliseconds depending on complexity. So you're looking at somewhere between 10x and 1000x faster.

**[ALEX]:** That's wild. And Redis is open source?

**[PRIYA]:** It started as open source, created by Salvatore Sanfilippo — the community calls him antirez — back in 2009. It's gone through some licensing changes recently, but the core Redis you'll use for most applications is still freely available, and there's also Valkey which is the truly open-source fork if that matters to you.

**[ALEX]:** OK so we know what it is. Let's talk about who uses it.

**[PRIYA]:** Twitter uses Redis for their timeline caches. GitHub uses it. Instagram, Pinterest, Airbnb, Stack Overflow — pretty much any high-scale web company has Redis somewhere in their stack.

**[ALEX]:** Which means if you're going for a backend role at basically any serious company, you need to know this.

**[PRIYA]:** Absolutely. And the good news is — it's genuinely fun to learn.

🎵 *Short musical sting — segment transition*

---

# SEGMENT 2: Installation & The Redis Mental Model `(7:00 – 14:00)`

**[PRIYA]:** OK so before we get into all the cool data structures, let's just get Redis running so people can follow along.

**[ALEX]:** Yes! How do you install it?

**[PRIYA]:** On a Mac you do `brew install redis` and then `brew services start redis`. On Ubuntu it's `sudo apt install redis-server`. On Windows, honestly, just use Docker — `docker run -d -p 6379:6379 redis`.

**[ALEX]:** 6379 — is that port significant?

**[PRIYA]:** It's the default Redis port. It's actually the number that MERZ — the name of a mobile phone — maps to on a phone keypad. antirez named it after a joke. Totally random but now it's iconic.

**[ALEX]:** Love it. OK so once it's running, how do you talk to it?

**[PRIYA]:** You use the Redis CLI — just type `redis-cli` in your terminal and you're dropped into an interactive shell. The most basic command is SET and GET.

**[ALEX]:** Like `SET name Alex` and `GET name`?

**[PRIYA]:** Exactly! And Redis responds with OK on the SET and "Alex" on the GET. That right there is the whole mental model — Redis is fundamentally a key-value store. Every piece of data has a key, and you use that key to retrieve it.

**[ALEX]:** It's basically a giant dictionary or HashMap.

**[PRIYA]:** Exactly. Like a Python dict or a JavaScript object — but one that can hold hundreds of gigabytes of data, handle millions of operations per second, and be shared across your entire fleet of servers.

**[ALEX]:** So you can scale Redis horizontally without changing your application code.

**[PRIYA]:** That's the key insight. It's a networked, shared, in-memory dictionary. And because it's separate from your application, it doesn't matter if you have 1 server or 100 servers — they all see the same data.

**[ALEX]:** Let's talk about keys for a second. Are there naming conventions?

**[PRIYA]:** Great question. Use colons as separators — like `user:1001:profile` or `session:abc123` or `rate_limit:ip:192.168.1.1`. Think of it like a namespace. It makes keys readable and lets you group related data.

**[ALEX]:** And keys can have a TTL — a time to live?

**[PRIYA]:** Yes! This is huge. You can say `SET mykey myvalue EX 3600` — that EX means expire in 3600 seconds, one hour. After that, Redis automatically deletes the key. This is perfect for session tokens, caches, rate limits — anything that should naturally expire.

**[ALEX]:** It's like a self-cleaning database.

**[PRIYA]:** Ha! That's a great way to put it. No more cron jobs to clean up stale data.

🎵 *Transition music sting*

---

# SEGMENT 3: Data Structures — The Real Power of Redis `(14:00 – 27:00)`

**[ALEX]:** Alright, so Redis isn't just strings right? There are multiple data types?

**[PRIYA]:** This is what separates Redis from a simple cache. Redis has Strings, Lists, Sets, Sorted Sets, Hashes, Bitmaps, HyperLogLog, Streams, and Geospatial indexes. Each one is perfectly designed for specific use cases.

**[ALEX]:** Let's go through them. Starting with Strings.

**[PRIYA]:** Strings are the most fundamental. Despite the name, they can store text, integers, or even binary data up to 512 megabytes. You can do `SET counter 0` and then `INCR counter` — Redis atomically increments it. No race conditions, no locks needed.

**[ALEX]:** Atomic operations — meaning no two requests can step on each other.

**[PRIYA]:** Right. Think of it like a bank teller. If two customers both try to deposit money at the exact same moment, the teller handles one, then the other — they don't accidentally double-count or miss one. Redis does the same for operations.

**[ALEX]:** OK. Next up — Hashes.

**[PRIYA]:** Hashes are perfect for storing objects. Imagine you want to store a user profile. Instead of serializing the whole object to JSON and storing it as a string, you can do:

```
HSET user:1001 name "Alice" email "alice@example.com" age 30
```

Each field is stored separately.

**[ALEX]:** And you can get just one field?

**[PRIYA]:** Yes! `HGET user:1001 email` — you don't have to retrieve and deserialize the whole object just to get one field. It's much more efficient. Think of a Hash like a hotel's guest card — the card has a room number, name, check-in date all in one place, and you can update just the check-in date without rewriting everything.

**[ALEX]:** Love that. What about Lists?

**[PRIYA]:** Lists are ordered sequences of strings. You can push to the left or right — `LPUSH` and `RPUSH`. They're doubly-linked lists internally. The classic use case is a job queue or a recent activity feed.

**[ALEX]:** Like push a job onto the list, and a worker pops it off?

**[PRIYA]:** Exactly. `BRPOP queue 0` — the B means blocking. A worker sits there waiting. The moment something appears in the queue, Redis immediately hands it to the worker. It's like a restaurant kitchen — tickets come in on one end, chefs grab them from the other.

**[ALEX]:** That's elegant. Sets?

**[PRIYA]:** Sets are unordered collections of unique strings. The killer feature is set operations — `SUNION`, `SINTER`, `SDIFF` for union, intersection, and difference. Think of it like Venn diagrams that can process millions of items instantly.

**[ALEX]:** Real-world use case?

**[PRIYA]:** Social networks. Store Alice's followers in `SET followers:alice`. Store Bob's followers in `SET followers:bob`. `SINTER followers:alice followers:bob` instantly gives you mutual followers. No JOIN query, no full table scan.

**[ALEX]:** Oh wow. Instagram could literally be using this.

**[PRIYA]:** They did! OK now Sorted Sets — this is my personal favorite.

**[ALEX]:** Why?

**[PRIYA]:** Sorted Sets are like Sets but every member has a score. Redis keeps everything sorted by score automatically. Think of a leaderboard — you add players with their scores:

```
ZADD leaderboard 9500 "Alice"
ZADD leaderboard 8200 "Bob"
ZADD leaderboard 11000 "Charlie"
```

Then `ZREVRANGE leaderboard 0 2` gives you the top 3 in order. Instantly.

**[ALEX]:** And updating a score?

**[PRIYA]:** `ZINCRBY leaderboard 500 "Alice"` — add 500 to Alice's score. Redis re-sorts automatically. It's like an Excel spreadsheet that re-sorts itself the moment you change any value.

**[ALEX]:** That's incredibly fast for a leaderboard with millions of users.

**[PRIYA]:** It's O(log N) time complexity. Even with 10 million users, it's blazing fast. `ZRANK leaderboard "Alice"` tells you her exact rank. You can even do range queries — get all players with scores between 5000 and 10000.

**[ALEX]:** One more — HyperLogLog. What on earth is that?

**[PRIYA]:** Ha! Great name right? HyperLogLog is for counting unique things with very low memory. Say you want to count unique visitors to your website. You could store every user ID in a Set — but with 100 million users that's gigabytes of memory. HyperLogLog uses just 12 kilobytes — regardless of how many items — and gives you an approximate count within about 0.8% accuracy.

**[ALEX]:** Wait, 12 kilobytes for 100 million items?

**[PRIYA]:** Yes! It's probabilistic — like sampling. Think of it like a polling company that surveys 1,000 people to estimate how 300 million people will vote. The sample is tiny but the estimate is remarkably accurate.

**[ALEX]:** That's mind-bending. `PFADD` and `PFCOUNT` are the commands right?

**[PRIYA]:** Exactly. PF stands for Philippe Flajolet, the mathematician who invented the algorithm.

🎵 *Transition music sting*

---

# SEGMENT 4: Caching — Redis's Most Common Use Case `(27:00 – 35:00)`

**[ALEX]:** OK let's get practical. Caching. Walk me through how you actually implement this.

**[PRIYA]:** Sure. The pattern is called Cache-Aside, and it's the most common approach. Here's how it works: when your app needs data, it first checks Redis. If the data is there — that's a cache hit — return it immediately. If it's not — that's a cache miss — go to the database, get the data, store it in Redis, then return it.

**[ALEX]:** So Redis is the first stop, database is the fallback.

**[PRIYA]:** Right. In pseudocode it's essentially:

```python
def get_user(user_id):
    cached = redis.get(f"user:{user_id}")
    if cached:
        return deserialize(cached)          # Cache hit ✅
    user = db.query("SELECT * FROM users WHERE id = ?", user_id)
    redis.setex(f"user:{user_id}", 3600, serialize(user))
    return user                             # Cache miss — now populated
```

**[ALEX]:** And you set a TTL so it doesn't get stale?

**[PRIYA]:** Always set a TTL. This is the rule. Imagine you cache a product price. If the price changes in the database but there's no TTL on the cache, you're serving stale prices forever. With a TTL of, say, 5 minutes, you accept that users might see a slightly old price for up to 5 minutes, but it automatically heals itself.

**[ALEX]:** Eventual consistency.

**[PRIYA]:** Exactly. Now let's talk about the other caching patterns. **Write-Through**: every write to the database is also written to Redis simultaneously. Always fresh, never stale. But double the write operations.

**[ALEX]:** Good for read-heavy workloads where staleness is not acceptable.

**[PRIYA]:** Exactly. Then there's **Write-Behind** — also called Write-Back. You write to Redis first, return success to the user immediately, and then asynchronously sync to the database in the background.

**[ALEX]:** That seems risky. What if Redis crashes before the data reaches the database?

**[PRIYA]:** Sharp question! That's the tradeoff — you get insanely fast writes but potential data loss. Use it when you can tolerate losing a tiny bit of data — analytics counters, view counts, that sort of thing. Not for financial transactions.

**[ALEX]:** Let's talk about cache eviction. What happens when Redis runs out of memory?

**[PRIYA]:** This is important to configure. Redis has several eviction policies. `allkeys-lru` is the most common — LRU means Least Recently Used. When Redis needs space, it evicts the key that hasn't been accessed in the longest time. Like a librarian making space on the shelf by removing books nobody has checked out recently.

**[ALEX]:** Sensible. What are the others?

**[PRIYA]:** `volatile-lru` only evicts keys that have a TTL set. `allkeys-random` evicts randomly. `noeviction` returns an error when full — don't use this in production caches! You configure it in redis.conf with:

```
maxmemory-policy allkeys-lru
maxmemory 2gb
```

**[ALEX]:** And what about cache stampede? I've heard that term.

**[PRIYA]:** Great one to know. Imagine your cache key for a popular product page expires at exactly midnight. Suddenly 10,000 users all try to load that page simultaneously. None of them find the cache. All 10,000 requests hit your database at the same moment. Your database has a heart attack.

**[ALEX]:** That sounds catastrophic.

**[PRIYA]:** It is! Solutions include adding random jitter to TTLs — instead of exactly 3600 seconds, use 3600 plus a random 0–60 seconds — so keys don't all expire simultaneously. Or use a mutex lock — only one request rebuilds the cache while others wait.

**[ALEX]:** Prevention is better than cure.

**[PRIYA]:** Always.

🎵 *Transition music sting*

---

# SEGMENT 5: Session Management & Authentication `(35:00 – 41:00)`

**[ALEX]:** Let's talk sessions. This is something every web app deals with.

**[PRIYA]:** Sessions are one of Redis's absolute sweetspots. Here's the problem: HTTP is stateless. Every request is independent. So how does a server know that this request is from a logged-in user?

**[ALEX]:** Traditionally you use a session cookie that maps to server-side session storage.

**[PRIYA]:** Right. And the old way was to store sessions in memory on a single server. That works great until you have two servers — suddenly half your users get logged out depending on which server they hit.

**[ALEX]:** The sticky session problem.

**[PRIYA]:** Exactly. Redis solves this because it's external — all your servers connect to the same Redis instance. Session is stored there, any server can retrieve it.

**[ALEX]:** Walk me through the flow.

**[PRIYA]:** User logs in, you verify credentials against your database. You generate a random session token — something like a UUID. You store it in Redis:

```
SETEX session:abc-uuid-123 86400 '{"userId": 1001, "role": "admin"}'
```

The 86400 is the TTL — 24 hours.

**[ALEX]:** And you send the token to the browser as a cookie.

**[PRIYA]:** Exactly. On the next request, the browser sends the cookie. Your server grabs the token, does `GET session:abc-uuid-123` in Redis, gets the user data back in under a millisecond. Done.

**[ALEX]:** And logging out is just deleting the key.

**[PRIYA]:** `DEL session:abc-uuid-123`. The session instantly dies. This is something JWTs can't do easily — once a JWT is issued, you can't un-issue it without maintaining a blocklist. With Redis sessions, logout is immediate and guaranteed.

**[ALEX]:** Oh that's a really good point. JWT vs Redis sessions is a whole debate.

**[PRIYA]:** It is! JWTs are stateless — good for distributed microservices where you don't want a central session store. Redis sessions are stateful — slightly more overhead but you get instant revocation, easy session inspection, the ability to store extra metadata. Choose based on your needs.

**[ALEX]:** And you can extend sessions too — touch the TTL on each request.

**[PRIYA]:** `EXPIRE session:abc-uuid-123 86400` — reset the timer on every active request. So the session expires 24 hours after the last activity, not 24 hours after login.

🎵 *Transition music sting*

---

# SEGMENT 6: Rate Limiting — Protecting Your APIs `(41:00 – 47:00)`

**[ALEX]:** Rate limiting is something I've always found tricky to implement. Redis makes it elegant?

**[PRIYA]:** Super elegant. Let me walk you through the Fixed Window algorithm first — simplest to understand. You have a key per user per time window: `rate_limit:user:1001:2024-01`. You INCR that key on every request. If the count exceeds your limit — say 100 — you return HTTP 429 Too Many Requests.

**[ALEX]:** And the key auto-expires at the end of the window.

**[PRIYA]:** Right, you set it to expire in 60 seconds on the first request. Here's the atomic way to do it:

```
SET rate:user:1001 0 EX 60 NX    # NX = only set if doesn't exist
INCR rate:user:1001               # Atomic increment
```

This is safe because INCR is atomic.

**[ALEX]:** What's the downside of fixed window?

**[PRIYA]:** The boundary problem. If your window is per-minute and the limit is 100, a user can make 100 requests at 11:59 and 100 more at 12:00 — 200 requests in 2 seconds. That's a burst attack.

**[ALEX]:** Yikes. So what's the better approach?

**[PRIYA]:** Sliding Window Log. You store a sorted set of timestamps:

```
ZADD rate:user:1001 {timestamp} {timestamp}
ZREMRANGEBYSCORE rate:user:1001 0 {60-seconds-ago}
ZCARD rate:user:1001
# If count < 100, allow the request
```

**[ALEX]:** And since it's a sliding window, you can't game the boundary.

**[PRIYA]:** Exactly. The window always looks at the last 60 seconds, not the current calendar minute. The downside is higher memory usage — you're storing every timestamp.

**[ALEX]:** What about limiting by IP?

**[PRIYA]:** Same patterns, just different key prefixes. `rate:ip:192.168.1.1` instead of `rate:user:1001`. You can layer them — per-user limits AND per-IP limits. This is what Nginx, Kong, AWS API Gateway all do under the hood — often with Redis as the backend store.

**[ALEX]:** This is also how DDoS protection works at a basic level.

**[PRIYA]:** Exactly. And you can add penalty boxes — if an IP exceeds the limit, block it for an hour:

```
SET blocked:ip:x.x.x.x 1 EX 3600
```

🎵 *Transition music sting*

---

# SEGMENT 7: Pub/Sub & Redis Streams — Real-Time Communication `(47:00 – 53:00)`

**[ALEX]:** Pub/Sub. I hear this in the context of event-driven systems. How does Redis handle it?

**[PRIYA]:** Redis Pub/Sub is a fire-and-forget messaging system. Publishers send messages to channels. Subscribers listen on channels and receive those messages in real-time.

**[ALEX]:** Think of it like a radio station?

**[PRIYA]:** Perfect analogy! The radio station broadcasts on a frequency — that's the channel. Anyone tuned in receives the broadcast. But if you missed it, it's gone — Pub/Sub doesn't store messages.

**[ALEX]:** That's the key limitation.

**[PRIYA]:** Right. If a subscriber goes offline for 10 minutes and comes back, they missed all messages during that time. For real-time notifications that are OK to lose — like a user's cursor position in a collaborative doc — Pub/Sub is perfect.

```
# Publisher
PUBLISH notifications "User Alice just joined"

# Subscriber (in another terminal)
SUBSCRIBE notifications
```

**[ALEX]:** What if you need delivery guarantees?

**[PRIYA]:** That's where Redis Streams come in. Added in Redis 5.0. Think of Streams like an append-only log. Like a ledger or an airline's flight manifest — every entry is permanent, ordered, and has a unique ID. Consumers can read from any position and they never miss a message.

**[ALEX]:** How do you add to a stream?

**[PRIYA]:** `XADD events * action buy product laptop` — the asterisk auto-generates a timestamp-based ID. To read:

```
XREAD COUNT 10 STREAMS events 0        # Read from the beginning
XREAD BLOCK 0 STREAMS events $         # Wait for new messages
```

**[ALEX]:** And consumer groups?

**[PRIYA]:** Consumer groups let multiple workers share a stream. Imagine a stream of orders. You have 5 worker processes. Each order should be processed exactly once.

```
XREADGROUP GROUP workers worker1 COUNT 5 STREAMS orders >
```

The `>` means give me undelivered messages. Worker 1 gets orders 1–5, worker 2 gets 6–10. No duplicates.

**[ALEX]:** And if a worker crashes?

**[PRIYA]:** Messages go into a Pending Entries List — PEL. Another worker can claim them with `XCLAIM`. This gives you at-least-once delivery guarantees. Redis Streams is basically Kafka-lite — not as powerful but no extra infrastructure required.

**[ALEX]:** When would you choose Kafka over Redis Streams?

**[PRIYA]:** Retention of millions of events for days or weeks, massive throughput requirements, complex consumer topologies. Redis Streams is perfect for moderate-scale event processing that needs to stay simple.

🎵 *Transition music sting*

---

# SEGMENT 8: Transactions, Lua Scripts & Atomic Operations `(53:00 – 57:30)`

**[ALEX]:** What about transactions? Can Redis do multi-step operations atomically?

**[PRIYA]:** Yes! Redis has MULTI and EXEC. You wrap commands in a block:

```
MULTI
DEBIT account:alice 100
CREDIT account:bob 100
EXEC
```

Redis buffers them all and executes them as a single atomic unit.

**[ALEX]:** What's WATCH for?

**[PRIYA]:** WATCH is for optimistic locking. Say you're implementing a transfer — debit one account, credit another. You WATCH both account keys. If anything modifies those keys between your WATCH and your EXEC, the transaction aborts. You retry.

```
WATCH account:alice account:bob
MULTI
DEBIT account:alice 100
CREDIT account:bob 100
EXEC    # Returns nil if any WATCHed key changed — retry!
```

**[ALEX]:** Like a compare-and-swap.

**[PRIYA]:** Exactly. And for complex atomic logic, use Lua scripts. `EVALSHA` lets you run Lua code server-side. Because Lua executes on the server, the entire script is atomic — no other command can run in between.

**[ALEX]:** So for our rate limiter — the check-and-increment — we could wrap that in Lua.

**[PRIYA]:** Exactly right. One Lua script that atomically checks the count, increments if below the limit, returns whether the request is allowed. No race conditions possible.

```lua
local current = redis.call('INCR', KEYS[1])
if current == 1 then
    redis.call('EXPIRE', KEYS[1], ARGV[1])
end
if current > tonumber(ARGV[2]) then
    return 0  -- Rate limited
end
return 1      -- Allowed
```

**[ALEX]:** Incredibly powerful.

**[PRIYA]:** And Lua scripts are cached by Redis with a SHA hash — so you don't re-send the script every time, just the hash. Very efficient.

🎵 *Transition music sting*

---

# SEGMENT 9: Persistence, Replication & High Availability `(57:30 – 62:00)`

**[ALEX]:** We've been talking about Redis as in-memory. But what happens if the server restarts? Does all the data vanish?

**[PRIYA]:** Great question. Redis has two persistence options. **RDB** — Redis Database Backup — takes point-in-time snapshots. Like a photograph of your data. Fast restarts, compact files, but you can lose data since the last snapshot.

**[ALEX]:** And the other option?

**[PRIYA]:** **AOF** — Append Only File. Every write operation is logged to a file in real-time. Like a continuous transaction log or a black box recorder on a plane. You can replay it to reconstruct the exact state. Slower writes, larger files, but minimal data loss — you can configure it to sync every second or even every write.

**[ALEX]:** In production, which do you use?

**[PRIYA]:** Most production setups use both — RDB for fast restarts and AOF for durability. You configure:

```
appendonly yes
save 900 1    # Save if at least 1 key changed in 900 seconds
save 300 10   # Save if at least 10 keys changed in 300 seconds
save 60 10000 # Save if at least 10,000 keys changed in 60 seconds
```

**[ALEX]:** What about replication for high availability?

**[PRIYA]:** Redis supports primary-replica replication. One primary accepts writes. Replicas sync from the primary and can serve reads. Redis Sentinel monitors the primary — if it goes down, Sentinel promotes a replica to primary automatically.

```
# In replica's redis.conf
replicaof primary-host 6379
```

**[ALEX]:** Automatic failover.

**[PRIYA]:** And for horizontal scaling — sharding across multiple nodes — you use Redis Cluster. It automatically partitions your data across up to 1,000 nodes using 16,384 hash slots. Each key is assigned to a slot, each slot to a node.

**[ALEX]:** So you can scale Redis horizontally without changing your application code.

**[PRIYA]:** Mostly, yes. Your Redis client handles the routing. This is how companies like Twitter handle billions of keys — they run Redis Cluster with dozens of nodes.

🎵 *Closing music begins to fade in softly*

---

# SEGMENT 10: Wrap-Up, Best Practices & What to Build Next `(62:00 – 65:00)`

**[PRIYA]:** OK — let's do our speed-round of best practices before we wrap up.

**[ALEX]:** Hit me.

**[PRIYA]:** Always set TTLs on cache keys. Use consistent key naming conventions with colons. Set a `maxmemory` limit and an eviction policy. Never use `KEYS *` in production — it's O(N) and blocks Redis. Use `SCAN` instead.

**[ALEX]:** Monitor your cache hit rate.

**[PRIYA]:** Yes — if your hit rate is below 80%, your caching strategy needs work. Use `INFO stats` to see `keyspace_hits` and `keyspace_misses`. And use Redis pipelines when you need to send many commands — batch them up instead of sending one by one.

**[ALEX]:** What should listeners build to practice?

**[PRIYA]:** Four projects, in order of complexity:

1. **URL Shortener** — The short code maps to the long URL with a visit counter using `INCR`.
2. **Real-Time Leaderboard** — Using Sorted Sets with live rank updates.
3. **Job Queue** — With Lists and `BRPOP` where workers process background tasks.
4. **Real-Time Chat System** — Using Pub/Sub with message history stored in a Stream.

**[ALEX]:** Those four projects cover pretty much everything we talked about today.

**[PRIYA]:** Intentionally! And once you've built those, you'll have a genuine feel for when Redis is the right tool. The answer, by the way, is almost always — Redis belongs somewhere in your stack.

**[ALEX]:** Alright — that is a wrap on Cache Me If You Can's deep dive into Redis. We covered what Redis is, data structures, caching patterns, sessions, rate limiting, pub/sub, streams, transactions, persistence, and scaling.

**[PRIYA]:** Links to all the documentation, the example code for everything we discussed, and a Redis cheatsheet are in the show notes. Go build something amazing.

**[ALEX]:** If you enjoyed this episode, please leave a review and share it with a fellow developer. Until next time —

**[PRIYA]:** Stay curious and keep caching. 🔴

🎵 *Outro music swells and fades out over 5 seconds*

---

## 🎯 Key Analogies Reference Card

| Concept | Analogy |
|---------|---------|
| Redis vs Database | Filing cabinet in storeroom vs. personal assistant with notepad at your desk |
| Atomic INCR | Bank teller handling one customer at a time — no double-counting |
| Redis Hashes | Hotel guest card — update any field independently without rewriting everything |
| Redis Lists as Queue | Restaurant ticket system — orders in one end, chefs grab from the other |
| Set Operations | Venn diagrams for millions of items — instant union/intersection/difference |
| Sorted Sets | Excel spreadsheet that auto-sorts the moment any value changes |
| HyperLogLog | Polling company surveying 1,000 to estimate how 300 million will vote |
| Cache TTL | Self-cleaning database — no manual cron jobs needed |
| Cache Stampede | Everyone calling the same phone number at midnight — stagger the calls! |
| Pub/Sub | Radio station — broadcast goes out, if you miss it, it's gone |
| Redis Streams | Airline flight manifest — every entry permanent, ordered, and replayable |
| AOF Persistence | Black box flight recorder — log every action to replay later |
| Redis Cluster | Post office sorting system — 16,384 slots assigned across nodes |

---

## 📚 Quick Command Cheatsheet

```bash
# ── STRINGS ──────────────────────────────────────────
SET key value EX 3600       # Set with 1hr expiry
GET key                     # Retrieve value
INCR counter                # Atomic increment
INCRBY counter 5            # Atomic increment by 5

# ── HASHES ───────────────────────────────────────────
HSET user:1001 name "Alice" email "a@b.com"
HGET user:1001 email
HGETALL user:1001

# ── LISTS ────────────────────────────────────────────
RPUSH queue "job1"          # Push to right (enqueue)
BRPOP queue 0               # Blocking pop from left (dequeue)
LRANGE mylist 0 -1          # Get all elements

# ── SETS ─────────────────────────────────────────────
SADD followers:alice "bob" "charlie"
SINTER followers:alice followers:bob   # Mutual followers
SUNION set1 set2

# ── SORTED SETS ──────────────────────────────────────
ZADD leaderboard 9500 "Alice"
ZINCRBY leaderboard 500 "Alice"
ZREVRANGE leaderboard 0 9 WITHSCORES   # Top 10
ZRANK leaderboard "Alice"

# ── HYPERLOGLOG ──────────────────────────────────────
PFADD visitors user123
PFCOUNT visitors

# ── STREAMS ──────────────────────────────────────────
XADD events * action "buy" product "laptop"
XREAD COUNT 10 STREAMS events 0
XREADGROUP GROUP workers w1 COUNT 5 STREAMS orders >

# ── PUB/SUB ──────────────────────────────────────────
PUBLISH channel "message"
SUBSCRIBE channel

# ── TRANSACTIONS ─────────────────────────────────────
MULTI
SET key1 val1
SET key2 val2
EXEC

# ── TTL & EXPIRY ─────────────────────────────────────
EXPIRE key 3600             # Set TTL in seconds
TTL key                     # Check remaining TTL
PERSIST key                 # Remove TTL

# ── ADMIN ────────────────────────────────────────────
INFO stats                  # Hit/miss stats
SCAN 0 MATCH user:* COUNT 100   # Safe key iteration
CONFIG SET maxmemory 2gb
CONFIG SET maxmemory-policy allkeys-lru
```

---

*Cache Me If You Can is produced independently. All trademarks belong to their respective owners.*
