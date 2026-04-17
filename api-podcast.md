# 🎙 API FRONTIERS — THE PODCAST FOR ENGINEERS WHO SHIP

**Episode 47**
## REST vs gRPC vs GraphQL: Building Production APIs the Right Way

> **Runtime:** ~60 minutes
> **Hosts:** Alex Chen & Priya Kapoor

---

### HOSTS

**ALEX CHEN** — Staff Engineer at a cloud-native fintech, 12 years building distributed systems. REST veteran, gRPC convert.

**PRIYA KAPOOR** — Principal API Architect at a global e-commerce platform, GraphQL evangelist, conference speaker.

---

## SEGMENT 1 — COLD OPEN `[0:00 – 2:00]`

*[ Upbeat intro music fades in, then under, then out ]*

**ALEX:** Welcome back to API Frontiers — I'm Alex Chen.

**PRIYA:** And I'm Priya Kapoor. If you're building anything on the web today, you are talking to an API. The question is, which kind?

**ALEX:** Today we're doing the episode a lot of you have been asking for. REST versus gRPC versus GraphQL. Not a surface-level comparison — we're going deep. Real production scenarios. War stories. Analogies that will actually stick.

**PRIYA:** By the end of this episode you will know exactly which protocol to reach for and why. No more cargo-culting REST just because that's what everyone does.

**ALEX:** Let's get into it.

*[ Music sting ]*

---

## SEGMENT 2 — SETTING THE STAGE `[2:00 – 8:00]`

### Why the choice matters

**PRIYA:** Before we judge the three contenders, let's agree on what we're optimising for. When you pick an API paradigm, you're making decisions that are really hard to undo. It's like choosing a city to build your house in — once the foundation is poured, you're not moving.

**ALEX:** Great analogy to kick off with. And the house here is your whole service mesh, your mobile apps, your third-party integrations…

**PRIYA:** Exactly. So the stakes are real. Now, to make sure everyone's on the same page — an API is simply a contract between a client and a server. You agree on: what can I ask for, in what format, over what transport, and what will you get back.

**ALEX:** And each of our three candidates — REST, gRPC, and GraphQL — answers those four questions very differently.

**PRIYA:** Think of it like ordering food. REST is a classic sit-down restaurant. You read the menu, you order a specific dish — let's say the salmon — and you get exactly the salmon with whatever sides the chef decided.

**ALEX:** You have no control over what comes with it. Sometimes the plate has too much on it, sometimes it's missing something. But the experience is familiar to everyone. Every waiter in the world knows how a sit-down restaurant works.

**PRIYA:** gRPC is more like a Japanese omakase counter. You're sitting right in front of the chef. Communication is tight, precise, fast. The chef — the server — is doing highly skilled work and you have a very direct, efficient conversation. But it's a specialised experience. Not every diner is comfortable with it.

**ALEX:** And GraphQL?

**PRIYA:** GraphQL is a buffet where you get to design your own plate. You walk up and say: I want exactly three pieces of salmon, half a scoop of rice, no vegetables, and a slice of that cake. You get precisely what you asked for. Amazing flexibility — but now you need to know what's available, and the restaurant needs to manage that buffet carefully.

**ALEX:** I love that. And just like choosing where to eat, the right choice depends on who your diners are and what experience you want to deliver. Let's meet each one properly.

---

## SEGMENT 3 — REST: THE WORKHORSE `[8:00 – 22:00]`

### What REST actually is

**ALEX:** REST — Representational State Transfer — was defined by Roy Fielding in his doctoral dissertation in the year 2000. Which, by software standards, makes it ancient.

**PRIYA:** And yet it powers the majority of public APIs today. GitHub, Twitter, Stripe, Twilio — REST everywhere.

**ALEX:** So what makes something REST? Six architectural constraints, but the ones that matter day-to-day are: statelessness, a uniform interface, and resources identified by URLs.

**PRIYA:** The statelessness one trips people up. Every single request must contain all the information the server needs to process it. The server has no memory of you between requests. It's like talking to someone with extreme short-term amnesia — every time you say hello, it's the first time you've met.

**ALEX:** Which is why we have tokens. You carry your identity with you — in the Authorization header — every single time. The server doesn't remember your session; you prove yourself on every call.

**PRIYA:** And resources are the core mental model. Not actions, resources. You don't call `getUser`. You `GET /users/42`. You don't call `deletePost`. You `DELETE /posts/99`. The HTTP verb does the heavy lifting.

