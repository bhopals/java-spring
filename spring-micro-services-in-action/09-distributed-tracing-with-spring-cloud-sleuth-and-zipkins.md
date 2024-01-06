## Distributed tracing with Spring Cloud Sleuth and Zipkin

- The microservices architecture is a powerful design paradigm for breaking down complex monolithic software systems into
  smaller, more manageable pieces.

- Microservices are distributed by nature.

### Topics

- Using Spring Cloud Sleuth to inject tracing information into service calls
- Using log aggregation to see logs for distributed transaction
- Querying via a log aggregation tool
- Using OpenZipkin to visually understand a user’s transaction as it flows across multiple microservice calls
- Customizing tracing information with Spring Cloud Sleuth and Zipkin

- Using correlation IDs to link together transactions across multiple services
- Aggregating log data from multiple services into a single searchable source
- Visualizing the flow of a user transaction across multiple services and understanding the performance characteristics
  of each part of the transaction

- Spring Cloud Sleuth
  - Add correlation IDs
- Papertrail
  - Aggregate logging data from multiple sources into single searchable database.
- Zipkin
  - A data-visualization tool

### Spring Cloud Sleuth Responsibilities

- Transparently create and inject a correlation ID into your service calls if one doesn’t exist.
- Manage the propagation of the correlation ID to outbound service calls so that the correlation ID for a
  transaction is automatically added to outbound calls.
- Add the correlation information to Spring’s MDC logging so that the generated correlation ID is automatically logged by
  Spring Boots default SL4J and Logback implementation.
- Optionally, publish the tracing information in the service call to the Zipkindistributed tracing platform.

### Log aggregation and Spring Cloud Sleuth

The combination of aggregated logs and a unique transaction ID across service log entries
makes debugging distributed transactions more manageable.

- Each individual service is producing logging data.
- An aggregation mechanism collects all of the data and funnels it to a common data store.
- As data comes into a central data store, it is indexed and stored in a searchable format
- The development and operations teams can query the log data to find individual transactions.
  The trace IDs from Spring Cloud Sleuth log entries allow us to tie log entries across services.

- Options for Log Aggregation Solutions for Use with Spring Boot
  - ELK (Elasticsearch, Logstash, Kibana)
  - Graylog
  - Splunk
  - Papertrail
  - Sumo Logic

#### A Spring Cloud Sleuth/Papertrail implementation in action

Using native Docker capabilities, logspot, and Papertrail allows you to quickly implement a
unified logging architecture.

- 1. The individual containers write their logging data to standard out. Nothing has changed in terms of their configuration.

- 2. In Docker, all containers write their standard out to an internal filesystem called Docker.sock.

- 3. A Logspout Docker container listens to Docker.sock and writes whatever goes to standard output to a remote syslog location.

- 4. Papertrail exposes a syslog port specific to the user’s application. It ingests incoming log data and indexes and stores it.

- 5. The Papertrail web application lets the user issue queries. Here you can enter a Spring Cloud Sleuth trace ID
     and see all of the log entries from the different services that contain that trace ID.

- Adding the correlation ID to the HTTP response with Zuul

### Distributed tracing with Open Zipkin

- Having a unified logging platform with correlation IDs is a powerful debugging tool.
- Distributed tracing involves providing a visual picture of how a transaction flows across your different microservices.
- Distributed tracing tools will also give a rough approximation of individual microservice response times.

- Spring Cloud Sleuth and the OpenZipkin (Zipkin)
- Zipkin is a distributed tracing platform that allows you to trace transactions across multiple service invocations.

- Setting up Spring Cloud Sleuth and Zipkin involves four activities:
  - 1.  Adding Spring Cloud Sleuth and Zipkin JAR files to the services that capture trace data
  - 2.  Configuring a Spring property in each service to point to the Zipkin server that will collect the trace data
  - 3.  Installing and configuring a Zipkin server to collect the data
  - 4.  Defining the sampling strategy each client will use to send tracing information to Zipkin

#### 01. Setting up the Spring Cloud Sleuth and Zipkin dependencies

```
<dependency>
 <groupId>org.springframework.cloud</groupId>
 <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
 <groupId>org.springframework.cloud</groupId>
 <artifactId>spring-cloud-sleuth-zipkin</artifactId>
</dependency>
```

#### 02. Configuring the services to point to Zipkin

With the JAR files in place, you need to configure each service that wants to communicate with Zipkin
`spring.zipkin.baseUrl=http://localhost:9411`

