## Spring Cloud Gateway

- Spring Cloud Gateway is an API Gateway
- An API Gateway provides a common access point to obscure running location of application
- Typically used with micro services to expose a common URL and abstract where the service is running
- API Gateway acts as a proxy to direct traffic to and from running instances
- Spring Cloud Gateway used WebFlux
- Spring Cloud Gateway is part of a larger family of projects for Spring Cloud

### Requirement

- To commplete this section, you wil need running:
- Set PORT via `application.properties` - `server.port=<port-number>`

  - spring-6-restmvc, server port 8081, My SQL running
  - spring6-auth-server, server port 9000
  - spring6-reactive, server port 8082
  - spring-6-reactive-mongo, server port 8083, MongoDB running

- Now Run `spring-cloud-gateway` application on server port 8080

### Create Spring Cloud Gateway Project

- Create Project using Spring Initializer

  - Spring Cloud Routing - Gateway

- Set/ADD/Update Spring Cloud version in `POM.XML`

  - `<spring.cloud.version>2022.0.0</spring.cloud.version>`

- All the Spring Cloud Projects we need to bring in Cloud Dependencies

```
<dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

- Add Spring boot Maven Plugin

```

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

- Here is the complete POM.XML

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.0.1</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>guru.springframework</groupId>
    <artifactId>spring-6-gateway</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring-6-gateway</name>
    <description>spring-6-gateway</description>
    <properties>
        <java.version>17</java.version>
        <spring-cloud.version>2022.0.0</spring-cloud.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

### Server Port Mapping

- Add Port details in `application.properties`

  - `server.port=8081`

- On Mac, if you want to check the port details, if it is used, below is the command

  - `lsof -i  :<port-number>` ==> - `lsof -i  :8082`

- To get running process details, use `ps -ef` details
  - `ps -ef | grep <process-id>` ==> `ps -ef | grep 4068`

### v1 Route Mapping

- Add configuration in `application.properties` OR `application.yml`
- Add new yml file under `resources/application.yml`
- Setup the mapping to send traffic to `http://localhost:8081` if matches `/api/v1/*`

```
spring:
  cloud:
    gateway:
      routes:
        - id: spring-6-rest-mvc
          uri: http://localhost:8081
          predicates:
            - Path=/api/v1/*
```

### Troubleshooting Spring Cloud Gateway

- Configure `HttpServer` and `HttpClient` to wiretap the info going in and out of the server

```
      httpserver:
        wiretap: true
      httpclient:
        wiretap: true
```

- Add Logging level

```
logging:
  level:
    root: error
    reactor.netty: trace
    org.springframework.cloud.gateway: trace
    org.springframework.http.server.reactive: trace
```

- Here would be complete YML with all the route details:

```
spring:
  cloud:
    gateway:
      routes:
        - id: spring-6-rest-mvc
          uri: http://localhost:8081
          predicates:
            - Path=/api/v1/*
        - id: spring-6-reactive
          uri: http://localhost:8082
          predicates:
            - Path=/api/v2/*
        - id: spring-6-reactive-mongo
          uri: http://localhost:8083
          predicates:
            - Path=/api/v3/*
# ENABLE BELOW FOR TROUBLESHOOTING
#      httpserver:
#        wiretap: true
#      httpclient:
#        wiretap: true
#logging:
#  level:
#    root: error
#    reactor.netty: trace
#    org.springframework.cloud.gateway: trace
#    org.springframework.http.server.reactive: trace
```

### Gateway Resource Server Configuration

Add security configuration in Cloud API Gateway

- Add OAuth dependency in `pom.xml`

```
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

- Add config in yml

```
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:9000
```

- Add Spring Security Cofig

```
/**
 * Created by jt, Spring Framework Guru.
 */
@Configuration
@EnableWebFluxSecurity
public class SpringSecConfig {
    @Bean
    public SecurityWebFilterChain securityFilterChain(ServerHttpSecurity http){
        http.csrf().disable()
                .authorizeExchange()
                .anyExchange().authenticated()
                .and()
                .oauth2ResourceServer().jwt();

        return http.build();
    }
}
```

### KEY TERMS

- API Gateway acts as a proxy to direct traffic to and from running instances
- Spring Cloud Gateway is Non-Blocking as it's using webflux under the hood
