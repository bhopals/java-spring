## JMS Messaging

### Introduction to JMS

- JMS is a Java API which allows a Java Application to send a message to another application

#### What is JMS?

- JMS - Java Messaging Service
- JMS is a Java API which allows a Java Application to send a message to another application
  - Generally the other application is a Java application - but not always!
- JMS is a standard Java API which requires an underlying implementation to be provided
  - Much like JPA - where JPA is the API standard, and Hibernate is the implementation
- JMS is highly scalable and allows you to loosely couple applications using asynchronous messaging

#### JMS Implementations

- Amazon SQS
- Apache ActiveMQ
- JBoss Messaging
- IBM MQ - (Closed source / paid)
- OracleAQ - (Closed Source / paid)
- RabbitMQ

#### Why Use JMS over REST?

- JMS is a true messaging service
- Asynchronous - send and forget!
- Greater through put - the HTTP protocol is slow comparatively
  - JMS protocols are VERY performant
- Flexibility in message delivery - Deliver to one or many consumers
- Security - JMS has a very robust security
- Reliability - Can guarantee message delivery

#### Types of Messaging

- Point to Point
  - Message is queued and delivered to one consumer
  - Can have multiple consumers - but message will be delivered only ONCE
  - Consumers `connect` to a `queue`
- Publish / Subscribe
  - Message is delivered to one or more subscribers
  - Subscribers will `subscribe` to a `topic`, then receive a copy of all messages sent to the topic

#### JMS KEY TERMS

- JMS Provider - JMS Implementation
- JMS Client - Application which sends or receives messages from the JMS provider
- JMS Producer or Publisher - JMS Client which sends messages
- JMS Consumer or Subscriber - JMS Client which receives messages
- JMS Message - the entity of data sent (details next slide!)
- JMS Queue - Queue for point to point messages. Often, not always, FIFO
- JMS Topic - Similar to a queue - but for publish and subscribe

#### JMS Message

- A JMS Message contains three parts:
  - Header - contains meta data about the message
  - Properties - Message properties are in 3 sections
    - Application - From Java Application sending message
    - Provider - Used by the JMS provider and are implementation specific
    - Standard Properties - Defined by the JMS API - Might not be supported by the provider
  - Payload - the message itself

##### JMS Header Properties

- JMSCorrelationID - String value, typically a UUID. Set by application, often used to trace a
  message through multiple consumers
- JMSExpires - Long - zero, does not expire. Else, time when message will expire and be removed from the queue
- JMSMessageId - String value, typically set by the JMS Provider
- JMSPriority - Integer - Priority of the message
- JMSTimestamp - Long - Time message was sent
- JMSType - String - The type of the message
- JMSReplyTo - Queue or topic which sender is expecting replies
- JMSRedelivery - Boolean - Has message been re-delivered?
- JMSDeliveryMode - Integer, set by JMS Provider for delivery mode
  - Persistent (Default) - JMS Provider should make best effort to deliver message
  - Non-Persistent - Occasional message lost is acceptable

#### JMS Message Properties

- JSMXUserId - (String) User Id sending message. Set by JMS Provider.
- JMSXAppID - (String) Id of the application sending the message. Set by JMS Provider.
- JMSXDeliveryCount - (Int) Number of delivery attempts. Set by JMS Provider.
- JMSXGroupID - (String) The message group which the message is part of. Set by Client.
- JMSXGroupSeq - (Int) Sequence number of message in group. Set by Client.
- JMSXProducerTDIX - (String) Transaction id when message was produced. Set by JMS Producer.
- JSMXConsumerTXID - (String) Transaction Id when the message was consumed. Set by JMS Provider.
- JMSXRcvTimestamp - (Long) Timestamp when message delivered to consumer. Set by JMS Provider.
- JMSXState - (Int) State of the JMS Message. Set by JMS Provider.

#### JMS Custom Properties

- The JMS Client can set custom properties on messages
- Properties are set as key / value pairs (String, value)
- Values must be one of:
  - String, boolean, byte, double, float, int, short, long or Object

