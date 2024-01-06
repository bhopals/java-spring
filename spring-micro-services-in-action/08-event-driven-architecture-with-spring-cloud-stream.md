## Event-Driven architecture with Spring Cloud Stream

- Using asynchronous messages to communicate between applications

- The concept of using messages to communicate events representing changes in state.
- This concept is called Event Driven Architecture (EDA). It’s also known as Message Driven Architecture (MDA).

- An EDA-based approach allows you to do is to build highly decoupled systems that can react to changes without being tightly
  coupled to specific libraries or services. When combined with microservices, EDA allows you to quickly add new functionality into your application by merely having the service listen to the stream of events (messages) being emitted by your application.

- Spring Cloud Stream allows you to easily implement message publication and consumption.

### The case for messaging, EDA (Event Driven Architecture), and microservices

- 1. synchronous request response model.
- 2. emit an asynchronous event (message)

#### 01. Using synchronous request-response approach to communicate state change

- 1. A licensing service user makes a call to retrieve licensing data.

- 2. The licensing service first checks the Redis cache for the organization data.

- 3. If the organization data isn’t in the Redis cache, the licensing service calls the organization service to retrieve it.

- 4. Organization data may be updated via calls to the organization service.

- 5. When organization data is updated, the organization service either calls back into the licensing service endpoint and
     tells it to invalidate its cache or talks to the licensing service’s cache directly.

- We see, in synchronous communication, there is a tight coupling between microservices

#### 02. Using messaging to communicate state changes between services

- 1. When the organization service communicates state changes, it publishes a message to a queue

- 2. The licensing service monitors the queue for any messages published by the organization service and can
     invalidate the Redis cache data as needed.

- When it comes to communicating state, the message queue acts as an intermediary between the licensing and organization service.
  - This approach offers four benefits:
    - Loose Coupling
    - Durability
    - Scalability
    - Flexibility

#### Downsides of a messaging architecture

Like any architectural model, a messaging-based architecture has tradeoffs. A messaging-based architecture can be complex and requires the development team to pay close attention to several key things, including

- Message handling semantics
  - Message Order, Failed messages reprocessing, Exception in messaging, Duplicate messages
- Message visibility
- Message choreography

### Introducing Spring Cloud Stream

- Spring Cloud makes it easy to integrate messaging into your Spring-based microservices.
- The Spring Cloud Stream project is an annotation-driven framework that allows you to easily build
  message publishers and consumers in your Spring application.

- Spring Cloud Stream also allows you to abstract away the implementation details of
  the messaging platform you’re using. Multiple message platforms can be used with
  Spring Cloud Stream (including the Apache Kafka project and RabbitMQ) and the
  implementation-specific details of the platform are kept out of the application code.
  The implementation of message publication and consumption in your application is
  done through platform-neutral Spring interfaces.

#### The Spring Cloud Stream architecture

- 1. The service client calls the service, and the service changes the state of the data it owns. This is done in the
     business logic of the service.

- 2. The source is the service’s Spring code that publishes the message.

- 3.  The message is published to a channel.

- 4. A binder is the Spring Cloud Stream’s framework code that communicates to the specific messaging system.

- 5. The message broker can be implemented using any number of messaging platforms, including Apache Kafka and RabbitMQ.

- 6. The order of message processing (binder, channel, sink) changes as a service receives a message.

- 7. A sink is the service-specific code that listens to a channel and then processes the incoming message.

- With the publication and consumption of a message in Spring Cloud, four components are involved in publishing and
  consuming the message:
  - Source
    - A source is a Spring annotated interface that takes a POJO that represents the message to be published.
  - Channel
    - A channel is an abstraction over the queue that’s going to hold the message after it has been published by
      the message producer or consumed by a message consumer
  - Binder
    - The binder is part of the Spring Cloud Stream framework. It’s the Spring code that talks to a specific message platform
  - Sink
    - In Spring Cloud Stream, when a service receives a message from a queue, it does it through a sink. A sink listens
      to a channel for incoming messages and de-serializes the message back into a plain old Java object.

#### Writing a simple message producer and consumer

##### Writing the message producer in the organization service

- 1. Organization client calls organization service’s REST endpoint; data is updated.

- 2. `Source` (SimpleSourceBean) - Name of the bean the organization service will use internally to publish the message.

- 3. `Channel` (output) - Name of the Spring Cloud Stream channel that will map to the Kafka topic (which will be orgChangeTopic).

- 4. `Binder` (Kafka) - The Spring Cloud Stream classes and configuration that bind to your Kafka server

STEPS:

- Add dependency in POM

```
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-stream</artifactId>
  </dependency>
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-stream-kafka</artifactId>
  </dependency>
```

