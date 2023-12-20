## Spring In Action

### Spring Intro

- What is Spring?
  Any non-trivial application is composed of many components, each responsible for
  its own piece of the overall application functionality, coordinating with the other
  application elements to get the job done. When the application is run, those components somehow need to be created and introduced to each other.
  At its core, Spring offers a container, often referred to as the Spring application context, that creates and manages application components. These components, or beans,
  are wired together inside the Spring application context to make a complete application, much like bricks, mortar, timber, nails, plumbing, and wiring are bound together
  to make a house.

- The act of wiring beans together is based on a pattern known as dependency injection
  (DI). Rather than have components create and maintain the lifecycle of other beans
  that they depend on, a dependency-injected application relies on a separate entity
  (the container) to create and maintain all components and inject those into the beans
  that need them. This is done typically through constructor arguments or property
  accessor methods.

- The @Configuration annotation indicates to Spring that this is a configuration class
  that will provide beans to the Spring application context. The configuration’s class methods are annotated with @Bean, indicating that the objects they return should be added
  as beans in the application context (where, by default, their respective bean IDs will
  be the same as the names of the methods that define them).

- Java-based configuration offers several benefits over XML-based configuration,
  including greater type safety and improved refactorability

- Automatic configuration has its roots in the Spring techniques known as `autowiring`
  and `component scanning`. With component scanning, Spring can automatically discover
  components from an application’s classpath and create them as beans in the Spring
  application context. With autowiring, Spring automatically injects the components
  with the other beans that they depend on.

- Spring Boot is an extension
  of the Spring Framework that offers several productivity enhancements. The most
  well-known of these enhancements is `autoconfiguration`, where Spring Boot can make
  reasonable guesses of what components need to be configured and wired together,
  based on entries in the classpath, environment variables, and other factors

- Spring Boot autoconfiguration has dramatically reduced the amount of explicit
  configuration (whether with XML or Java) required to build an application.

- Benefits of Spring Boot, including starter dependencies and autoconfiguration.
- But in addition to starter dependencies and autoconfiguration,
  Spring Boot also offers a handful of other useful features:
  - The Actuator provides runtime insight into the inner workings of an application, including metrics, thread dump information, application health, and environment properties available to the application.
  - Flexible specification of environment properties.
  - Additional testing support on top of the testing assistance found in the core
    framework.

#### Initializing a Spring application

- The Spring Initializr is both a browser-based web application and a REST API,
  which can produce a skeleton Spring project structure that you can flesh out with
  whatever functionality you want.

```
@SpringBootApplication
public class TacoCloudApplication {
 public static void main(String[] args) {
 SpringApplication.run(TacoCloudApplication.class, args);
 }
}
```

- `@SpringBootApplication` is a composite application that combines three other
  annotations:

  - `@SpringBootConfiguration` — Designates this class as a configuration class.
    Although there’s not much configuration in the class yet, you can add Javabased Spring Framework configuration to this class if you need to. This annotation is, in fact, a specialized form of the @Configuration annotation.
  - `@EnableAutoConfiguration` — Enables Spring Boot automatic configuration.
    For now, know that this annotation tells Spring Boot to automatically configure any
    components that it thinks you’ll need.
  - `@ComponentScan` — Enables component scanning. This lets you declare other
    classes with annotations like @Component, @Controller, @Service, and others,
    to have Spring automatically discover them and register them as components in
    the Spring application context.

- Spring aims to make developer challenges easy, like creating web applications,
  working with databases, securing applications, and microservices.
- Spring Boot builds on top of Spring to make Spring even easier with simplified
  dependency management, automatic configuration, and runtime insights.
- Spring applications can be initialized using the Spring Initializr, which is webbased and supported
  natively in most Java development environments.
- The components, commonly referred to as beans, in a Spring application context can be declared explicitly with Java or XML, discovered by component scanning, or automatically configured with Spring Boot autoconfiguration.

### Spring Web

In a Spring web application, it’s a controller’s job to fetch and process data. And
it’s a view’s job to render that data into HTML that will be displayed in the browser.