#### JMS Provider Properties

- The JMS Client can also set JMS Provider Specific properties
- These properties are set as JMS\_<provider name>
- JMS Provider specific properties allow the client to utilize features specific to the JMS Provider

#### JMS Message Types

- Message - Just a message, no payload. Often used to notify about events
- BytesMessage - Payload is an array of bytes
- TextMessage - Message is stored as a string. (Often JSON or XML)
- StreamMessage - sequence of Java primitives
- MapMessage - message is name value pairs
- ObjectMessage - Message is a serialized Java object

#### Which Message Type to Use?

- JMS 1.0 was originally released in 1998 - Initial focus was on Java to Java messaging
- Since 1998 Messaging and technology has grown and evolved beyond the Java ecosystem
- JMS TextMessages with JSON or XML payloads are currently favored
  - Decoupled from Java - can be consumed by any technology
  - Not uncommon to ‘bridge’ to non-java providers
  - Makes migration to a non-JMS provider less painful
    - Important since messaging is becoming more and more generic and abstracted

### Initial Project and Maven Dependencies

- Create new project using Spring Initilizer

  - Spring Boot DevTools
  - Lombok
  - Spring Web Starter
  - Spring for Apache ActiveMQ Artemis

- Add artimis embedded server dependency

```
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>artemis-server</artifactId>
</dependency>

<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>artemis-jms-server</artifactId>
</dependency>
```

#### Java Message Object

- Setup a POJO (`HelloWorldMessage`) that we will be using to send messages

```
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class HelloWorldMessage implements Serializable {
    static final long serialVersionUID = -6703826490277916847L;
    private UUID id;
    private String message;
}
```

#### Embedded Server Configuration

- In the main Spring Boot starter, we can add ActiveMQ server configuration

```
		ActiveMQServer server = ActiveMQServers.newActiveMQServer(new ConfigurationImpl()
			.setPersistenceEnabled(false)
			.setJournalDirectory("target/data/journal")
			.setSecurityEnabled(false)
			.addAcceptorConfiguration("invm", "vm://0"));

		server.start();
```

#### Task Configuration

- Configure a Task executor for Spring
- To send out message periodically by enabling scheduling
- Create a Config class and annotate it with - `@EnableScheduling`, `@EnableAsync`, `@Configuration`

```
/**
 * Created by jt on 2019-07-17.
 */
@EnableScheduling
@EnableAsync
@Configuration
public class TaskConfig {

    @Bean
    TaskExecutor taskExecutor(){
        return new SimpleAsyncTaskExecutor();
    }
}
```

#### Message Converter Configuration

- Configure a Message converter with JMS Messages

```
@Configuration
public class JmsConfig {
    public static final String MY_QUEUE = "my-hello-world";
    public static final String MY_SEND_RCV_QUEUE = "replybacktome";
    @Bean
    public MessageConverter messageConverter(){
        MappingJackson2MessageConverter converter = new MappingJackson2MessageConverter();
        converter.setTargetType(MessageType.TEXT);
        converter.setTypeIdPropertyName("_type");
        return converter;
    }
}
```

#### Sending JMS Messagess

- Create Message Sender class

```
@Scheduled(fixedRate = 2000)
public void sendMessage(){
    HelloWorldMessage message = HelloWorldMessage
            .builder()
            .id(UUID.randomUUID())
            .message("Hello World!")
            .build();
    jmsTemplate.convertAndSend(JmsConfig.MY_QUEUE, message);
}
```

#### Receiving JMS Messages (Listener)

- Create a Class to Listen for a message - To receive message

```
private final JmsTemplate jmsTemplate;
    @JmsListener(destination = JmsConfig.MY_QUEUE)
    public void listen(@Payload HelloWorldMessage helloWorldMessage,
                       @Headers MessageHeaders headers, Message message){
       System.out.println("I Got a Message!!!!!");
       System.out.println(helloWorldMessage);
       // uncomment and view to see retry count in debugger
       // throw new RuntimeException("foo");

    }
```

