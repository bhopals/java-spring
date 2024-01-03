## Welcome to the Cloud, Spring

- Small, Simple, and Decoupled Services = Scalable, Resilient, and Flexible Applications

- Microservices are distriburted, loosely coupled software services that carry out a small
  number of well-defined tasks.

- Build Mircoservices using:

  - Java and
  - two Spring framework projects: Spring Boot and Spring Cloud.

- Traditional Monolith Spring Application to Microservice applications

- Monolith Architecture Style

  - Single Deployable Software artifact
  - Packed together into a single application artifact
  - The size and complexity of Monolith App

- Microservice Architecture Style

  - Microservices allow you to take a large application and decompose it into
    easy-to-manage components with narrowly defined responsibilities

- Invoke all business logic as REST-based service calls

### A microservice architecture has the following characteristics:

- Application logic is broken down into small-grained components with welldefined boundaries
  of responsibility that coordinate to deliver a solution.

- Each component has a small domain of responsibility and is deployed completely independently
  of one another.

- Microservices communicate based on a few basic principles (notice I said principles,
  not standards) and employ lightweight communication protocols such as
  HTTP and JSON (JavaScript Object Notation) for exchanging data between the
  service consumer and service provider.

- The underlying technical implementation of the service is irrelevant because
  the applications always communicate with a technology-neutral protocol (JSON
  is the most common).

- Microservices—by their small, independent, and distributed nature—allow
  organizations to have small development teams with well-defined areas of
  responsibility.

### What is Spring and why is it relevant to microservices?

- Spring has become the de facto development framework for building Java-based applications.
- A light weight alternative for enterprise Applications

- Microservices are a common Architectural patterns for building cloud-based applications.

#### Spring Boot and Spring Cloud

Spring Boot is a re-envisioning of the Spring framework. While it embraces core
features of Spring, Spring Boot strips away many of the “enterprise” features found in
Spring and instead delivers a framework geared toward Java-based, REST-oriented
(Representational State Transfer)1
microservices. With a few simple annotations, a
Java developer can quickly build a REST microservice that can be packaged and
deployed without the need for an external application container.

The Spring Cloud framework makes it simple to operationalize and
deploy microservices to a private or public cloud. Spring Cloud wraps several popular
cloud-management microservice frameworks under a common framework and makes
the use and deployment of these technologies as easy to use as annotating your code

### Building a microservice with Spring Boot

- Flow of Spring Boot Microservice

  - A Client Makes a HTTP Request
  - Route mapping
    - Spring Boot will parse the HTTP request and map the route based on the HTTP Verb, the URL,
      and potential parameters defined for the URL. A route maps to a method in a Spring RestController class
  - Parameter destructuring
    - Once Spring Boot has identified the route it will map any parameters defined inside
      the route to a Java method that will carry out the work.
  - JSON->Java object mapping
    - For an HTTP PUT or Post, a JSON passed in the HTTP body is mapped to a Java class.
  - Business logic execution
    - Once all of the data has been mapped, Spring Boot will execute the business logic.
  - Java->JSON object mapping
    - Once the business logic is executed, Spring Boot will convert a Java object to JSON.

- Spring Boot abstracts away the common REST microservice task (routing to business logic, parsing
  HTTP parameters from the URL, mapping JSON to/from Java Objects), and lets the developer focus on the business logic for the service.

- Command to run services - `mvn spring-boot:run`

- 3 cloud-based computing model:

  - Infrastructure as a Service (IaaS)
  - Platform as a Service (PaaS)
  - Software as a Service (SaaS)

  - Functions as a Service (FaaS) - Lambda
  - Container as a Service (CaaS) - Docker

### Six categories of microservice patterns

- 1. Core development patterns
- 2. Routing patterns
- 3. Client resiliency patterns
- 4. Security patterns
- 5. Logging and tracing patterns
- 6. Build and deployment patterns

#### 01. Core development patterns

- The core development microservice development pattern addresses the basics of
  building a microservice.

- When designing your microservice, you have to think about how the service will be consumed
  and communicated with

  - Service granularity
  - Communication protocols
  - Interface design
  - Configuration management of service
  - Event processing between services

#### 02. Microservice routing patterns

- Service Routing - acts as a policy enforcement point
- Service Discovery - abstracts away the physical location of the service from the client

- Service discovery and routing are key parts of any large-scale microservice application.

- The microservice routing patterns deal with how a client application that wants to
  consume a microservice discovers the location of the service and is routed over to it.

- Service routing gives the microservice client a single logical URL to talk to and acts
  as a policy enforcement point for things like authorization, authentication, and
  content checking.

- Service discovery abstracts away the physical location of the service from the client.
  New microservice instances can be added to scale up, and unhealthy service instances can
  be transparently removed from the service.

- Service Discovery

  - discoverable
  - location of service

- Service Routing
  - single entry point
  - apply security policies
  - policy enforcement point