#### Create Data/Domain/Model

```
//Using  'lombok' library - import lombok.Data;import lombok.RequiredArgsConstructor;
@Data
@RequiredArgsConstructor
public class Ingredient {
 private final String id;
 private final String name;
 private final Type type;
 public static enum Type {
 WRAP, PROTEIN, VEGGIES, CHEESE, SAUCE
 }
}
```

- @Data annotation at the class level is to generate all of those missing methods as well as
  a constructor that accepts all final properties as arguments

#### Creating a controller class

Controllers are the major players in Spring’s MVC framework. Their primary job is to
handle HTTP requests and either hand a request off to a view to render HTML
(browser-displayed) or write data directly to the body of a response (RESTful).

- Spring MVC request-mapping annotations

  - @RequestMapping General-purpose request handling
  - @GetMapping Handles HTTP GET requests
  - @PostMapping Handles HTTP POST requests
  - @PutMapping Handles HTTP PUT requests
  - @DeleteMapping Handles HTTP DELETE requests
  - @PatchMapping Handles HTTP PATCH requests

- To Register a Controller

```
@Configuration
public class WebConfig implements WebMvcConfigurer {
 @Override
 public void addViewControllers(ViewControllerRegistry registry) {
 registry.addViewController("/").setViewName("home");
 }
}
```

- Spring offers a powerful web framework called Spring MVC that can be used to
  develop the web frontend for a Spring application.

- Spring MVC is annotation-based, enabling the declaration of request-handling
  methods with annotations such as @RequestMapping, @GetMapping, and @PostMapping.

- Most request-handling methods conclude by returning the logical name of a
  view, such as a Thymeleaf template, to which the request (along with any model
  data) is forwarded.

- Spring MVC supports validation through the Java Bean Validation API and
  implementations of the Validation API such as Hibernate Validator.

- View controllers can be used to handle HTTP GET requests for which no
  model data or processing is required.

- In addition to Thymeleaf, Spring supports a variety of view options, including
  FreeMarker, Groovy Templates, and Mustache

### Spring Data

- Spring support for JDBC (Java Database Connectivity) to eliminate boilerplate code.
- Spring support for JPA (Java Persistence API), eliminating even more code.

- Spring’s JdbcTemplate greatly simplifies working with JDBC.
- PreparedStatementCreator and KeyHolder can be used together
  when you need to know the value of a database-generated ID.
- For easy execution of data inserts, use SimpleJdbcInsert.
- Spring Data JPA makes JPA persistence as easy as writing a repository interface

### Spring Security

- To Enable Spring Authentication

```
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {}
```

- Spring Security offers several options for configuring a user store, including
  these: (by overriding `configure` method)

  - In-Memory user store
  - JDBC-based user store
  - LDAP-backed user store (LDAP - Lightweight directory access protocol )
  - Custom User authentication - (User Details stored in DB)

- `AuthenticationManagerBuilder auth` to specify how users will be looked up during authentication.

#### LDAP

```
@Override
protected void configure(AuthenticationManagerBuilder auth)
 throws Exception {
 auth
 .ldapAuthentication()
 .userSearchBase("ou=people")
 .userSearchFilter("(uid={0})")
 .groupSearchBase("ou=groups")
 .groupSearchFilter("member={0}")
 .passwordCompare()
 .passwordEncoder(new BCryptPasswordEncoder())
 .passwordAttribute("passcode")
 .contextSource()
 .url("ldap://tacocloud.com:389/dc=tacocloud,dc=com");
}

```

- Spring Security autoconfiguration is a great way to get started with security, but most applications
  will need to explicitly configure security to meet their unique security requirements.
- User details can be managed in user stores backed by relational databases, LDAP, or
  completely custom implementations.
- Spring Security automatically protects against CSRF attacks.
- Information about the authenticated user can be obtained via the SecurityContext object (returned
  from SecurityContextHolder.getContext()) or injected into controllers using `@AuthenticationPrincipal`.

### Working with Configuration Properties

- Spring Boot autoconfiguration is like this. Autoconfiguration greatly
  simplifies Spring application development.