**ALEX:** That's the uniform interface. GET, POST, PUT, PATCH, DELETE — five verbs to rule them all. Any developer who has ever touched HTTP knows these verbs. That is REST's superpower: ubiquity.

---

### REST in the real world — the good parts

**PRIYA:** Let me give a real scenario. You're building a public API for a SaaS product. Your customers are enterprise developers at companies you don't control. What's the most important thing?

**ALEX:** Ease of adoption. They need to be up and running fast. No special tooling, no code gen, no learning curve.

**PRIYA:** REST wins there, hands down. You can call a REST API with `curl`. With Postman. With any HTTP client in any language. No schemas to download, no IDL to compile. You paste a URL into a browser and you get data back. That frictionlessness is enormous.

**ALEX:** And caching. HTTP caching is baked in. GET requests are cacheable by default. CDNs understand them. Your infrastructure already knows how to handle them. That's free performance.

**PRIYA:** Stripe is a masterclass in REST API design. Their API has been mostly stable for over a decade. Clear resource hierarchy: `/v1/customers`, `/v1/customers/:id/subscriptions`. Versioned. Self-documenting via OpenAPI. It's a benchmark.

---

### REST — the real pain points

**ALEX:** Now the ugly truth. REST has two problems that bite you hard at scale: over-fetching and under-fetching.

**PRIYA:** Explain those for the folks at home.

**ALEX:** Over-fetching: you ask for a user, and the API returns fifty fields — name, email, address, phone, profile picture URL, subscription status, created date, last login, and forty more. Your mobile client only needed the name and profile picture. You just wasted bandwidth on forty-eight fields.

**PRIYA:** Under-fetching is the opposite. You need to show a user's profile page, which requires the user, their recent orders, and their saved addresses. REST gives you three separate endpoints. So you make three sequential requests: `GET /users/42`, `GET /users/42/orders`, `GET /users/42/addresses`. Three round trips. On a mobile connection, that's painful.

**ALEX:** This is called the N+1 problem. You need one user and their ten orders — that's eleven requests minimum. And if each order has line items… you see where this goes.

**PRIYA:** The workaround is to build custom endpoints — `GET /users/42/profile` that bundles everything together. But now you're maintaining a bespoke endpoint for every client's specific needs. Your API becomes a snowflake. It grows tentacles.

**ALEX:** Versioning is the other nightmare. You add a field, remove a field, rename something — suddenly you're on v2, and you're maintaining v1 forever because someone didn't read the deprecation notice.

---

### ✅ When to use REST

**PRIYA:** So given all that — when is REST the right call?

**ALEX:** **Public-facing APIs.** Any time your consumers are external developers you don't control. The adoption cost of anything else is too high.

**PRIYA:** **Simple CRUD services.** If you're managing resources — create, read, update, delete — REST is a perfect fit. Don't overcomplicate it.

**ALEX:** **Browser-based applications.** Browsers are built for HTTP. Service workers cache HTTP responses. CORS is designed for HTTP. REST fits naturally.

**PRIYA:** **Microservices that need to be discoverable and loosely coupled.** Teams that need to work independently. REST's uniform interface means any team can consume any service without tight coordination.

**ALEX:** The rule I tell junior engineers: if you're not sure, start with REST. It's the safe default. You can always migrate later. You can't easily un-ring the bell if you pick something more complex prematurely.

---

## SEGMENT 4 — gRPC: THE FORMULA ONE CAR `[22:00 – 38:00]`

### What gRPC actually is

**PRIYA:** gRPC — Google Remote Procedure Call — is Google's open-source RPC framework, released in 2016. It's built on two technologies: Protocol Buffers and HTTP/2.

**ALEX:** Let's unpack both. Protocol Buffers — protobuf — is a binary serialisation format. Instead of sending JSON like `"name": "Alex Chen"`, you define a schema in a `.proto` file and the data is compiled to a dense binary blob. Think of JSON as sending a handwritten letter and protobuf as sending a ZIP file — same information, radically smaller and faster to parse.

**PRIYA:** And HTTP/2 is the next generation of HTTP. It supports multiplexing — multiple requests over a single connection simultaneously. No head-of-line blocking. Header compression. And — crucially for gRPC — streaming.

**ALEX:** gRPC gives you four communication patterns. Unary — one request, one response, just like REST. Server streaming — server sends a stream of responses to one request. Client streaming — client sends a stream, server responds once. And bidirectional streaming — both sides streaming simultaneously.

**PRIYA:** That last one is wild. It's like a phone call instead of email. Both parties talking at the same time, in real time. That's not something REST can touch.

