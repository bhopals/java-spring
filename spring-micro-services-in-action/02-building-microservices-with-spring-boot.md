## Building Microservices with Spring Boot

The microservices architecture is a powerful design paradigm for breaking down
complex monolithic software systems into smaller, more manageable pieces.

- Monolith Applications

  - Tightly coupled
  - Leaky
  - Monolithic

- Microservices

  - Constrained
  - Loosely coupled
  - Abstracted
  - Independent

- Cloud-based applications in general have the following:
  - A large and diverse user base
  - Extremely high uptime requirements
  - Uneven volume requirements

### The architect’s story: designing the microservice architecture

- Decomposing the business problem
- Establishing service granularity
- Defining the service interfaces
  - Embrace the REST philosophy
  - Use URI’s to communicate intent
  - Use JSON for your requests and responses
  - Use HTTP status codes to communicate results

### The DevOps Story: building for the rigors of runtime

From a DevOps perspective, you must address the operational needs of a microservice up front and translate these four principles into a standard set of lifecycle events:

- Service assembly
- Service bootstrapping
- Service registration/discovery
- Service monitoring

#### Building the Twelve-Factor microservice service application

- I. Codebase - One codebase tracked in revision control, many deploys
- II. Dependencies - Explicitly declare and isolate dependencies
- III. Config - Store config in the environment
- IV. Backing services - Treat backing services as attached resources
- V. Build, release, run - Strictly separate build and run stages
- VI. Processes - Execute the app as one or more stateless processes
- VII. Port binding - Export services via port binding
- VIII. Concurrency - Scale out via the process model
- IX. Disposability - Maximize robustness with fast startup and graceful shutdown
- X. Dev/prod parity - Keep development, staging, and production as similar as possible
- XI. Logs - Treat logs as event streams
- XII. Admin processes - Run admin/management tasks as one-off processes

- More - https://12factor.net/

#### Summary

- To be successful with microservices, you need to integrate in the architect’s,
  software developer’s, and DevOps’ perspectives.

- Microservices, while a powerful architectural paradigm, have their benefits and
  tradeoffs. Not all applications should be microservice applications.

- From an architect’s perspective, microservices are small, self-contained, and distributed.
  Microservices should have narrow boundaries and manage a small set of data.

- From a developer’s perspective, microservices are typically built using a RESTstyle of design,
  with JSON as the payload for sending and receiving data from the service.

- Spring Boot is the ideal framework for building microservices because it lets
  you build a REST-based JSON service with a few simple annotations.

- From a DevOp’s perspective, how a microservice is packaged, deployed, and
  monitored are of critical importance.

- Out of the box, Spring Boot allows you to deliver a service as a single executable
  JAR file. An embedded Tomcat server in the producer JAR file hosts the service.

- Spring Actuator, which is included with the Spring Boot framework, exposes
  information about the operational health of the service along with information
  about the services runtime.

### KEY TERMS

- self-contained
- independently deployable