- Bean wiring—Configuration that declares application components to be created as beans in
  the Spring application context and how they should be injected into each other.
- Property injection—Configuration that sets values on beans in the Spring application context.

#### Understanding Spring’s environment abstraction

The Spring environment pulls from several property sources, including:

- JVM system properties
- Operating system environment variables
- Command-line arguments
- Application property configuration files (`application.properties`, `application.yml`)

- Spring beans can be annotated with @ConfigurationProperties to enable injection of values from
  one of several property sources.

- Configuration properties can be set in command-line arguments, environment variables, JVM system
  properties, properties files, or YAML files, among other options.

- Configuration properties can be used to override autoconfiguration settings,
  including the ability to specify a data-source URL and logging levels.

- Spring profiles can be used with property sources to conditionally set configuration
  properties based on the active profile(s).

### Creating REST Services

#### Enabling hypermedia

The API you’ve created thus far is fairly basic, but it does work as long as the client
that consumes it is aware of the API’s URL scheme. For example, a client may be hardcoded to know that it can obtain a list of recently created tacos by issuing a GET request
for /design/recent. Likewise, it may be hardcoded to know that it can append the ID
of any taco in that list to /design to get the URL for that particular taco resource.
Using hardcoded URL patterns and string manipulation is common among API
client code. But imagine for a moment what would happen if the API’s URL scheme
were to change. The hardcoded client code would have an obsolete understanding of
the API and would thus be broken. Hardcoding API URLs and using string manipulation on them makes the client code brittle.

Hypermedia as the Engine of Application State, or HATEOAS, is a means of creating
self-describing APIs wherein resources returned from an API contain links to related
resources. This enables clients to navigate an API with minimal understanding of the
API’s URLs. Instead, it understands relationships between the resources served by the API
and uses its understanding of those relationships to discover the API’s URLs as it traverses those relationships.

