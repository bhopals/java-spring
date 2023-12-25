## Spring Boot Microservices on Kubernetes

- `kubectl get nodes`
- `kubectl config get-context`
- `kubectl config use-context docker-desktop`

### Introduction to Spring Boot Microservices on Kubernetes

- 3 Spring boot Micro Services

  - 1. `Beer Inventory Service` - Holds information about inventory levels

    - Rest API used to get inventory information
    - JMS messaging used to increase inventoryu from brewing, decrease inventory from order allocation

  - 2. `Beer Service` - Holds information about beers brewed, Rest API for Beer CRUD Operations

    - JMS messaging to `brew` beer when invetory is low
    - Scheduled job to `brew` beer

  - 3. `Beer Order Service` - Holds information about beer Orders
    - Rest API for Beer Order Operations
    - JMS Messaging to `allocate` beer orders
    - Tasting Room Service - Scheduled job to `order` beers, which decreases inventory, which triggers brewing

- Important to understand `Beer Order Service` will generate orders, request allocations from
  `Beer Inventory Service` (reducing inventory), which triggers `Beer Service` to `brew` beer (adding to the inventory)

- `Inventory Failover Service` - Service to provide a positive responce if Inventory Service is not available.

  - Always returns an on-hand quantity of 999.

- Original Design
  - `Spring Cloud Gateway` in front of the services
  - `Eureka` for service discovery,
  - `Spring Cloud Config` config for configuration services.

And then we can see that the three services are deployed, each with their own MySQL backend and then

- Changes for Kubernetes
  - `Eureka` - is not used, Kubernetes us used for service discovery
  - `Spring Cloud Config` - is not used, Kubernetes used to manage environment properties
  - `Single MySql Instance` - One Instance used. Just for simplicity, a production deployment should
    have independent MySQL database instances
  - `Single GitHub Repo` - Again for simplicity. Microservices typically would have independent source
    code repositories

### Source Code Review

- Repo - https://github.com/springframeworkguru/kbe-sb-microservices

### Running Services via Docker Compose

- Run `compose-local.yaml`

  - `docker compose -f compose-local-yaml up -d` (`-d` for Detached mode)

  - At the end, everthing is up and running
  - Check by running `docker ps`

- Kibana init

  - `http://localhost:5601/app/home`
  - Create Index Pattern

    - Select "filebeat\*" (`Filebeat`: Lightweight Log Analysis & Elasticsearch | Elastic)
    - In Configure Settings, select `@timestamp`
    - Click on "Create Index Pattern"

    - Go to Home page, select "Discover", you should be able to see all the logs
    - So effectively, we have the three microservices running in independent containers. Filebeat is
      monitoring each container, the council logs and shipping those off to the last database.

- Shut Down all the services

  - `docker compose -f compose-local-yaml down`

- YML Source code

