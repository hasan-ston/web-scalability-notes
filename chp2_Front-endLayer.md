# Chapter 2 — Managing Complexity and System Design

## Local Simplicity

As systems grow, maintaining overall simplicity becomes impractical. The focus shifts to **local simplicity** - ensuring individual components remain understandable in isolation.

Key principles:
- When examining a class, its purpose and behavior should be quickly apparent
- When reviewing a module (collection of classes), the individual method implementations can be abstracted away to understand the module's overall purpose

Systems can be visualized as graphs where classes are nodes and dependencies are edges. Complexity stems not from the number of classes, but from the density of dependencies between them. A class depending on five others is significantly harder to maintain than one with one or two dependencies.

## Loose Coupling

It means minimizing the knowledge each component has about others.

Benefits:
- Changes to one component have minimal impact on others
- Components can be replaced or upgraded independently
- Parallel development is facilitated as teams don't need complete system knowledge
- Debugging becomes more tractable when concerns are separated

Structural hierarchy reminder:
- Applications → Modules → Classes
- Classes represent the fundamental unit of abstraction

## Common Productivity Issues

### DRY Principle (Don't Repeat Yourself)
Code duplication creates maintenance burden. When logic is duplicated across multiple locations, bug fixes and updates must be applied consistently everywhere. This increases both development time and the chances of introducing inconsistencies.

### Manual Processes
Manual deployment, testing, and environment configuration may seem manageable initially but scales poorly. These tasks compound over time, eventually consuming more resources than feature development. Early automation investment pays off significantly.

## Contracts in Object-Oriented Programming

A contract defines the interface between system components, establishing clear expectations for interaction.

**Method contract:** The method signature specifies expected parameters and return values
**Class contract:** The public interface exposes available behaviors and operations

Contracts establish boundaries that allow components to interact predictably without exposing internal implementation details.

## Open/Closed Principle

It states that classes should be open for extension but **closed for modification.

This means new functionality should be addable without modifying existing, stable code. This approach reduces the risk of introducing bugs into working systems when requirements evolve.

## Interfaces and Dependencies

**Interfaces** define contracts (what operations are available).  
**Concrete classes** provide implementations (how operations are performed).

Dependency guidelines:
- Interfaces should depend only on other interfaces, never on concrete implementations
- Concrete classes should depend on interfaces wherever practical

When interfaces reference concrete classes directly, it creates rigid dependencies that eliminate flexibility, which is why they should reference other interfaces.

## Dependency Injection (DI)

**Dependency Injection** inverts the responsibility for creating dependencies. Instead of a class instantiating its own dependencies, they are provided externally.

Without DI:
```java
class ReportService {
    Database db = new Database();  // Creates own dependency
}
```

With DI:
```java
class ReportService {
    Database db;
    ReportService(Database db) {  // Dependency injected
        this.db = db;
    }
}
```

This approach decouples `ReportService` from specific database implementations. It also allows mock dependencies to be injected, making testing easier.

## Inversion of Control (IoC)

Dependency Injection is one aspect of the broader **Inversion of Control** pattern.

In traditional programming, application code calls libraries and frameworks. With IoC, this relationship inverts - the framework calls application code at appropriate times.

Example: Web frameworks call application-defined request handlers when HTTP requests arrive. The framework controls execution flow; the application defines behavior.

IoC doesn't reduce overall system complexity, but it does simplify application level code by delegating control flow management to the framework.

## Functional Partitioning

**Functional partitioning** organizes infrastructure by distinct roles or functions.

Common functional partitions:
- Web servers (HTTP request handling)
- Cache servers (temporary data storage)
- Message queue servers (asynchronous task distribution)
- Worker servers (background job processing)
- Database servers (persistent data storage)
- Load balancers (traffic distribution)

Each function operates independently, allowing targeted scaling and replacement without affecting other system components.

## Data Partitioning

It distributes data across multiple servers rather than replicating complete datasets.

**Advantages:**
- Enables horizontal scaling - capacity increases by adding partitions
- Theoretically unlimited scalability

**Challenges:**
- Requires mechanism to locate data before querying (routing logic)
- Cross-partition queries are complex and expensive
- Rebalancing partitions during growth is non-trivial

Data partitioning provides the best scalability characteristics but is also the most complex and expensive partitioning strategy to implement correctly.

## Reliability and Fault Tolerance

### Single Point of Failure (SPOF)

It is any component whose failure causes complete system failure.

Example: A master database without replication. If it fails, the entire application becomes unavailable.

Design goal: Eliminate single points of failure through redundancy and failover mechanisms. In practice, achieving zero SPOFs is challenging, but identifying and mitigating critical failure points significantly improves reliability.

### Self-Healing Systems

They detect failures and automatically recover without manual intervention.

Common self-healing mechanisms:
- Automatic service restarts after crashes
- Traffic rerouting around failed servers
- Automatic storage node repair and rebalancing

### Availability Calculation

```
Availability = MTBF / (MTBF + MTTR)
```

Where:
- **MTBF** = Mean Time Between Failures
- **MTTR** = Mean Time To Recovery

Availability improves by either reducing failure frequency (higher MTBF) or decreasing recovery time (lower MTTR). Both approaches are valuable but address different aspects of system reliability.
