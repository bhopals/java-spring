## Introduction to Microservices

### The Traditional Monolith Application

- mon·o·lith - /ˈmänəˌliTH/

#### Monolith Defined

- mon·o·lith - /ˈmänəˌliTH/

  - Origin Greek

    - monos - single
    - lithos - stone

  - Noun - a large single upright block of stone, especially one shaped into or serving
    as a pillar or monument.

#### Single Application

- One Code Base
- One Build System
- Single executional program (ie WAR or EAR file)
- In Enterprise system - an application can become very big
  - 10’s of thousands of packages, classes.

#### Traits of a Monolithic Architecture

- Code is stored together
- Typically will use one database
- Code Releases are done as one big version
- Scaling is an all or nothing situation
  - If one component needs to increase scale, the whole application needs to scale

#### Benefits of Monoliths

- Development is easy - everything is in one project
- Deployment is easy - One app to deploy
- Testing is simplified - One app to test

#### Problems with Monoliths

- As the business requirements of Monoliths grow, so does their complexity
- Can lead to anti-patterns - such as Spaghetti Code and Big Ball of Mud design patterns
- Difficult to modify - Even the smallest change will require a full deployment of
  the application
- Technology Lock In - The monolith becomes tightly coupled to the technology stack
- Difficult to introduce new technologies
- CI/CD difficult

#### Are Monoliths Bad?

- Answer - It Depends

### What are Microservices?

- Microservices are small targeted services
- Each service has its own repository
- Microservices are isolated from other services
  - Should not be bundled with other services when deployed
- Microservices are loosely coupled
  - When interacting with other services, should be done in a technology agnostic manner
  - ie - Restful web services - HTTP / JSON

#### Microservice Architecture

- With a Microservice Architecture:
  - Applications are composed using individual microservices
  - Each service will typically have its own database
  - Each microservice is independently deployable
  - Scaling of individual services is now possible
  - CI/CD becomes easier since services are smaller and less complex to deploy

#### Benefits (Pros) of Microservices

- Easy to understand & develop - Services are smaller and more targeted
- Software Quality - Since services are more targeted and have a limited scope
- Scalability - Independent services can be scaled up and down to the application’s demands.
- Reliability - Software bugs are isolated
- Technology flexibility - Services can be developed using any language or technology stack.

#### Cons of Microservices

- Integration testing can be difficult
- Deployments are more complex. Rather than one application to deploy, you now have many.
- Operational cost with each service - Each service is a small application
  - Needs own repo, own deployment process, own database, etc
- Additional hardware resources - Additional services need additional hardware to run on

#### How ‘Big’ Should a Microservice Be?

- A microservice can be as small as a single API endpoint
  - ie - ‘Get Orders’
- A microservice can be several or even dozens of API endpoints
- Answer is a topic of much debate
- Guideline - Amazon’s Two Pizza Team - A microservice should be able to be supported be
  a team you can fed with two pizzas. (~12 people)
- Scalability - This can also be a consideration in the size of a microservice
  - The higher the scalability, the more specialized the service should be

#### What is the Cloud?

- The ‘Cloud’?
- The Cloud is not a physical object, but more of a concept.
- The ‘Cloud’ allows you to use virtual servers and services.
- The ‘Cloud’ abstracts the physical underlying hardware and services
- Amazon Web Services for example:
- Allows provisioning in zones - a physical region made of many data centers,appearing as one

#### Cloud != Virtual Machines

- Virtual Machines are easy to provision in cloud environments
- SFG Website runs on a AWS Virtual Machine
- This is not ‘Cloud’ Computing
- Virtualization is not Cloud Based Architecture
- Subject of heated debates!

#### Cloud Based Architectures

- Microservices are a key aspect of Cloud Based Architectures
- Cloud based architectures focus on abstraction, redundancy, and avoidance of single
  point of failures
- For example saving a file to AWS S3
  - File is copied to multiple servers, and in multiple data centers before the
    ‘save’ is confirmed.
  - Thus protected from server failure, and even loss of a data center.

#### Microservices in Cloud Based Architectures

- Typically multiple instances of microservices are deployed in a cloud environment
- Important to reliability of the application as a whole
- If a service instance is terminated, another running instance can assume the workload
- Netflix as a tool called Chaos Monkey who’s job is to randomly terminate components to
  ensure there are no single points of failure

#### Common Microservice Deployment Tools

- This is very large and diverse area!

  - AWS Beanstalk
  - AWS ECS / EKS
  - Kubernetes
  - Docker Swarm
  - Red Hat OpenShift
  - Cloud Foundry

### Adopting Microservices