#### Microservice client resiliency patterns

Because microservice architectures are highly distributed, you have to be extremely
sensitive in how you prevent a problem in a single service (or service instance) from
cascading up and out to the consumers of the service. To this end, we’ll cover four client resiliency patterns:

- Client-side load balancing
  - The service client caches microservice endpoints retrieved from the service discovery and
    ensures that the service calls are load balanced between instances.
- Circuit breakers pattern
  - The circuit breaker pattern ensures that a service client does not repeatedly call a
    failing service. Instead, a circuit breaker "fails fast" to protect the client.
- Fallback pattern
  - When a client does fail, is there an alternative path the client can take to retrieve data
    from or take action with?
- Bulkhead pattern
  - How do you segregate different service calls on a client to make sure one misbehaving
    service does not take up all the resources on the client?

### Microservice security patterns

- Authentication
- Authorization
- Credential management and propagation

### Microservice logging and tracing patterns

- Log correlation
  - Log correlation: All service log entries have a correlation ID that ties the log entry to a
    single transaction.
- Log aggregation
  - Log aggegration: An aggregation mechanism collects all of the logs from all the services instances.
- Microservice transaction tracing
  - Microservice transaction tracing: The development and operations teams can query the log
    data to find individual transactions. They should also be able to visualize the flow of
    all the services involved in a transaction.

### Microservice build/deployment patterns

- Build and deployment pipeline
- Infrastructure as code
- Immutable servers
- Phoenix servers
  - The long running servers

### Using Spring Cloud in building your microservices

- 1. Core development patterns

  - Core microservice patterns - Spring Boot
  - Configuration management - Spring Cloud Config
  - Asynchronous messaging - Spring Cloud Stream

- 2. Routing patterns

  - Service discovery patterns - Spring Cloud / Netflix Eureka
  - Service routing patterns - Spring Cloud / Netflix Zuul

- 3. Client resiliency patterns

  - Client-side load balancing - Spring Cloud / Netflix Ribbon
  - Circuit breaker pattern - Spring Cloud / Netflix Hystrix
  - Fallback pattern - Spring Cloud / Netflix Hystrix
  - Bulkhead pattern - Spring Cloud / Netflix Hystrix

- 4. Security patterns

  - Authorization - Spring Cloud Security / OAuth2
  - Authentication - Spring Cloud Security / OAuth2
  - Credential management and propagation - Spring Cloud Security / OAuth2 / JWT

- 5. Logging and tracing patterns

  - Log correlation - Spring Cloud Sleuth
  - Log aggregation - Spring Cloud Sleuth (with Papertrail)
  - Microservice tracing - Spring Cloud Sleuth / Zipkin

- 6. Build and deployment patterns

  - Continuous integration - Travis CI
  - Infrastructure as code - Docker
  - Immutable servers - Docker
  - Phoenix servers - Travis CI / Docker

### Spring Boot

Spring Boot is the core technology used in our microservice implementation. Spring
Boot greatly simplifies microservice development by simplifying the core tasks of
building REST-based microservices. Spring Boot also greatly simplifies mapping HTTPstyle verbs (GET, PUT, POST, and DELETE) to URLs and the serialization of the JSON
protocol to and from Java objects, as well as the mapping of Java exceptions back to
standard HTTP error codes

### Spring Cloud Config

Spring Cloud Config handles the management of application configuration data
through a centralized service so your application configuration data (particularly
your environment specific configuration data) is cleanly separated from your
deployed microservice

It integrates with:

- Git - Open source version control system that allows you to manage and track changes to
  any type of text file
- Consul - An open source service discovery
  tool that allows service instances to register themselves with the service.
- Eureka - An open source Netflix project that, like Consul, offers similar service discovery capabilities.

### Spring Cloud service discovery

With Spring Cloud service discovery, you can abstract away the physical location (IP
and/or server name) of where your servers are deployed from the clients consuming
the service.

- Consul
- Eureka

### Spring Cloud/Netflix Hystrix and Ribbon