```
version: '3.8'
services:
    mysql:
        image: mysql:5.7
        environment:
            MYSQL_ROOT_PASSWORD: dbpassword
            MYSQL_DATABASE: beerservice
    elasticsearch:
        image: docker.elastic.co/elasticsearch/elasticsearch:7.12.1
        ports:
            - 9200:9200
        environment:
            discovery.type: single-node
    kibana:
        image: docker.elastic.co/kibana/kibana:7.12.1
        ports:
            - 5601:5601
        restart: on-failure
        depends_on:
            - elasticsearch
    filebeat:
        image: docker.elastic.co/beats/filebeat:7.12.1
        volumes:
            - ./filebeat/filebeat.docker.yml:/usr/share/filebeat/filebeat.yml:ro # Configuration file
            - /var/lib/docker/containers:/var/lib/docker/containers:ro           # Docker logs
            - /var/run/docker.sock:/var/run/docker.sock:ro                       # Additional information about containers
        user: root                                                             # Allow access to log files and docker.sock
        restart: on-failure
    jms:
        image: vromero/activemq-artemis
        ports:
            - 8161:8161
            - 61616:61616
    inventory-service:
        image: springframeworkguru/kbe-brewery-inventory-service
        ports:
            - 8082:8082
        depends_on:
            - jms
        environment:
            SPRING_DATASOURCE_USER: root
            SPRING_DATASOURCE_PASSWORD: dbpassword
            SPRING_JPA_HIBERNATE_DDL-AUTO: update
            SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/beerservice?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
            SPRING_ARTEMIS_HOST: jms
        restart: on-failure
        labels:
            collect_logs_with_filebeat: "true"
            decode_log_event_to_json_object: "true"
    inventory-failover:
        image: springframeworkguru/kbe-brewery-inventory-failover
        ports:
            - 8083:8083
    beer-service:
        image: springframeworkguru/kbe-brewery-beer-service
        ports:
            - 8080:8080
        depends_on:
            - inventory
            - jms
        restart: on-failure
        environment:
            SPRING_DATASOURCE_USER: root
            SPRING_DATASOURCE_PASSWORD: dbpassword
            SPRING_JPA_HIBERNATE_DDL-AUTO: update
            SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/beerservice?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
            SPRING_ARTEMIS_HOST: jms
            SFG_BREWERY_BEER-INVENTORY-SERVICE-HOST: http://inventory-service:8082
        labels:
            collect_logs_with_filebeat: "true"
            decode_log_event_to_json_object: "true"
    order-service:
        image: springframeworkguru/kbe-brewery-order-service
        ports:
            - 8081:8081
        depends_on:
            - beer-service
            - jms
        restart: on-failure
        environment:
            SPRING_DATASOURCE_USER: root
            SPRING_DATASOURCE_PASSWORD: dbpassword
            SPRING_JPA_HIBERNATE_DDL-AUTO: update
            SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/beerservice?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
            SPRING_ARTEMIS_HOST: jms
            SFG_BREWERY_BEER-SERVICE-HOST: http://beer-service:8080
        labels:
            collect_logs_with_filebeat: "true"
            decode_log_event_to_json_object: "true"
    gateway:
        image: springframeworkguru/kbe-brewery-gateway
        ports:
            - 9090:9090
        depends_on:
            - inventory
            - beer-service
            - order-service
            - inventory-failover
        restart: on-failure
        labels:
            collect_logs_with_filebeat: "true"
            decode_log_event_to_json_object: "true"
```

### Infrastructure Services

- MySQL
- JMS

#### MySql Service

- In the above YML config, we are defining MySQL Service

```
mysql:
        image: mysql:5.7
        environment:
            MYSQL_ROOT_PASSWORD: dbpassword
            MYSQL_DATABASE: beerservice
```

- Then we are setting up two environment properties. So we need to replicate this in a Kubernetes context.

- First, Kubernete `Deployment` (pod)

  - Go to terminal
  - Run `kubectl create deployment mysql --image=mysql:5.7 --dry-run=cilent -o=yaml > mysql-development.yml`
  - `vi mysql-development.yml`
  - Add properties under `spec container image`
    - `name: MYSQL_ROOT_PASSWORD`
      `value: dbpassword`
    - `name: MYSQL_DATABASE`
      `value: beerservice`
  - Save `deployment.yml`

  - Reapply the change
    - `kubectl apply -f mysql-deployment.yml`
  - `kubectl get all`

- Second, Kubernete `Service` (Exposed outside the cluster)

  - `kubectl create service clusterip mysql --tcp=3306:3306 --dry-run=cilent -o=yaml > mysql-service.yml`
  - `kubectl apply -f mysql-service.yml`
  - `kubectl get all`

- Files
  - `mysql-deployment.yml`
  - `mysql-service.yml`

#### JMS Service

- In the YML config, we are defining JMS Service

```
jms:
        image: vromero/activemq-artemis
        ports:
            - 8161:8161
            - 61616:61616
```

- First, Kubernete `Deployment` (pod)

  - Go to terminal
  - Run `kubectl create deployment jms --image=vromero/activemq-artemis --dry-run=cilent -o=yaml > jms-development.yml`

  - Reapply the change
    - `kubectl apply -f jms-development.yml`
  - `kubectl get all`

- Second, Kubernete `Service` (Exposed outside the cluster)

  - `kubectl create service clusterip jms --tcp=8161:8161 --61616:61616 --dry-run=cilent -o=yaml > jms-service.yml`
  - `kubectl apply -f jms-service.yml`
  - `kubectl get all`

  - `docker ps`

- Files
  - `jms-deployment.yml`
  - `jms-service.yml`

### Intro to Spring Boot Microservices

- Create deployments
- Create Services
- Edit the YAML for environment specific properties
- Set up Readiness and Liveness probe on Each Service
- Graceful Shutdown
- Set up Spring Cloud Gateway to access Services