---

### The .proto file — your contract

**ALEX:** The centrepiece of gRPC is the `.proto` file. This is your API contract, written in the Protocol Buffer Interface Definition Language. You define your messages and your services in one place.

**PRIYA:** And here's the magic: you run the protobuf compiler, and it generates client and server code in any language you want. Go, Java, Python, Rust, C++, Node — the toolchain handles it. Your Go service and your Python client are talking through generated stubs that are type-safe by construction.

**ALEX:** This is a massive shift from REST. With REST, you write docs and hope the client developer reads them. With gRPC, the contract is the code. If it compiles, the types are right. Period.

**PRIYA:** The analogy I use: REST documentation is like a recipe written in English — descriptive, but humans have to interpret it correctly. A `.proto` file is like a CNC machine program — precise, unambiguous, the machine just runs it.

---

### gRPC war story — microservices at scale

**ALEX:** Let me give a concrete production scenario. At a previous company, we had a recommendation engine. It was called by twenty other internal services, hundreds of millions of times a day. The payloads were small — here's a user ID and a context, give me back ten product IDs.

**PRIYA:** Classic high-throughput, low-latency internal service.

**ALEX:** We started with REST over JSON. It worked. But at scale, the JSON parsing overhead was measurable. We were spending significant CPU just deserialising. We migrated to gRPC over protobuf and saw a **forty percent reduction in CPU** on that service. Latency dropped by roughly a third. Because we're talking microseconds per request multiplied by hundreds of millions of requests, that difference was enormous.

**PRIYA:** And the bidirectional streaming?

**ALEX:** We used server streaming for a different service — real-time inventory updates. Instead of clients polling "has the stock level changed?" every second — which is a thundering herd problem — the inventory service streams updates to all subscribers. The moment stock changes, every interested service knows. Push, not pull.

**PRIYA:** This is where gRPC is genuinely transformative. REST can't do this cleanly. You'd hack it with WebSockets or Server-Sent Events and you're reinventing the wheel badly.

---

### gRPC — the pain points

**PRIYA:** Now the honest downsides. gRPC is not browser-native. You can't call a gRPC service directly from a browser today without a proxy layer like gRPC-Web or Envoy in front. That's a significant constraint.

**ALEX:** And debugging is harder. Binary over the wire means you can't just open Chrome DevTools, look at the Network tab, and read the response. You need tooling. You need `grpcurl` or Postman's gRPC support. It's not impossible, but it's friction.

**PRIYA:** Schema changes are also more careful. Proto files are versioned, and you have to be very deliberate about backwards compatibility. Adding fields is safe. Removing or renumbering fields — that's a breaking change and you can corrupt data silently if you're not careful.

**ALEX:** And the cognitive overhead of the toolchain. Junior engineers who have never touched protobuf have a learning curve. It's not just a `curl` command anymore.

---

### ✅ When to use gRPC

**PRIYA:** So the verdict — when does gRPC shine?

**ALEX:** **Internal microservice-to-microservice communication.** This is the sweet spot. You control both ends. Performance matters. The binary efficiency and generated client stubs are worth the complexity.

**PRIYA:** **Real-time, streaming use cases.** Live dashboards, event feeds, IoT data streams, financial tick feeds — anything that needs continuous data flow.

**ALEX:** **Polyglot environments.** When your backend is Go, your ML service is Python, your mobile SDK is Swift, and your legacy system is Java — gRPC's code generation unifies them all under one contract.

**PRIYA:** **Latency-critical paths.** If you have a service that is in the critical path of every user request and is called thousands of times a second, the performance gains from gRPC are real and measurable.

**ALEX:** The Formula One analogy holds. A Formula One car is faster than any road car. But you can't drive your kids to school in it, you can't fuel it at a normal petrol station, and it requires a team of specialists to maintain. gRPC is your Formula One car. Use it on the track — your internal services — not on the school run.

---

## SEGMENT 5 — GRAPHQL: THE SWISS ARMY KNIFE `[38:00 – 52:00]`

### What GraphQL actually is

**PRIYA:** GraphQL was created at Facebook in 2012, open-sourced in 2015. The origin story tells you everything: Facebook's mobile app was fetching enormous JSON blobs over 3G networks. They were drowning in over-fetching. They needed a way for clients to ask for exactly what they needed — no more, no less.

**ALEX:** And so GraphQL was born. It's a query language — emphasis on language — for your API. There's a single endpoint, typically `/graphql`. And instead of calling different URLs for different resources, you send a query document describing exactly the shape of the data you want.

