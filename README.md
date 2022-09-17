This is my summary of Monolith to Microservices, by Sam Newman.

I recommend you to buy and read the book to learn any concept directly from its author.

# Contents

- [1. Just Enough Microservices](#1-just-enough-microservices)
  - [What Are Microservices?](#what-are-microservices)
  - [The Monolith](#the-monolith)
  - [On Coupling and Cohesion](#on-coupling-and-cohesion)
- [2. Planning a Migration](#2-planning-a-migration)
  - [Why Might You Choose Microservices?](#why-might-you-choose-microservices)
  - [When Might Microservices Be a Bad Idea?](#when-might-microservices-be-a-bad-idea)
  - [Importance of Incremental Migration](#importance-of-incremental-migration)
  - [How Do We Start?](#how-do-we-start)
  - [How Will You Know if the Transition Is Working?](#how-will-you-know-if-the-transition-is-working)
- [3. Splitting the Monolith](#3-splitting-the-monolith)
  - [Pattern: Strangler Fig Application](#pattern-strangler-fig-application)
  - [Pattern: UI Composition](#pattern-ui-composition)
  - [Pattern: Branch by Abstraction](#pattern-branch-by-abstraction)
  - [Pattern: Parallel Run](#pattern-parallel-run)
  - [Pattern: Decorating Collaborator](#pattern-decorating-collaborator)
  - [Pattern: Change Data Capture (CDC)](#pattern-change-data-capture-cdc)
- [4. Decomposing the Database](#4-decomposing-the-database)
  - [The Shared Database](#the-shared-database)
  - [Transferring Ownership](#transferring-ownership)
  - [Data Synchronization](#data-synchronization)
  - [Splitting Apart the Database](#splitting-apart-the-database)
  - [Schema Separation Examples](#schema-separation-examples)
  - [Transactions](#transactions)
  - [Sagas](#sagas)
- [5. Growing Pains](#5-growing-pains)
  - [Ownership at Scale](#ownership-at-scale)
  - [Breaking Changes](#breaking-changes)
  - [Reporting](#reporting)
  - [Monitoring and Troubleshooting](#monitoring-and-troubleshooting)
  - [Local Developer Experience](#local-developer-experience)
  - [Running Too Many Things](#running-too-many-things)
  - [End-to-End Testing](#end-to-end-testing)
  - [Global Versus Local Optimization](#global-versus-local-optimization)
  - [Robustness and Resiliency](#robustness-and-resiliency)
  - [Orphaned Services](#orphaned-services)
- [Patterns Index](#patterns-index)

# 1. Just Enough Microservices

## What Are Microservices?

Microservices are independently deployable services modeled around a business domain. They comunicate to each other via networks making them a kind of distributed system.

They also encapsulate data storage and retrieval, exposing data, via well defined interfaces. So databases are hidden inside the service boundary.

| Advantages | Disadvantages |
| ---------- | ------------- |
| Flexibility<br>Scalability<br>Robustness<br>Mixed technologies<br>Easy to understand | Communication between services<br>Network latency<br>Transactions |

## The Monolith

When all the functionality in a system had to be deployed together, it is considered a monolith.

- **Single-Process Monolith**: All the code is packed into a single process.
- **Modular Single-Process Monolith**: Same as single-process monolith but separated in modules.
- **Distributed Monolith**: System that consists of multiple services, but the entire system has to be deployed together.

| Advantages | Disadvantages |
| ---------- | ------------- |
| Simpler deployment topology<br>Simpler development workflow<br>Simpler monitoring<br>Simpler troubleshooting<br>Simpler End-to-End testing<br>Simpler code reuse | Coupling<br>Crashes between developers<br>Confusion around ownership |

## On Coupling and Cohesion

**Cohesion**

"Code that changes together, stays together". Microservices architecture should be optimized around ease of making changes. We want functionality grouped in such way that we can make changes in as few places as possible.

**Coupling**

- Implementation coupling: A is coupled to B in terms of how B is implemented. When the implementation of B changes, A also changes.
- Temporal coupling: When a message is sent, and how that message is handled is connected in time.
- Deployment coupling: Everything must be deployed together.
- Domain coupling: The interactions between services model the interactions in our real domain.

# 2. Planning a Migration

Microservices are not the goal. You don't "win" by having microservices. You should be thinking of migration to a microservices architecture in order to achieve something that you can't currently achieve with your existing system architecture.

You should ask these three question if you are considering to adopt microservices architecture:

- What are you hoping to achieve?
- Have you considered alternative to using microservices?
- How will you know if the transition is working?

## Why Might You Choose Microservices?

| Reason | Alternatives |
| ------ | ------------ |
| Improve Team Autonomy | Modular monolith<br>Assign responsibilities based on functional grounds |
| Reduce Time to Market | Path-to-production modeling to identify bottlenecks |
| Scale Cost-Effectively for Load | Vetical scaling<br>Horizontal scaling<br>Technology replacement |
| Improve Robustness | Multiple copies of your monolith behind load distribution mechanism<br>Use more reliable hardware and software<br>Automate manual processes |
| Scale the Number of Developers | Modular monolith (but requires deployment coordination) |
| Embrace New Technology |  |

## When Might Microservices Be a Bad Idea?

- **Unclear Domain**: Can lead with high cost of change and ownership. Get a firmer understanding of the domain before decomposing the system.
- **Startups**: Once you have initial success as startup, microservices could be a great way of solving the sort of problems you could have, not before.
- **Customer-Installed and Managed Software**: Increase of operational complexity.
- **Not Having a Good Reason**: Don't do Microservices just because everyone else is doing it.

## Importance of Incremental Migration

- **Step by Step**: Break the big journey into little steps. Each step can be carried out and learned from and allow you to to identify quick wins. The smaller the change, the smaller the potential negative impacts you’ll see, and the faster you can address them when they occur.
- **It's Production That Counts**: The vast majority of the important lessons will not be learned until your service hots production.

## How Do We Start?

- **Identify Bounded Contexts** and the relationships between them (Event Storming can help).
- Services with **less in-bound dependencies** would be easier to extract (Code could not be aligned with the model design).
- **Esase vs Benefit** of decomposition, we want to start with the easiest services that bring most benefit.
- **Reorganize teams** to align architecture and organization, and more responsible of the end-to-end delivery cycle.

## How Will You Know if the Transition Is Working?

**Having Regular Checkpoints**:  
1. Restate what you are trying to achieve. Does it still make sense?
2. Review any quantitative and qualitative measures you have in place to see whether you’re making progress.
3. Ask for qualitative feedback. Do people think things are still working out?
4. Check and decide if any change is needed.

**Avoiding the Sunk Cost Fallacy**: It occurs when people become so invested in a previous approach to doing something that even if evidence shows the approach isn’t working, they’ll still proceed anyway.

**Being Open to New Approaches**: Try to embrace a culture of constant improvement, to always have something new you’re trying, then it becomes much more natural to change direction when needed.

# 3. Splitting the Monolith

## Pattern: Strangler Fig Application

The idea is that the old and the new can coexist, giving the new system time to grow and potentially entirely replace the old system. The key benefit to this pattern, is that it supports incremental migration to a new system and it gives us the ability to pause and even stop the migration altogether, while still taking advantage of the new system delivered so far and also ensure that each step is easily reversible, reducing the risk of each incremental step.

**Steps**:  
1. Identify parts of the existing system that you wish to migrate.
2. Implement this functionality in your new microservice (avoid any functionality change).
3. Reroute calls from the monolith over to the new microservice (reverse proxy if HTTP is used).
4. Do a incremental rollout (canary release, parallel run,...)

## Pattern: UI Composition

Compose a user interface from multiple different parts that can be managed and deployed separately. These components can be served from different backing systems (perhaps separate microservices), and can be changed independently from each other, allowing different teams to work in parallel and push out changes as and when they are ready:

- Page composition
- Widget composition
- Micro-frontends

## Pattern: Branch by Abstraction

Create a single abstraction point over the functionality to be reimplemented, allowing both the existing functionality and the new implementation to co-exist inside the same running process at the same time. With this pattern long lived source code branches can be avoided. Feature toggles can allow which implementation of the abstraction is live, potentially allowing you to switch between implementations in a running system.

**Steps**:
1. Create an abstraction for the functionality to be replaced.
2. Change clients of the existing functionality to use the new abstraction.
3. Create a new implementation of the abstraction with the reworked functionality. In our case, this new implementation will call out to our new microservice.
4. Switch over the abstraction to use our new implementation.
5. Clean up the abstraction and remove the old implementation.

## Pattern: Parallel Run

Rather than calling either the old or the new implementation, instead we call both, allowing us to compare the results to ensure they are equivalent. Despite calling both implementations, only one is considered the source of truth at any given time. Typically, the old implementation is considered the source of truth until the ongoing verification reveals that we can trust our new implementation.

This pattern is typically used when the functionality being changed is considered to be high risk.

## Pattern: Decorating Collaborator

Rather than intercepting these calls before they reach the monolith, we allow the call to go ahead as normal. Then, based on the result of this call, we can call out to our external microservices through whatever collaboration mechanism we choose.

Proxy works as a decorator that calls new microservice in addition to old monolith. This pattern works best where the required information can be extracted from the inbound request, or the response back from the monolith. Where more information is required for the right calls to be made to your new service, the more complex and tangled this implementation ends up being.

## Pattern: Change Data Capture (CDC)

With change data capture, rather than trying to intercept and act on calls made into the monolith, we react to changes made in a datastore. For change data capture to work, the underlying capture system has to be coupled to the monolith’s datastore. It would be better if this pattern can be avoided.

**How to implement it:**
- Database triggers
- Transaction log pollers
- Batch delta copier

# 4. Decomposing the Database

## The Shared Database

There are some of issues related to sharing a single database among multiple services:

- It can be difficult to understand what parts of a schema can be changed safely, since we don't know what part of the schema is used by each service (can be mitigated with views).
- Lack of cohesion of business logic since it can be spread around the system.

It can be used for:
- Read-only static reference data.
- A service is directly exposing a database as a defined endpoint that is designed and managed in order to handle multiple consumers [Database-as-a-Service interface](#pattern-database-as-a-service-interface).

### Pattern: Database View

In a situation where we want a single source of data for multiple services, a view can be used to mitigate the concerns regarding coupling. With a view, a service can be presented with a schema that is a limited projection from an underlying schema. This projection can limit the data that is visible to the service, hiding information it shouldn’t have access to.

Use a database view in situations where it is impractical to decompose the existing monolithic schema but it would be better to avoid the need for a view if possible. **It is read-only!**

### Pattern: Database Wrapping Service

Hide the database behind a service that acts as a thin wrapper, moving database dependencies to become service dependencies. The use of a wrapping service allows us to control what is shared and what is hidden. It presents an interface to consumers that can be fixed, while changes are made under the hood to improve the situation.

Better than [Pattern: Database View](#pattern-database-view):
- More testable
- Not constrained to table structures (logic can be added)
- Can take writes
- Aligned to [incremental steps](#importance-of-incremental-migration)

Of course, adopting this pattern does require upstream consumers to make changes; they have to shift from direct DB access to API calls.

### Pattern: Database-as-a-Service Interface

This pattern is about create a dedicated database designed to be exposed as a read-only endpoint, and have this database populated when the data in the underlying database changes.

It requires to create a mapping engine and it can be done in some ways:
- Dedicated change data capture system (CDC)
- Batch process
- Event log

## Transferring Ownership

If we embrace the idea of a microservice encapsulating the logic associated with one or more aggregates, we also need to move the management of their state and associated data into the microservice’s own schema. On the other hand, if our new microservice needs to interact with an aggregate that is still owned by the monolith, we need to expose this capability via a well-defined interface.

### Pattern: Aggregate Exposing Monolith

Beyond just exposing data, we’re exposing operations that allow external parties to query the current state of an aggregate, and to make requests for new state transitions. We can still decide to restrict what state of an aggregate is exposed from our service boundary and to limit what state transition operations can be requested from the outside.

This pattern can be used as a pathway to more services.

If the monolith in question cannot be changed to expose these new endpoints, better to use:
- [Database view](#pattern-database-view)
- [Dedicated change data capture system](#pattern-change-data-capture-cdc)
- [Database wrapping service](#pattern-database-wrapping-service)

### Pattern: Change Data Ownership

If the newly extracted service encapsulates the business logic that changes some data, that  data should be under the new service’s control. The data should be moved from where it is, over into the new service. The monolith should be changed to make calls to the new service directly and use it as the source of truth.

## Data Synchronization

When we are in the process of switching over to a new service, but the new service and the existing equivalent code in the monolith also manages the same data data. To maintain the ability to switch between implementations, we need to ensure that both sets of code can see the same data, and that this data can be maintained in a consistent way.

Once the new service related data has been copied over into our new microservice, it can start serving traffic. However, what happens if we need to fall back to using the functionality in the existing monolithic system? Data changed in the microservices’ schema will not be reflected in the state of the monolithic database, so we could end up losing state.

### Pattern: Synchronize Data in Application

1. **Bulk Synchronize Data**: Migrate data from de old system to the new database.
2. **Synchronize on Write, Read from Old Schema**: Application writes all data into both databases but only read from the old one.
3. **Synchronize on Write, Read from New Schema**: Application reads and writes into the new database, only writes to the onld one (to keep as fallback).
4. **Remove old schema**.

### Pattern: Tracer Write

The reason this pattern is called a tracer write is that you can start with a small set of data being synchronized and increase this over time, while also increasing the number of consumers of the new source of data.

The biggest problem that needs to be addressed with the tracer write pattern is the issue that plagues any situation where data is duplicated-inconsistency. To resolve this, you have a few options:

- **Write to one source**: All writes are sent to one of the sources of truth. Data is synchronized to the other source of truth after the write occurs.
- **Send writes to both sources**: All write requests made by upstream clients are sent to both sources of truth. This occurs by making sure the client makes a call to each source of truth itself, or by relying on an intermediary to broadcast the request to each downstream service.
- **Seed writes to either source**: Clients can send write requests to either source of truth, and behind.

## Splitting Apart the Database

### Split the Database First
When concerned about the potential performance or data consistency issues.

- **Pattern: Repository per bounded context**: Helps to better understand how to split the monolith apart.
- **Pattern: Database per bounded context**: Keep schema separation where you think you may have service separation in the future. That way, you get some of the benefits of decoupling these ideas, while reducing the complexity of the system.

### Split the Code First

By splitting out the application tier, it becomes much easier to understand what data is needed by the new service. You also get the benefit of having an independently deployable code artifact earlier. Be careful to not stop here leaving a shared database!

- **Pattern: Monolith as data access layer**: Same as [Aggregate Exposing Monolith](#pattern-aggregate-exposing-monolith).
- **Pattern: Multischema storage**: New microservice uses new schema for new functionality/data,  old schema for existing data. Works well in conjunction with **Monolith as data access layer**.

### Split Database and Code Together

It is a big step to take. It is better to avoid this approach and instead splitting either the schema or application tier first.

### So, Which Should I Split First?
If you are able to change the monolith, and if you are concerned about the potential impact to performance or data consistency, **split the schema apart first**. Otherwise, **split the code out**, and use that to help understand how that impacts data ownership.

## Schema Separation Examples

### Pattern: Split Table

When the table is owned by two or more bounded contexts in your current monolith, you need to split the table along those lines. If you find specific columns in that table that seem to be updated by multiple parts of your codebase, you need to make a judgment call as to who should “own” that data.

### Pattern: Move Foreign-Key Relationship to Code

- Replace joins by service calls (increases latency)
- Data consistency
    - Check before deletion with other services (avoid it): More coupling and reverse dependency.
    - Handle deletion gracefully: Be able to handle missing record with 410 Http code instread of 404.
    - Don’t allow deletion (or use soft delete).

### Shared Static Data

- Pattern: Duplicate static reference data (table duplication).
- Pattern: Dedicated reference data schema (any change will affect some services).
- Pattern: Static reference data library (versioning and only for small datasets).
- Pattern: Static reference data service.

## Transactions

When we split data across databases, we lose the benefit of using a database transaction to apply changes in state in an atomic fashion. **Avoid using distributed transactions!**

## Sagas

- Avoids the need for locking resources for long periods of time.
- Forces us to model transactions as business process.
- Useful for long lived transactions (LLT).
- Rollback with compensating actions in case of failure:
    - Backward recovery involves reverting the failure, and cleaning up afterwards.
    - Forward recovery allows us to pick up from the point where the failure occurred, and keep processing.
    - Reorder steps to reduce rollbacks

### Orchestrated sagas (command-and-control)

Central coordinator (orchestrator) to define the order of execution and to trigger any required compensating action. These orchestrated processors tend to make **heavy use of request/response** calls between services.

| Advantages | Disadvantages |
| ---------- | ------------- |
| Good degree of visibility<br>Easy to understand the process | Coupling ([Domain coupling](#on-coupling-and-cohesion))<br>Anemic microservices (too much logic in the orchestrator) |

One of the ways to avoid too much centralization with orchestrated flows can be to
ensure you have different services playing the role of the orchestrator for different
flows.

### Choreographed sagas (trust-but-verify)

Choreographed sagas aim to distribute responsibility for the operation of the saga among multiple collaborating services. Choreographed sagas will often make **heavy use of events** for
collaboration between services.

| Advantages | Disadvantages |
| ---------- | ------------- |
| Less coupling<br> | Difficult to understand the process |

With this approach we lack a way of knowing what state a saga is in, which can also deny us the chance to attach compensating actions when required.

Use correlation Id into all the events emmited as part of the saga to build/know the state of the saga. We could then have a service whose job is to receive all the events and present a view of what state each saga is in.

# 5. Growing Pains

## Ownership at Scale

- **Strong code ownership**: All services have owners. Any change have to be submitted to the owners.
- **Weak code ownership**: All services have owners. Anyone can do changes directly.
- **Collective code ownership**: No one owns anything, and anyone can change anything they want.

Without strong code ownership a microservices architecture will grow into a distributed monolith.

## Breaking Changes

We are striving for independent deployability, but for that to happen, we need to make sure that
when we make a change to a microservice we don’t break our consumers.

- **Eliminate accidental breaking changes**: Explicit schema to avoid structural or semantic breakages.
- **Think twice before making a breaking change**: If possible, prefer expansion changes to your contract.
- **Give consumers time to migrate**:
    - Run two versions of your microservice at the same time (only for short period of time).
    - Run one version of your microserice but supporting different contract versions (preferred).

## Reporting

Create a dedicated database for reporting:
- Change data capture system (CDC)
- Views
- Copy data programmatically
- Intermediate component listening events

## Monitoring and Troubleshooting

- **Use a log aggregation system**: First thing to do when implementing microservices.
- **Tracing**: To collate a series of flows and look at them as a whole (correlation id).
- **Test in production**: Use fake users (synthetic transactions).
- **Toward observability**: Make tracing and logs easy to query and view in context.

## Local Developer Experience

As you have more and more services, the developer experience can start to suffer because you can't run the entire system on one machine. Solutions:

- Stub services
- Point local services against instances running elsewhere.

## Running Too Many Things

The more microservices you have, and the more instances you have of those microservices, the more manual processes or more traditional automated configuration management tools no longer fit the bill. Solutions:

- Kubernetes (requires containers).
- Function-as-a-Service (FaaS).

## End-to-End Testing

In a microservice architecture, the “scope” of end-to-end tests gets very large.

- Difficult to see the real issue.
- Difficult to create/maintain.
- As test suite grows, it takes longer to complete.

 As the scope of the tests increase, you’ll spend more of your time fighting the problems that arise, to the point where trying to create and maintain end-to-end tests becomes a huge time sink. Solutions:

 - **Limit scope of functional automated tests**: Avoid avoid larger-scoped tests that cross team boundaries.
 - **Use consumer-driven contracts**:  Avoid cross-service test cases.
 - **Use automated release remediation and progressive delivery**: Ie. canary release.
 - **Continually refine your quality feedback cycles**

## Global Versus Local Optimization

Multiple teams have solved the same problem in different ways, but were never aware that they were all trying to fix the same issue. A possible solution could be a **cross-cutting group to raise awareness/make tech decisions**.

## Robustness and Resiliency

As the number of services increases, and the number of service calls increase, you’ll become more and more vulnerable to resiliency issues. The more interconnected your services are, the more likely you’ll suffer from things like cascading failures and back pressure. Solutions:

- Isolating services more from each other.
- Avoid temporal coupling.
- Time-outs in conjuntion with circuit breakers.
- Running multiple copies of services.
- State management (to restart a service if it crashes).
- Document production issues.

## Orphaned Services

Services that no one is taking ownership or responsibility for them.
Create service registries by adding service metadata in the source code repository to ensure the service can be easily operated.

# Patterns Index

| Pattern | Description |
| ------- | ----------- |
Aggregate exposing monolith | Exposing domain aggregates from a monolith to allow microservices to access entities managed by the monolith. |
| Branch by abstraction | Coexisting two implementations of the same functionality in the same codebase at the same time, allowing for a new implementation to be incrementally developed until it can replace the old implementation. |
| Change data capture | Transmit changes made to an underlying datastore to other interested parties. |
| Change data ownership | Moving the source of truth from the monolith to a microservice. |
| Database as a Service interface | Using a dedicated database to provide read-only access to internal service data.| 
| Database view | A view is projected from an underlying database, allowing for parts of the database to be hidden. |
| Database wrapping service | A facade service is placed in front of an existing shared database, allowing for services to migrate away from direct use of the database. |
| Decorating collaborator | Trigger functionality running in a separate microservice by sniffing requests sent to the monolith, and the responses that are sent back in return.
| Dedicated reference data schema | A dedicated database to house all static reference data. This database can be accessed by multiple different services. |
| Duplicate static reference data | Copy static reference data into microservice databases. |
| Monolith as data access layer | Accessing data managed by the monolith via APIs rather than directly accessing the database. |
| Move foreign key to code | Move management and enforcement of foreign-key relationships from a single database up into your service tier. |
| Multischema storage | Managing data in different databases, typically while migrating from a shared database to a database-per-service model. |
| Parallel run | Run two implementations of the same functionality side by side, to ensure that the new functionality behaves appropriately. |
| Repository per bounded context | Break apart a single repository layer around different parts of the domain, making decomposition into services easier. |
| Shared database | A single database is shared between more than one service. |
| Split table | Breaking a table into two parts prior to service decomposition. | 
| Static reference data library | Move static reference data into a library or configuration file that can be packaged with each microservice that needs it. |
| Static reference data service | A dedicated microservice that provides access to static reference data. |
| Strangler fig application | Wrap your new microservice architecture around the existing monolith. Calls to use functionality that has been migrated from the monolith to your microservices are diverted; other calls are left unchanged. |
| Synchronize data in application | Synchronize data between two sources of truth from inside a single application. |
| Tracer write | Incrementally migrate data from one source of truth to another, tolerating two sources of truth during the migration. |
| UI composition | Presenting a single user interface by assembling many small parts together. |