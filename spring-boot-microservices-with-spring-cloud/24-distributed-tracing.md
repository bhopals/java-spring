## Distributed Tracing

- Allows us to see things going through the Services

### What is Distributed Tracing?

- Monoliths have the luxury of being self contained, thus tracing typically is not needed
- Transactions in microservices can span many services / instances, and even data centers
- Distributed tracing provides the tools to trace a transaction across services and nodes
- Distributed tracing is used for two aspects:
  - Performance monitoring across steps
  - Logging / troubleshooting

#### Spring Cloud Sleuth

- Spring Cloud Sleuth is the distributed tracing tool for Spring Cloud
- Spring Cloud Sleuth uses an open source distributed tracing library called Brave
- Conceptually what happens:
  - Headers on HTTP requests or messages are enhanced with trace data
  - Logging is enhanced with trace data
  - Optionally trace data can be reported to Zipkin

#### Tracing Terminology

- Spring Cloud Sleuth uses terminology established by Dapper
- Dapper is a distributed tracing system created by Google for their production systems
- `Span` - is a basic unit of work. Typically a send and receive of a message.
- `Trace` - A set of spans for a transaction
- `cs` / `sr` - `Client Sent` / `Server Received` - aka the request
- `ss` / `cr` - `Server Sent` / `Client Received` - aka the response

#### Zipkin Server

- Zipkin is an open source project used to report distributed tracing metrics
- Information can be reported to Zipkin via webservices via HTTP
  - Optionally metrics can be provided via Kafka or Rabbit
- Zipkin is a Spring MVC project
  - Recommended to use binary distribution or Docker image
  - Building your own is not supported
- Uses in memory database for development
  - Cassandra or Elasticsearch should be used for production

#### Zipkin Quick Start

- Via Curl:
  - curl -sSL https://zipkin.io/quickstart.sh | bash -s
  - java -jar zipkin.jar
- Via Docker (Recommend for course):
  - docker run -d -p 9411:9411 openzipkin/zipkin
- View traces in UI at:
  - http://your_host:9411/zipkin/

#### Installing Spring Cloud Sleuth

- org.springframework.cloud:spring-cloud-starter-sleuth
  - Starter for logging only
- org.springframework.cloud:spring-cloud-starter-zipkin
  - Starter for Sleuth with Zipkin - includes Sleuth dependencies
- Property spring.zipkin.baseUrl is used to configure Zipkin server

#### Logging Output

- Example: - DEBUG [beer-service,39853b63c1c3f919,419b9ac9a073bbba,true]
  - [Appname, TraceId, SpanId, exportable]
- `Appname` - Spring Boot Application Name
- `TraceId` - Id value of the trace
- `SpanId` - Id of the Span
- `Exportable` - Should span be exported to Zipkin? (Programmatic configuration option)

#### Logging Configuration

- Microservices typically will use consolidated logging
- Number of different approaches for this - highly dependent on deployment environment
- Consolidated logging will be covered in a future section of the course
- To support consolidated logging, log data should be available in JSON
- Spring Boot by default uses logback, which is easy to configure for JSON output

### Zipkin Server

- More - https://zipkin.io/
- Docker
  `docker run -d -p 9411:9411 openzipkin/zipkin`

- Go to `http://localhost:9411/zipkin`

### Setup Spring Cloud Sleuth

- Go to any service - Beer Serice
- Add zipkin (it internally has spring cloud sleuth) dependency in `POM.XML`

```
 <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

- Add the above zipkin depndencies in `sping-cloud-gateway` project as well.
- Add zipkin base url property in `sping-cloud-gateway` and Other services
  application properties - `spring.zipkin.baseUrl=http://localhost:9411/`

- Restart the service, hit the request - beerService getList
- To see Trace logs, copy traceId from logs and got to - http://localhost:9411/zipkin

### Logging Config for JSON

- Add logstash dependency is Service - beerService (To configure for JSON output)

```
 <dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>6.3</version>
</dependency>
```

- Need to add custom configuration for logstash (`resources/logback-spring.xml`)

```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
    ​
    <springProperty scope="context" name="springAppName" source="spring.application.name"/>

    <!-- You can override this to have a custom pattern -->
    <property name="CONSOLE_LOG_PATTERN"
              value="%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}"/>

    <!-- Appender to log to console in a JSON format -->
    <appender name="jsonConsole" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
            <providers>
                <timestamp>
                    <timeZone>UTC</timeZone>
                </timestamp>
                <version/>
                <logLevel/>
                <message/>
                <loggerName/>
                <threadName/>
                <context/>
                <pattern>
                    <omitEmptyFields>true</omitEmptyFields>
                    <pattern>
                        {
                        "severity": "%level",
                        "service": "${springAppName:-}",
                        "trace": "%X{X-B3-TraceId:-}",
                        "span": "%X{X-B3-SpanId:-}",
                        "parent": "%X{X-B3-ParentSpanId:-}",
                        "exportable": "%X{X-Span-Export:-}",
                        "baggage": "%X{key:-}",
                        "pid": "${PID:-}",
                        "thread": "%thread",
                        "class": "%logger{40}",
                        "rest": "%message"
                        }
                    </pattern>
                </pattern>
            </providers>
        </encoder>
    </appender>
    ​
    <root level="INFO">
        <appender-ref ref="jsonConsole"/>
    </root>
</configuration>
```

### Refactor Zipkin Configuration

- Disable zipking at each service application level
  `spring.zipkin.enabled=false`
- Enable zipkin at cloud config (parent) level
  `spring.zipkin.enabled=true`

### KEY TERMS

- Distrubuted Tracing with spring boot microservices

- Chain of events that lead to that failure
- Spring Cloud Sleuth

- Span
- Trace
- Client Sent
- Server Received
- Server Sent
- Client Received

- Distributed Tracing
- Performance Monitoring
- Logging and Troubleshooting
- Distributed Tracing Tool
- Report Distributed Tracing metrics
- Trace is a set of Spans for a transaction

- Consolidate Logging / Log Centralization

- Logstash - JSON output of the logs - To make it searchable