This particular flavor of `HATEOAS` is known as `HAL` (Hypertext Application Language; http://stateless.co/hal_specification.html), a simple and commonly used format for embedding hyperlinks in JSON responses

```
{

 "_embedded": {
 "tacoResourceList": [
 {
 "name": "Veg-Out",
 "createdAt": "2018-01-31T20:15:53.219+0000",
 "ingredients": [
 {
 "name": "Flour Tortilla", "type": "WRAP",
 "_links": {
 "self": { "href": "http://localhost:8080/ingredients/FLTO" }
 }
 },
 {
 "name": "Corn Tortilla", "type": "WRAP",
 "_links": {
 "self": { "href": "http://localhost:8080/ingredients/COTO" }
 }
 },
 {
 "name": "Diced Tomatoes", "type": "VEGGIES",
 "_links": {
 "self": { "href": "http://localhost:8080/ingredients/TMTO" }
 }
 },
 {
 "name": "Lettuce", "type": "VEGGIES",
 "_links": {
 "self": { "href": "http://localhost:8080/ingredients/LETC" }
 }
 },
 {
 "name": "Salsa", "type": "SAUCE",
 "_links": {
 "self": { "href": "http://localhost:8080/ingredients/SLSA" }
 }
 }
 ],
 "_links": {
 "self": { "href": "http://localhost:8080/design/4" }
 }
 },
 ...
 ]
 },
 "_links": {
 "recents": {
 "href": "http://localhost:8080/design/recent"
 }
 }
}
```

- Spring HATEOAS provides two primary types that represent hyperlinked resources:
  `Resource` and `Resources`. You build resources using `ControllerLinkBuilder`

```
@GetMapping("/recent")
public Resources<TacoResource> recentTacos() {
 PageRequest page = PageRequest.of(
 0, 12, Sort.by("createdAt").descending());
 List<Taco> tacos = tacoRepo.findAll(page).getContent();
 List<TacoResource> tacoResources =
 new TacoResourceAssembler().toResources(tacos);
 Resources<TacoResource> recentResources =
 new Resources<TacoResource>(tacoResources);
 recentResources.add(
 linkTo(methodOn(DesignTacoController.class).recentTacos())
 .withRel("recents"));
 return recentResources;
}
```

- REST endpoints can be created with Spring MVC, with controllers that follow
  the same programming model as browser-targeted controllers.
- Controller handler methods can either be annotated with @ResponseBody or
  return ResponseEntity objects to bypass the model and view and write data
  directly to the response body.
- The @RestController annotation simplifies REST controllers, eliminating the
  need to use @ResponseBody on handler methods.
- Spring HATEOAS enables hyperlinking of resources returned from Spring MVC
  controllers.
- Spring Data repositories can automatically be exposed as REST APIs using Spring
  Data REST.

### Consuming REST Services

A Spring application can consume a REST API with

- RestTemplate — A straightforward, synchronous REST client provided by the core Spring Framework.
- Traverson — A hyperlink-aware, synchronous REST client provided by Spring HATEOAS.
  Inspired from a JavaScript library of the same name.
- WebClient — A reactive, asynchronous REST client introduced in Spring 5.

#### RestTemplate

RestTemplate defines 12 unique operations, each of which is overloaded, providing a total
of 41 methods.

- delete(…) - Performs an HTTP DELETE request on a resource at a specified URL

- exchange(…) - Executes a specified HTTP method against a URL, returning a
  ResponseEntity containing an object mapped from the response body

- execute(…) - Executes a specified HTTP method against a URL, returning an object
  mapped from the response body

- getForEntity(…) Sends an HTTP GET request, returning a ResponseEntity containing
  an object mapped from the response body

- getForObject(…) Sends an HTTP GET request, returning an object mapped from a
  response body

- headForHeaders(…) Sends an HTTP HEAD request, returning the HTTP headers for the specified resource URL

- optionsForAllow(…) Sends an HTTP OPTIONS request, returning the Allow header for the
  specified URL

- patchForObject(…) Sends an HTTP PATCH request, returning the resulting object mapped
  from the response body

- postForEntity(…) POSTs data to a URL, returning a ResponseEntity containing an
  object mapped from the response body

- postForLocation(…) POSTs data to a URL, returning the URL of the newly created resource

- postForObject(…) POSTs data to a URL, returning an object mapped from the response body

- put(…) PUTs resource data to the specified URL

#### Examples:

```
public Ingredient getIngredientById(String ingredientId) {
 ResponseEntity<Ingredient> responseEntity =
 rest.getForEntity("http://localhost:8080/ingredients/{id}",
 Ingredient.class, ingredientId);
 log.info("Fetched time: " +
 responseEntity.getHeaders().getDate());
 return responseEntity.getBody();
}

public void updateIngredient(Ingredient ingredient) {
 rest.put("http://localhost:8080/ingredients/{id}",
 ingredient,
 ingredient.getId());
}

public Ingredient createIngredient(Ingredient ingredient) {
 return rest.postForObject("http://localhost:8080/ingredients",
 ingredient,
 Ingredient.class);
}
```

#### Navigating REST APIs with Traverson

Traverson comes with Spring Data HATEOAS as the out-of-the-box solution for consuming hypermedia APIs in Spring applications. This Java-based library is inspired by a similar JavaScript library of the same name (https://github.com/traverson/traverson).
You might have noticed that Traverson’s name kind of sounds like “traverse on”,
which is a good way to describe how it’s used. In this section, you’ll consume an API by
traversing the API on relation names.
Working with Traverson starts with instantiating a Traverson object with an API’s
base URI:

```
Traverson traverson = new Traverson(
URI.create("http://localhost:8080/api"), MediaTypes.HAL_JSON);
Resources<Ingredient> ingredientRes =
 traverson
 .follow("ingredients")
 .toObject(ingredientType);
Collection<Ingredient> ingredients = ingredientRes.getContent();
```

##### Summary

- Clients can use RestTemplate to make HTTP requests against REST APIs.
- Traverson enables clients to navigate an API using hyperlinks embedded in the responses.

### Sending messages asynchronously

- Java Message Service (JMS),
- RabbitMQ
- Advanced Message Queueing Protocol (AMQP)
- Apache Kafka

#### Sending messages with JMS

- JMS is a Java standard that defines a common API for working with message brokers

- Before JMS, each message broker had a proprietary API, making an application’s messaging code
  less portable between brokers. But with JMS, all compliant implementations can be worked with via a common
  interface in much the same way that JDBC has given relational database operations a common interface

- Spring supports JMS through a template-based abstraction known as JmsTemplate.
- Using JmsTemplate, it’s easy to send messages across queues and topics from the producer side and to
  receive those messages on the consumer side.
- Spring also supports the notion of message-driven POJOs: simple Java objects that react to messages
  arriving on a queue or topic in an asynchronous fashion.

##### Setting up JMS

If you’re using ActiveMQ, you’ll need to add the following dependency to your
project’s pom.xml file:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-activemq</artifactId>
</dependency>
```

If ActiveMQ Artemis is the choice, the starter dependency should look like this:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-artemis</artifactId>
</dependency>
```

- Set connection details in Properties file.

- More Details
  - Artemis — https://activemq.apache.org/artemis/docs/latest/using-server.html
  - ActiveMQ — http://activemq.apache.org/getting-started.html#GettingStarted-PreInstallationRequirements

##### Sending messages with JmsTemplate

With a JMS starter dependency (either Artemis or ActiveMQ) in your build, Spring
Boot will autoconfigure a JmsTemplate (among other things) that you can inject and
use to send and receive messages.

JmsTemplate has several methods that are useful for sending messages, including the following:

```
// Send raw messages
void send(MessageCreator messageCreator) throws JmsException;
void send(Destination destination, MessageCreator messageCreator)
 throws JmsException;
void send(String destinationName, MessageCreator messageCreator)
 throws JmsException;
// Send messages converted from objects
void convertAndSend(Object message) throws JmsException;
void convertAndSend(Destination destination, Object message)
 throws JmsException;
void convertAndSend(String destinationName, Object message)
 throws JmsException;
// Send messages converted from objects with post-processing
void convertAndSend(Object message,
 MessagePostProcessor postProcessor) throws JmsException;
void convertAndSend(Destination destination, Object message,
 MessagePostProcessor postProcessor) throws JmsException;
void convertAndSend(String destinationName, Object message,
 MessagePostProcessor postProcessor) throws JmsException;
```

##### Receiving JMS messages

When it comes to consuming messages, you have the choice of a `pull` model, where your
code requests a message and waits until one arrives, or a `push` model, in which messages
are handed to your code as they become available.

JmsTemplate offers several methods for pulling methods from the broker, including the following:

```
Message receive() throws JmsException;
Message receive(Destination destination) throws JmsException;
Message receive(String destinationName) throws JmsException;
Object receiveAndConvert() throws JmsException;
Object receiveAndConvert(Destination destination) throws JmsException;
Object receiveAndConvert(String destinationName) throws JmsException;
```

#### Working with RabbitMQ and AMQP

As arguably the most prominent implementation of AMQP, RabbitMQ offers a more
advanced message-routing strategy than JMS. Whereas JMS messages are addressed
with the name of a destination from which the receiver will retrieve them, AMQP messages are addressed with the name of an exchange and a routing key, which are decoupled from the queue that the receiver is listening to.

Messages sent to a RabbitMQ exchange are routed to one or more queues, based on routing keys and bindings.

- TERMS - Sender, Receiver, Exchange, Binding, Queue, RabbitMQ Broker, Message Routing Key,
  Binding key, value of Message

  - When a message arrives at the RabbitMQ broker, it goes to the exchange for which it
    was addressed. The exchange is responsible for routing it to one or more queues,depending on the type of exchange, the binding between the exchange and queues, and the value of the message’s routing key.

- There are several different kinds of exchanges, including the following:
  - `Default` — A special exchange that’s automatically created by the broker. It routes
    messages to queues whose name is the same as the message’s routing key. All
    queues will automatically be bound to the default exchange.
  - `Direct` — Routes messages to a queue whose binding key is the same as the message’s routing key.
  - `Topic` — Routes a message to one or more queues where the binding key (which
    may contain wildcards) matches the message’s routing key.
  - `Fanout` — Routes messages to all bound queues without regard for binding keys or routing keys.
  - `Headers` — Similar to a topic exchange, except that routing is based on message
    header values rather than routing keys.
  - `Dead letter` — A catch-all for any messages that are undeliverable (meaning they
    don’t match any defined exchange-to-queue binding).

The most important thing to understand is that messages are sent to `exchanges` with
`routing keys` and they’re consumed from queues. How they get from an `exchange` to a
`queue` depends on the `binding` definitions and what best suits your use cases

You need to define `exchange` and `routing keys`

```
spring:
 rabbitmq:
 template:
 exchange: tacocloud.orders
 routing-key: kitchens.central
```

#### Messaging with Kafka

At a glance, Kafka is a message broker just like ActiveMQ, Artemis, or Rabbit. But Kafka
has a few unique tricks up its sleeves.

Kafka is designed to run in a cluster, affording great scalability. And by partitioning its topics across all instances in the cluster, it’s very resilient. Whereas RabbitMQ deals primarily with queues in exchanges, Kafka utilizes topics only to offer pub/sub messaging

Kafka topics are replicated across all brokers in the cluster. Each node in the cluster acts as a leader for one or more topics, being responsible for that topic’s data and
replicating it to the other nodes in the cluster.

Going a step further, each topic can be split into multiple partitions. In that case,
each node in the cluster is the leader for one or more partitions of a topic, but not for
the entire topic. Responsibility for the topic is split across all nodes.

- A Kafka cluster is composed of multiple brokers, each acting as a leader for
  partitions of the topics.

- Each Topic replicated across all Brokers, Each Topic split into multiple partitions

#### Summary

- Asynchronous messaging provides a layer of indirection between communicating applications,
  which allows for looser coupling and greater scalability.
- Spring supports asynchronous messaging with JMS, RabbitMQ, or Apache Kafka.
- Applications can use template-based clients (JmsTemplate, RabbitTemplate, or
  KafkaTemplate) to send messages via a message broker.
- Receiving applications can consume messages in a pull-based model using the
  same template-based clients.
- Messages can also be pushed to consumers by applying message listener annotations
  (@JmsListener, @RabbitListener, or @KafkaListener) to bean methods.

### Integrating Spring

- Spring Integration enables the definition of flows through which data can be
  processed as it enters or leaves an application.
- Integration flows can be defined in XML, Java, or using a succinct Java DSL configuration style.
- Message gateways and channel adapters act as entry and exit points of an integration flow.
- Messages can be transformed, split, aggregated, routed, and processed by service activators
  in the course of a flow.
- Message channels connect the components of an integration flow.

### Introducing Reactor

- Imperative Code
- Reactive Code

- Example

  - Imperative Programming - A water balloon carries its payload all at once, soaking its intended target at the
    moment of impact.

  - Reactive Programming - A garden hose carries its payload as a stream of water that flows from the spigot
    to the nozzle.

There’s nothing inherently wrong with water balloons (or `imperative` programming),
but the person holding the garden hose (or applying `reactive` programming) has an
advantage in regard to scalability and performance.

#### Understanding Reactive Programming

The idea is simple: you write code as a list of instructions to be followed, one at a time, in the order
that they’re encountered. A task is performed and the program waits for it to complete before moving on to the next task. At each step along the way, the data that’s to be processed must be fully available so that it can be processed as a whole.

- Managing concurrency in multiple threads is challenging. More threads mean more complexity

- In contrast, reactive programming is functional and declarative in nature.

#### Defining Reactive Streams

`Reactive Streams` is an initiative started in late 2013 by engineers from Netflix, Lightbend, and Pivotal (the company behind Spring). Reactive Streams aims to provide a standard for asynchronous stream processing with non-blocking backpressure

The Reactive Streams specification can be summed up by four interface definitions:
`Publisher`, `Subscriber`, `Subscription`, and `Processor`

#### Summary

- Reactive programming involves creating pipelines through which data flows.
- The Reactive Streams specification defines four types: Publisher, Subscriber,
  Subscription, and Transformer (which is a combination of Publisher and Subscriber).
- Project Reactor implements Reactive Streams and abstracts stream definitions
  into two primary types, Flux and Mono, each of which offers several hundred operations.
- Spring 5 leverages Reactor to create reactive controllers, repositories, REST clients,
  and other reactive framework support.

### Developing Reactive APIs

- Blocking web frameworks won’t scale effectively under heavy request volume

Typical Servlet-based web frameworks, such as Spring MVC, are blocking and multithreaded in nature, using a single thread per connection. As requests are handled,
a worker thread is pulled from a thread pool to process the request. Meanwhile, the
request thread is blocked until it’s notified by the worker thread that it’s finished.
Consequently, blocking web frameworks won’t scale effectively under heavy
request volume. Latency in slow worker threads makes things even worse because it’ll
take longer for the worker thread to be returned to the pool, ready to handle another
request. In some use cases, this arrangement is perfectly acceptable. In fact, this is
largely how most web applications have been developed for well over a decade. But
times are changing.

- Asynchronous web frameworks, in contrast, achieve higher scalability with fewer
  threads—generally one per CPU core. By applying a technique known as `event looping`, these frameworks are able to handle many requests per thread, making the per-connection cost more economical.

- In an event loop, everything is handled as an event, including requests and callbacks
  from intensive operations like database and network operations. When a costly operation is needed, the event loop registers a callback for that operation to be performed
  in parallel, while it moves on to handle other events.

- When the operation is complete, it’s treated as an event by the event loop, the
  same as requests. As a result, asynchronous web frameworks are able to scale better
  under heavy request volume with fewer threads, resulting in reduced overhead for
  thread management.

- Spring WebFlux - Spring MVC Reactive web framework

- Flux v/s Mono
  While both Mono and Flux can be used for reactive programming, they have different use cases and behaviors. As we have seen, Mono is used when you want to work with a single value, while Flux is used when you want to work with a potentially unbounded stream of values.

- Observable v/s Single - RxJava options (equivalent for Flux / Mono in Spring WebFlux)

### KEY TERMS

- At its core, Spring offers a container, often referred to as the `Spring application context`,
  that creates and manages application components.
- persisting objects to a database
- The Spring Environment
- Beans in Spring Application Context
- Spring Property Soruces to fetch/pull property sources
- Make properties available to beans in the Application Context
- Defining profile-specific properties
- Configuring Profiles
- Conditionally creating beans with profiles (`@Profile("dev")`)

- `Hypermedia as the Engine of Application State`, or `HATEOAS`, is a means of creating
  self-describing APIs wherein resources returned from an API contain links to related
  resources.
- `Traverson` — A hyperlink-aware, synchronous REST client provided by Spring HATEOAS.
- Using RestTemplate, you are on your way writing resource-consuming REST clients.
- Traverson comes with Spring Data HATEOAS as the out-of-the-box solution for Consuming Hypermedia APIs
  in Spring Applications

- Asynchronous Messaging/Communication
- Making application messaging code less portable between brokers
- With JMS, all compliant implementations can be worked with via a common interface
- Artemis is a next-generation reimplementation of ActiveMQ
- Messages sent to a RabbitMQ exchange are routed to one or more queues, based on routing keys and bindings.
- An Exchange and a Routing Key
- TERMS - Sender, Receiver, Exchange, Binding, Queue, RabbitMQ Broker
- AMQP - Advanced Message Queue Protocol
- How the message get from an `exchange` to a `queue` depends on the `binding` definitions
- Kafka is designed to run in a cluster, affording great scalability. And by partitioning its topics across all instances in the cluster, it’s very resilient

- Nature of Imperative Programming
- Reactive programming has an advantage in regard to scalability and performance.
- `Reactive Streams` is an initiative started in late 2013 by engineers from Netflix, Lightbend, and Pivotal
- Reactive Streams aims to provide a standard for asynchronous stream processing with non-blocking backpressure
- Asynchronous Stream Processing
- Non-Blocking Backpressure