#### Inventory Service

- YML Configuration from `compose-local.yml`

```
inventory-service:
    image: springframeworkguru/kbe-brewery-inventory-service
    ports:
        - 8082:8082
    depends_on:
        - jms
    environment:
        SPRING_DATASOURCE_USER: root
        SPRING_DATASOURCE_PASSWORD: dbpassword
        SPRING_JPA_HIBERNATE_DDL-AUTO: update
        SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/beerservice?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
        SPRING_ARTEMIS_HOST: jms
    restart: on-failure
    labels:
        collect_logs_with_filebeat: "true"
        decode_log_event_to_json_object: "true"
```

- First, Kubernete `Deployment` (pod)

  - Go to terminal
  - Run `kubectl create deployment inventory-service --image=springframeworkguru/kbe-brewery-inventory-service --dry-run=cilent -o=yaml > inventory-development.yml`
  - `vi inventory-development.yml`
  - Add properties under `spec container env`
    - `name: SPRING_DATASOURCE_USER`
      `value: root`
    - `name: SPRING_DATASOURCE_PASSWORD`
      `value: dbpassword`
    - `name:  SPRING_JPA_HIBERNATE_DDL-AUTO`
      `value: update`
    - `name: SPRING_DATASOURCE_URL`
      `value: jdbc:mysql://mysql:3306/beerservice?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC`
    - `name: SPRING_ARTEMIS_HOST`
      `value: jms`
    - `name: MANAGEMENT_ENDPOINT_HEALTH_PROBES_ENABLED`
      `value: "true"`
    - `name: MANAGEMENT_HEALTH_READINESSTATE_ENABLED`
      `value: "true"`
    - `name: MANAGEMENT_HEALTH_LIVENESSSTATE_ENABLED`
      `value: "true"`
    - `name: SERVER_SHUTDOWN`
      `value: "graceful"`
      readinessProbe:
      httpGet:
      port: 8080
      path: /actuator/health/readiness
      livenessProbe:
      httpGet:
      port: 8080
      path: /actuator/health/liveness
      lifecycle:
      preStop:
      exec:
      command: ["sh", "-c", "sleep 10"]
  - Save `inventory-deployment.yml`

  - Reapply the change - `kubectl apply -f inventory-deployment.yml`
  - `kubectl get all`

- Second, Kubernete `Service` (Exposed outside the cluster)

  - `kubectl create service clusterip inventory-service --tcp=8082:8082 --dry-run=cilent -o=yaml > jms-service.yml`
  - `kubectl apply -f inventory-service.yml`
  - `kubectl get all`

  - `docker ps`

- Files
  - `inventory-deployment.yml`
  - `inventory-service.yml`

#### Inventory Failover Service

- YML Configuration from `compose-local.yml`

```
inventory-failover:
        image: springframeworkguru/kbe-brewery-inventory-failover
        ports:
            - 8083:8083
```

- First, Kubernete `Deployment` (pod)

  - Go to terminal
  - Run `kubectl create deployment inventory-failover --image=springframeworkguru/kbe-brewery-inventory-failover --dry-run=cilent -o=yaml > inventory-failover-deployment.yml`

  - Reapply the change - `kubectl apply -f inventory-failover-deployment.yml`
  - `kubectl get all`

- Second, Kubernete `Service` (Exposed outside the cluster)

  - `kubectl create service clusterip inventory-failover-service --tcp=8083:8083 --dry-run=cilent -o=yaml > inventory-failover-service.yml`
  - `kubectl apply -f inventory-failover-service.yml`
  - `kubectl get all`

  - `docker ps`

- Files
  - `inventory-failover-deployment.yml`
  - `inventory-failover-service.yml`

#### Beer Service

- YML Configuration from `compose-local.yml`

```
beer-service:
        image: springframeworkguru/kbe-brewery-beer-service
        ports:
            - 8080:8080
        depends_on:
            - inventory
            - jms
        restart: on-failure
        environment:
            SPRING_DATASOURCE_USER: root
            SPRING_DATASOURCE_PASSWORD: dbpassword
            SPRING_JPA_HIBERNATE_DDL-AUTO: update
            SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/beerservice?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
            SPRING_ARTEMIS_HOST: jms
            SFG_BREWERY_BEER-INVENTORY-SERVICE-HOST: http://inventory-service:8082
        labels:
            collect_logs_with_filebeat: "true"
            decode_log_event_to_json_object: "true"
```

