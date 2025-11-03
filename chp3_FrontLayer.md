# Chapter 3 — Building the Front-End Layer

## What is the Front-End Layer?

The front-end layer spans multiple components:
- The client (web browser or mobile app)
- Network components between client and data center
- Parts of the data center that respond directly to client connections

This layer receives the most traffic since all user interactions must pass through it. This creates the highest throughput and concurrency demands, making front-end scalability critical.

## Modern Web Application Types

**Traditional Multi-Page Applications**
Every interaction causes a full page reload. Click a link or button, browser makes a new request, server sends back complete HTML page.

**Single Page Applications (SPAs)**
Built primarily in JavaScript. Web servers are reduced to providing a data API and security layer. The client handles most of the rendering and logic.

**Hybrid Applications**
Some interactions trigger full page loads, others use AJAX for partial view updates. This combines benefits of both approaches depending on the use case.


**State** is any information that would need to be synchronized between servers to make them identical. Examples include - User session data, shopping cart contents, in-progress form data, and authentication status.

**Stateless services** don't hold data themselves. They delegate to external services whenever client state needs to be accessed.

### Stateful Services

Stateful services keep some "knowledge" between requests that isn't available to every instance. Clients must stick to the selected instance to prevent side effects.

**Problem:** This breaks horizontal scalability since you can't freely distribute requests across all servers.

## HTTP Sessions

Since HTTP itself is stateless, applications developed the concept of sessions on top of HTTP. This allows servers to recognize multiple requests from the same user as part of a larger session.

## Session Storage Approaches

### 1. Cookie-Based Session Storage

1. Application uses session scope normally during request processing
2. Before sending response, framework serializes session data
3. Framework encrypts the serialized data
4. Framework includes it in response headers as session cookie value

**Advantage:** No session storage needed in the data center. Entire session state travels with every request, making the application stateless.

**Problems:**
- Cookies are sent with **every single request**, regardless of resource type (images, CSS, JavaScript, AJAX requests)
- Session storage becomes expensive due to bandwidth
- Encrypting and Base64 encoding increases size by ~33%
- This overhead applies to every request and response
- Browser cookie size limits (typically 4KB)

### 2. Dedicated Data Store

1. Client sends request with session ID cookie (just an ID, not the data)
2. Server loads session data from external data store (Redis, Memcached)
3. Application processes request with session data
4. Before sending response, server saves session back to data store

**Advantages:**
- Small cookies (just session ID)
- Lower bandwidth overhead
- Can store larger sessions
- Stateless web servers

**Problems:**
- Need to maintain external data store
- Additional latency for data store lookups
- Data store becomes a dependency

### 3. Sticky Sessions (Load Balancer Approach)

Push session responsibility onto the load balancer. The load balancer inspects request headers and ensures requests with the same session cookie always go to the server that initially issued the cookie.

**Problem:** This fundamentally breaks statelessness. Servers become stateful, which limits scalability and creates problems during server failures or deployments.

---

When you need globally available state, a common scaling pattern is:
1. Isolate the functionality requiring global state
2. Remove it from the main application
3. Create a new independent service encapsulating this functionality

**Downside:** Increased latency since applications must make remote calls for what used to be local operations.

## Load Balancers

Load balancers sit in front of server pools and present a single IP address while distributing traffic across backend machines.

### SSL Termination

Load balancers can perform SSL termination. Connections from clients to load balancer use HTTPS, but connections from load balancer to web servers use HTTP.

**Advantages:** Significantly reduces resources needed by backend servers since they don't need to handle SSL encryption/decryption.

### Connection Draining

Also called graceful termination. Allows removing a web server from the load balancer pool without terminating existing connections. New requests go to other servers while existing connections finish naturally.

This enables maintanence without disrupting user experience.

### Hardware Load Balancers

**Benefits:**
- High throughput
- Extremely low latencies
- Consistent performance
- Reliability

**Tradeoffs:** Expensive, vendor lock-in, may require specialized knowledge to configure.

## Auto-Scaling

Auto-scaling automatically adjusts the number of servers based on demand.

### Requirements for Auto-Scaling:
1. Web server instances (e.g., Amazon EC2)
2. Machine image (AMI) that can bootstrap itself
3. Ability for new instances to automatically join the cluster
4. Auto-scaling group with defined scaling policies
5. Monitoring metrics (e.g., CloudWatch)

### How It Works:
- Auto-scaling service monitors cluster metrics
- When thresholds are crossed (high CPU, high request rate), launches new instances
- New instances automatically added to load balancer
- When load decreases, removes instances
- Scales both up and down based on actual demand

## File Storage and CDNs

### Hosting on S3 (or Similar Object Storage)

Public buckets become automatically available over HTTP. You can point CDN directly to them without additional infrastructure.

### Hosting in Private Data Center

Need to put a layer of web servers in front of file storage to allow public HTTP access. CDN then pulls from these web servers.

**Pattern:**
```
CDN → Web Server Layer → File Storage System
```

This allows CDN to cache and serve files globally while your storage remains private.

---

## Key Takeaways

- Front-end layer handles all client connections, making it the highest traffic component
- Stateless design is critical for horizontal scalability
- Session storage has tradeoffs between bandwidth (cookies), latency (data store), and scalability (sticky sessions)
- Load balancers enable transparent scaling and provide features like SSL termination and graceful shutdowns
- Auto-scaling adjusts capacity based on demand, but requires proper infrastructure setup
- CDNs reduce load on origin servers and improve global performance
