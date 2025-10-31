# Chapter 1 — Web Scalability Basics

## 1. Concurrency vs Interaction Rate
**Concurrency** is how many clients your system can handle at the same time.  
Higher concurrency means:
- More open connections
- More active threads
- More messages being processed at once
- More CPU context switches

**Interaction rate** is how often clients talk to the server.  
This creates pressure on **latency**. If users are constantly hitting you with requests, your system has to respond faster (faster reads/writes, faster turnaround). That usually means you’re forced into higher concurrency too.

When concurrency goes up, context switching goes up. A context switch = CPU pauses one process, saves its state, loads another one, runs it. Doing that too often burns CPU time and slows everything down.

---

## 2. Hosting: DNS, VPS, Shared Hosting
Typical request flow:
1. User asks DNS for the IP of your server.
2. Once they have the IP, they send HTTP requests directly to your web server.

- **DNS** is usually not something you run yourself. You normally just use your hosting company’s DNS.
- **VPS (Virtual Private Server)**: You “rent” a virtual machine on a bigger physical machine. You get your own OS and admin/root access, even though you’re sharing hardware with other VPS instances.
- **Shared hosting**: Cheapest option. You only get a user account on a machine, no admin privileges, and you share resources.

---

## 3. Vertical Scaling (Scale Up)
Vertical scaling = make one machine stronger.

Ways to scale vertically:
- Add more disk I/O by using RAID arrays.  
  RAID basically bundles multiple physical drives and exposes them like one logical volume. Reads/writes get spread out. RAID 10 is popular because you get both redundancy and higher throughput.
  - Main point: in database servers, I/O throughput and disk saturation are usually the big bottlenecks.

- Use SSDs instead of HDDs.  
  Random reads/writes are ~10–100x faster on SSDs. Sequential reads/writes are not that much faster. A lot of databases (like MySQL) try to make operations sequential anyway, to avoid depending on random access. Some systems (like Cassandra) are designed around sequential I/O almost entirely, which makes SSDs less of a huge win.

- Add more RAM.  
  More memory = bigger filesystem cache, more working set in memory, fewer disk hits. This is especially important for databases.

- Add more CPU cores / processors.  
  More cores lets you run more things in parallel without making threads fight each other for CPU time. With more cores, the OS does fewer context switches, which helps performance.

Why vertical scaling isn’t perfect:
- It gets extremely expensive after a certain point. You can’t just keep buying “a bigger box” forever.
- Software will limit you. Example: MySQL won’t scale forever just because you keep adding CPUs. You eventually hit **lock contention**.
  - A lock is used so that multiple threads don’t smash the same shared resource (like memory, files, etc.).
  - If your code uses coarse locks, threads end up waiting for each other all the time.
  - Once you hit a lock contention wall, adding more cores doesn’t increase throughput anymore.

One good thing about vertical scaling:  
It doesn’t force you to redesign the whole system. You’re still running the same architecture, just on stronger hardware.

---

## 4. Isolation of Services / Functional Partitioning
A common step after basic scaling is: split services onto different physical machines.

Example:
- Web server on one box
- Database on another
- Cache on another
- DNS, FTP, etc., on their own

The idea is to stop running everything as one giant monolith on one server.  
You break the system into distinct functional parts and host them separately.  
This is called **functional partitioning**.

You also introduce **cache** as its own thing:
- A cache is a service that stores precomputed results / previously generated content.
- Goal: reduce latency and reduce how much work your app has to redo.

---

## 5. Content Delivery Networks (CDNs)
As you get more users, you don’t want every static file request (images, JavaScript, CSS, video, etc.) hitting your own servers.

A **CDN** is basically an HTTP proxy layer for static content:
- The client downloads static files from the CDN instead of directly from you.
- If the CDN already has that file cached, it serves it immediately.
- If not, the CDN asks your server once, caches it, and then serves future users itself.

Benefits:
- Your servers use less bandwidth.
- You need fewer web servers just to push static files.
- Users get lower latency because CDN providers have data centers all over the world.
  - If your app lives in North America and a user is in Europe, normally they’d get higher latency.
  - With a CDN, static content comes from the nearest CDN location, so pages load faster.