**PRIYA:** The classic example: I want a user, their name and email, and their last three orders with just the order ID and total. In REST, that's potentially three requests. In GraphQL, it's one query, and you get exactly that shape back. Nothing extra.

**ALEX:** The schema is at the heart of everything. You define your types — `User`, `Order`, `Product` — and the relationships between them form a graph. That's where the name comes from. Your data is a graph, and GraphQL lets you traverse it in one query.

---

### The three operations

**PRIYA:** GraphQL has three operation types. **Queries** — read data. **Mutations** — write or change data. **Subscriptions** — real-time, like gRPC streaming but over WebSockets.

**ALEX:** Subscriptions are underused and underappreciated. You can subscribe to events on the server and get pushed updates. Live sports scores, collaborative document editing, live order tracking — subscriptions handle it cleanly.

**PRIYA:** And the introspection system is a superpower. Any GraphQL API can be queried about its own schema. Tools like GraphiQL and Apollo Studio give you an interactive playground where you can explore the entire API, autocomplete your queries, and see documentation — all generated automatically from the schema. No separate doc site to maintain.

---

### GraphQL at scale — a real scenario

**ALEX:** Paint the picture for a complex product — you've run GraphQL in anger.

**PRIYA:** Absolutely. Our e-commerce platform serves a mobile app, a web app, a third-party seller portal, and a smart TV app. Each of these clients needs a slightly different shape of product data. Mobile needs small images and key specs. Web needs the full description, all images, related products. The TV app needs almost nothing — just title, price, and a single image.

**ALEX:** With REST, that's four custom endpoints — or one giant endpoint serving the worst case.

**PRIYA:** With GraphQL, it's one schema and four queries. Each client asks for exactly what it needs. Our **mobile data payload dropped by sixty percent** on the product detail page. That's real. That translates to faster load times, lower data bills for users, and better ratings on the App Store.

**ALEX:** And for the seller portal — sellers can query their products, inventory, and order data in a single round trip, joining across what used to be completely separate microservices.

**PRIYA:** GraphQL acts as a federation layer here. We use Apollo Federation, which lets you define a unified graph across multiple backend services. The Product service owns the product types, the Order service owns orders, the User service owns users — and GraphQL stitches them together transparently.

**ALEX:** The schema is the contract, but it's a living, flexible contract. Adding new fields is non-breaking. Old clients just don't query for them. You can deprecate fields and they remain available until you're ready to remove them.

---

### GraphQL — the honest pain points

**ALEX:** Now the things people don't tell you in conference talks.

**PRIYA:** The N+1 problem does not go away — it just moves. In GraphQL, if you fetch a list of users and for each user you resolve their orders, you've got a database query per user unless you implement DataLoader. DataLoader batches and caches resolver calls. But you have to build that. It doesn't come for free.

**ALEX:** Caching is also hard. HTTP caching relies on URLs. GraphQL has one URL. Everything is a POST request. Your CDN has no idea what to cache. You need client-side caching via Apollo Client or Relay, or you need persisted queries to get some GET-based caching back. Manageable, but real complexity.

**PRIYA:** Query complexity is a security concern. Because clients can ask for anything, a malicious or naive client can craft a deeply nested query — give me all users, and for each user all their orders, and for each order all its line items, and for each line item the product — and bring your server to its knees. You must implement query depth limiting and complexity analysis in production.

**ALEX:** And the resolver pattern has a learning curve. Instead of one handler for an endpoint, you have resolvers for every field on every type. That mental model shift trips people up.

---

### ✅ When to use GraphQL

**PRIYA:** So the decision criteria for GraphQL?

**ALEX:** **Multiple different client types with different data needs.** Mobile, web, TV, partner portals — if they all need different slices of the same underlying data, GraphQL is the answer.

**PRIYA:** **Rapid product development.** When your front-end team is iterating fast and they keep asking the back-end team for new endpoints — replace that with GraphQL. Front-end queries what they need without a back-end deploy.

**ALEX:** **Complex, interconnected data.** If your domain is naturally a graph — users have orders, orders have products, products have reviews, reviews have users — GraphQL maps directly to that structure in a way REST's flat URL hierarchy doesn't.

**PRIYA:** **Teams where front-end and back-end are moving at different speeds.** GraphQL creates a stable contract but maximum flexibility for the consumer.

**ALEX:** The buffet analogy really does hold. The power is real — but buffets are also expensive to run. You need kitchen infrastructure that most sit-down restaurants don't have. Know the cost before you commit.

