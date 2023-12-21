## 04. Introduction to Restful Web Services

### HTTP History

- HTTP/1
  - Optionall uses TLS/SSL
- HTTP/2
  - Lower Latency
  - Higher throughput
  - Uses TLS 1.2+
- HTTP/3
  - Uses QUIC (Quick UDP Internet Connections, pronounced quick)network protocol rather than TCP
  - Uses TLS 1.3

### HTTP Request Methods

- Idempotent Methods

  - Idempotence - A quality of an action such that repetitions of the action have no
    further effect on the outcome
  - `PUT` and `DELETE` are Idempotent Methods
  - Safe Methods (GET, HEAD, TRACE, OPTIONS) are also development
  - Being truly Idemotent is not enforced by the protocol

- Non-Idempotent Methods
  - POST is NOT Idempotent
  - Multiple Posts are likely to create multiple resources
  - That is why websites asking only to click Submit once, OR they disable the button after the click

#### HTTP Status Code

- 100 series are informational in nature
- 200 series indicate successful request - (200 Okay; 201 Created; 204 Accepted)
- 300 series are redirections - (301 Moved Permanently)
- 400 series are client errors - (400 Bad Request; 401 Not Authorized; 404 Not Found)
- 500 series are server side errors - (500 Internal Server Error; 503 Service Unavailable)

### Guide of REST

#### RESTful Web Services

- Because of their simplicity and versatility, RESTful web services have become the de facto
  standard for web services.
- REST - Representational State Transfer
  - Representational - Typically JSON or XML
  - State Transfer - Typically via HTTP
  - Established by Roy Fielding from his 2000 doctoral dissertation

#### What is REST?

- REST APIs use HTTP Verbs to CREATE, MANAGE, DELETE server resources
- For Example:
  - GET: is used to get a resource
  - DELETE: is used to delete a resource
- Resources are typically data structures presented by JSON or XML
- HTTP Status codes are used to communicate success, failure, or errors
- REST is not a formal standard, more generally agreed upon methods and techniques

#### RESTful Terminology

- Verbs - HTTP Methods: GET, POST, PUT, DELETE
- Messages - The payload of the action (JSON/XML)
- URI - Uniform Resource Identifier
  - A unique string identifying a resource
- URL - Uniform Resource Locator

  - A URI with network informaton

- All URLs are URIs, but not all URIs are URLs.
- A URI is an identifier of a specific resource. Examples: Books, Documents
- A URL is special type of identifier that also tells you how to access it. Examples: HTTP, FTP, MAILTO
- If the protocol (https, ftp, etc.) is either present or implied for a domain, you should call it a URL—even though it’s also a URI.
- Reference - https://danielmiessler.com/p/difference-between-uri-url/

- Statless - Service does not maintain any client state
- HATEOAS - Hypermedia As the Engine of Application State
  - A REST Client should then be able to us service-provided links dynamically to discover
    all the available actions and resources it needs. As access proceeds, the server response with
    text that includes hyperlinks to other actions that are currently available.

#### More on REST

- REST (Representational State Transfer)
- REST architecture streamlines communication between web components.
- REST, or REpresentational State Transfer, is an architectural style for providing standards between
  computer systems on the web, making it easier for systems to communicate with each other
- REST-compliant systems, often called RESTful systems, are characterized by how they are stateless
  and separate the concerns of client and server

- The key abstraction of information in REST is a resource.
- REST is a set of architectural constraints, not a protocol or a standard
- REST is a set of architectural principles.
- More - https://www.codecademy.com/article/what-is-rest

#### The Six Guiding Principles of REST

REST is based on some constraints and principles that promote simplicity, scalability, and statelessness in the design. The six guiding principles or constraints of the RESTful architecture are:

- 1. Uniform Interface
     REST defines a consistent and uniform interface for interactions between clients and servers. For example, the HTTP-based REST APIs make use of the standard HTTP methods (GET, POST, PUT, DELETE, etc.) and the URIs (Uniform Resource Identifiers) to identify resources.
     This is a fundamental principle for the design of any RESTful API. There should be a uniform and standard way of interacting with a given server for all client types. The uniform interface helps to simplify the overall architecture of the system