So the final pattern looks like:
1. Client hits DNS.
2. Client requests HTML/pages from your servers.
3. Client requests static assets (CSS, JS, images, video) from the CDN.
Result: your servers see less traffic, and users get faster loads.

---

## 6. Horizontal Scaling (Scale Out)
Horizontal scaling = instead of one giant box, you add more boxes.

Why people worship this:
- You avoid the “supercomputer tax.” You’re not forced to buy insanely expensive hardware.
- You can keep growing by just adding more servers. There’s no hard “ceiling” like there is with vertical scaling.
- Cost per capacity unit goes down at large scale. Cloud providers will often give cheaper rates per unit once you’re high volume because supporting one huge fleet is more efficient for them.

### Round-Robin DNS
- Normal DNS: `domain.com -> one IP`.
- Round-robin DNS: `domain.com -> multiple IPs`.  
  Each IP is a different server. DNS rotates which IP it hands out.
  - Different clients get routed to different servers without knowing.
  - After a client gets one IP, it keeps talking to that same server.

---

## 7. Global Scalability
Once you’re operating in multiple regions, you’re basically running multiple data centers.

Two important tools:

### GeoDNS
GeoDNS is DNS that returns **different IPs depending on where the user is**.
- A user in Europe and a user in Australia can get different IPs.
- That way each user gets sent to the closest data center.
- Goal: minimize network latency across the planet.

### Edge Cache
An **edge cache** is an HTTP cache server physically near the user.
- The browser sends the request to the edge cache first.
- The edge cache can:
  - Serve a full cached page immediately,
  - Assemble missing pieces of the page by making background requests to your real application servers,
  - Or, if the page can’t be cached, just forward the request to your app.

You usually use edge cache + CDN together:
- Dynamic/semi-dynamic stuff can be served or partially built from the edge cache in, say, Europe.
- Static files (CSS, JS, images, video) come from the CDN node closest to that user.
- Your core application servers don’t have to be physically near every user, but the perceived latency drops anyway.

---

## 8. High-Level Data Center Layout
A modern setup usually looks like this:

1. **geoDNS**  
   User asks DNS for your domain. GeoDNS picks the “closest” data center and returns the IP of a load balancer in that region.

2. **Load Balancer**  
   A load balancer sits in front of a pool of servers and exposes a single IP.  
   - It spreads traffic across all the backend machines.
   - You can add or remove servers behind it without users noticing.

3. **Front Cache / Front-End Web App Servers**  
   Traffic from the load balancer gets distributed across:
   - Front cache servers, or
   - Front-end web application servers.

### Web Application Layer
- These front-end app servers generate HTML and respond to HTTP requests.
- Usually run something like PHP / Java / Ruby / Groovy frameworks.
- They’re supposed to stay pretty “dumb”: mostly just handle user interaction, render UI, and translate requests into calls to internal services.
- They should be stateless.

### Web Services Layer
- This layer contains most of the actual business logic.
- It’s often split into multiple specialized services (functional partitioning again).
- Front-end talks to these services over HTTP, usually using REST or SOAP.
- Because services are separate, you can scale each service independently.

### Extra Components
- **Object cache servers**: store precomputed / partially computed results to reduce database load and speed up responses.
- **Message queues**: let you delay work and process it asynchronously.
  - Front-end and services both push messages onto queues.
  - Dedicated workers (queue worker machines / batch processors) consume those messages later.
  - These workers also handle stuff like notifications, order fulfillment, anything that’s high latency and doesn’t need to block the user’s request.

### Persistence Layer
- There’s no rule that says “one database only.”
- Companies use multiple different data stores at the same time (“polyglot persistence”) to get the best scalability behavior for each kind of data.

---

## 9. Architecture and Business Reality
Your architecture should reflect your **business model**, not just your favorite framework.

To do that we define a **domain model**:
- The domain model is written in business language, not tech jargon.
- It lists the important concepts, the actors, and what they can do.
- Example for an ATM: account, cash, debit, credit, authentication, security policy.
- It intentionally ignores technical implementation details.
- It’s there so everyone has the same mental model of “what problem are we actually solving?”

