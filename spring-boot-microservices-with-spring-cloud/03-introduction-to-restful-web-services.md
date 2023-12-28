## Introduction to Restful Web Services

### Idempotemt Methods

- Idempotence - A Quality of an ACTION such that repetitions of the action have
  no further effect on the outcome.

- PUT and DELETE are Idempotent Methods
- Safe Methods (GET, HEAD, TRACE, OPTIONS) are also Idempotent Methods

- POST and PATCH are generally considered non-idempotent,
  as their outcomes may vary with each request

#### RESTful Web Services

- Because of their simplicity and versatility, RESTful web services have become the de facto
  standard for web services.
- REST - Representational State Transfer
  - Representation - Typically JSON or XML
  - State Transfer - Typically via HTTP
  - Established by Roy Fielding from his 2000 doctoral dissertation

#### RESTful Terminology

- Verbs - HTTP Methods: GET, PUT, POST, DELETE
- Messages - the payload of the action (JSON/XML)
- URI - Uniform Resource Identifier
  - A unique string identifying a resource
- URL - Uniform Resource Locator

  - A URI with network information - http://www.example.com

- HTTP POST
  - Non-Idempotent - Multiple POSTs is expected to create multiple objects
  - Not Safe Operations - does change state of resource
  - Only Non-Idempotent, Non-safe HTTP verb

### Richardson Maturity Model

- Richardson Maturity Model: Steps Towards the Glory of REST

#### Richardson Maturity Model (RMM)

- Established by Leonard Richardson in a 2008 Q-Con Presentation
- A model used to describe the maturity of RESTful services
- Unlike SOAP, there is no formal specification for REST
- RMM is used to describe the quality of the RESTful service

#### RMM Levels

Level 3: Hypermedia Controls
Level 2: HTTP Verbs
Level 1: Resources
Level 0: The Swap of POX

#### Core Technologies

- HYPERMEDIA
- HTTP
- URI

#### Level 0: Swamp of POX

- POX - Plain Old XML
- Uses implementing protocol as a transport protocol
- Typically uses one URI and one kind of method
- Examples - RPC, SOAP, XML-RPC

#### Level 1: Resources

- Uses Multiple URIs to identify specific resources
- Examples:
  - http://www.example.com/product/1234
  - http://www.example.com/product/5687
- Still uses a single method (ie GET)

#### Level 2: HTTP Verbs

- HTTP Verbs are used with URIs for desired actions
- Examples:
  - GET /products/1234 - to return data for product 1234
  - PUT /products/1234 (with XML body) to update data for product 1234
  - DELETE /products/1234 to delete product 1234
- Most common in practical use

#### Level 3: Hypermedia

- Representation now contains URIs which may be useful to consumers
- Helps client developers explore the resource
- No clear standard at this time
- Spring provides an implementation of HATEOS

#### Summary

- Level 3 - provides discoverability, making the API more self documenting
- Level 2 - Introduces Verbs to implement actions
- Level 1 - breaks large service into distinct URIs

### Spring Framework and RESTful Services

- The Spring Framework has very robust support for creating and consuming RESTFul Web Services
- Spring Framework has 3 Distinct web frameworks for creating RESTful services
  - Spring MVC - Web Framework
  - Spring WebFlux - Web Framework
  - WebFlux.fn - Web Framework
- Spring Framework has 2 Distinct web client for consuming RESTful services
  - Spring RestTemplate - Web Client
  - Spring WebClient - Web Client
- There are also several popular libraries for creating and consuming RESTful
  services frequently used with Spring
  - Not covered in this course

#### Spring MVC - Web Framework

- Spring MVC is the oldest and most commonly used library for creating RESTful web services
- Part of the core Spring Framework
  - Compatible with Java EE (Jakarta EE in Spring 6)
- MVC - Model View Controller
- Has robust support for traditional Web Applications
- Based on traditional Java Servlet API
  - By nature this is blocking, non-reactive

#### Spring WebFlux - Web Framework

- Spring WebFlux was introduced with version 5 of the Spring Framework
- WebFlux uses project Reactor to provide reactive web services
  - Does not use Java Servlet API, thus is non-blocking
- Follows very closely to the configuration model of Spring MVC
  - Provides an easy transition for developers accustomed to traditional Spring MVC

#### WebFlux.fn - Web Framework

- Also introduced in Spring Framework 5
- WebFlux.fn is a functional programming model used to define endpoints
- Alternative to annotation based configuration
- Designed to rapidly and simply define microservice endpoints

#### Spring RestTemplate - Web Client

- RestTemplate is Spring’s primary library for consuming RESTFul web services
- Very mature - been a part of Spring for a very long time
- Highly configurable
- As of Spring Framework 5 RestTemplate is in maintenance mode
  - No new features are planned
  - Step before deprecation, Spring recommends using WebClient for new development

#### Spring WebClient - Web Client

- Spring WebClient was introduced in Spring Framework version 5
- This is Spring’s reactive web client.
- By default uses Reactor Netty, a non-blocking HTTP Client library

#### Marshalling / Unmarshalling

- Converting Java POJOs to JSON or XML is called Marshalling
- Converting JSON or XML to Java Objects is called Unmarshalling
- By default Spring Boot configures Jackson to facilitate Marshalling and Unmarshalling
- Spring Boot does support several other libraries, however Jackson is by far the most popular
- Jackson will be the focus of the course

#### SPA - Single Page Applications

- RESTFul APIs are often combined with SPA applications for rich user web applications
- Popular client side SPA frameworks include Vue, BackBoneJS, ReactJS, AngularJS, and EmberJS
- Frequent question is which framework is the “best” to use with Spring Boot
- The correct answer is that it does not matter
- The framework used is decoupled from Spring via the HTTP / JSON (or XML) layer
- All of these libraries can consume RESTful APIs
- Server side can be Spring Boot, .NET, Ruby on Rails, etc - the RESTful API abstracts the
  implementation

### KEY TERMS

- Idempotent Methods
- Non-idempotent Methods

- URL = URI + Network protocol details (`http`)

- Richard Maturity Model (RMM)
- RMM is used to describe the quality of the RESTful service
- A model used to describe the maturity of RESTful service

- blocking, non-reactive
- reactive, non-blocking

- Spring RestTemplate (non-reactive and blocking web client - To be deprecated, not preferred)
- Spring WebClient (non-blocking and reactive web client - Preferred)