- 2. Client-Server Design Pattern
     The client-server design pattern enforces the separation of concerns, which helps the client and the server components evolve independently. While the client and the server evolve, we have to make sure that the interface/contract between the client and the server does not break.
     REST architecture imposes the client-server design pattern, which enforces the separation of concerns and helps the client and server function independently. In REST, a client is an entity that requests a resource. A server is an entity responsible for storing these resources, having business logic, and sending the response to the client. In many cases, one server (API) can have multiple clients using different client-side technologies, or even other servers that act as a client of the API.

- 3. Stateless
     Statelessness mandates that each request from the client to the server must contain all of the information necessary to understand and complete the request. The server cannot take advantage of any previously stored context information on the server. For this reason, the client application must entirely keep the session state.
     REST architecture mandates that the server will not store anything related to a session. The server should process and complete each request independently. Each client request should contain all of the information necessary for understanding, processing, and completing it. Clients can do this via query parameters, headers, URIs, request body, etc. This principle of stateless versus stateful web app design will be discussed in more detail below.

- 4. Cacheable
     The cacheable constraint requires that a response should implicitly or explicitly label itself as cacheable or non-cacheable. If the response is cacheable, the client application gets the right to reuse the response data later for equivalent requests and a specified period.
     very response sent by the server should contain information regarding its cacheability. Simply, the clients should be able to determine whether or not this response can be cached from their side, and if so, for how long. If a response is cacheable, the client has the right to return the data from its cache for an equivalent request and specified period, without sending another request to the server. A well-managed caching mechanism greatly improves the availability and performance of an API by completely or partially eliminating some of the client-server interactions. However, this increases the chances of users receiving stale data.

- 5. Layered System
     The layered system style allows an architecture to be composed of hierarchical layers by constraining component behavior. In a layered system, each component cannot see beyond the immediate layer they are interacting with. A layman’s example of a layered system is the MVC pattern. The MVC pattern allows for a clear separation of concerns, making it easier to develop, maintain, and scale the application.
     An application with layered architecture is composed of various hierarchical layers. Each layer has information from only itself, and no other layer. There can be multiple intermediate layers between the client and server. You can design your REST APIs with multiple layers, each having tasks such as security, business logic, and application, all working together to fulfill client requests. These layers are invisible to the client and only interact with intermediate servers. This architecture improves the overall system availability and performance by enabling load balancing and shared caches.

#### Richardson Maturity Model(RMM)

- Established by Leonard Richardson in a 2008 Q-Con Presentation
- A model used to describe the maturity of RESTful Services
- Unlike SOAP, there is no formal specification for REST
- RMM is used to describe the quality of the RESTful service

- RMM LEVELS

  - Level 3: Hypermedia Controls
  - Level 2: HTTP Verbs
  - Level 1: Resources
  - Level 0: The Swap of POX

- Level 0: The Swap of POX

  - POX - Plain Old XML
  - Uses Implementing Protocol as a transport protocol
  - Typically uses one URI and one kind of method
  - Examples - RPC, SOAP, XML-RPC

- Level 1: Resources

  - Uses Multiple URIs to dentify specific resources
  - Examples:
    - http://www.example.com/product/1234
    - http://www.example.com/product/5678
  - Still Uses a Single method (ie `GET`)

- Level 2: HTTP Verbs

  - HTTP Verbs are used with URIs for desired actions
  - Examples:
    - GET /products/1234 - to return data for product 1234
    - PUT /products/1234 (with XML body) - to update data for product 1234
    - DELETE /products/1234 - to delete product 1234
  - Most common in practical use