---

## 10. Design Principles for Scalable Systems
### 1. Simplicity
This is the most important rule.
- If the system gets too complex, nobody understands it.
- If nobody understands it, you can’t safely scale it.
- Simple + working beats clever + broken.

### 2. Loose Coupling
Second most important rule.
- Low coupling = different parts of the system don’t depend too tightly on each other.
- That means engineers can work in parallel without needing to understand the entire codebase.
- You can also scale teams (hire more devs) without total chaos.

### Waste / Time Sinks
Things that repeatedly kill engineering time:
- **Bad process** (slow reviews, slow releases, etc.).
- **No automation**: manually deploying, testing, provisioning servers, configuring dev machines, etc. At first it feels “fast,” but it slowly eats your entire day.
- **Reinventing the wheel / Not Invented Here**: writing code that already exists in mature form (hashing functions, trees, frameworks, DB layers, etc.). This is super common with junior devs who just want to build everything themselves.
- **“It’s just a one-off hack, I won’t need it again.”**  
  Then you *do* need it again, except now the code is garbage, undocumented, untested, and other people start copy-pasting it.

---

## 11. Diagrams and Thinking Before You Code
If you struggle to design first, start by documenting things you’ve *already* built. Draw diagrams of features you know well. That builds design instinct.

Then move toward sketching designs earlier:
- Think from the client’s point of view.
- Sketch class boundaries.
- Write high-level unit tests for interfaces before you fully implement them.
- Use diagrams to expose flaws before you ship broken code.

### Use Case Diagrams
- Show which users exist and what actions they need to perform.
- They focus on business needs, not implementation.
- Good for clarifying requirements when you add a new feature.

### Class Diagrams
Some rules:
- Interfaces should depend on other interfaces, not on concrete classes.
- Classes should depend on interfaces as much as possible.
- **Single Responsibility Principle (SRP):**  
  A class should do one thing. If you keep dumping random behavior into one giant class, it becomes impossible to reason about, and everything gets tightly coupled.
- **Open/Closed Principle:**  
  Code should be open to extension but closed to modification.  
  Translation: you should be able to add new behavior without constantly editing old code in risky ways.

---

## 12. Layered / Service-Oriented Architecture
When you split a system into services, you’re basically moving toward **service-oriented architecture (SOA)**: multiple loosely coupled, mostly independent services that solve specific business needs.

Think of something like a ride-sharing app:
- One service for maps/location
- One for payments
- One for social / identity
- One for the core ride logic
- The user just sees “the app,” but behind it is a bunch of services talking to each other.

In a multilayer architecture:
- Upper layers are closer to the UI. They move faster, have richer features, and tend to have unstable APIs because they keep changing.
- Lower layers are more stable, simpler, and expose cleaner APIs.
- The whole point is to hide complexity behind abstractions, limit dependencies, let each part scale on its own, and let teams work in parallel.

---

## 13. Big Picture
Designing scalable web applications means understanding how all of this interacts:
- Architecture (layers, services, caches, queues, etc.)
- Infrastructure (DNS, load balancers, CDNs, edge caches, data centers)
- Technology choices (databases, frameworks, messaging)
- Algorithms and data access patterns
- The actual business requirements (what are we even building?)

---

## IMPORTANT (Ultra Short)

- Bottleneck at high traffic = latency. Faster replies need faster I/O and higher concurrency.
- VPS = you get a virtual server on shared hardware. Shared hosting = same idea but no admin control.
- DBs choke on disk I/O. RAID helps by spreading reads/writes across multiple disks.
- Too many context switches wastes CPU.
- Vertical scaling (bigger box) is easy architecturally but gets insanely expensive and hits limits like lock contention.
- CDN/edge cache push content closer to the user to cut latency and offload your servers.
- Round-robin DNS and GeoDNS split users across servers / data centers (ideally closest to them).
- Load balancer = single entry point that fans traffic out and lets you add/remove servers live.
- Front-end app servers = render HTML + handle HTTP.
- Web services layer = core business logic, can be split into independent services (SOA / functional partitioning).
- Good architecture is layered, loosely coupled, and built around the business, not the framework.

