## Decomposition

You are developing a large, complex application and want to use the microservice architecture. The microservice architecture structures an application as a set of loosely coupled services. The goal of the microservice architecture is to accelerate software development by enabling continuous delivery/deployment.

The microservice architecture does this in two ways:

- Simplifies testing and enables components to deployed independently
- Structures the engineering organization as a collection of small (6-10 members), autonomous teams, each of which is
  responsible for one or more services

These benefits are not automatically guaranteed. Instead, they can only be achieved by the careful functional decomposition of the application into services.

- How to decompose an application into services?

- 1. Decompose by business capability - define services corresponding to business capabilities
- 2. Decompose by subdomain - define services corresponding to DDD subdomains
- 3. Self-contained Service - design services to handle synchronous requests without waiting for other services to respond
- 4. Service per team

### Problem

- How to decompose an application into services?

### Forces

- The architecture must be stable
- Services must be cohesive. A service should implement a small set of strongly related functions.
- Services must conform to the Common Closure Principle - things that change together should be packaged
  together - to ensure that each change affect only one service
- Services must be loosely coupled - each service as an API that encapsulates its implementation. The implementation
  can be changed without affecting clients
- A service should be testable
- Each service be small enough to be developed by a “two pizza” team, i.e. a team of 6-10 people
- Each team that owns one or more services must be autonomous.
  A team must be able to develop and deploy their services with minimal collaboration with other teams.

### 01. Decompose by business capability

- Define services corresponding to business capabilities

#### Solution

Define services corresponding to business capabilities. A business capability is a concept from business architecture modeling. It is something that a business does in order to generate value. A business capability often corresponds to a business object, e.g.

- Order Management is responsible for orders
- Customer Management is responsible for customers

Business capabilities are often organized into a multi-level hierarchy. For example, an enterprise application might have top-level categories such as Product/Service development, Product/Service delivery, Demand generation, etc.

### Decompose by subdomain

- Define services corresponding to DDD subdomains

#### Solution:

Define services corresponding to Domain-Driven Design (DDD) subdomains. DDD refers to the application’s problem space - the business - as the domain. A domain is consists of multiple subdomains. Each subdomain corresponds to a different part of the business.

Subdomains can be classified as follows:

- Core - key differentiator for the business and the most valuable part of the application
- Supporting - related to what the business does but not a differentiator. These can be implemented in-house or outsourced.
- Generic - not specific to the business and are ideally implemented using off the shelf software

### Self-contained Service

- Design services to handle synchronous requests without waiting for other services to respond
- Implement a Service Module rather than a separate service.

#### Problem:

- How should a service collaborate with other services when handling a synchronous request?

#### Solution:

The key drawback of using synchronous request/response is that it reduces availability. That’s because if any of the Order Sevice’s collaborators are unavailable, it will not be able to create the order and must return an error to the client.

An alternative approach is to eliminate all synchronous communication between the Order Service and its collaborators by using the CQRS and Saga patterns.

One way to make a service self-contained is to implement needed functionality as a service module rather than a separate service. We could, for example, merge the Order Service and Restaurant Service.

Another way to make a service self-contained is for it to collaborate with other services using the CQRS and the Saga patterns. A self-contained service uses the Saga pattern to asynchronously maintain data consistency. It uses the CQRS pattern to maintain a replica of data owned by other services.

### Service per team

- A high performance development organization consist of multiple teams

#### Solution:

Each service is owned by a team, which has sole responsibility for making changes. Ideally each team has only one service:

- A team should be small, e.g. 5-9 people
- A team should be autonomous and loosely coupled
- The size and complexity of the team’s code base must not exceed the team’s cognitive capacity.
- Finer-grained service decomposition improves -ilities including maintainability, testability, deployability
- Finer-grained service decomposition adds complexity

### KEY TERMS

- Accelerate Software development by enabling Continuous Delivery and Deployment
- Functional Decomposition for one or more services
- Careful Functional Decomposition of application into services

- OOO (Object Oriented) Design
- Common Closure Principle (CCP)
- Single Responsibility Principle

- Services must be cohesive
- Service must conform to the Common Closure Principle - Things that changes together should be packaged together
- Services must be loosely coupled
- Business Capability
- Business Architect Modeling

- Another useful principle from OOD is the Common Closure Principle (CCP), which states that classes that change for the
  same reason should be in the same package

- Drawback of using synchronous request/response is that it reduces availability
- Subscribing the domain events published by the service that own the data

- High performance Development organization
- High performance Team
- Coarse grained services
- A team should be autonomous and loosely coupled