- Add Boot Strater with Enabling Spring Cloud Stream

```
 @SpringBootApplication
 @EnableEurekaClient
 @EnableCircuitBreaker
 // The @EnableBinding annotation tells Spring Cloud Stream to bind the application to a message broker.
 @EnableBinding(Source.class)
 public class Application {
     @Bean
     public Filter userContextFilter() {
         UserContextFilter userContextFilter = new UserContextFilter();
         return userContextFilter;
     }
     public static void main(String[] args) {
         SpringApplication.run(Application.class, args);
     }
 }
```

- Add Message Publication Code

```
 @Component
  public class SimpleSourceBean {
  private Source source;
  private static final Logger logger =
  LoggerFactory.getLogger(SimpleSourceBean.class);
      @Autowired
      public SimpleSourceBean(Source source){
          this.source = source;
      }
      public void publishOrgChange(String action,String orgId){
          logger.debug("Sending Kafka message {} for Organization Id: {}", action, orgId);

      OrganizationChangeModel change = new OrganizationChangeModel(OrganizationChangeModel.class.getTypeName(),
      action, orgId, UserContext.getCorrelationId());
      source.output().send(MessageBuilder.withPayload(change).build());
      }
  }
```

- Add KAFKA Broker Configuration

```
spring.application.name="organizationservice"
spring.stream.bindings.output.destination="orgChangeTopic"
spring.stream.bindings.output.content-type="application/json"
spring.stream.bindings.kafka.binder.zkNodes= "localhost"
spring.stream.bindings.kafka.binder.brokers= "localhost"
```

##### Writing the message consumer in the licensing service

- 1. A change message comes into the Kafka orgChangeTopic.
- 2. `Binder` (Kafka) - Spring Cloud Stream classes and configuration
- 3. `Channel` (inboundOrgChanges) - You’ll use both the default input channel and a custom
     channel (inboundOrgChanges) to communicate the incoming message.
- 4. `Sink` (OrganizationChangeHandler) - The OrganizationChangeHandler class processes each incoming message.

- Add dependency in POM

```
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream</artifactId>
 </dependency>
 <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-kafka</artifactId>
</dependency>
```

- Enable Binding

```
//Imports removed for conciseness
@EnableBinding(Sink.class)
public class Application {
    //Code removed for conciseness
    @StreamListener(Sink.INPUT)
    public void loggerSink(
        OrganizationChangeModel orgChange) {logger.debug("Received an event for organization id {}" , orgChange.getOrganizationId());
    }
}
```

- Add KAFKA Broker Configuration

```
spring.application.name="licensingservice"
spring.stream.bindings.input.destination="orgChangeTopic"
spring.stream.bindings.input.content-type="application/json"
spring.stream.bindings.input.group="licensingGroup"
spring.stream.bindings.binder.zkNodes= "localhost"
spring.stream.bindings.binder.brokers= "localhost"
```

The concept of a consumer group is this: You might have multiple services with
each service having multiple instances listening to the same message queue. You want
each unique service to process a copy of a message, but you only want one service
instance within a group of service instances to consume and process a message.

### A Spring Cloud Stream use case: distributed caching

//TODO

### Summary

- Asynchronous communication with messaging is a critical part of microservices architecture.

- Using messaging within your applications allows your services to scale and become more fault tolerant.

- Spring Cloud Stream simplifies the production and consumption of messages by using simple annotations and
  abstracting away platform-specific details of the underlying message platform.

- A Spring Cloud Stream message `source` is an annotated Java method that’s used to publish messages to a message broker’s queue.

- A Spring Cloud Stream message `sink` is an annotated Java method that receives messages off a message broker’s queue.

- Redis is a key-value store that can be used as both a database and cache.

### KEY TERMS

- Asynchronous messages to communicate between applications
- Event Driven Architecture (EDA)
- Message Driven Architecture (MDA)
- Messaging based solutions
- Spring Cloud Stream
- Emit an asynchronous event (message)

- Redis
  - A distributed KEY-VALUE Store database
- Invalidate cache

- Loose Coupling
- Durability
- Scalability
- Flexibility

- A synchronous HTTP response creates a hard dependency between services

- Message handling semantics
- Message visibility
- Message choreography
- Annotation-driven framework

- Message broker
- Message Queue
- Service Client
- Source
- Sink
- Channel
- Binder

- Bind the application to a message broker
- Bind to a message broker
- Distributed Caching
- Use Redis as Distributed Cache
- Increase Resiliency

- Using Messaging/Asyncronous Communication allows your APP. to scale and become more Fault Tolerant
- Abstracting away platform-specific details of the underlying message platform