- Zipkin, RabbitMQ, and Kafka
  Zipkin does have the ability to send its tracing data to a Zipkin server via RabbitMQ
  or Kafka. From a functionality perspective, there’s no difference in Zipkin behavior if
  you use HTTP, RabbitMQ, or Kafka. With the HTTP tracing, Zipkin uses an asynchronous thread to send performance data. The main advantage to using RabbitMQ or
  Kafka to collect your tracing data is that if your Zipkin server is down, any tracing messages sent to Zipkin will be “enqueued” until Zipkin can pick up the data.

#### 03. Installing and configuring a Zipkin server

- Add Dependencies

```
<dependency>
 <groupId>io.zipkin.java</groupId>
 <artifactId>zipkin-server</artifactId>
</dependency>
<dependency>
 <groupId>io.zipkin.java</groupId>
 <artifactId>zipkin-autoconfigure-ui</artifactId>
</dependency>
```

- Add Boot class

```
@SpringBootApplication
// The @EnableZipkinServer allows you to quickly start Zipkin as a Spring Boot project.
@EnableZipkinServer
public class ZipkinServerApplication {
public static void main(String[] args) {
    SpringApplication.run(ZipkinServerApplication.class, args);
 }
}
```

- Zipkin Backend store
  - Zipkin supports four different back end data stores.
    - 1.  In-memory data
    - 2.  MySQL: http://mysql.com
    - 3.  Cassandra: http://cassandra.apache.org
    - 4.  Elasticsearch: http://elastic.co

#### 04. Setting tracing levels

- Define how often each service should write data to Zipkin.
  `spring.sleuth.sampler.percentage=1`
- The property takes a value between 0 and 1:
  - A value of 0 means Spring Cloud Sleuth won’t send Zipkin any transactions.
  - A value of .5 means Spring Cloud Sleuth will send 50% of all transactions.

### Summary

- Spring Cloud Sleuth allows you to seamlessly add tracing information (correlation ID) to your microservice calls.

- Correlation IDs can be used to link log entries across multiple services. They allow you to see the behavior of a
  transaction across all the services involved in a single transaction.

- While correlation IDs are powerful, you need to partner this concept with a log aggregation platform that will
  allow you to ingest logs from multiple sources and then search and query their contents.

- While multiple on-premise log aggregation platforms exist, cloud-based services allow you to manage your logs without having
  to have extensive infrastructure in place. They also allow you to easily scale as your application logging volume grows.

- You can integrate Docker containers with a log aggregation platform to capture all the logging data being written to the
  containers stdout/stderr. In this chapter, you integrated your Docker containers with Logspout and an online cloud
  logging provider, Papertrail, to capture and query your logs.

- While a unified logging platform is important, the ability to visually trace a transaction through its microservices
  is also a valuable tool.

- Zipkin allows you to see the dependencies that exist between services when a call to a service is made.

- Spring Cloud Sleuth integrates with Zipkin. Zipkin allows you to graphically see the flow of your transactions and
  understand the performance characteristics of each microservice involved in a user’s transaction.

- Spring Cloud Sleuth will automatically capture trace data for an HTTP call and inbound/outbound message channel used
  within a Spring Cloud Sleuth enabled service.

- Spring Cloud Sleuth maps each of the service call to the concept of a span. Zipkin allows you to see the performance of a span.

- Spring Cloud Sleuth and Zipkin also allow you to define your own custom spans so that you can understand the performance
  of non-Spring-based resources (a database server such as Postgres or Redis).

### KEY TERMS

- Single Searchable Source
- Log Aggregating
- Distributed Tracing
  - Spring Cloud Sleuth and the OpenZipkin (Zipkin)
- Correlation ID

- Spring Cloud Sleuth
  - Add correlation IDs
- Papertrail
  - Aggregate logging data from multiple sources into single searchable database.
- Zipkin

  - A data-visualization tool

- A transcation flows across multiple services
- A Distributed Transaction
- Funnel Data to common data store

- Trace Id
- Span Id

- Unified Logging Architecture
- Unified Logging Platform
- Zipkin

  - Zipkin is a distributed tracing platform that allows you to trace transactions across multiple service invocations.

- Correlation IDs can be used to link log entries across multiple services
- A Log Aggregation platform
- Zipkin allows you to graphically see the flow of your transactions and understand the performance characteristics