- First, Kubernete `Deployment` (pod)

  - Go to terminal
  - Run `kubectl create deployment beer-service --image=springframeworkguru/kbe-brewery-beer-service --dry-run=cilent -o=yaml > beer-service-deployment.yml`

  - `vi beer-service-deployment.yml`
  - Add properties under `spec container env`

        - `name: SPRING_DATASOURCE_USER`
            `value: root`
        - `name: SPRING_DATASOURCE_PASSWORD`
            `value: dbpassword`
        - `name: SPRING_JPA_HIBERNATE_DDL-AUTO`
            `value: update`
        - `name: SPRING_DATASOURCE_URL`
            `value: jdbc:mysql://mysql:3306/beerservice?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC`
        - `name: SPRING_ARTEMIS_HOST`
            `value: jms`
        - `name: SFG_BREWERY_BEER-INVENTORY-SERVICE-HOST`
            `value: http://inventory-service:8082`
        - `name: MANAGEMENT_ENDPOINT_HEALTH_PROBES_ENABLED`
            `value: "true"`
        - `name: MANAGEMENT_HEALTH_READINESSTATE_ENABLED`
            `value: "true"`
        - `name: MANAGEMENT_HEALTH_LIVENESSSTATE_ENABLED`
            `value: "true"`
        - `name: SERVER_SHUTDOWN`
            `value: "graceful"`
        readinessProbe:
          httpGet:
            port: 8080
            path: /actuator/health/readiness
        livenessProbe:
          httpGet:
            port: 8080
            path: /actuator/health/liveness
        lifecycle:
          preStop:
            exec:
              command: ["sh", "-c", "sleep 10"]

  - Reapply the change - `kubectl apply -f beer-service-deployment.yml`
  - `kubectl get all`

- Second, Kubernete `Service` (Exposed outside the cluster)

  - `kubectl create service clusterip  beer-service --tcp=8080:8080 --dry-run=cilent -o=yaml > beer-service.yml`
  - `kubectl apply -f beer-service.yml`
  - `kubectl get all`

  - `docker ps`

- Files
  - `beer-service-deployment.yml`
  - `beer-service.yml`

#### Order Service

- YML Configuration from `compose-local.yml`

```
order-service:
        image: springframeworkguru/kbe-brewery-order-service
        ports:
            - 8081:8081
        depends_on:
            - beer-service
            - jms
        restart: on-failure
        environment:
            SPRING_DATASOURCE_USER: root
            SPRING_DATASOURCE_PASSWORD: dbpassword
            SPRING_JPA_HIBERNATE_DDL-AUTO: update
            SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/beerservice?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
            SPRING_ARTEMIS_HOST: jms
            SFG_BREWERY_BEER-SERVICE-HOST: http://beer-service:8080
        labels:
            collect_logs_with_filebeat: "true"
            decode_log_event_to_json_object: "true"
```

- First, Kubernete `Deployment` (pod)

  - Go to terminal
  - Run `kubectl create deployment order-service --image=springframeworkguru/kbe-brewery-order-service --dry-run=cilent -o=yaml > order-service-deployment.yml`

  - `vi order-service-deployment.yml`
  - Add properties under `spec container env`

    - `name: SPRING_DATASOURCE_USER`
      `value: root`
    - `name: SPRING_DATASOURCE_PASSWORD`
      `value: dbpassword`
    - `name: SPRING_JPA_HIBERNATE_DDL-AUTO`
      `value: update`
    - `name: SPRING_DATASOURCE_URL`
      `value: jdbc:mysql://mysql:3306/beerservice?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC`
    - `name: SPRING_ARTEMIS_HOST`
      `value: jms`
    - `name: SFG_BREWERY_BEER-SERVICE-HOST`
      `value: http://beer-service:8080`
    - `name: MANAGEMENT_ENDPOINT_HEALTH_PROBES_ENABLED`
      `value: "true"`
    - `name: MANAGEMENT_HEALTH_READINESSTATE_ENABLED`
      `value: "true"`
    - `name: MANAGEMENT_HEALTH_LIVENESSSTATE_ENABLED`
      `value: "true"`
    - `name: SERVER_SHUTDOWN`
      `value: "graceful"`
      `readinessProbe:`
      `   httpGet:`
      `      port: 8080`
      `      path: /actuator/health/readiness`
      `livenessProbe:`
      `   httpGet:`
      `      port: 8080`
      `      path: /actuator/health/liveness`
      `lifecycle:`
      `   preStop:`
      `      exec:`
      `          command: ["sh", "-c", "sleep 10"]`

  - Reapply the change - `kubectl apply -f order-service-deployment.yml`
  - `kubectl get all`

