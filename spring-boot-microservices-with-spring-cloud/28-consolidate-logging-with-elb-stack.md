## Consolidated Logging with ELB Stack

- E - `Elasticsearch`
- L - `Logstash`
- K - `Kibana`

- All products open source, supported company called elastic
- Elasticsearch - JSON based search engine based on Lucene
  - Highly scalable - 100s of nodes (cloud scale)

#### Logstash

- Data processing pipeline for log data
- Allows to:
  - Collect from multiple sources
  - Transform
  - Send

#### Kibana

- Data visualization tool for Elasticsearch
- Can query data and act as a dashboard
- Can also create charts, graphs, and alerts
  - Many many more features

#### Filebeat

- Filebeat is a log shipper
- Moves log data to a destination
- Often destination is a logstash server

  - Logstash is used for further transformation before sending to Elasticseach

- Take the log data from a Client Machine and Move it to Logstash Server.
- Logstash is going to transform to a common format and then send that data on Elasticsearch

#### ELK without Logstash

- Filebeat has ability to do some transformations
- Thus, possible to skip Logstash and write directly to Elasticsearch
- Previously we setup JSON logout put
- Filebeat can convert JSON logs to JSON objects for Elasticsearch

### Logging Configuration

- Add `logback-spring.xml` - The Log configuration in `resource` directory

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

### Add Elastic Search, Kibana, Filebeat

- Add docker compose - `compose-logging.yaml` for spinning up services

```
version: '3.8'
services:
    elasticsearch:
        image: docker.elastic.co/elasticsearch/elasticsearch:7.7.0
        ports:
            - 9200:9200
        environment:
            discovery.type: single-node
    kibana:
        image: docker.elastic.co/kibana/kibana:7.7.0
        ports:
            - 5601:5601
        restart: on-failure
        depends_on:
            - elasticsearch
    filebeat:
        image: docker.elastic.co/beats/filebeat:7.7.0
        volumes:
            - ./filebeat/filebeat.docker.yml:/usr/share/filebeat/filebeat.yml:ro # Configuration file
            - /var/lib/docker/containers:/var/lib/docker/containers:ro           # Docker logs
            - /var/run/docker.sock:/var/run/docker.sock:ro                       # Additional information about containers
        user: root                                                             # Allow access to log files and docker.sock
        restart: on-failure
```

- Add flags in other services to send logs to filebeat
  `label: collect_logs_with_filebeat: "true"`
  `label: decode_log_event_to_json_object: "true"`

- Reference - https://github.com/sfg-beer-works/sfg-brewery-beer-service/tree/master/src/main/docker

### View Logs in Kibana

- Run `compose-logging.yaml` file
- Go to - http://localhost:app/kibana#/home
- Discover
  - Setup Index Pattern - `@timestamp`
  - Hit Save button

### KEY TERMS

- Logstash
- Logstash Server
- Logstash is going to transform to a common format and then send that data on Elasticsearch

- Kibana
- Data visualization Tool

- Filebeat
- Moves log data to a destination
- Take the log data from a Client Machine and Move it to Logstash Server.

- Elasticsearch
- A Very Fast Search Engine
- Robust, widely used, highly scalable
