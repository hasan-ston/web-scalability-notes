# Chapter 1 — Web Scalability Basics

## Concurrency vs Interaction Rate

Concurrency refers to the number of clients a system can handle at a time. Higher concurrency implies:
- More open connections
- More active threads
- Increased message processing load
- More frequent CPU context switches

Interaction rate measures how frequently clients communicate with the server. This metric directly impacts latency requirements. When users generate frequent requests, the system must respond quickly (faster I/O operations, reduced turnaround time), which typically requires higher concurrency support.

As concurrency increases, context switching overhead grows. A context switch occurs when the CPU pauses one process, saves its state, loads another process, and executes it. Excessive context switching consumes CPU resources and degrades overall system performance.

## Hosting Infrastructure: DNS, VPS, Shared Hosting

Typical request flow:
1. Client queries DNS to resolve the domain to an IP address
2. Client sends HTTP requests directly to the web server at that IP

**DNS:** Typically managed by hosting providers rather than operated independently.

**VPS (Virtual Private Server):** A virtualized environment on shared physical hardware. Provides dedicated OS instance and administrative access. Resourcs shared with other VPS instances.

**Shared Hosting:** Most economical option. Users receive basic level access on shared machines without administrative privileges.

## Vertical Scaling (Scale Up)

It involves increasing server(s) capacity.

### Common vertical scaling approaches:

**Disk I/O Enhancement via RAID**
RAID (Redundant Array of Independent Disks) aggregates multiple physical drives into a logical volume, sharing read/write operations between HDDs. RAID 10 is widely adopted for providing both redundancy and improved throughput. Database servers frequently experience I/O saturation as their primary bottleneck.

**SSD Adoption**
SSDs provide 10–100x performance improvement for random access operations compared to HDDs. Sequential operation performance gains are much less. Many databases (including MySQL) optimize for sequential access patterns to minimize random I/O dependency. Some systems like Cassandra are architected almost entirely around sequential I/O, which makes SSDs a less attractive option.

**Memory Expansion**
Additional RAM enables larger filesystem caches, increases working set capacity in memory, and reduces disk access frequency. This is especially useful for database performance.

**CPU Core/Processor Addition**
More cores enable increased parallelization without thread contention for CPU time. Reduced context switching improves performance as the OS has to manage fewer process transitions.

### Limitations of vertical scaling:

**Cost escalation:** Hardware costs increase exponentially beyond certain thresholds, making it economically impractical.

**Software constraints:** Applications don't scale infinitely with hardware. MySQL, for example, encounters lock contention that limits CPU scalability. Locks coordinate access to shared resources (i.e memory, files) between threads. Coarse-grained locking causes threads to wait on each other. Once lock contention becomes the bottleneck, additional cores no longer improve throughput (hard limit).

**Advantage:** Vertical scaling doesn't require architectural redesign. The system runs on upgraded hardware.

## Service Isolation and Functional Partitioning

A common scaling step involves partitioning services across dedicated physical machines.

Example distribution:
- Web server on dedicated hardware
- Database on separate server
- Cache infrastructure independent

This approach replaces monolithic single-server deployments with functional partitioning - organizing the system into separate functional components hosted separately.

**Cache systems** emerge as independent services in this model:
- Store prefetched results and previously generated content
- Reduce latency and eliminate redundant processing

## Content Delivery Networks (CDNs)

At scale, serving all static file requests (images, JavaScript, CSS, video) from application servers becomes inefficient. Users can benefit from this integration as CDNs have many servers around the world, so they are likely to experience less latency.

**CDNs** function as HTTP proxy layers for static content:
- Clients retrieve static files from CDN infrastructure rather than origin servers
- CDNs serve cached content immediately when available, reducing latency.
- On cache miss, CDN retrieves content from original server once, caches it, then serves subsequent requests independently.

### Benefits:
- Reduced origin server bandwidth consumption
- Fewer web servers required for static content delivery
- Decreased user latency through geographically distributed CDN nodes
  - Users far from main servers experience higher latency accessing content directly
  - CDN edge locations serve content from nodes nearest to users

### Resulting architecture:
1. Client performs DNS lookup
2. Client requests dynamic content (HTML/pages) from application servers
3. Client retrieves static assets from CDN infrastructure

This pattern reduces load on main server while improving user experience.

## Horizontal Scaling (Scale Out)

Horizontal scaling adds additional machines rather than upgrading individual servers.

### Advantages:
- Avoids premium pricing for high-end hardware
- Enables continued growth through incremental server additions without hard capacity ceilings
- Reduces per-unit cost at scale as cloud providers offer volume discounts

### Round-Robin DNS
Standard DNS maps domains to single IP addresses. Round-robin DNS returns multiple IPs:
- Each IP represents a different server
- DNS rotates which IP is provided to different clients
- Clients maintain connections to their assigned server
- Distribution occurs transparently to end users
  
