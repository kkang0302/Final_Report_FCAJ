---
title: "Blog 2"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# Monolith or Microservices: Which Architecture is Suitable for a Personal Project?

When starting a new project, I used to think that the more services an architecture had, the more "professional" the system would be. A backend split into Identity Service, Payment Service, Notification Service, API Gateway, Message Queue, and multiple individual databases on a diagram is clearly more appealing than an application with a single backend.

Microservices also appear frequently in articles about system design, cloud-native architecture, and the systems of large tech companies. Therefore, choosing this architecture for a personal project seemed like a reasonable way to learn technologies used in practice.

But after digging deeper into building and operating a distributed system, I started asking myself:

> "If the goal is to deliver a well-functioning product, is Microservices really the right starting point?"

My short answer: **not necessarily**.

Not because Microservices is a bad architecture, but because most of its benefits only make sense once the system reaches a certain scale. Meanwhile, the associated costs and complexities appear almost from day one.

![System Architecture](/images/3-BlogPosted/Screenshot%202026-07-31%20202713.png)

## Microservices Solve Real Problems

Microservices did not appear just because developers wanted to split source code. This architecture is used to solve very specific problems of large-scale systems.

When a product has multiple development teams, each team can own a service and deploy on its own release cycle. When one function receives more traffic than the rest, that service can be scaled independently. When one component fails, the scope of the impact can be limited instead of taking down the entire system.

These are all important benefits.

But if the project only has one to a few developers, the user base is negligible, and features are still constantly changing, most of the problems that Microservices solves may not actually exist yet.

At that point, splitting the system into multiple services sometimes does not reduce complexity. It only shifts complexity from within the source code to the infrastructure and operations.

## The Real Costs Begin After Splitting Services

Suppose a system initially has domains like users, courses, payments, and notifications. When placing them all in a single application, modules can call each other directly and participate in the same database transaction.

When splitting them into microservices, those internal function calls become network requests.

From there, a series of new questions arise:

* How does this service find the address of that service?
* If a request times out, how many times should it retry?
* If one service succeeds but the next fails, how is data rolled back or reconciled?
* How do you trace a request across multiple services?
* How do you authenticate and authorize internal calls?
* If an event is sent twice, does the system handle it safely (idempotency)?
* When a service's API changes, how are dependent services updated?

These are no longer pure source code organization issues. They are classic distributed system problems: network failure, eventual consistency, idempotency, observability, and fault tolerance.

In a Monolith, errors are contained within a single process and can be traced through a single log stream. In Microservices, a single request may traverse an API Gateway, Load Balancer, multiple containers, message queues, and databases before returning a result.

Debugging changes completely.

## Infrastructure Increases, but Features Do Not Necessarily Follow

A small Monolith system might only need:

* One backend application
* One database
* One CI/CD pipeline
* One deployment environment
* One centralized logging system

After moving to Microservices, each service might need its own Docker image, pipeline, environment configurations, health check, autoscaling policy, and IAM permissions.

The system may also require an API Gateway, service discovery, container registry, message broker, distributed tracing, and centralized monitoring.

All these components have technical value. But there is one thing to note: **they do not directly create new features for users**.

Users do not care whether the backend has one service or ten. They care if the page loads quickly, if data is accurate, if payments succeed, and if the system is stable.

For a personal project, time spent managing multiple services means less time for business logic, user experience, testing, and polishing the product.

## Modular Monolith: A More Balanced Starting Point

If building a new project with the goal of creating an MVP or a portfolio, my preferred choice is a **Modular Monolith**.

The system is still deployed as a single application, but internally structured into modules with clear domain boundaries, such as:

```text
identity/
course/
enrollment/
payment/
notification/
```

Each module owns its business logic, interfaces, and scope of responsibility. Modules should not arbitrarily access each other's implementation details but rather communicate through predefined interfaces.

This organization preserves the most crucial aspect of software design: **separation of concerns and domain boundaries**, without paying the immediate cost of a distributed system.

Deployment remains simple because there is only one application. Testing is easier. Transactions between modules can be handled directly. Logs remain in a centralized stream. When a feature change spans multiple domains, developers do not need to synchronize multiple repositories and API versions.

More importantly, a Modular Monolith does not mean abandoning Microservices forever.

If module boundaries are well-designed, a module can be extracted into an independent service when a real need arises. For example, Payment might need specific compliance standards and deployment cycles, Notification might need to handle large asynchronous tasks, or Media Processing might need to scale CPU independently from the rest.

At that point, splitting a service is a response to an observed problem, rather than anticipating a problem that may never happen.

## When Does Microservices Start Making Sense?

In my view, Microservices should be considered when the system starts showing specific signs.

First, multiple teams need to develop and deploy independently. If every change has to wait for a shared pipeline, or if modifying one module frequently breaks other modules, splitting services can improve team autonomy.

Second, components have very different scaling needs. A system might have read operations receiving thousands of requests, while the admin section is used only a few times a day. If everything resides in the same application, scaling the entire system can be wasteful.

Third, certain domains have special operational or security requirements. Payment, media processing, or analytics might require different technologies, resources, and deployment policies than a standard backend.

Fourth, the Monolith has become a measurable bottleneck: build times are too long, deployment blast radius is too large, codebase ownership is hard to define, or a minor bug frequently brings down the entire system.

The common thread in these cases is that Microservices is chosen to solve an existing problem, not just because the architecture looks good on a system design diagram.

## Microservices is Still Worth Learning

Just because a Monolith is more suitable for many personal projects does not mean students or junior developers should avoid building Microservices.

If the primary goal of the project is to learn Cloud Engineering, DevOps, or Distributed Systems, Microservices provides an excellent playground.

A multi-service system forces you to tackle issues such as:

* Container orchestration
* Service discovery
* Load balancing
* Message queueing
* Distributed tracing
* Centralized logging
* Retries and circuit breakers
* CI/CD for multiple services
* Secret management and authorization
* Autoscaling and monitoring

These skills are hard to master fully if you only deploy a small application to a single server.

Therefore, an architecture that might not be optimal for launching a product can still be ideal for learning. The key is to distinguish between these two goals from the start.

## Key Takeaways

The most interesting part of comparing Monolith and Microservices is not finding which one is better, but realizing that each choice is a trade-off.

Monolith trades scalability and independent deployment for simplicity.

Microservices trades simplicity for division of ownership, fault isolation, and independent scalability.

Modular Monolith sits in the middle: keeping the system running as a single unit while striving to build clean boundaries so it can scale or be split when needed.

Choosing a Monolith for a small project does not mean poor design. On the contrary, avoiding unnecessary complexity is also a key architectural decision.

A good architecture is not the one that uses the most technologies, but the one that fits the current stage, resources, and problem at hand.

So, when starting a new project, the first question probably shouldn't be:

> "Should we use Monolith or Microservices?"

But rather:

> "What problem does the current system have, and how does this architecture help solve it?"

For a personal project with no real users, no multiple teams, and no clear scaling needs, the logical answer is usually to start simple.

Then, only split the system when you know exactly why you need to.