- Second, Kubernete `Service` (Exposed outside the cluster)

  - `kubectl create service clusterip  order-service --tcp=8080:8080 --dry-run=cilent -o=yaml > order-service.yml`
  - `kubectl apply -f order-service.yml`
  - `kubectl get all`

  - `docker ps`

- Files
  - `order-service-deployment.yml`
  - `order-service.yml`

### Add Readiness and Liveness Probe Configuration

- Refer Above Update configuration for all 4 services

### Add Graceful Shutdown

- Refer Above Update configuration for all 4 services

### Kubernetes Ingress Controllers

- Refer - https://kubernetes.io/docs/concepts/services-networking/ingress/
- So ingress is how we get in and out of the cluster as far as in a networking perspective.
- Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster.
  Traffic routing is controlled by rules defined on the Ingress resource.

- A Cluster is going to be a set of Nodes

- The Edge router, again, that's going to be some type of device that could be routing traffic into

- Now, some type of endpoint, that ingress endpoint that is going to be coming into the cluster.

- Ingress Controller - https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/

- We are going to set up `Spring Cloud Gateway` as our ingress controller
- However, there are lots of different ways to do it (See above link for the potential controllers list)

### Spring Cloud Gateway Service

- We are going to use a spring cloud gateway instance as a service and deployment inside of Kubernetes.
- Again, this is going to be mimicking some type of Kubernetes ingress.

- Below is the Spring Cloud Gateway Configuration

```
@SpringBootApplication
public class SfgBreweryGatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(SfgBreweryGatewayApplication.class, args);
    }
    @Bean
    public RouteLocator gatewayRoutes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("beer-service", r -> r.path("/api/v1/beer/*", "/api/v1/beerUpc/*")
                .uri("http://beer-service:8080"))
        .route("inventory-service", r -> r.path("/api/v1/beer/*/inventory")
                .uri("http://inventory-service:8082"))
        .route("order-service", r -> r.path(("/api/v1/customers/**"))
                .uri("http://order-service:8081"))
        .build();
    }
}
```

- From `docker-compose.yml`

```
 gateway:
        image: springframeworkguru/kbe-brewery-gateway
        ports:
            - 9090:9090
        depends_on:
            - inventory
            - beer-service
            - order-service
            - inventory-failover
        restart: on-failure
        labels:
            collect_logs_with_filebeat: "true"
            decode_log_event_to_json_object: "true"
```

- First, Kubernete `Deployment` (pod)

  - Go to terminal
  - Run `kubectl create deployment gateway --image=springframeworkguru/kbe-brewery-gateway --dry-run=cilent -o=yaml > gateway-deployment.yml`

  - Reapply the change - `kubectl apply -f gateway-deployment.yml`
  - `kubectl get all`

- Second, Kubernete `Service` (Exposed outside the cluster)

  - `kubectl create service nodeport gateway-service --tcp=9090:9090 --dry-run=cilent -o=yaml > gateway-service.yml`
  - `kubectl apply -f gateway-service.yml`
  - `kubectl get all`

  - `docker ps`

- Files
  - `gateway-deployment.yml`
  - `gateway-service.yml`

### Deleting Services and Deployments

- `kubectl get all`
- `kubectl delete -f gateway-service.yml` (Delete Service)
- `kubectl delete -f gateway-deployment.yml` (Delete Deployment)

* Similarly we can do for all the desired services and deployments.

### Introduction to Consolidated Logging

Now, the problem of running in a distributed environment. We have logs running in these containers all over the place, which if you're troubleshooting something that can be become very difficult, especially if you're going from multiple services. So what we are going to do is we want to take all that log data, consolidate it and get it up to `Elasticsearch`, where it can be searched using `Kibana`.

`Kibana` is a great graphical user face to look at data inside of `Elasticsearch`, so the two work really
well. Together we will be using `File beat`, which is a very efficient tool to go and grab the log data to
get that in. So a lot of moving parts here and let me prefix this, there's a lot of different ways to do consolidated logging.