## Global Scalability

Require multiple data center deployments.

### GeoDNS
GeoDNS returns location-specific IP addresses based on client geography:
- European and Australian users receive different IPs
- Routes users to nearest data center, minimizing latency 

### Edge Cache
Edge caches are HTTP cache servers positioned near users geographically:
- Browsers initially contact edge cache
- Edge cache can:
  - Serve fully cached pages immediately
  - Assemble partial pages through background requests to application servers
  - Forward uncacheable requests to application servers

Edge caches typically work in conjunction with CDNs:
- Dynamic/semi-dynamic content served from regional edge caches
- Static assets delivered from nearest CDN node
- Core application servers need not be geographically distributed, yet perceived latency remains low

## Data Center Architecture

Modern data center infrastructure typically follows this pattern:

### 1. GeoDNS Layer
DNS queries return IP addresses for load balancers in the user's nearest region.

### 2. Load Balancer
Presents single IP address while distributing traffic across server pools:
- Spreads load across backend machines
- Enables transparent server additions/removals

### 3. Front Cache / Application Server Layer
Load balancer distributes traffic to:
- Front cache servers
- Front-end web application servers

### Web Application Layer
Front-end servers handle HTTP requests and generate HTML responses:
- Common technologies: PHP, Java, Ruby, Groovy frameworks
- Primary responsibilities: user interaction, UI rendering, request translation to service calls
- Should maintain stateless design

### Web Services Layer
Contains core business logic:
- Often split into specialized services (functional partitioning)
- Front-end communicates via HTTP (typically REST or SOAP)
- Independent service scaling becomes possible

### Additional Infrastructure Components

**Object cache servers:** Store precomputed/partially computed results to reduce database load and improve response times

**Message queues:** Enable asynchronous work processing:
- Front-end and services publish messages to queues
- Dedicated workers consume messages independently
- Handle high-latency operations (notifications, order fulfillment) without blocking user requests

### Persistence Layer
Modern architectures employ multiple specialized data stores (polyglot persistence) to optimize scalability characteristics for different data types.


## Architecture and Business Alignment

Architecture should reflect business requirements rather than framework preferences.

### Domain Model
Defines the system in business terminology:
- Lists key concepts, actors, and available operations
- Example for ATM system: account, cash, debit, credit, authentication, security policy
- Deliberately excludes technical implementation details
- Establishes shared understanding of the problem domain

## Design Principles for Scalable Systems

### Simplicity
The primary design principle. Complex systems become incomprehensible, and incomprehensible systems cannot be safely scaled or maintained.

### Loose Coupling
Second critical principle. Minimal coupling between components allows:
- Parallel development without requiring complete codebase understanding
- Team scaling without coordination problems

### Progressive design approach:
- Consider user perspective first
- Sketch component boundaries
- Write high-level interface tests before full implementation
- Use diagrams to identify design flaws before implementation

### Use Case Diagrams
- Identify user types and required actions
- Focus on business requirements rather than implementation
- Clarify feature requirements

### Class Diagrams
Key principles:
- Interfaces should depend on other interfaces, not concrete implementations
- Classes should depend on interfaces wherever practical
- **Single Responsibility Principle:** Classes should have focused, singular purposes. Accumulating diverse behaviors in single classes creates reasoning difficulty and tight coupling.
- **Open/Closed Principle:** Code should support extension without requiring modification. New functionality should be addable without risky changes to existing code.

## Service-Oriented Architecture

It consists of multiple loosely coupled, largely independent services addressing specific business needs.

### Layered Architecture Characteristics:
- Upper layers (closer to UI): rapid iteration, rich features, frequently changing APIs
- Lower layers: stable, simplified, clean APIs
- Benefits: complexity abstraction, limited dependencies, independent scaling, parallel team development

## Key Takeaways

- High traffic bottlenecks manifest as latency issues. Reduced response times require faster I/O and higher concurrency support.
- VPS provides virtualized servers on shared hardware. Shared hosting offers similar resource sharing without administrative access.
- Database performance often bottlenecks on disk I/O. RAID distributes operations across multiple disks.
- Excessive context switching wastes CPU cycles.
- Vertical scaling (hardware upgrades) is architecturally straightforward but becomes prohibitively expensive and encounters limits like lock contention.
- CDN/edge cache infrastructure reduces latency by serving content from locations near users, offloading origin servers.
- Round-robin DNS and GeoDNS distribute users across servers/data centers, ideally routing to nearest locations.
- Load balancers provide single entry points that distribute traffic and enable live server additions/removals.
- Front-end application servers handle HTTP requests and HTML rendering.
- Web services layer contains core business logic, can be decomposed into independent services (SOA/functional partitioning).
- Effective architecture is layered, loosely coupled, and aligned with business requirements rather than framework constraints.