- `Often applications will start as monoliths
  - Might be because of being older legacy applications
  - Or a development choice
    - Remember there is a ‘cost’ to splitting into microservices
  - Its not uncommon to start development of an application as a monolith
- Monolithic architectures are well established in companies
- Many companies are just starting to adopt Microservices

#### Decomposing to Services

- Decomposing is the process of taking a larger monolithic application and breaking it up
  into microservices
- Decomposition is more of an `art` than a science
- Strategies you can use:
  - By Business Capability - ie Order Service
  - By Domain Objects - ie Product Service (services over domain object ‘Product’)
  - By action verbs - Payment Service
  - By Nouns - Customer Service

#### Single Responsibility Principle

- Single Responsibility Principle (SRP) is a term coined by Uncle Bob Martin about
  object oriented programming.
  - SRP says a class should have just one reason to change.
  - Meaning your classes should be very specific in what they do
  - Do one thing, and do it very well
- SRP can also be applied to microservices
  - Do one thing, and do it very well

#### Microservices and Development Teams

- Larger organizations might have hundreds of developers
- When possible small teams should be responsible for specific microservices
- This will often lend itself to business functions
  - An account team would work on accounting related services
  - An Customer Order team would work on Customer Order related services
  - An Order Fulfillment team would work on Order Fulfillment related services
- Often you will see a lot of overlap of business domain with the domain of the services

### Microservice Architecture and Design

The traffic coming in through the Gateway / Load Balancer that is gonna be distributed
to kind of like a front line set of microservices.

#### Gateway

- Endpoint that is exposed to other services
  - Can be internet for public APIs
  - More likely to be internal
- Abstracts implementation of services
- Client calls URL, is unaware of routing taking place to running instance
- Acts as roughly a proxy for network traffic
- Can also act as a load balancer

#### Service Instances

- Expect to be running N number of services
- Exact number depends on reliability and load requirements
- Minimum might be 3, for high availability
- Some tools allow you to dynamically scale based on load or anticipated load
  - Think Netflix at night
  - In evenings, Netflix traffic is 1/3 of US internet traffic
  - Netflix will scale up and down with load

#### Database Tier

- Typically one database per microservice
  - Guideline - not a hard ‘rule’
- Highly scalable services will often have one transactional database
  - And one or more read database (replicas)
- Organizations will often have more than one database technology
- Not uncommon to see mix of SQL and NoSQL database technologies

#### Messaging

- A common pattern is to expose an API endpoint via a RESTFul API
  - Dependent microservices are often message based
  - Messages follow an event or command pattern
- Messaging allows for decoupling and scalability
- Messaging can be used to define a work flow
  - New Order, Validate Order, Charge Credit Card, Allocate Inventory, Ship Order

#### Downstream Services

- Often an action on a microservice will invoke actions on multiple down stream services
- For example, it is rumoured a search on Amazon will invoke over 100 services to return
  the search results - search, sponsors, your history, logging your search, etc
- Placing a new order might invoke the following:
  - Validate Order
  - Pay Credit Card
  - Allocate Inventory
  - Ship Order

### 12 Factor Applications

In the modern era, software is commonly delivered as a service: called web apps, or software-as-a-service. The twelve-factor app is a methodology for building software-as-a-service apps that:

- I. Codebase - One codebase tracked in revision control, many deploys
- II. Dependencies - Explicitly declare and isolate dependencies
- III. Config - Store config in the environment
- IV. Backing services - Treat backing services (DB,Queue) as attached resources
- V. Build, release, run - Strictly separate build and run stages
- VI. Processes - Execute the app as one or more stateless processes
- VII. Port binding - Export services via port binding
- VIII. Concurrency - Scale out via the process model
- IX. Disposability - Maximize robustness with fast startup and graceful shutdown
- X. Dev/prod parity - Keep development, staging, and production as similar as possible
- XI. Logs - Treat logs as event streams
- XII. Admin processes - Run admin/management tasks as one-off processes

- More - https://12factor.net/

### KEY TERMS

- Abstraction
- Redundancy
- Single Point of Failures
- Scalability
- Provisoning
- Assuming workload
- Technology agnostic
- Legacy Applications
- Monolith
- Microservices
- Anti-pattern
- Business Capability
- Domain Object
- Decomposing
- Decoupling
- loosely coupled
- loosely coupled in Technology agnostic manner
- Handle Anticipated load
- Dynamically scale
- Dynamically scale based on load
- Dynamically scale Horizentely / Vertically
- Scale up
- Scale Down
- Scale up and Scale Down with Load
- Define a workflow
- Dependent services
- Messages follow an event or command pattern
- Invoke Multiple Down Stream Services
- Gateway abstract implementations of Services (To and From traffic)
- Stateless
- Maximise Robustness
- Maximise Robustness with fast startup and graceful shutdown

- Spaghetti Code
- Big Ball of Mud
- Can lead to anti-patterns - such as Spaghetti Code and Big Ball of Mud design patterns
- Becomes tightly coupled to the Technology stack
- There is no Silver Bullet for every situation

- Increased Software Quality
- Scalability
- Reliability
- Technology agnostic
- Technology Flexibility

- Increased Operational Cost
- Additional Hardware Resources

- More Specialized service
- Higher the scaliblity, the more specialized the service should be

- Abstract the physical underlying Hardware and Services
- Allowing provisioning in zones - A Physical Region made of many data centers

- Avoidance of Single Point of Failures
- Cloud based architectures focus on Abstraction, Redundancy, and Avoidance of single
  point of failures.

- Assume workload
- If a Service instance is terminated, another Running instance can assume the workload
- Netflix's `Chaos Monkey`

- Overlap of Business domain with the Domain of the services

- Highly Scalable Services
- One Transactional Database
- One or more Read databases (Replicas)

- Oftern action on a Microservice will invoke Actions on Multiple Down Stream Services

- Database Tier
- Gateway/Load Balancer
- Service Instances
- Message Queues

- Deploy Build artifact
- Strict separation between the Build, Release and Run stages
- The twelve-factor app is a methodology for building software-as-a-service apps
- Create a view across multiple services

- Explicitly declare and isolate dependencies
- strict separation of config from code.
- Build stage, Release stage, Run stage
- Stateless processes
- Maximize robustness with fast startup and graceful shutdown
- Processes should strive to minimize startup time