The `Elasticsearch`, `File beat` and `Cabana` are very popular.

We will also be utilizing `Spring Cloud Sleuth`, which will give us a trace ID that if we have a request
going across multiple services, Spring Cloud Sleuth enables that.
So for a spring context allows us to trace hops across multiple services.

### Logging Configuration Code Review

- Logging Configuration for Consolidated Logging

- Logstash and Sleuth dependency

```
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>${logstash.logback.version}</version>
</dependency>

<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-sleuth</artifactId>
        </dependency>
```

- Logstash: Collect, Parse, Transform Logs - Elastic
- It takes the Log Data and creates JSON output.

- Spring Cloud Sleuth. Sleuth is a project that is going to add tracing elements so you'll
  get a correlation ID pass through. So if service A call service B Spring Cloud Sleuth is going to provide data in those logging calls so that we can see a trace ID to help us trace a call through multiple services.

- `logback-spring.xml` - Log and Sleuth configuration
- So this is the primary piece that is going to be taking the log data and making a JSON object out of it.

- And that is important because we're going to be posting JSON to the console and then file beat is going
  to be picking up the counsel logs and shipping those to Elasticsearch.

### Elastic Search

- So Elasticsearch is the database that is going to be holding our consolidated log data.

- Referring `compose-local.yml`

```
 elasticsearch:
        image: docker.elastic.co/elasticsearch/elasticsearch:7.12.1
        ports:
            - 9200:9200
        environment:
            discovery.type: single-node
```

- First, Kubernete `Deployment` (pod)

  - Go to terminal
  - Run `kubectl create deployment elasticsearch --image=docker.elastic.co/elasticsearch/elasticsearch:7.12.1 --dry-run=cilent -o=yaml > elasticsearch-deployment.yml`
  - `vi elasticsearch-deployment.yml`
  - Update Property
    `name:discovery.type`
    `value:single-node`
  - Reapply the change - `kubectl apply -f elasticsearch-deployment.yml`
  - `kubectl get all`

- Second, Kubernete `Service` (Exposed outside the cluster)

  - `kubectl create service clusterid elasticsearch --tcp=9200:9200 --dry-run=cilent -o=yaml > elasticsearch-service.yml`
  - `kubectl apply -f elasticsearch-service.yml`
  - `kubectl get all`
  - `docker ps`

- Files
  - `elasticsearch-deployment.yml`
  - `elasticsearch-service.yml`

### Kibana

- Referring `compose-local.yml`

```
kibana:
        image: docker.elastic.co/kibana/kibana:7.12.1
        ports:
            - 5601:5601
        restart: on-failure
        depends_on:
            - elasticsearch
```

- First, Kubernete `Deployment` (pod)

  - Go to terminal
  - Run `kubectl create deployment kibana --image=docker.elastic.co/kibana/kibana:7.12.1 --dry-run=cilent -o=yaml > kibana-deployment.yml`
  - Apply the change - `kubectl apply -f elasticsearch-deployment.yml`
  - `kubectl get all`

- Second, Kubernete `Service` (Exposed outside the cluster)

  - `kubectl create service nodeport kibana --tcp=5601:5601 --dry-run=cilent -o=yaml > kibana-service.yml`
  - `kubectl apply -f kibana-service.yml`
  - `kubectl get all`
  - `docker ps`

- Files
  - `kibana-deployment.yml`
  - `kibana-service.yml`

### Filebeat

- Setting up Filebeat to run on Kubernetes.
- Refer - https://www.elastic.co/guide/en/beats/filebeat/current/running-on-kubernetes.html

### KEY TERMS

- Service - (Exposed outside the cluster)
- Deployment - (Pod)

- YAML - Yet Another Markup Language

- Ingress
- Edge Router - Routing traffic into your Kubernetes Cluster
- Node
- Cluster - Set of Nodes
- Ingress-managed Load Balancer
- Ingress Controller

- ClusterIP is used for Pod-to-Pod communication within the same Kubernetes cluster.
- In contrast, NodePort and LoadBalancer Services are used for communication between applications
  within the cluster and external clients outside the cluster

- Elastic Search
- Kiabana
- Filebeat
- Spring Cloud Sleuth

- Consolidate Logging

- Nodeport v/s ClusterIp
