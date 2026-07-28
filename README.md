# 📘 System Design Fundamentals

> **A comprehensive guide to distributed systems, architecture patterns, and system design principles for engineers and interview preparation.**

---

## 📋 Table of Contents

- [Core Architecture Principles](#-core-architecture-principles)
- [Networking and Communication](#-networking-and-communication)
- [Database and Storage Internals](#-database-and-storage-internals)
- [Reliability and Fault Tolerance](#-reliability-and-fault-tolerance)
- [Caching and Messaging](#-caching-and-messaging)
- [Observability and Security](#-observability-and-security)

---

## 🏗️ Core Architecture Principles

### 1. Vertical vs Horizontal Scaling

#### What is it?
- **Vertical Scaling (Scale Up)**: Adding more power (CPU, RAM, storage) to a single machine
- **Horizontal Scaling (Scale Out)**: Adding more machines to distribute the workload

#### Why do we need it?
- Applications grow, and a single server cannot handle increasing traffic and data
- Need to maintain performance, availability, and handle growth

#### How does it work?
- **Vertical**: Upgrade existing hardware (more cores, RAM, faster SSDs)
- **Horizontal**: Add new servers behind a load balancer

#### Real Example
- **Vertical**: Upgrading AWS EC2 from t2.micro to t2.large
- **Horizontal**: Adding more web servers behind an AWS ELB (Elastic Load Balancer)

#### Advantages
| Vertical | Horizontal |
|----------|------------|
| Simple to implement | Unlimited scalability |
| No application changes needed | Better fault tolerance |
| Less complex networking | Cost-effective at scale |
| Lower latency between components | Geographic distribution possible |

#### Disadvantages
| Vertical | Horizontal |
|----------|------------|
| Hardware limits | Complex architecture |
| Expensive at high end | Requires stateless services |
| Single point of failure | Load balancing needed |
| Downtime for upgrades | Data consistency challenges |

#### Interview Questions
- **Q:** When would you choose vertical over horizontal scaling?
  **A:** During early growth stages, for legacy applications, or when your database can't be easily distributed.
  
- **Q:** How do you handle state with horizontal scaling?
  **A:** Make services stateless by moving session data to external stores (Redis, databases) or use sticky sessions.

---

### 2. CAP Theorem

#### What is it?
A distributed system can only guarantee **two** of three properties simultaneously:
- **Consistency (C)**: All nodes see the same data at the same time
- **Availability (A)**: Every request receives a response
- **Partition Tolerance (P)**: System continues despite network failures

#### Why do we need it?
Helps make informed trade-offs when designing distributed systems based on business requirements.

#### How does it work?
During a network partition (P), you must choose:
- **CP (Consistency + Partition Tolerance)**: Sacrifice availability for data correctness
- **AP (Availability + Partition Tolerance)**: Sacrifice consistency for uptime

#### Real Example
- **CA**: Single-node databases (PostgreSQL, MySQL - but can't tolerate partitions)
- **CP**: MongoDB (chooses consistency during partitions)
- **AP**: Cassandra, DynamoDB (chooses availability during partitions)

#### Advantages
- Provides clear framework for design decisions
- Prevents unrealistic expectations
- Helps justify trade-offs to stakeholders

#### Disadvantages
- Simplifies reality (network issues are complex)
- Doesn't cover latency considerations
- PACELC extends it with more nuance

#### Interview Questions
- **Q:** Why can't we have all three in a distributed system?
  **A:** During a network partition, you either serve stale data (choose A) or become unavailable (choose C).
  
- **Q:** Which would you choose for a banking system vs a social media feed?
  **A:** Banking → CP (consistency critical). Social media → AP (availability more important).

---

### 3. PACELC Theorem

#### What is it?
Extends CAP theorem:
- **If Partition**: Choose Availability or Consistency (CAP)
- **Else (normal operation)**: Choose Latency or Consistency

#### Why do we need it?
Even without network failures, there's a trade-off between speed and consistency.

#### How does it work?
Systems must decide:
- **PA/EL**: Partition → Availability; Else → Latency (Cassandra, DynamoDB)
- **PC/EC**: Partition → Consistency; Else → Consistency (MongoDB)
- **PC/EL**: Partition → Consistency; Else → Latency (rare)

#### Real Example
- **Cassandra**: During partition → AP; Normal → fast but eventually consistent
- **MongoDB**: During partition → CP; Normal → may trade latency for consistency

#### Advantages
- More realistic than CAP alone
- Covers system behavior in all scenarios
- Helps choose database for specific use cases

#### Disadvantages
- Adds complexity to decision-making
- Not as widely known as CAP

#### Interview Questions
- **Q:** How is PACELC different from CAP?
  **A:** CAP only covers partition scenarios, PACELC also covers normal operation trade-offs.
  
- **Q:** Give an example of choosing latency over consistency.
  **A:** Social media likes - users don't mind if they don't see the count update instantly.

---

### 4. ACID vs BASE

#### ACID

##### What is it?
Properties for reliable transactions:
- **Atomicity**: All or nothing
- **Consistency**: Data remains valid
- **Isolation**: Concurrent transactions don't interfere
- **Durability**: Committed changes persist

##### Why do we need it?
Data integrity and reliability for critical operations where mistakes are costly.

##### How does it work?
Uses locking, write-ahead logging, and strict transaction management.

##### Real Example
- Banking: Transferring money between accounts must be atomic and consistent
- E-commerce: Inventory updates during checkout

##### Advantages
- Strong data guarantees
- Predictable behavior
- Simple to reason about

##### Disadvantages
- Lower performance
- Less scalable
- Higher latency

#### BASE

##### What is it?
Properties for large distributed systems:
- **Basically Available**: System remains operational
- **Soft state**: State may change over time
- **Eventual consistency**: System becomes consistent eventually

##### Why do we need it?
Scaling requires trading some consistency for availability and performance.

##### How does it work?
Accepts temporary inconsistencies that resolve themselves.

##### Real Example
- Social media news feeds
- Analytics counters
- Activity logs

##### Advantages
- High availability
- Better performance
- Horizontal scaling possible

##### Disadvantages
- Complex to reason about
- Temporary inconsistencies
- Harder to debug

#### Interview Questions
- **Q:** When would you choose ACID over BASE?
  **A:** Financial systems, inventory management, anything where data accuracy is critical.
  
- **Q:** How do you handle eventual consistency in a user-facing application?
  **A:** Use UI cues (loading states, "recent updates" indicators), or implement strong consistency for critical operations.

---

### 5. Throughput vs Latency

#### What is it?
- **Throughput**: Number of requests processed per unit time (req/s)
- **Latency**: Time taken to process a single request (ms)

#### Why do we need it?
Understanding both helps balance performance and user experience.

#### How does it work?
Optimizing one often impacts the other:
- More parallelism → Higher throughput but may increase latency (queues)
- Lower latency → Often means less batching → Lower throughput

#### Real Example
- **Restaurant**: Taking many orders (throughput) vs serving each order quickly (latency)
- **Database**: Batch inserts (high throughput, higher latency for individual records)

#### Advantages
- Helps set realistic performance goals
- Guides capacity planning
- Identifies bottlenecks

#### Disadvantages
- Trade-off isn't always obvious
- Different metrics for different parts of system

#### Interview Questions
- **Q:** How do you improve throughput without increasing latency?
  **A:** Add resources (horizontal scaling), optimize algorithms, use caching, implement asynchronous processing.
  
- **Q:** Your latency is high but throughput is low. What do you check?
  **A:** Database queries, network issues, resource contention, inefficient algorithms.

---

### 6. Amdahl's Law

#### What is it?
Speedup from parallelization is limited by the sequential portion of the system:
```
Speedup = 1 / (1 - p + p/n)
```
Where p = parallelizable portion, n = processors

#### Why do we need it?
Reminds us that throwing more resources at a problem has diminishing returns.

#### How does it work?
If 20% of your system is sequential, maximum speedup is 5x (1/0.2) no matter how many servers you add.

#### Real Example
- **Database**: If all writes go to a single master, that's your bottleneck
- **Data pipeline**: If one step must be completed before the next can start

#### Advantages
- Identifies real bottlenecks
- Prevents wasting resources
- Guides optimization efforts

#### Disadvantages
- Assumes perfect parallelization
- Doesn't account for overhead
- Real systems are more complex

#### Interview Questions
- **Q:** Your CEO wants to add 100 servers to speed up a service. What do you tell them?
  **A:** Identify the sequential bottleneck first. If it's a single database master, 100 servers won't help.
  
- **Q:** How do you identify bottlenecks in your system?
  **A:** Profiling, monitoring, distributed tracing, load testing.

---

### 7. Strong vs Eventual Consistency

#### Strong Consistency

##### What is it?
All reads reflect the most recent write. Every user sees the same data at the same time.

##### Why do we need it?
Critical for data integrity where old data is unacceptable.

##### How does it work?
Uses synchronous replication, quorum reads/writes, or consensus algorithms.

##### Real Example
- Bank account balance after transfer
- Inventory system during checkout

##### Advantages
- Easy to reason about
- No stale data confusion
- Predictable behavior

##### Disadvantages
- Higher latency
- Lower availability during partitions
- Harder to scale

#### Eventual Consistency

##### What is it?
Updates propagate over time. Nodes may temporarily disagree but converge.

##### Why do we need it?
Enables high availability and low latency at scale.

##### How does it work?
Uses asynchronous replication, conflict resolution, and anti-entropy mechanisms.

##### Real Example
- Social media follower counts
- Analytics dashboards
- DNS records

##### Advantages
- High availability
- Low latency
- Scalable

##### Disadvantages
- Stale data possible
- Complex logic needed
- Harder to debug

#### Interview Questions
- **Q:** How would you implement eventual consistency in a microservices architecture?
  **A:** Use message queues, event-driven communication, idempotent operations, and compensation mechanisms.
  
- **Q:** When is eventual consistency unacceptable?
  **A:** Financial transactions, medical records, inventory in checkout flows.

---

### 8. Stateful vs Stateless Architecture

#### Stateful Architecture

##### What is it?
Service remembers user context/state between requests, often storing session data locally.

##### Why do we need it?
Simpler code for certain applications (games, chat, shopping carts).

##### How does it work?
Session data stored in memory or local disk. Requests must go to the same instance.

##### Real Example
- Online multiplayer game server
- Video streaming session

##### Advantages
- Simpler code
- Lower latency for state access
- Less external dependency

##### Disadvantages
- Hard to scale horizontally
- Session stickiness required
- Recovery on failure is complex

#### Stateless Architecture

##### What is it?
Service treats every request as new. Any instance can handle any request.

##### Why do we need it?
Essential for horizontal scaling and resilience.

##### How does it work?
State is stored externally in databases, caches, or client-side.

##### Real Example
- Web application servers (REST APIs)
- Microservices
- Serverless functions

##### Advantages
- Easy horizontal scaling
- Fault tolerant
- Simple load balancing

##### Disadvantages
- External state access latency
- More network calls
- Cache invalidation challenges

#### Interview Questions
- **Q:** How do you handle user sessions in a stateless architecture?
  **A:** Use JWT tokens (stateless), Redis (external store), or database sessions.
  
- **Q:** When would you prefer stateful over stateless?
  **A:** When you need extremely low latency and high performance for state access (gaming, real-time processing).

---

### 9. Microservices vs Monoliths

#### Monolith Architecture

##### What is it?
Single application containing all features in one deployable unit.

##### Why do we need it?
Simplest starting point for most applications.

##### How does it work?
All code in one codebase, deployed as one application.

##### Real Example
- Early versions of most startups
- Basecamp, Shopify (initially)

##### Advantages
- Simple development and deployment
- Easy to debug
- Fast initial development
- Simple testing

##### Disadvantages
- Scaling limited
- Team coordination overhead
- Long build/deploy times
- Technology lock-in

#### Microservices Architecture

##### What is it?
Features split into independent services communicating over network.

##### Why do we need it?
Scalability, team autonomy, and independent deployment.

##### How does it work?
Each service has its own codebase, database, deployment pipeline.

##### Real Example
- Netflix, Uber, Amazon
- Each feature (recommendations, payments, etc.) is separate

##### Advantages
- Independent scaling
- Team autonomy
- Technology diversity
- Smaller codebases
- Independent deployment

##### Disadvantages
- Network complexity
- Data consistency challenges
- Debugging difficulty
- Operational overhead
- DevOps expertise needed

#### Interview Questions
- **Q:** Should we start with microservices or monolith?
  **A:** Start with monolith. Migrate when pain points emerge (team size, deployment bottlenecks, scaling issues).
  
- **Q:** How do you handle distributed transactions in microservices?
  **A:** Use SAGA pattern, eventual consistency, or two-phase commit (2PC) with caution.

---

### 10. Serverless Architecture

#### What is it?
Run code without managing servers. Platform handles scaling and infrastructure.

#### Why do we need it?
Reduces operational overhead, scales automatically, and charges per execution.

#### How does it work?
Code runs in response to events (HTTP requests, message queue events). Platform spins up/down containers as needed.

#### Real Example
- AWS Lambda processing S3 uploads
- Google Cloud Functions for webhooks
- Azure Functions for background jobs

#### Advantages
- No server management
- Auto-scaling
- Pay-per-use pricing
- Quick deployment
- Event-driven support

#### Disadvantages
- Cold start latency
- Execution time limits
- Vendor lock-in
- Debugging difficulty
- Cost at high volume

#### Interview Questions
- **Q:** When is serverless NOT appropriate?
  **A:** Long-running tasks, high-traffic consistent workloads, applications requiring custom runtime environments.
  
- **Q:** How do you handle cold starts?
  **A:** Provisioned concurrency, keep-warm strategies, or optimize function startup time.

---

## 🌐 Networking and Communication

### 11. Load Balancing

#### What is it?
Distributes incoming traffic across multiple servers to prevent overload.

#### Why do we need it?
- No single server gets overwhelmed
- Improves reliability and performance
- Enables horizontal scaling

#### How does it work?
Sits between clients and servers, directs requests based on algorithms, performs health checks.

#### Real Example
- AWS ELB (Elastic Load Balancer)
- Nginx, HAProxy
- Google Cloud Load Balancing

#### Advantages
- High availability
- Scalability
- Fault tolerance
- Health checks
- SSL termination

#### Disadvantages
- Single point of failure (if not redundant)
- Added latency
- Cost
- Complexity

#### Interview Questions
- **Q:** What happens if a server behind the load balancer fails?
  **A:** Load balancer's health checks detect failure and stop routing traffic to it. New servers can be added automatically.
  
- **Q:** How does a load balancer handle session persistence?
  **A:** Using sticky sessions (cookie-based or IP-hash), or move sessions to a shared external store.

---

### 12. Load Balancing Algorithms

#### What is it?
Rules that determine how traffic is distributed across backend servers.

#### Why do we need it?
Different workloads require different distribution strategies for optimal performance.

#### How does it work?

| Algorithm | How It Works |
|-----------|--------------|
| **Round Robin** | Cycles through servers in order |
| **Least Connections** | Sends to server with fewest active connections |
| **IP Hash** | Uses client IP to consistently route to same server |
| **Weighted Round Robin** | Routes proportionally to server capacity |
| **Least Response Time** | Routes to fastest responding server |

#### Real Example
- **Round Robin**: Equal capacity web servers
- **Least Connections**: Video processing servers with varying task lengths
- **IP Hash**: Services with simple caching needs

#### Advantages
- Tailored to workload
- Better resource utilization
- Improved user experience

#### Disadvantages
- Algorithm selection requires understanding workloads
- Dynamic workloads need adaptive algorithms

#### Interview Questions
- **Q:** When would you choose least connections over round robin?
  **A:** When requests take varying time (API calls, file processing). Round robin would overload servers handling long requests.
  
- **Q:** How does IP hash help with caching?
  **A:** Same user always hits same server, improving cache hit rates.

---

### 13. Reverse Proxy vs Forward Proxy

#### Reverse Proxy

##### What is it?
Sits in front of servers, represents them to clients, hides internal topology.

##### Why do we need it?
Security, load balancing, caching, SSL termination, compression.

##### How does it work?
Clients connect to reverse proxy, which forwards requests to appropriate internal servers.

##### Real Example
- Nginx as reverse proxy for web application
- API Gateway
- Load balancer in cloud

##### Advantages
- Hides internal infrastructure
- SSL termination
- Caching and compression
- Load balancing
- Security

##### Disadvantages
- Added complexity
- Potential bottleneck
- Configuration overhead

#### Forward Proxy

##### What is it?
Sits in front of clients, represents them to the internet.

##### Why do we need it?
Security, content filtering, caching, anonymity.

##### How does it work?
Clients connect through proxy to reach external resources.

##### Real Example
- Corporate internet gateways
- VPN services
- Content filtering systems

##### Advantages
- Content filtering
- Privacy/anonymity
- Caching for bandwidth savings
- Access control

##### Disadvantages
- Single point of failure
- Performance impact
- SSL complications

#### Interview Questions
- **Q:** What's the main difference between forward and reverse proxy?
  **A:** Forward proxy sits in front of clients (represents them), reverse proxy sits in front of servers (represents them).
  
- **Q:** When would you use both?
  **A:** Corporate network with forward proxy for internet access and reverse proxy for internal services.

---

### 14. API Gateway

#### What is it?
Single entry point for all API calls in a microservices system.

#### Why do we need it?
Reduces client complexity, handles cross-cutting concerns centrally.

#### How does it work?
- Receives requests, routes to appropriate services
- Handles rate limiting, authentication, logging
- Can aggregate responses from multiple services

#### Real Example
- AWS API Gateway
- Kong
- Spring Cloud Gateway
- Netflix Zuul

#### Advantages
- Single entry point
- Centralized security
- Rate limiting
- Monitoring
- Response transformation

#### Disadvantages
- Performance bottleneck
- Single point of failure
- Overloading with business logic
- Requires careful scaling

#### Interview Questions
- **Q:** How does an API gateway differ from a load balancer?
  **A:** Load balancer distributes traffic; API gateway also handles routing, security, aggregation, and protocol translation.
  
- **Q:** What's the risk of putting too much logic in the gateway?
  **A:** Becomes a bottleneck, mini-monolith, hard to scale, single point of failure.

---

### 15. CDN (Content Delivery Network)

#### What is it?
Geographically distributed servers that cache static content closer to users.

#### Why do we need it?
Reduce latency, offload origin servers, improve user experience globally.

#### How does it work?
- Content cached at edge nodes worldwide
- Users routed to nearest node
- Origin server only updated when needed

#### Real Example
- Cloudflare
- Amazon CloudFront
- Akamai
- Fastly

#### Advantages
- Lower latency (local delivery)
- Reduced origin server load
- Scalability
- DDoS protection
- Global availability

#### Disadvantages
- Cost (at scale)
- Cache invalidation challenges
- Limited to static content
- Infrastructure complexity

#### Interview Questions
- **Q:** How do you handle cache invalidation in a CDN?
  **A:** Versioning (URL-based), TTL expiration, purging, or using ETags.
  
- **Q:** What types of content should NOT be served via CDN?
  **A:** Dynamic content, user-specific data, content that changes frequently.

---

### 16. DNS (Domain Name System)

#### What is it?
Maps human-readable domain names to IP addresses.

#### Why do we need it?
Humans remember names better than numbers. Provides abstraction and flexibility.

#### How does it work?
- Hierarchical lookup: root → TLD → authoritative nameserver
- Caching at multiple levels (browser, OS, ISP)
- Multiple record types (A, CNAME, MX, etc.)

#### Real Example
- `google.com` → `142.250.185.78`
- Cloudflare DNS (1.1.1.1), Google DNS (8.8.8.8)

#### Advantages
- Human-readable addresses
- Load balancing support
- Geographic routing
- Failover capabilities
- Abstraction from IP changes

#### Disadvantages
- Propagation delays (TTL)
- Attack vector (DDoS, cache poisoning)
- Single point of failure (if not redundant)

#### Interview Questions
- **Q:** How does DNS contribute to load balancing?
  **A:** Round-robin DNS returns different IPs for the same name, distributing load across multiple servers.
  
- **Q:** Why do DNS changes take time to propagate?
  **A:** Caching at various levels (browser, OS, ISP) with TTL values that may be hours.

---

### 17. TCP vs UDP

#### TCP

##### What is it?
Reliable, connection-oriented protocol with guaranteed ordered delivery.

##### Why do we need it?
Data integrity matters. File transfers, web pages, APIs need complete data.

##### How does it work?
- Three-way handshake (SYN, SYN-ACK, ACK)
- Acknowledgments and retries
- Flow control and congestion control

##### Real Example
- HTTP/HTTPS (web browsing)
- FTP (file transfers)
- SMTP (email)

##### Advantages
- Reliable delivery
- Ordered packets
- Error checking
- Congestion control

##### Disadvantages
- Higher overhead
- Slower
- Connection setup overhead

#### UDP

##### What is it?
Connectionless, unreliable, faster protocol.

##### Why do we need it?
Speed over reliability for real-time applications.

##### How does it work?
- No handshake
- No acknowledgments
- Fire-and-forget

##### Real Example
- Video streaming (YouTube, Netflix)
- Online gaming
- VoIP (Zoom, Skype)
- DNS queries

##### Advantages
- Fast
- Low overhead
- No connection setup
- Suitable for real-time

##### Disadvantages
- No guaranteed delivery
- No ordering
- No error correction
- Packet loss possible

#### Interview Questions
- **Q:** Why would you choose UDP over TCP for a video streaming service?
  **A:** TCP's retransmission would cause buffering; UDP allows skipping lost frames for smooth playback.
  
- **Q:** How does HTTP/3 (QUIC) use UDP to overcome TCP limitations?
  **A:** QUIC implements reliability and congestion control at application layer, without TCP's head-of-line blocking.

---

### 18. HTTP/2 and HTTP/3 (QUIC)

#### HTTP/2

##### What is it?
Major revision of HTTP protocol introduced in 2015.

##### Why do we need it?
Address limitations of HTTP/1.1 (head-of-line blocking, multiple connections needed).

##### How does it work?
- Multiplexing: Multiple requests over single TCP connection
- Header compression (HPACK)
- Server push
- Binary framing

##### Real Example
- Modern web applications
- Major websites (Google, Facebook)
- Supported by all modern browsers

##### Advantages
- Reduced latency
- Better connection utilization
- Header compression saves bandwidth
- Server push for resources

##### Disadvantages
- Still TCP-based (head-of-line blocking remains)
- More complex than HTTP/1.1
- Requires TLS (usually)

#### HTTP/3 (QUIC)

##### What is it?
HTTP over QUIC (UDP-based), introduced in 2022.

##### Why do we need it?
Solve TCP's head-of-line blocking, faster connection setup.

##### How does it work?
- Based on UDP
- Built-in encryption (TLS 1.3)
- Independent streams
- Zero-RTT connection setup

##### Real Example
- Google services
- Cloudflare
- YouTube
- Facebook

##### Advantages
- Faster connection setup (0-RTT)
- No head-of-line blocking
- Better performance on unreliable networks
- Connection migration

##### Disadvantages
- New protocol, less adoption
- More complex implementation
- Firewall issues (UDP)

#### Interview Questions
- **Q:** How does multiplexing improve performance in HTTP/2?
  **A:** Multiple requests share one connection, avoiding the overhead of creating multiple TCP connections.
  
- **Q:** Why is HTTP/3 better for mobile networks?
  **A:** QUIC handles packet loss better (no head-of-line blocking) and supports connection migration when IP changes.

---

### 19. gRPC vs REST

#### REST

##### What is it?
Architectural style for APIs using HTTP/JSON with resource-oriented endpoints.

##### Why do we need it?
Simplicity, human-readable, widely understood, browser compatibility.

##### How does it work?
- Resource-based URLs (/users, /orders)
- HTTP methods (GET, POST, PUT, DELETE)
- JSON/XML payloads
- Stateless

##### Real Example
- Public APIs (Twitter, GitHub, Stripe)
- Web applications
- Mobile apps

##### Advantages
- Human-readable
- Browser support
- Simple to implement
- Wide ecosystem
- Language agnostic

##### Disadvantages
- Higher payload size (JSON)
- No strict contract
- No streaming (without WebSockets)
- Limited error handling

#### gRPC

##### What is it?
High-performance RPC framework using HTTP/2 and Protocol Buffers.

##### Why do we need it?
Performance, strong typing, streaming, cross-language support.

##### How does it work?
- Protocol Buffers (binary format)
- Code generation from .proto files
- HTTP/2 transport
- Bidirectional streaming

##### Real Example
- Microservices communication (Uber, Netflix)
- Backend-to-backend services
- Streaming applications

##### Advantages
- High performance (binary protocol)
- Strong typing and contracts
- Streaming support
- Multi-language support
- Better for microservices

##### Disadvantages
- Not human-readable
- Browser support limited (needs gRPC-Web)
- Steeper learning curve
- Less ecosystem

#### Interview Questions
- **Q:** When would you choose gRPC over REST?
  **A:** Service-to-service communication, high-throughput systems, streaming, strong contract requirements.
  
- **Q:** Why is REST still preferred for public APIs?
  **A:** REST is human-readable, browser-friendly, and widely understood with extensive tooling.

---

### 20. WebSocket and Server-Sent Events (SSE)

#### WebSocket

##### What is it?
Full-duplex communication where client and server can send messages anytime.

##### Why do we need it?
Real-time bidirectional communication without polling.

##### How does it work?
- Upgrade from HTTP to WebSocket protocol
- Persistent connection
- Messages sent as frames
- Full-duplex

##### Real Example
- Chat applications (Slack, WhatsApp)
- Multiplayer games
- Live collaboration (Google Docs)
- Stock trading apps

##### Advantages
- Low latency
- Bidirectional
- Real-time
- Efficient (no HTTP headers on each message)

##### Disadvantages
- Complex to implement
- Firewall issues
- Connection management needed
- Not HTTP (different protocol)

#### Server-Sent Events (SSE)

##### What is it?
Server-to-client only, one-way channel over HTTP.

##### Why do we need it?
Simple real-time push from server to client.

##### How does it work?
- Client opens HTTP connection
- Server sends `text/event-stream` responses
- Client receives events as they arrive
- Automatic reconnection

##### Real Example
- Live score updates
- News feeds
- Notification systems
- Real-time dashboards

##### Advantages
- Simpler than WebSockets
- HTTP-based (works with proxies)
- Automatic reconnection
- Built-in event IDs

##### Disadvantages
- Server-to-client only
- Limited concurrency (HTTP/1.1 connection limit)
- No binary data support

#### Interview Questions
- **Q:** When would you choose SSE over WebSockets?
  **A:** When only the server pushes updates (news, scores, notifications) and full-duplex not needed.
  
- **Q:** How do you handle WebSocket reconnection?
  **A:** Implement exponential backoff, use heartbeat pings, store session data externally.

---

### 21. Long Polling

#### What is it?
Technique where client requests and server holds it open until new data or timeout.

#### Why do we need it?
Simulate real-time updates over plain HTTP without special protocols.

#### How does it work?
- Client sends request
- Server holds response until data available
- Response sent, client immediately opens new request
- Repeats continuously

#### Real Example
- Facebook chat (historically)
- Real-time notifications before WebSockets
- Lightweight applications

#### Advantages
- Simple to implement
- Works through proxies
- No special protocols
- Reliable

#### Disadvantages
- Less efficient than WebSockets
- Potential for timeouts
- Higher latency than WebSockets
- Consumes server resources

#### Interview Questions
- **Q:** How is long polling different from regular polling?
  **A:** Regular polling makes repeated requests at intervals; long polling holds connections open until data arrives.
  
- **Q:** When would you use long polling instead of WebSockets?
  **A:** When WebSocket support is limited (old browsers, corporate firewalls) or for simpler applications.

---

### 22. Gossip Protocol

#### What is it?
Protocol where nodes share information by periodically talking to random peers.

#### Why do we need it?
Distributed information sharing without central authority, fault-tolerant.

#### How does it work?
- Nodes randomly select peers
- Exchange information
- Information spreads exponentially
- Eventually everyone knows

#### Real Example
- Membership in Cassandra
- Health in Consul
- Cluster management in DynamoDB
- Bitcoin blockchain (partially)

#### Advantages
- Decentralized
- Fault-tolerant
- Scalable
- Eventual convergence
- No single point of failure

#### Disadvantages
- Eventual consistency (not immediate)
- Increased network chatter
- Harder to reason about
- Security concerns (gossip can spread bad info)

#### Interview Questions
- **Q:** How does gossip protocol handle node failures?
  **A:** Nodes stop hearing heartbeats from failed nodes; eventually the cluster marks them as dead.
  
- **Q:** Why is gossip used in distributed databases?
  **A:** Provides eventual consistency without central coordination, scales well with large clusters.

---

## 💾 Database and Storage Internals

### 23. Sharding (Data Partitioning)

#### What is it?
Splitting data across multiple machines, each holding a subset.

#### Why do we need it?
Scale beyond single machine limits in storage and throughput.

#### How does it work?

| Strategy | How It Works |
|----------|--------------|
| **Range-based** | Partition by value ranges (user IDs 1-1000, 1001-2000) |
| **Hash-based** | Distribute via hash function |
| **Directory-based** | Lookup service maps keys to shards |
| **Geographic** | By region (EU, US, Asia) |

#### Real Example
- Social media (shard by user_id)
- E-commerce (shard by order_id)
- Multi-tenant SaaS (shard by tenant_id)

#### Advantages
- Horizontal scaling
- Improved performance
- Geographic distribution
- Smaller datasets

#### Disadvantages
- Complex shard key selection
- Resharding challenges
- Hot spots possible
- Cross-shard queries difficult
- Transaction complexity

#### Interview Questions
- **Q:** How do you choose a shard key?
  **A:** Consider access patterns, avoid hot spots, ensure even distribution, support your queries.
  
- **Q:** What happens when you need to add new shards?
  **A:** Resharding is complex—use consistent hashing to minimize movement, or plan for downtime.

---

### 24. Replication Patterns (Master-Slave, Master-Master)

#### Master-Slave (Primary-Replica)

##### What is it?
One node handles writes, replicates changes to others that serve reads.

##### Why do we need it?
Read scalability, fault tolerance, backup.

##### How does it work?
- Master processes writes
- Changes streamed to slaves (replication log)
- Slaves serve read requests
- Failover possible

##### Real Example
- PostgreSQL replication
- MySQL primary-replica
- MongoDB replica sets

##### Advantages
- Read scaling
- Backups without downtime
- Disaster recovery
- Simple to understand

##### Disadvantages
- Write bottleneck
- Replication lag
- Failover complexity
- Single point of failure (master)

#### Master-Master (Multi-Primary)

##### What is it?
Multiple nodes accept writes and reconcile conflicts.

##### Why do we need it?
High availability, writes everywhere, geographic distribution.

##### How does it work?
- All nodes accept writes
- Conflict resolution needed
- Asynchronous or synchronous replication

##### Real Example
- Multi-master MySQL
- Cassandra (peer-to-peer)
- DynamoDB (multi-region)

##### Advantages
- High write availability
- Geographic distribution
- No single failure point
- Writes in any location

##### Disadvantages
- Conflict resolution complexity
- More complex to operate
- Consistency challenges
- Increased latency for consistency

#### Interview Questions
- **Q:** What's replication lag and how do you handle it?
  **A:** Delay between master write and slave update. Use monitoring, read from master for critical reads, use leader leases.
  
- **Q:** When would you choose master-master over master-slave?
  **A:** Global applications requiring writes in multiple regions, or for high availability with minimal downtime.

---

### 25. Consistent Hashing

#### What is it?
Technique to distribute keys across nodes minimizing data movement when nodes change.

#### Why do we need it?
Scale distributed systems without massive data redistribution.

#### How does it work?
- Keys and nodes placed on logical ring
- Each key belongs to next node on ring
- Adding/removing node moves only small portion

#### Real Example
- DynamoDB
- Cassandra
- Redis Cluster
- CDNs (for content routing)

#### Advantages
- Minimal data movement
- Scalability
- Fault tolerance
- Even distribution (with virtual nodes)

#### Disadvantages
- Complexity
- Virtual nodes needed for even distribution
- Not human-readable
- Hot spots possible

#### Interview Questions
- **Q:** How does consistent hashing handle node failure?
  **A:** Keys from failed node are reassigned to next node(s), only a small portion of keys move.
  
- **Q:** Why are virtual nodes used in consistent hashing?
  **A:** Even out distribution, especially with heterogeneous hardware.

---

### 26. Database Indexing (B-Trees, LSM Trees)

#### B-Trees

##### What is it?
Balanced tree structure for sorted data with efficient range lookups.

##### Why do we need it?
Fast read operations, range queries, sorted data access.

##### How does it work?
- Balanced tree structure
- Each node has multiple keys
- Pointers to child nodes
- Efficient range scans

##### Real Example
- PostgreSQL (B-tree indexes)
- MySQL InnoDB
- Oracle

##### Advantages
- Fast reads
- Range queries efficient
- Stable performance
- Widely supported

##### Disadvantages
- Write amplification
- Fragmentation over time
- Slower writes

#### LSM Trees (Log-Structured Merge-Trees)

##### What is it?
Batches writes in memory, periodically flushes to disk.

##### Why do we need it?
Write-heavy workloads, high throughput.

##### How does it work?
- Memtable (in-memory write buffer)
- SSTables (sorted tables on disk)
- Background compaction
- Bloom filters for reads

##### Real Example
- Cassandra
- RocksDB
- MongoDB (WiredTiger)
- LevelDB

##### Advantages
- High write throughput
- Compression friendly
- Good for write-heavy workloads
- Low write amplification

##### Disadvantages
- Slower reads
- Compaction overhead
- More complex
- Disk space overhead

#### Interview Questions
- **Q:** When would you choose LSM tree over B-tree?
  **A:** Write-heavy workloads (logs, time-series data, event sourcing) where write performance is critical.
  
- **Q:** How do indexes affect write performance?
  **A:** Each index must be updated on write, slowing inserts/updates. More indexes = slower writes.

---

### 27. Write Ahead Logging (WAL)

#### What is it?
Records changes to a log before applying them to the main database.

#### Why do we need it?
Ensures durability and atomicity; allows recovery from crashes.

#### How does it work?
- Changes written to WAL first
- Then applied to database (in background)
- On crash, replay WAL to recover
- Enables replication from log

#### Real Example
- PostgreSQL WAL
- MySQL binary log
- MongoDB oplog
- Most relational databases

#### Advantages
- Durability guarantee
- Atomic transactions
- Replication support
- Point-in-time recovery
- Performance (sequential writes)

#### Disadvantages
- Extra storage
- Write overhead
- Log management
- Recovery time

#### Interview Questions
- **Q:** What happens if a crash occurs mid-transaction with WAL?
  **A:** On restart, system replays WAL, applying committed transactions and rolling back uncommitted ones.
  
- **Q:** How does WAL enable replication?
  **A:** Replication logs (stream of WAL entries) are sent to replicas, which replay them.

---

### 28. Normalization vs Denormalization

#### Normalization

##### What is it?
Organizing data to reduce redundancy and dependencies (1NF, 2NF, 3NF).

##### Why do we need it?
Avoid update anomalies, insert anomalies, delete anomalies.

##### How does it work?
- Split data into related tables
- Use foreign keys
- Reduce duplication
- Each fact stored once

##### Real Example
- Customer orders: separate tables for customers, orders, products
- Third normal form design

##### Advantages
- Data integrity
- No redundancy
- Easier updates
- Smaller storage

##### Disadvantages
- Slower reads (joins)
- Complex queries
- Performance at scale

#### Denormalization

##### What is it?
Intentionally duplicating data to speed up reads.

##### Why do we need it?
Read-heavy workloads, performance optimization.

##### How does it work?
- Duplicate data across tables
- Store computed values
- Reduce joins
- Cache frequently accessed data

##### Real Example
- E-commerce: store product name with each order item (not just product_id)
- Social media: store user name in posts table

##### Advantages
- Faster reads
- Fewer joins
- Simpler queries
- Better performance

##### Disadvantages
- Data redundancy
- Update complexity
- Storage overhead
- Consistency maintenance

#### Interview Questions
- **Q:** When should you denormalize?
  **A:** Read-heavy applications, analytics, reporting, when join performance is problematic.
  
- **Q:** How do you maintain consistency in a denormalized design?
  **A:** Use triggers, application logic, or eventual consistency with scheduled reconciliation.

---

### 29. Polyglot Persistence

#### What is it?
Using multiple database types within the same system.

#### Why do we need it?
Different workloads have different requirements (ACID vs BASE, read vs write, structural vs unstructured).

#### How does it work?
- Choose database per use case
- Integrate through application or services
- Each handles its strength

#### Real Example
- **PostgreSQL**: Financial transactions
- **MongoDB**: Product catalogs
- **Redis**: Caching
- **Elasticsearch**: Search
- **Neo4j**: Recommendations

#### Advantages
- Best tool for each job
- Optimized performance
- Flexibility
- Innovation

#### Disadvantages
- Operational complexity
- Data consistency challenges
- Higher cost
- Team expertise needed
- Integration complexity

#### Interview Questions
- **Q:** How do you manage data consistency across different databases?
  **A:** Use event-driven architecture, eventual consistency, compensate on failure, or use a single source of truth.
  
- **Q:** When would polyglot persistence NOT be appropriate?
  **A:** Small teams, simple applications, where simplicity outweighs benefits.

---

### 30. Bloom Filters

#### What is it?
Space-efficient probabilistic data structure for set membership.

#### Why do we need it?
Quickly answer "is this item definitely not in the set?" without storing actual items.

#### How does it work?
- Multiple hash functions
- Set bits in bit array on insert
- Check bits on lookup
- False positives possible, no false negatives

#### Real Example
- Cassandra (avoid disk reads)
- HBase (filter in MemStore)
- Content filtering (SpamAssassin)
- Database query optimization

#### Advantages
- Space efficient
- Fast (O(k) where k is hash functions)
- No false negatives
- Can tune false positive rate

#### Disadvantages
- False positives possible
- Cannot remove items (without counting)
- Cannot expand easily
- Not exact

#### Interview Questions
- **Q:** How would you handle false positives with bloom filters?
  **A:** After bloom filter says "maybe," check the actual data structure for confirmation.
  
- **Q:** When would you NOT use a bloom filter?
  **A:** When exact membership is required, or when items need to be removed frequently.

---

### 31. Vector Databases

#### What is it?
Database designed to store and query high-dimensional vectors (embeddings).

#### Why do we need it?
Modern AI/ML applications require similarity search (find similar items).

#### How does it work?
- Store vectors (numeric representations)
- Use distance metrics (cosine similarity, Euclidean)
- Build indexes for fast search
- Support nearest neighbor queries

#### Real Example
- Pinecone
- Weaviate
- Milvus
- pgvector (PostgreSQL extension)

#### Advantages
- Fast similarity search
- Scalable to billions
- AI-native features
- Integration with ML models

#### Disadvantages
- New, evolving technology
- Complex to operate
- Cost at scale
- Indexing overhead

#### Interview Questions
- **Q:** What's the difference between vector databases and traditional databases?
  **A:** Vector databases optimize for similarity search (nearest neighbors) while traditional DBs handle equality/range queries.
  
- **Q:** How do vector databases work with recommendation systems?
  **A:** User and item embeddings are stored; find items closest to user's embedding.

---

## 🛡️ Reliability and Fault Tolerance

### 32. Rate Limiting

#### What is it?
Controls how many requests a user/IP can make in a time window.

#### Why do we need it?
Protect from abuse, accidental spikes, and runaway loops.

#### How does it work?

| Strategy | How It Works |
|----------|--------------|
| **Fixed Window** | Reset at fixed time intervals |
| **Sliding Window** | Rolling time windows |
| **Token Bucket** | Tokens replenish over time |
| **Leaky Bucket** | Constant output rate |

#### Real Example
- Twitter API rate limits
- Stripe API (read rate limits)
- AWS API Gateway throttling

#### Advantages
- Prevents abuse
- Protects resources
- Ensures fairness
- Predictable system load

#### Disadvantages
- User frustration
- Configuration complexity
- May block legitimate users
- Performance overhead

#### Interview Questions
- **Q:** How would you implement rate limiting in a distributed system?
  **A:** Use Redis (atomic increments), with consistent hashing, or centralized rate limiter service.
  
- **Q:** What happens when a user exceeds the rate limit?
  **A:** Return HTTP 429 (Too Many Requests), with Retry-After header.

---

### 33. Circuit Breaker Pattern

#### What is it?
Pattern that "breaks" calls to a service when failures exceed threshold.

#### Why do we need it?
Prevent cascading failures, fail fast, allow service recovery.

#### How does it work?
- **Closed**: Normal operation, count failures
- **Open**: Fail fast, no calls to service
- **Half-open**: Test service, attempt few calls

#### Real Example
- Netflix Hystrix
- Resilience4j
- Spring Cloud Circuit Breaker

#### Advantages
- Prevents cascading failures
- Graceful degradation
- Fast failure
- Self-healing capability

#### Disadvantages
- Configuration complexity
- May hide transient issues
- Additional latency
- Recovery tuning needed

#### Interview Questions
- **Q:** How do you determine when to open the circuit breaker?
  **A:** Based on error rate percentage, threshold over time window, or consecutive failures.
  
- **Q:** How does a circuit breaker differ from a timeout?
  **A:** Timeout handles individual slow calls; circuit breaker prevents calls entirely when service is unhealthy.

---

### 34. Bulkhead Pattern

#### What is it?
Isolates system components so failure in one area doesn't sink everything.

#### Why do we need it?
Limit blast radius, prevent resource exhaustion.

#### How does it work?
- Separate connection pools
- Separate thread pools
- Isolated services/clusters
- Resource quotas

#### Real Example
- Microservices: separate deployments per service
- Database: separate connection pools per tenant
- Netflix: service isolation patterns

#### Advantages
- Fault isolation
- Improved resilience
- Resource guarantee
- Graceful degradation

#### Disadvantages
- Resource overhead
- Management complexity
- Potentially inefficient
- Harder to tune

#### Interview Questions
- **Q:** How do you implement bulkhead at the service level?
  **A:** Separate deployments, separate databases, circuit breakers per service, resource limits.
  
- **Q:** What's the trade-off with smaller vs larger bulkheads?
  **A:** Smaller = better isolation but less efficiency; larger = more efficient but bigger blast radius.

---

### 35. Retry Patterns and Exponential Backoff

#### What is it?
Retry failed operations with increasing delays between attempts.

#### Why do we need it?
Recover from transient errors without overwhelming the system.

#### How does it work?
- Initial retry after delay
- Each retry multiplies delay (1s, 2s, 4s, 8s)
- Add jitter (randomness)
- Stop after max attempts

#### Real Example
- AWS SDK automatic retries
- Database connection retries
- HTTP client retry policies

#### Advantages
- Increased success rate
- Graceful recovery
- System-friendly
- Handles transient failures

#### Disadvantages
- Increased latency for failures
- Configuration overhead
- Potential for cascading if not careful

#### Interview Questions
- **Q:** Why use exponential backoff instead of constant retry?
  **A:** Prevents thundering herd, gives service time to recover, reduces load on failing services.
  
- **Q:** What is jitter and why is it important?
  **A:** Randomization added to backoff to prevent retries from synchronizing (thundering herd).

---

### 36. Idempotency

#### What is it?
Operation that produces the same result regardless of how many times it's executed.

#### Why do we need it?
Handle duplicate requests safely in distributed systems with retries.

#### How does it work?

| ✅ Idempotent | ❌ Not Idempotent |
|---------------|-------------------|
| `UPDATE user SET active = true WHERE id = 123` | `UPDATE user SET balance = balance + 10` |
| `DELETE /order/123` (if exists) | `POST /order` (create order) |
| `PUT /user/123` with full object | `PATCH /user/123` (increment) |

#### Real Example
- Payment processing with idempotency keys
- Stripe idempotency keys
- State updates (set vs increment)

#### Advantages
- Safe retries
- Duplicate handling
- Payment integrity
- Better user experience

#### Disadvantages
- Design complexity
- State tracking needed
- Storage overhead
- May hide bugs

#### Interview Questions
- **Q:** How do you implement idempotency in a REST API?
  **A:** Use idempotency keys (client-provided), store results, return cached response on duplicate.
  
- **Q:** What's the difference between idempotent and safe (GET) methods?
  **A:** Safe methods have no side effects (idempotent by default); idempotent methods can have side effects but only once.

---

### 37. Heartbeat

#### What is it?
Periodic signal sent by a service/node to indicate it's alive.

#### Why do we need it?
Liveness detection for failure detection and failover.

#### How does it work?
- Node sends heartbeats periodically
- Monitoring system tracks
- Missing heartbeats trigger actions

#### Real Example
- Kubernetes liveness probes
- AWS ELB health checks
- Etcd leader heartbeats

#### Advantages
- Simple to implement
- Low overhead
- Early failure detection
- Enables auto-recovery

#### Disadvantages
- Timing complexity (too short = false positives, too long = slow detection)
- Network issues can cause false negatives
- Additional traffic

#### Interview Questions
- **Q:** What's a good interval for heartbeats?
  **A:** Balance detection speed vs overhead. Common: 5-30 seconds with multiple missed allowed.
  
- **Q:** How does a heartbeat differ from a health check?
  **A:** Heartbeat is push (node says "I'm alive"); health check is pull (system asks "are you alive?").

---

### 38. Leader Election (Paxos, Raft)

#### What is it?
Process of choosing a single coordinator among many nodes.

#### Why do we need it?
Need single decision-maker for consistency, avoids split-brain.

#### How does it work?
- **Raft**: Leader election, log replication, safety
- **Paxos**: Proposal-based consensus
- Nodes vote, leader emerges, heartbeat maintains

#### Real Example
- ZooKeeper (uses ZAB)
- Etcd (Raft)
- Consul (Raft)
- Kafka controller

#### Advantages
- Consistent coordination
- Fault-tolerant
- Self-healing
- Single decision-maker

#### Disadvantages
- Complexity
- Performance overhead
- Network latency impact
- Tuning required

#### Interview Questions
- **Q:** What's the difference between Paxos and Raft?
  **A:** Raft is designed to be more understandable (leader-based); Paxos is more abstract and harder to implement.
  
- **Q:** How does leader election handle network partitions?
  **A:** Majority-based; if split, only side with majority can elect a leader, prevents split-brain.

---

### 39. Distributed Transactions

#### SAGA Pattern

##### What is it?
Sequence of local transactions with compensating actions for rollbacks.

##### Why do we need it?
Distributed transactions without locking across services.

##### How does it work?
- Each step has compensate action
- Execute sequentially
- On failure, execute compensations

##### Real Example
- E-commerce: order → payment → inventory → shipping (each with compensation)
- Choreography vs Orchestration

##### Advantages
- No distributed locking
- Eventual consistency
- Microservices-friendly
- Works at scale

##### Disadvantages
- Complex compensation logic
- Partial failures
- Hard to debug
- Eventually consistent

#### Two Phase Commit (2PC)

##### What is it?
Protocol for atomic transactions across multiple nodes.

##### Why do we need it?
Strong consistency across distributed systems.

##### How does it work?
- **Phase 1 (Prepare)**: All participants must be ready
- **Phase 2 (Commit/Rollback)**: All commit or all rollback

##### Real Example
- Distributed relational databases
- XA transactions
- Few modern distributed systems

##### Advantages
- Strong consistency
- ACID guarantees
- Atomic across nodes

##### Disadvantages
- Blocking
- Single point of failure (coordinator)
- Performance overhead
- Not scalable

#### Interview Questions
- **Q:** When would you use SAGA over 2PC?
  **A:** Microservices, high-scale systems, where availability is more important than immediate consistency.
  
- **Q:** How do you handle failures in SAGA compensation?
  **A:** Retry compensation with exponential backoff, dead letter queues, manual intervention if needed.

---

## 💰 Caching and Messaging

### 40. Caching

#### What is it?
Storing frequently accessed data in fast storage (usually memory).

#### Why do we need it?
Reduce latency, reduce backend load, improve performance.

#### How does it work?
- Check cache for data
- If present (hit), return quickly
- If not (miss), fetch from source
- Store in cache for future

#### Real Example
- Redis (in-memory cache)
- Memcached
- Browser cache
- CDN cache
- In-process caches

#### Advantages
- Lower latency
- Reduced database load
- Scalability
- Cost savings

#### Disadvantages
- Cache invalidation complexity
- Stale data
- Memory cost
- Consistency challenges

#### Interview Questions
- **Q:** What are the challenges with caching in distributed systems?
  **A:** Cache invalidation, consistency, stale reads, cache stampede, cold start.
  
- **Q:** When should you NOT cache?
  **A:** Frequently changing data, when staleness is unacceptable, when storage cost outweighs benefit.

---

### 41. Caching Strategies

#### Cache Aside (Lazy Loading)

##### How it works?
- Read: Check cache, load from DB if miss
- Write: Update DB, invalidate cache

##### When to use?
Most common, works for most applications.

##### Advantages
- Simple
- Lazy population
- Best for read-heavy

##### Disadvantages
- Cache misses on first request
- Stale data possible

#### Write Through

##### How it works?
- Write: Update cache and DB simultaneously
- Read: Always consistent

##### When to use?
When consistency is critical.

##### Advantages
- Always consistent
- No stale reads

##### Disadvantages
- Higher write latency
- Writes propagate

#### Write Back (Write Behind)

##### How it works?
- Write: Update cache first, flush later
- Read: From cache

##### When to use?
Write-heavy, tolerant of data loss.

##### Advantages
- Very fast writes
- Reduces DB load

##### Disadvantages
- Data loss risk
- Consistency challenges

#### Real Example
- **Cache Aside**: Most web applications with Redis
- **Write Through**: Financial systems
- **Write Back**: Analytics, logging

#### Interview Questions
- **Q:** Which strategy would you use for an e-commerce product catalog?
  **A:** Cache aside (read-heavy, updates infrequent, stale okay momentarily).
  
- **Q:** When would you use write-back caching?
  **A:** When write speed is critical and some data loss is acceptable (analytics, metrics).

---

### 42. Cache Eviction Policies

#### LRU (Least Recently Used)

##### What is it?
Evicts items not accessed recently.

##### When to use?
When recent access predicts future access.

##### Advantages
- Works for many workloads
- Simple
- Effective

##### Disadvantages
- Not always optimal
- Overhead with frequent updates

#### LFU (Least Frequently Used)

##### What is it?
Evicts items accessed least often.

##### When to use?
When frequency matters more than recency.

##### Advantages
- Good for long-term popularity

##### Disadvantages
- Not adaptive to new trends
- Overhead tracking counts

#### Real Example
- Redis: LRU or LFU configurable
- Memcached: LRU
- Browser: LRU for cache
- Database buffer: LRU variants

#### Interview Questions
- **Q:** When would you choose LFU over LRU?
  **A:** When access patterns are stable (popular items remain popular) or for long-term caching.
  
- **Q:** How does Redis implement LRU?
  **A:** Approximate LRU with sampling (not perfect), configurable sample size for accuracy.

---

### 43. Message Queues (Point-to-Point)

#### What is it?
Asynchronous communication where messages are consumed by one receiver.

#### Why do we need it?
Decouple producers and consumers, handle load spikes, process asynchronously.

#### How does it work?
- Producer sends messages to queue
- Consumer picks messages
- Messages removed after consumption
- At-least-once, exactly-once, at-most-once delivery

#### Real Example
- RabbitMQ
- Amazon SQS
- ActiveMQ

#### Advantages
- Loose coupling
- Load balancing
- Fault tolerance
- Asynchronous processing
- Reliable delivery

#### Disadvantages
- Complexity
- Additional infrastructure
- Latency for synchronous needs
- Monitoring needed

#### Interview Questions
- **Q:** How do you handle message processing failures in a queue?
  **A:** Use dead letter queues, retry with exponential backoff, manual intervention, or skip for non-critical.
  
- **Q:** What's the difference between at-least-once and exactly-once delivery?
  **A:** At-least-once: message may be delivered multiple times (needs idempotency). Exactly-once: delivered once (complex to implement).

---

### 44. Pub/Sub (Publish Subscribe)

#### What is it?
Broadcast communication where publishers send to topics, subscribers receive copies.

#### Why do we need it?
One-to-many communication, event-driven architectures, decoupled systems.

#### How does it work?
- Publisher sends to topic
- All subscribers receive copies
- Typically push-based
- Parallel processing possible

#### Real Example
- Google Pub/Sub
- Amazon SNS
- Apache Kafka
- Redis Pub/Sub

#### Advantages
- Multiple consumers
- Loose coupling
- Scalable
- Real-time
- Event-driven

#### Disadvantages
- Ordering challenges
- Duplicate messages
- Subscriber management
- Delivery guarantees

#### Interview Questions
- **Q:** How is pub/sub different from message queues?
  **A:** Pub/sub: one message to many subscribers; queue: one message to one consumer.
  
- **Q:** When would you use Kafka vs RabbitMQ?
  **A:** Kafka: event streaming, large data, replayability; RabbitMQ: traditional messaging, routing, transactions.

---

### 45. Dead Letter Queues

#### What is it?
Queue for messages that cannot be processed successfully.

#### Why do we need it?
Prevent poison messages from blocking, enable debugging, ensure reliability.

#### How does it work?
- Failed messages moved after max retries
- Separate queue for inspection
- Can be replayed after fix
- Alerting on DLQ growth

#### Real Example
- AWS SQS dead-letter queues
- RabbitMQ dead letter exchanges
- Kafka dead letter topics

#### Advantages
- Prevents message blocking
- Debugging capability
- Recovery possible
- Monitoring signal

#### Disadvantages
- Additional infrastructure
- Complex handling
- Delay in processing
- Storage overhead

#### Interview Questions
- **Q:** How do you handle messages in a dead letter queue?
  **A:** Monitor, alert on growth, investigate root cause, fix consumer, replay messages.
  
- **Q:** When would a message end up in DLQ?
  **A:** Processing errors, validation failures, dependency unavailability, malformed messages.

---

## 🔍 Observability and Security

### 46. Distributed Tracing

#### What is it?
Tracking a request as it flows through multiple services.

#### Why do we need it?
Debugging, performance analysis, understanding distributed flows.

#### How does it work?
- Trace ID identifies request
- Spans for each operation
- Context propagation between services
- Visualization of complete flow

#### Real Example
- Jaeger
- Zipkin
- AWS X-Ray
- Google Cloud Trace

#### Advantages
- End-to-end visibility
- Performance analysis
- Dependency mapping
- Root cause analysis

#### Disadvantages
- Additional instrumentation
- Performance overhead
- Storage costs
- Sampling complexity

#### Interview Questions
- **Q:** How do you propagate trace context between services?
  **A:** Through HTTP headers (W3C Trace Context, B3), gRPC metadata, or message headers.
  
- **Q:** When would you use distributed tracing vs logs?
  **A:** Tracing for request flow and performance; logs for events, errors, and debugging.

---

### 47. SLA vs SLO vs SLI

#### What is it?
Service measurement and commitment framework.

#### Why do we need it?
Define, measure, and guarantee service reliability.

#### How does it work?

| Term | Definition | Example |
|------|------------|---------|
| **SLI** (Service Level Indicator) | Actual metric | 99.92% uptime, 120ms latency |
| **SLO** (Service Level Objective) | Internal target | 99.95% uptime target |
| **SLA** (Service Level Agreement) | External promise | 99.9% guaranteed, compensation if missed |

#### Real Example
- AWS: 99.99% SLA for EC2
- Google: 99.9% SLO, 99.95% SLA
- Internal APIs: SLOs for teams

#### Advantages
- Clear expectations
- Measurable reliability
- Customer trust
- Team accountability
- Improvement focus

#### Disadvantages
- Enforcement complexity
- Penalties on failures
- Measurement overhead
- Gaming potential

#### Interview Questions
- **Q:** What's the relationship between SLI, SLO, and SLA?
  **A:** SLI measures, SLO sets target, SLA guarantees (with consequences).
  
- **Q:** How do you choose SLO targets?
  **A:** Based on user expectations, business requirements, technical capability, and competitive position.

---

### 48. OAuth 2.0 and OIDC

#### OAuth 2.0

##### What is it?
Framework for delegated authorization (access without sharing passwords).

##### Why do we need it?
Security, user experience, least privilege access.

##### How does it work?
- Authorization server issues tokens
- Client presents token for access
- Resource server validates token
- Scopes define permissions

##### Real Example
- "Login with Google/Facebook"
- API access tokens
- Mobile app authentication

##### Advantages
- No password sharing
- Scoped access
- Granular permissions
- Revocation possible

##### Disadvantages
- Complex flow
- Token management
- Security considerations

#### OIDC (OpenID Connect)

##### What is it?
Authentication layer on top of OAuth 2.0.

##### Why do we need it?
Verify user identity (who they are), not just authorization.

##### How does it work?
- Adds ID token (JWT) to OAuth flow
- Contains user identity claims
- Standardized user info endpoint

##### Real Example
- Login with Google (returns ID token)
- Corporate SSO
- Multi-tenant applications

##### Advantages
- Standardized authentication
- User identity verification
- Simple integration
- Widely supported

##### Disadvantages
- Added complexity
- Security considerations
- Token handling

#### Interview Questions
- **Q:** What's the difference between authentication and authorization?
  **A:** Authentication = who you are (login); Authorization = what you can do (permissions).
  
- **Q:** How do you validate a JWT token?
  **A:** Check signature, verify claims (exp, iss, aud), use JWKS endpoint for public keys.

---

### 49. TLS/SSL Handshake

#### What is it?
Protocol for secure communication over network.

#### Why do we need it?
Encrypt data in transit, prevent eavesdropping, verify identity.

#### How does it work?
- Client hello (supported ciphers)
- Server hello (chosen cipher, certificate)
- Key exchange
- Encrypted connection established

#### Real Example
- HTTPS websites
- Database connections (MongoDB, PostgreSQL)
- API communications
- Internal microservices

#### Advantages
- End-to-end encryption
- Authentication
- Data integrity
- Widely supported

#### Disadvantages
- Performance overhead
- Certificate management
- Complexity
- Potential vulnerabilities

#### Interview Questions
- **Q:** What happens during the TLS handshake?
  **A:** Client and server agree on encryption, exchange keys, verify certificate, establish secure connection.
  
- **Q:** Why is TLS termination at load balancer a good practice?
  **A:** Offloads CPU-intensive decryption, enables SSL inspection, centralizes certificate management.

---

### 50. Zero Trust Security

#### What is it?
Security model: "Never trust, always verify."

#### Why do we need it?
Traditional perimeter security is insufficient for modern systems.

#### How does it work?
- Verify every request
- Least privilege access
- Continuous validation
- Identity-based, not network-based

#### Real Example
- Google BeyondCorp
- AWS VPC with micro-segmentation
- Zero Trust VPNs
- API authentication for all requests

#### Advantages
- Stronger security
- Defense in depth
- Compliant
- Adaptable to cloud

#### Disadvantages
- Complex implementation
- Performance impact
- Culture change required
- Tooling needed

#### Interview Questions
- **Q:** How does Zero Trust differ from traditional security?
  **A:** Traditional trusts inside network; Zero Trust trusts nothing, verifies everything everywhere.
  
- **Q:** What are the principles of Zero Trust?
  **A:** Verify explicitly, least privilege access, assume breach, all requests authenticated/authorized/encrypted.

---

## 🎯 Quick Reference

### Decision Matrix

| Scenario | Recommendation |
|----------|----------------|
| High traffic growth | Horizontal scaling, sharding |
| Financial transactions | ACID, strong consistency, 2PC |
| Social media feeds | BASE, eventual consistency, pub/sub |
| Global users | CDN, multi-region replication |
| Microservices communication | gRPC, API Gateway |
| Real-time features | WebSockets, SSE |
| Background processing | Message queues |
| Frequent reads | Caching, denormalization |
| Write-heavy workloads | LSM trees, write-back caching |
| Service resilience | Circuit breakers, bulkheads |
| System visibility | Distributed tracing, metrics |
| Security | Zero Trust, TLS, OAuth 2.0 |

### System Design Checklist

- [ ] **Clarify** requirements, scale, and constraints
- [ ] **Estimate** traffic, storage, and bandwidth
- [ ] **Choose** appropriate patterns
- [ ] **Design** data model and APIs
- [ ] **Plan** for scaling and bottlenecks
- [ ] **Handle** failures and retries
- [ ] **Implement** observability
- [ ] **Consider** security from the start
- [ ] **Document** trade-offs and decisions

---

## 📚 Further Reading

- *Designing Data-Intensive Applications* by Martin Kleppmann
- *System Design Interview* by Alex Xu
- *Distributed Systems: Concepts and Design*
- *The Art of Scalability* by Martin Abbott

---

## 🤝 Contributing

This guide is a living resource. Contributions, corrections, and examples are welcome!

---

## 📝 License

This documentation is for educational purposes. Use it as a reference for learning and interview preparation.

---

*Last Updated: 2026*

---

> ⭐ **Star this guide if you found it helpful!** ⭐