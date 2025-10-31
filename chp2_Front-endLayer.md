# Chapter 2 — Managing Complexity and System Design

## 1. The Idea of Local Simplicity
As systems grow, they can’t stay completely simple — so we aim for **local simplicity** instead.

Local simplicity means:
- When you open a *class*, you can quickly understand what it does.
- When you look at a *module* (a collection of classes), you can ignore the individual methods and see the bigger picture:  
  “These classes work together to accomplish X.”

You can imagine the system as a **graph**, where:
- Classes = nodes  
- Dependencies between them = edges  

Complexity isn’t about how many classes you have — it’s about how many connections exist between them.  
A class depending on five other classes is far harder to maintain than one depending on just one or two.

> **Lesson:** Simplicity is the most fundamental design value. Without it, engineers can’t understand the code, and if they can’t understand it, they can’t safely extend or maintain it. Growth dies when comprehension dies.

---

## 2. Loose Coupling
**Loose coupling** means each component knows as little as possible about the others.

### Why It Matters
- Changing one component shouldn’t break everything else.  
- When components are independent, you can replace or upgrade one without rewriting the whole system.  
- It also makes debugging and scaling teams easier since no one needs to understand *everything* to work on *something*.

**Structure reminder:**
- You have **applications** → made of **modules** → made of **classes**.  
- **Classes** are the smallest unit of abstraction.

---

## 3. Common Productivity Killers

### DRY (Don’t Repeat Yourself)
Copy/paste programming breaks this rule.  
If you duplicate code across apps, every bug must be fixed in multiple places. That wastes time and invites errors.

### Lack of Automation
Manually deploying, testing, building, or configuring development environments might seem harmless early on, but it snowballs.  
Eventually, most of your day is spent doing repetitive setup instead of writing actual code.  
**Automate early.**

---

## 4. Contracts in OOP
In programming, a **contract** defines the rules of interaction between parts of your system.

- For **methods**, the contract = the method signature (what parameters it expects and what it returns).  
- For **classes**, the contract = its public interface (what behaviors it exposes).

Contracts help maintain boundaries between parts of your code — each piece promises to behave a certain way, without revealing how.

---

## 5. The Open/Closed Principle
The **Open/Closed Principle** says:  
> Classes should be *open for extension* but *closed for modification*.

That means you should be able to add new functionality without rewriting old code.  
This keeps existing, stable code from constantly being touched (and possibly broken) when requirements change.

---

## 6. Interfaces and Dependencies
**Interfaces** describe *what* something can do.  
**Concrete classes** define *how* they do it.

### Rules to Remember
- Interfaces should only depend on other interfaces — never on specific implementations.  
- Concrete classes should depend on interfaces whenever possible.  

This keeps your code flexible and reduces how tightly components are bound together.  
If an interface mentions a concrete class, it’s like hardcoding a dependency — that kills flexibility.

---

## 7. Dependency Injection & Inversion of Control (IoC)

### Dependency Injection (DI)
**Dependency Injection (DI)** is a technique to reduce coupling and encourage the open/closed principle.  
Instead of a class *creating* what it needs, you *give* it what it needs from the outside.

```java
// Instead of this:
class ReportService {
    Database db = new Database();
}

// Do this:
class ReportService {
    Database db;
    ReportService(Database db) {
        this.db = db;
    }
}
Now `ReportService` doesn’t care *which* database it gets — just that it gets one.

---

## 7. Inversion of Control (IoC)

**Dependency Injection (DI)** is part of a bigger idea called **Inversion of Control (IoC)**.

IoC means something external (like a framework) controls when your code runs.  
Instead of your code calling the framework, the **framework calls you**.

### Example
A web framework might call your `handleRequest()` method whenever a new request arrives.  
You don’t decide when this happens — the framework does.  
You just define what to do when it happens.

IoC doesn’t reduce *overall* system complexity, but it simplifies your local application code because the framework handles much of the control flow for you.

---

## 8. Functional and Data Partitioning

### Functional Partitioning
In infrastructure terms, **functional partitioning** means isolating server roles.

You divide your system into different server types:
- Web servers  
- Object cache servers  
- Message queue servers  
- Queue worker machines  
- Data stores  
- Load balancers  

Each role focuses on one job, allowing you to scale or replace that function independently.

---

### Data Partitioning
**Data partitioning** means dividing your data across multiple servers instead of cloning everything.

#### Benefits
- Enables massive (potentially endless) scalability — you can just add more partitions.

#### Challenges
- You must be able to **find** which server has the data before querying.  
- Queries that touch multiple partitions are much slower and more complex to manage.

> So while data partitioning offers great scalability, it’s also the hardest and most expensive technique to get right.

---

## 9. Reliability and Self-Healing

### Single Point of Failure (SPOF)
A **single point of failure** is any part of your system that, if it fails, brings everything down.  

Example:
- A master database server with no replica.  
If it dies, your entire app goes offline.

**Goal:** design systems with *no single points of failure.*

---

### Self-Healing Systems
The **holy grail** of web operations.  
A self-healing system doesn’t just fail gracefully — it *detects problems and fixes them automatically*, without human intervention.

#### Examples
- Restarting crashed services automatically  
- Rebalancing traffic when servers fail  
- Repairing failed storage nodes in a cluster  

---

### Availability Formula
To measure how available a system is:

