# Chapter 4 — Web Services

Developing with an **API-first approach** is a cleaner way to build software, but it requires:
- More upfront planning
- Clear understanding of final requirements
- Experienced engineering resources
- Knowledge of how to design scalable and flexible web services

The tradeoff is that you get better long-term maintainability and clearer separation between front-end and back-end.

## Types of Web Services

### Function-Centric Services

The concept is to call functions or object methods on remote machines without needing to know how they're implemented.

**SOAP (Simple Object Access Protocol)** is the dominant technology in this category. It focuses on letting client code invoke functions implemented on remote machines.

How it Works

SOAP is a system that lets different programs communicate over the internet, even if they're written in different languages or run on different machines.

For this to work, both sides (client and server) need to agree on how to communicate:
- What messages they'll send
- What data types they'll use
- What functions exist
- What each function expects

This shared agreement is written down in a contract -> (WSDL + XSD). It is defined using XML files: 
- Describes what the service does
- Lists available functions (endpoints)
- Specifies function names
- Defines parameters each function takes
- Describes return data types

**XSD (XML Schema Definition)**
- Describes what the data looks like
- Defines exact structure of messages being sent and received

#### SOAP Request Flow

1. Client sends XML request to SOAP server (usually via HTTP POST)
2. XML body includes method name and parameters
3. Server processes the request
4. Server sends back XML response
5. Tools/libraries handle serialization, deserialization, and network communication automatically

#### SOAP Scalability Problem: No HTTP-Level Caching

- SOAP requests are sent as POST requests to a single endpoint (like `/soap-service`)
- The URL never changes, even when request content differs
- Request details (like `getUser(123)`) are hidden inside the XML body
- HTTP caching relies on URL and method, so SOAP messages cannot be cached

To HTTP, every request looks identical even though the data inside is different.

Consequences:
- Every request goes to the application server directly
- No reverse proxies, CDNs, or HTTP caches can reduce load
- Makes SOAP less scalable in high-traffic systems with repeated queries
- REST and modern APIs solve this by making the URL itself meaningful

### Resource-Centric Services

This approach focuses on the concept of a **resource** rather than a function.

**REST (Representational State Transfer)** is the primary example. It's basically an HTTP server with a routing mechanism to map URL patterns to your code.

REST uses meaningful URLs and HTTP methods:
- `GET /users/123` - retrieve user 123
- `POST /users` - create new user
- `PUT /users/123` - update user 123
- `DELETE /users/123` - delete user 123

Making web services stateless provides several advantages:

### Load Balancing: Deploy a load balancer and distribute requests in round-robin fashion. Any server can handle any request.
### Failure Handling: Take crashed service machines out of the pool immediately without affecting user experience. No sessions or state is lost because none was stored on the server.
### Graceful Decommissioning: Can remove servers from rotation cleanly for maintenance or updates.
### Zero-Downtime Updates. Roll out changes one server at a time. Users never experience downtime.
### Easy Scaling: Scale by adding more identical clones of your service. All instances are interchangeable.
### Auto-Scaling: Implement auto-scaling the same way as for the front-end layer. Add/remove instances based on demand.

## Resource Locking in Stateless Services

A common problem with stateless web services is supporting **resource locking**.
This can be fixed by buuilding your own lock serviceusing a data store to track which resources are locked and by whom. Or Use existing lock systems like ZooKeeper that provide distributed coordination.

### Challenges of Distributed Locking:

- Each lock requires a remote call, creating opportunities for the service to stall or fail
- Network can drop packets
- Clients can time out
- Increases latency
- Reduces the number of parallel clients your web service can handle

## Distributed Transactions

When you need to coordinate changes across multiple services or databases, you need distributed transactions.

**Most common method:** 2PC (Two-Phase Commit) algorithm

2PC ensures all participating services either commit the transaction or all roll it back. This maintains consistency across distributed systems, though it comes with performance costs and increased complexity.

## HTTP Method Semantics

### GET Requests Must Be Read-Only

**Rule:** GET requests must only retrieve data and never change server state.

**Why this matters:**

**Enables HTTP caching:** Browsers and proxies can reuse previous GET responses instead of re-fetching them.

**Consistency:** If GET modifies data, caches may serve outdated or incorrect results, breaking data consistency.

**Performance:** Proper GET usage improves scalability and speed by reducing server load and network traffic.