Spring Cloud heavily integrates with Netflix open source projects. For microservice client resiliency patterns, Spring Cloud wraps the Netflix Hystrix libraries (https://github
.com/Netflix/Hystrix) and Ribbon project (https://github.com/Netflix/Ribbon) and
makes using them from within your own microservices trivial to implement.

- Netflix Hystrix - Implement service client resiliency patterns such as the circuit
  breaker and bulkhead patterns.

- Netflix Ribbon - Simplifies integrating with service discovery
  agents such as Eureka, it also provides client-side load-balancing of service calls from a
  service consumer

### Spring Cloud/Netflix Zuul

Spring Cloud uses the Netflix Zuul project (https://github.com/Netflix/zuul) to provide service routing capabilities for your microservice application.

- Netflix Zuul - Zuul is a service
  gateway that proxies service requests and makes sure that all calls to your microservices go through a single “front door” before the targeted service is invoked

### Spring Cloud Stream

To easily integrate lightweight message processing into your microservice.
With Spring Cloud Stream, you can quickly integrate your microservices with message brokers such as
RabbitMQ (https://www.rabbitmq.com/) and Kafka (http://kafka.apache.org/).

### Spring Cloud Sleuth

Spring Cloud Sleuth (https://cloud.spring.io/spring-cloud-sleuth/) allows you to
integrate unique tracking identifiers into the HTTP calls and message channels (RabbitMQ, Apache Kafka) being used within your application.

The real beauty of Spring Cloud Sleuth is seen when it’s combined with logging
aggregation technology tools such as Papertrail (http://papertrailapp.com) and tracing tools such as Zipkin (http://zipkin.io).

### Spring Cloud Security

Spring Cloud Security is token-based and allows
services to communicate with one another through a token issued by an authentication server. Each service receiving a call can check the provided token in the HTTP
call to validate the user’s identity and their access rights with the service.

### What about provisioning?

- Travis CI
- Docker

### Example

```

@SpringBootApplication
@RestController
@RequestMapping(value="hello")
@EnableCircuitBreaker
@EnableEurekaClient
public class Application {
 public static void main(String[] args) {
     SpringApplication.run(Application.class, args);
 }

 @HystrixCommand(threadPoolKey = "helloThreadPool")
 public String helloRemoteServiceCall(String firstName, String lastName){
    ResponseEntity<String> restExchange = restTemplate.exchange( "http://logical-service-id/name/
    [ca]{firstName}/{lastName}", HttpMethod.GET, null, String.class, firstName, lastName);
    return restExchange.getBody();
 }
 @RequestMapping(value="/{firstName}/{lastName}", method = RequestMethod.GET)
 public String hello( @PathVariable("firstName") String firstName, @PathVariable("lastName") String lastName) {
    return helloRemoteServiceCall(firstName, lastName)
 }
}
```

- `@EnableCircuitBreaker` - annotation tells your Spring microservice that you’re going to use
  the Netflix Hystrix libraries in your application

- `@EnableEurekaClient` - The @EnableEurekaClient annotation tells your microservice to
  register itself with a Eureka Service Discovery agent.

- `@HystrixCommand(threadPoolKey = "helloThreadPool")` - The @HystrixCommand annotation is doing
  two things. First, any time the helloRemoteServiceCall method is called, it won’t be directly invoked. Instead, the method will be delegated to a thread pool managed by Hystrix. If the call takes too long (default is one second), Hystrix steps in and interrupts the call. This is the implementation of the circuit breaker pattern. The second thing this annotation does is create a thread pool called helloThreadPool that’s managed by Hystrix. All calls to helloRemoteServiceCall method will only occur on this thread pool and will be isolated from any other remote service calls being made.

### SUMMARY

- Microservices are extremely small pieces of functionality that are responsible
  for one specific area of scope.

- No industry standards exist for microservices. Unlike other early web service
  protocols, microservices take a principle-based approach and align with the
  concepts of REST and JSON.

- Writing microservices is easy, but fully operationalizing them for production
  requires additional forethought. We introduced several categories of microservice development patterns, including core development, routing patterns, client resiliency, security, logging, and build/deployment patterns.

- While microservices are language-agnostic, we introduced two Spring frameworks that
  significantly help in building microservices: Spring Boot and Spring Cloud.

- Spring Boot is used to simplify the building of REST-based/JSON microservices.
  Its goal is to make it possible for you to build microservices quickly with nothing
  more than a few annotations.

- Spring Cloud is a collection of open source technologies from companies such
  as Netflix and HashiCorp that have been “wrapped” with Spring annotations to
  significantly simplify the setup and configuration of these services.

### KEY TERMS

- Distributed and loosely coupled
- Traditional Monolith Spring Application
- Microservice applications
- Monolith Architecture Style
- Single Deployable Software artifact
- Packed together into a single application artifact
- The size and complexity of Monolith App

- Easy-to-manage components
- Narrowly defined responsibilities

- Rest-Based Services calls
- Small-grained components with well-defined boundaries of responsibilities
- Small domain of responsibilities
- Empploy light-weight communication protocol
- Communicate with Technology-neutral protocol (JSON)

- Highly distribute services
- Microservices based applications using Spring Boot and Spring Cloud

- Small, Simple, and Decoupled Services = Scalable, Resilient, and Flexible Applications

- Deployed as its own discrete and independent artifact

- Simplified Infrastructure management
- Massive horizontal scalability
- High redundancy through geographic distribution
- coarse-grained service
- fine-grained service

- policy enforcement point
- policy enforcement point for things like Authorization, Authentication, Content Checking

- Service discovery
- Service Routing

- Highly distributed
- Poorly Behaving Service

- SIX Microservices Pattern
  - Core development patterns
  - Routing patterns
  - Client resiliency patterns
  - Security patterns
  - Logging and tracing patterns
  - Build and deployment patterns
