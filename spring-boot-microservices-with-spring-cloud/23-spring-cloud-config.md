## Spring Cloud Config

- Repo - https://github.com/bhopals/mssc-config-server
- Repo - https://github.com/bhopals/mssc-brewery-config-repo

- Spring Cloud Config provides externalized configuration for distributed environments
- Provides a RESTFul style API for Spring services to lookup configuration values
- Spring Boot applications on startup obtain configuration values from Spring Cloud
  Config server
- Properties can be global and application specific
- Properties can also be stored by Spring profiles
- Easily encrypt and decrypt properties

### Property Storage

- Spring Cloud Config provides a number of options for property storage:
  - Git (default) or SVN
  - File System
  - HashiCorp’s Vault
  - JDBC, Redis
  - AWS S3
  - CredHub

### Spring Cloud Config Client

- Spring Cloud Config Client by default will look for a URL property
  - ‘spring.cloud.config.url’ - default is http://localhost:8888
- If using discovery client, client will look for service called ‘configserver’
- Fail Fast - optionally configure client to fail with exception if config server cannot
  be reached

### Configuration Resources

- Resources served as: /application/profile/label
  - application = spring.application.name
  - profile = Spring active profile(s)
  - label = spring.cloud.config.label

### Brewery Services

- Spring Boot Micro Services

  - mssc-beer-service - 8080
  - mssc-beer-inventory-service - 8082
  - mssc-inventory-failover - 8083
  - mssc-beer-order-service - 8081

- Spring Cloud Services
  - mssc-brewery-gateway - 9090
  - mssc-brewery-eureka - 8761
  - mssc-config-server - 8888

### Create Config Server

- Use Spring Initialzr
- Spring Boot (Latest Release)
- Config Server
- Eureka Discovery Client

### Spring Cloud Config Server Server Configuration

- Annotate Main class with `@EnableConfigServer`

```
@EnableConfigServer
@SpringBootApplication
public class MsscConfigServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(MsscConfigServerApplication.class, args);
	}

}
```

- Add application properties using `application.properties`

```
server.port=8888
spring.application.name=mssc-brewery-config

spring.cloud.config.server.git.uri=https://github.com/bhopals/mssc-brewery-config-repo
spring.cloud.config.server.git.clone-on-start=true
spring.cloud.config.server.git.search-paths={application}
logging.level.org.springframework.cloud=debug
logging.level.org.springframework.web=debug
```

### Server Side Application Configuration

- Adding Centralized configuration of each application
- Create Directories for each of the Spring Boot Microservices
- Add Configuration for active profiles

### Spring Cloud Config RESTFul Endpoints

- Configuration Endpoints available endpoints:

  - /{application}/{profile}[/{lable}]
  - /{application}-{profile}.yml
  - {lable}/{application}-{profile}.yml
  - /{application}-{profile}.properties
  - /{lable}/{application}-{profile}.properties

- Repo - https://github.com/bhopals/mssc-brewery-config-repo

- Lookup

  - `spring.cloud.config.server.git.search-paths={application}`
  - So what's going to happen here is we're telling the Spring Cloud Config to look in a folder of the
    application named for configuration for that application.

  - Example
    `http://localhost:8080/conf/beer-service/local` => returns `application-local.properties` file details
    as JSON response.

### Spring Clound Client Configuration

- Configure Spring Cloud Config Discovery Client

- Add spring cloud starter cloud config dependency

```
 <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

So this is setting up this spring cloud config client. So that is what we need to use connect now.
That's where we're going to get into some more advanced things. The way with spring cloud config
is gonna work it uses a bootstrap. So it actually has like a two phase configuration step. It's going to start up, that's going to look at a `bootstrap.properties` file to find the environment and then it's going to go out and get
that and then go through all the normal stuff.

```
spring.cloud.discovery.enabled=true
spring.cloud.config.discovery.service-id=mssc-brewery-config
spring.cloud.config.fail-fast=true
```

### KEY TERMS

- Externalized Configuration in Distributed Environment

- Spring Boot Microservices
- Spring Cloud Services

- Spring Cloud Config Server
- Spring Cloud Config Repo

- Spring Cloud Configuration