- Level 3: Hypermedia

  - Representation now contains URIs which may be useful to consumers
  - Helps client developers exploure the resource
  - No clear standard at this time
  - Spring provides an implementation of HATEAOS (`Traverson` is the library that's been used to traverse)

- SUMMARY
  - Level 1 - Breaks large service into disting URIs
  - Level 2 - Introduces Verbs to implment actions (Most popularly used)
  - Level 3 - Provides discoverability, making the API more self documenting

#### Spring Framework and RESTful Services

- The Spring framework has very robust support for creating and consuming RESTFul Web Services
- Spring Framework has 3 Distint `Web Frameworks` for creating RESTful Services
  - Spring MVC
  - Spring Web Flux
  - WebFlux.fn
- Srping Framework has 2 Distinct `Web clients` for consuming RESTful Services
  - Spring RestTemplate - Web Client
  - Spring WebClient - Web Client
- There are also several popular libraries for creating and consuming RESTful Services frequently
  used with Spring

- Spring MVC - Web Framework

  - Spring MVC is the oldest and most commonly used library for creating and consuming RESTful Services
  - Part of the core Spring Framework
    - Compatible with JAVA EE (Jakarta EE in Spring 6)
  - MVC - Model View Controller
  - Has Robust support for traditional Web Applications
  - Based on Traditional Java Servlet API
    - By nature this is Blocking, non-reactive

- Spring WebFlux - Web Framework

  - Spring WebFlix was introduced with version 5 of the Spring Framework
  - WebFlux uses project Reactor to provide reactive web services
    - It does not use Java Servlet API, thus it is NON-BLOCKING
  - Follows very closely to the configuration model of Spring MVC

- WebFlux.fn - Web Framework

  - Also introduced in Spring Framework 5
  - WebFlux.fn is a functional programming model used to define endpoints
  - Alternative to annotation based configuration
  - Designed to repidly and simply define microservice endpoints

- Spring RestTemplate - Web Client

  - RestTemplate is Spring's primary library by consuming RESTFul Web Services
  - Very mature - been a part of Spring for a very long time
  - Highly Configurable
  - As of Spring Framework 5 RestTemplate is in maintenance mode
    - No new features are planned
    - Step before deprecation, Spring recommends using `WebClient` for new development

- Spring WebClient - Web Client

  - Spring WebClient was introduced in Spring Framework version 5
  - This is Spring's reactive web client
  - By default uses Reactor Netty, a non-blocking HTTP Client Library

- Marshalling/ Unmarshalling

  - Converting Java POJSs to JSON or XML is called Marshalling
  - Converting JSON or XML to Java Objects is called Unmarshalling
  - By default Spring Boot configures Jackson to facilitate Marshalling and Unmarshalling
  - Spring Boot does not support several other libraries, however jackson is by far the most popular

- SPA - Single Page Applications
  - RESTFul APIs are often combined with SPA applications for rich user web applications
  - Popular client side SPA frameworks include Vue, BackBoneJS, ReactJS, AngularJS, and EmberJS
  - Frequent question is which framework is the "BEST" to use with Spring Boot
  - The correct answer is that it does not matter
    - The framework used is decoupled from Spring via the HTTP/JSON (or XML) layer
    - All of these libraries can consume RESTful APIs
    - Server side can be Spring Boot, .NET, Ruby on Rails, etc - The RESTful API
      abstracts the implementation

### KEY TERMS

- SOLID Principles of OOP
  - S - Single Responsibility Principle
  - O - Open Closed Principle
  - L - Liskov Substitution Principle
  - I - Interface Segregation Principle
  - D - Dependency Inversion Principle
- Software entities (e.g., classes, modules, functions)
- REST, or REpresentational State Transfer, is an architectural style for providing standards between
  computer systems on the web, making it easier for systems to communicate with each other

- REST Principles

  - Uniform Interface
  - Stateless
  - Cacheable
  - Layered System
  - Client Server Design Pattern

- Richardson Maturity Model for REST

  - Level 0
  - Level 1 (Resources) - Breaks large services into distinct URIs
  - Level 2 (HTTP Verbs) - Introduces Verbs to implement actions (Most Popularly used)
  - Level 3 (Hypermedia) - Provides discoverability, making the API more self documenting (HATEAO)

- Spring Framework has 3 Distint `Web Frameworks` for creating RESTful Services
  - Spring MVC
  - Spring Web Flux
  - WebFlux.fn
- Srping Framework has 2 Distinct `Web clients` for consuming RESTful Services
  - Spring RestTemplate - Web Client
  - Spring WebClient - Web Client
