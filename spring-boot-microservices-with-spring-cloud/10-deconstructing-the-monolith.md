## Deconstructing the Monolith

- Deconstruction Strategies

### Domain Driven Design

- Domain Driven Design - is a methodology to bring clarity to complexity
- aka - DDD
- Some call DDD an extension to Object Oriented Programming
- DDD concepts can be used to model complex systems
- DDD Concept is accredited to Eric Evans from his 2003 book -
  - “Domain-Driven Design: Tackling Complexity in the Heart of Software”
- See lesson resources for detailed PDF about DDD from Eric Evans

#### DDD - Definitions

- Following are definitions from Eric Evens ‘Domain-Driven Design Reference”, 2015

  - Via Creative Commons Attribution License

- `Domain` - A sphere of knowledge, influence or activity. The subject area to which the
  user applies a program is the domain of the software.

- `Model` - A system of abstractions that describes the selected aspects of a domain and can be
  used to solve problems related to that domain

- `Ubiquitous Language` - A language structured around the domain model and used by all team
  members within a bounded context to connect all the activities of the team with the software.

- `Context` - The setting in which a word or statement appears that determines its
  meaning. Statements about a model can only be understood in a context.

- `Bounded Context` - A description of a boundary (typically a subsystem, or the work
  of a particular team) within which a particular model is defined and applicable

#### Building Blocks

- `Entities` - Not a traditional Object. Represent a thread of identity that runs through
  time and often across distinct representations.

- `Value Objects` - Some objects describe or compute some characteristic of a thing.
  Immutable object, with attributes - but no identity.

- `Domain Events` - Something happened that domain experts care about. An object that
  is used to record a discrete event related to model activity.

- `Services` - Sometimes it just isn’t a thing. Some concepts are not natural to
  model as objects.

- `Aggregates` - are a cluster of entities and value objects

- `Repositories` - Query access to aggregates express in the ubiquitous language.
  Like a specialized service.

- `Factories` - Like OOP, factories create aggregates.

### DDD for Microservice Design

- Think in Bounded Contexts -
  - `Bounded Context` - A description of a boundary (typically a subsystem, or the work of
    a particular team) within which a particular model is defined and applicable.
- A bounded context will help you contain complexity
- Contexts will define common terminology
- DDD Bounded Contexts helps you with organization
- DDD Building blocks help you with defining implementation details

#### Example

- Warehouse Management System - ie software to run a large warehouse. Receives orders,
  selects inventory, ships products
- Some ‘microservices’ in terms of bounded contexts might be:
  - Inventory
  - Order Allocation
  - Manifest (shipping interface with parcel carrier details)
  - Labor Tracking
  - Returns

### Deconstructing plan

- Divide the monolith into 3 services

  - Beer Service

    - Will manage ‘Beers’ - aka products
    - Will manage brewing - aka manufacturing
    - Will list beers, and validate order lines for valid beers
    - Table Name - `BEER`

  - Beer Order Service

    - Manages Beer Orders
    - Will manage the lifecycle of an order - from order placement, to order shipment
    - Manages customers
    - Listens to order events
    - Implements call backs (aka Webhooks) to customers
    - Table Name - `BEER_INVENTORY`

  - Beer Inventory Service
    - Manages Beer Inventory
    - Takes in inventory from brewing operations
    - Provides inventory for orders
    - Implementation will be simple
    - Real use would be more complex
    - Table Name - `BEER_ORDER`, `BEER_ORDER_LINE`, `CUSTOMER`

#### Setting Default Ports for Services

- Brewery Beer Service - 8080
- Brewery Beer Order Service - 8081
- Brewery Beer Inventory Service - 8082

- Update PORT details in `application.properties` in each service
  - `server.port=<port-number>`

### Deconstruction

- Where We Are At

  - Beer Monolith has been broken into 3 independent microservices
  - Each is using its own in-memory database
  - Setup inter-service communication for read operations

- What’s Not Working

  - Order Allocation is not working
  - Beer ‘Brewing’ is not working
  - Monolith was using Spring Events and scheduled jobs for these features
    - Events and consumers of events is broken
  - The 3 services each are using Maven, BUT there is a high degree of duplication in Maven POM files

- Next Steps

  - Establish Maven BOM to reduce duplication in Maven POMs
    - Also - good technique for standardization and compliance in the enterprise
  - Setup MySQL for database - will help with troubleshooting, and configuration for target deployment
  - Transaction to JMS to publish events as messages
  - Build Saga’s to coordinate microservice actions for events
    - ie Order Allocation

- A BOM is a special kind of POM that is used to control the versions of a project's dependencies and provide a central place to define and update those versions

- A Maven pom defines the project structure including the stated dependencies. A bom defines the complete bill of materials of what dependencies are actually used - the effective dependencies

- What Is Maven POM?

  - Maven POM is an XML file that contains information and configurations (about the project) that are
    used by Maven to import dependencies and build the project.

- What Is Maven BOM?

  - BOM stands for `Bill Of Materials`. A BOM is a special kind of POM that is used to control the versions
    of a project’s dependencies and provide a central place to define and update those versions.

- Dependency Management is a mechanism to centralize the dependency information.

### KEY TERMS

- Domain Driven Design
- Domain
- Model
- Ubiquitous Language
- Context
- Bounded Context

  - A description of a boundary (typically a subsystem, or the work of
    a particular team) within which a particular model is defined and applicable.

- Bounded Context
- Contexts will define common terminology

- Domain Driven Design Building Blocks
- Maven BOM
- BOM - Bill of Materials