#### Send and Receive of JMS Messages

- Send a JMS message and expects a reply

- There are use cases where you want to send and receive across the JMS broker. And what happens in
  this scenario is you send out to a queue, the message consumer then replies back on a temporary queue.
  And, this is all kind of managed transparently for us.

- Sender.java

```
@Scheduled(fixedRate = 2000)
    public void sendandReceiveMessage() throws JMSException {
        HelloWorldMessage message = HelloWorldMessage
                .builder()
                .id(UUID.randomUUID())
                .message("Hello")
                .build();
        Message receviedMsg = jmsTemplate.sendAndReceive(JmsConfig.MY_SEND_RCV_QUEUE, new MessageCreator() {
            @Override
            public Message createMessage(Session session) throws JMSException {
                Message helloMessage = null;
                try {
                    helloMessage = session.createTextMessage(objectMapper.writeValueAsString(message));
                    helloMessage.setStringProperty("_type", "guru.springframework.sfgjms.model.HelloWorldMessage");
                    System.out.println("Sending Hello");
                    return helloMessage;
                } catch (JsonProcessingException e) {
                   throw new JMSException("boom");
                }
            }
        });
        System.out.println(receviedMsg.getBody(String.class));
    }
```

- Add this in Listener.java

```
 @JmsListener(destination = JmsConfig.MY_SEND_RCV_QUEUE)
    public void listenForHello(@Payload HelloWorldMessage helloWorldMessage,
                       @Headers MessageHeaders headers, Message jmsMessage,
                               org.springframework.messaging.Message springMessage) throws JMSException {

        HelloWorldMessage payloadMsg = HelloWorldMessage
                .builder()
                .id(UUID.randomUUID())
                .message("World!!")
                .build();

        //example to use Spring Message type
       // jmsTemplate.convertAndSend((Destination) springMessage.getHeaders().get("jms_replyTo"), "got it!");

        jmsTemplate.convertAndSend(jmsMessage.getJMSReplyTo(), payloadMsg);

    }
```

#### Running Active MQ in Docker

- Running the image on local

```
docker run -it --rm \
  -p 8161:8161 \
  -p 61616:61616 \
  vromero/activemq-artemis
```

- Go to `http://127.0.0.1:8161/console/login`
- User name and password - `artemis` / `simetraehcapa`

- More - https://github.com/vromero/activemq-artemis-docker

#### Using Local ActiveMQ Broker with Spring Boot

- Run the docker container from the above - `Running Active MQ in Docker`

- Comment ActiveMQ server from Spring Boot Application

```
ActiveMQServer server = ActiveMQServers.newActiveMQServer(new ConfigurationImpl()
    .setPersistenceEnabled(false)
    .setJournalDirectory("target/data/journal")
    .setSecurityEnabled(false)
    .addAcceptorConfiguration("invm", "vm://0"));
server.start();
```

- Remove below `artemis` dependency from POM

```
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>artemis-server</artifactId>
</dependency>

<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>artemis-jms-server</artifactId>
</dependency>
```

- Add Queue username/password in `application.properties`

```
spring.artemis.user=artemis
spring.artemis.password=simetraehcapa
```

- To see, all the messages sent
  - Go to `http://127.0.0.1:8161/console/login`
  - User name and password - `artemis` / `simetraehcapa`

### KEY TERMS

- JMS is an API Standard
- Asynchronous messaging
- Greater Throughput
- Robust Security
- Guaranteed Delivery
- Loosely couple application
- JMS is highly scalable and allows you to loosely couple applications using asynchronous messaging

- Point to Point
  - Consumers `connect` to a `queue`
- Publish / Subscribe
  - Subscribes`subscribe` to a `topic`
- Message
- Producer / Publisher
- Consumer / Subscriber
- Queue
- Topic

- JMS Broker
- JMS Message (Header, Properties, Message)
- JMS Delivery Mode (Persistent, NonPersistent)
