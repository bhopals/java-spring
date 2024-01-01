## Spring Registration with Eureka

- Eureka - Service Discovery and Registration Service
- More - https://spring.io/guides/gs/service-registration-and-discovery/

### Netflix Eureka

- Netflix Eureka is a Service Discovery and Registration service
  - When a microservice instance starts, it registers itself with the Eureka service
    - Provides host name / IP, port, and service name
    - Known as Service Registration
- Service Discovery is the process of discovering the available service instances

### Service Registration

- Spring Boot provides a Spring Boot Starter for a Eureka Server
- Spring Boot also provides a starter for Eureka clients
  - Both will self configure for localhost operations
- Important to configure service name in application.properties
  - This is the value used to lookup the service in Eureka

### Service Discovery

- Spring Cloud Open Feign allows for easy service discovery between microservices
- Works in connection with Eureka and Ribbon
- Spring Cloud Gateway can be configured to lookup services in Eureka
- Works in conjunction with Ribbon to load balance requests

### Set Steps

- Create Eureka Server
- Configure Services to Register with Eureka
- Create OpenFeign Service for Inventory Requests
- Configure Spring Cloud Gateway to use Eureka for Service Discovery

### Eurka Server Creation

- Create a Spring boot project with Spring Cloud Eureka Dependency

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

- Annotate main starter class with `@EnableEurekaServer`

```
@EnableEurekaServer
@SpringBootApplication
public class MsscBreweryEurekaApplication {
	public static void main(String[] args) {
		SpringApplication.run(MsscBreweryEurekaApplication.class, args);
	}
}
```

- Add configuration details in application property file

```
server.port=8761
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false // Enable if it's in cluster with eureka server (Mutliple Eurekas)
logging.level.com.netflix.eureka=OFF
logging.level.com.netflix.discovery=OFF
```

### Eureka Client Configuration for Beer Service

- Add Eureka Client dependency

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

- Add application name in `application.properties`

  - `spring.application.name=beer-service`

- Add Configuraton for Discovery

```
@Profile("local-discovery")
@EnableDiscoveryClient
@Configuration
public class LocalDiscovery {
}
```

### KEY TERMS

- Netflix Eureka - Service Discovery and Registration Service
- Service Registry
- Service Discovery

- Service Node(s)
  - Register to Eureka on Startup (Registration Process: Service Node ===> EUREKA)
- API Gateway
  - Query Eureka to know Service Details (Discovery Process: API Gateway ===> EUREKA)
- Eureka

- Ribbon - Software LoadBalancer

- Spring Cloud Feign - Service Discovery between Microservices