---

## SEGMENT 6 — DECISION FRAMEWORK `[52:00 – 57:00]`

### The one-page cheat sheet

**PRIYA:** Alright, let's put this all together. Decision tree style.

**ALEX:** **Question one:** Is this a public-facing API that external developers will consume?

**PRIYA:** Yes → **REST.** The ubiquity and low friction win every time. Stop overthinking.

**ALEX:** **Question two:** Is this internal service-to-service communication with high throughput or real-time streaming requirements?

**PRIYA:** Yes → **gRPC.** Binary efficiency, generated clients, streaming. This is what it was made for.

**ALEX:** **Question three:** Do you have multiple client types — mobile, web, partners — all needing different shapes of the same data?

**PRIYA:** Yes → **GraphQL.** The query flexibility eliminates the N+1 endpoint problem.

**ALEX:** And the fourth question, which is actually the most important one: What is your team's experience? What does your infrastructure support?

**PRIYA:** Technology is half the decision. The team operating it is the other half. A REST API run by engineers who deeply understand it will outperform a GraphQL setup where nobody really knows what they're doing. Every time.

**ALEX:** And I'll add — these are not mutually exclusive. Our recommendation engine uses gRPC internally. Our product API uses GraphQL for the front-end. Our payment webhooks and partner integrations use REST. Pick the right tool for each job.

**PRIYA:** The analogy: you don't use a hammer for every job just because hammers are the most familiar tool. Screws exist. Wrenches exist. The best engineers know when to reach for which one.

---

## SEGMENT 7 — PRODUCTION TIPS `[57:00 – 59:30]`

### Hard-won wisdom

**ALEX:** Three rapid-fire production tips for each. REST first.

**PRIYA:** **Version from day one.** Always. `/v1/`. Even if you think the API will never change.

**ALEX:** **Design for idempotency.** Make sure your POST endpoints have idempotency keys. Stripe's approach is the gold standard.

**PRIYA:** **Use OpenAPI / Swagger.** Your documentation, client SDK generation, and contract testing can all flow from one spec file. Don't write docs by hand.

**ALEX:** gRPC tips. **Use deadlines everywhere.** Every RPC call should have a context with a timeout. Never fire and forget without a deadline.

**PRIYA:** **Keep your proto files in a shared repository — a schema registry.** Every team pulls from the same source of truth. Proto drift across services is a nightmare.

**ALEX:** **Enable TLS.** gRPC over cleartext is a red flag in production. Use mutual TLS for internal service authentication.

**PRIYA:** GraphQL tips. **Implement persisted queries in production.** You get caching, you get security — clients can only run pre-approved queries.

**ALEX:** **Instrument resolver performance from day one.** You need to know which resolvers are slow before your users find out.

**PRIYA:** **Use schema linting and breaking change detection in your CI pipeline.** Apollo Rover, GraphQL Inspector — these tools will save you from shipping breaking changes accidentally.

---

## SEGMENT 8 — WRAP-UP & OUTRO `[59:30 – 60:00]`

*[ Outro music begins to fade in ]*

**ALEX:** That's the episode. REST, gRPC, GraphQL — the when, the why, and the war stories.

**PRIYA:** The big takeaway: there is no universally correct answer. There is the right answer for your specific context, your team, your clients, and your performance requirements.

**ALEX:** Understand the trade-offs, make a deliberate choice, and build something great. Links to all the resources we mentioned — Stripe API docs, Apollo Federation, gRPC best practices — in the show notes.

**PRIYA:** If this episode helped you, leave us a review. And hit us on Twitter — I'm @PriyaAPIKapoor.

**ALEX:** And I'm @AlexChenEng. Until next time — ship well.

**PRIYA:** Ship well!

*[ Music out ]*

---

## SHOW NOTES & RESOURCES

**REST**
- Roy Fielding's original dissertation — ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm
- Stripe API Reference — stripe.com/docs/api
- OpenAPI Specification — openapis.org

**gRPC**
- gRPC Official Docs — grpc.io
- Protocol Buffers Language Guide — protobuf.dev
- grpcurl (CLI tool) — github.com/fullstorydev/grpcurl

**GraphQL**
- GraphQL Foundation — graphql.org
- Apollo Federation — apollographql.com/docs/federation
- DataLoader — github.com/graphql/dataloader
- GraphQL Inspector — graphql-inspector.com

---

*API Frontiers — Episode 47 | © 2026 API Frontiers Podcast*
