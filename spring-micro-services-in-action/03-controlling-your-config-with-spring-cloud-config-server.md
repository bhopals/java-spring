## Controlling your configuration with Spring Cloud Configuration Server

- Separate configuration information from the code.
- The loading of configuration management for a microservice occurs during the bootstrapping
  phase of the microservice

Application configuration data written directly into the code is often problematic because
every time a change to the configuration has to be made the application has to be recompiled and/or redeployed.

To avoid this, developers will separate the configuration information from the application code completely. This makes it easy to make changes to configuration without going through a
recompile process, but also introduces complexity because you now have another
artifact that needs to be managed and deployed with the application.

Many developers will turn to the lowly property file (or YAML, JSON, or XML) to
store their configuration information. This property file will sit out on a server often
containing database and middleware connection information and metadata about the
application that will drive the application’s behavior. Segregating your application
into a property file is easy and most developers never do any more operationalization
of their application configuration then placing their configuration file under source
control (if that) and deploying it as part of their application.
This approach might work with a small number of applications, but it quickly falls
apart when dealing with cloud-based applications that may contain hundreds of
microservices, where each microservice in turn might have multiple service instances
running.

Suddenly configuration management becomes a big deal as application and operations team in a cloud-based environment have to wrestle with a rat’s nest of which configuration files go where. Cloud-based microservices development emphasizes

- 1. Completely separating the configuration of an application from the actual code being deployed

- 2. Building the server and the application and an immutable image that never changes as
     it’s promoted through your environments

- 3. Injecting any application configuration information at startup time of the server
     through either environment variables or through a centralized repository the application’s microservices read on startup

Let’s begin our discussion about application configuration management by establishing four principles we want to follow:

- Segregate
- Abstract
- Centralize
- Harden

### Building our Spring Cloud configuration server

The Spring Cloud configuration server is a REST-based application that’s built on top
of Spring Boot. It doesn’t come as a standalone server. Instead, you can choose to
either embed it in an existing Spring Boot application or start a new Spring Boot project with the server embedded it.

- `@EnableConfigServer` annotation start this as Config server running on the Port you specified
  in `licensingservice-dev.properties`.

- It can be queries using `http://localhost:<port>licensingservice/dev`

- Spring framework implements a hierarchical mechanism for resolving properties.

```
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
 public static void main(String[] args) {
 SpringApplication.run(ConfigServerApplication.class, args);
 }
}
```

### Configuring the licensing service to use Spring Cloud Config

- To access, above configuration from Licensing Service we need to tell the licensing service
  where to contact the Spring Cloud configuration server.

- In a Spring Boot service that uses Spring Cloud Config, configuration information can be
  set in one of two configuration files: `bootstrap.yml` and `application.yml`.

- The `bootstrap.yml` file reads the application properties before any other configuration
  information used. In general, the bootstrap.yml file contains the application name for the service, the application profile, and the URI to connect to a Spring Cloud Config server.

```
spring.application.name=licensingservice
spring.profiles.active=default
spring.cloud.config.uri=http://localhost:888
```

- Any other configuration information that you want to keep local
  to the service (and not stored in Spring Cloud Config) can be set locally in the services in the application.yml file

### Using Spring Cloud configuration server with Git

- Spring Cloud configuration server with a Git source control repository.

```
spring.cloud.config.server.git.uri=https://github.com/carnellj/config-repo/
spring.cloud.config.server.git.searchPaths: licensingservice,organizationservice
spring.cloud.config.server.git.username: native-cloud-apps
spring.cloud.config.server.git.password: 0ffended
```

#### Refreshing your properties using Spring Cloud configuration server

Spring Boot applications will only read their properties at startup time,
so property changes made in the Spring Cloud configuration server won’t be automatically picked up by the Spring Boot application. Spring Boot Actuator does offer a `@RefreshScope` annotation that will allow a development team to access a /refresh endpoint that will force the Spring Boot application to reread its application configuration. The following listing shows the @RefreshScope annotation in action.

```
@SpringBootApplication
@RefreshScope
public class Application {
  public static void main(String[] args) {
   SpringApplication.run(Application.class, args);
 }
}
```

- To perform the refresh, you can hit the `http://<yourserver>:8080/refresh` endpoint.

##### On refreshing microservices

When using Spring Cloud configuration service with microservices, one thing you
need to consider before you dynamically change properties is that you might have
multiple instances of the same service running, and you’ll need to refresh all of those
services with their new application configurations. There are several ways you can
approach this problem:

Spring Cloud configuration service does offer a `push` -based mechanism called
Spring Cloud Bus that will allow the Spring Cloud configuration server to publish to all
the clients using the service that a change has occurred. Spring Cloud configuration
requires an extra piece of middleware running (RabbitMQ). This is an extremely useful
means of detecting changes, but not all Spring Cloud configuration backends support
the “push” mechanism (that is, the Consul server).

#### Protecting sensitive configuration information

By default, Spring Cloud configuration server stores all properties in plain text within
the application’s configuration files. This includes sensitive information such as database credentials.

- JCE Jars (JCE - Java Cryptography Extension)

- `http://localhost:8888/encrypt` - To encrypt
- `http://localhost:8888/decrypt` - To decrypt

##### Configuration

- Spring Cloud Configuration will store encrypted properties as:
  - {cipher}<your encrypted value here>
- When a Spring Cloud Config client requests an encrypted property the value is
  decrypted and presented to the client in the request
- Must set a symmetric key in property ‘encrypt.key’ - should prefer setting this
  as environment variable
- asymmetric (public / private) keys also supported - see documentation for details

##### Encryption / Decryption

- Spring Cloud Configuration provides endpoints for property encryption / decryption

  - POST /encrypt - will encrypt body of post
  - POST /decrypt - will decrypt body of post

- Configuration
  - Add encrypt key in bootstrap properties of Cloud Config project
    // should be env. variable
    `encrypt.key=MySuperSecretKey`
  - Encrpty - `POST http://localhost:8888/encrypt` - Add Body params that we want to Encrypt
  - Decrpty - `POST http://localhost:8888/decrypt` - Add Body params that we want to Decrypt

##### Encrypt Beer Service Passwords

- In application properties, we have credentials for JMS (artemis) and SQL

```
spring.artemis.user=artemis
spring.artemis.password=simetraehcapa

spring.datasource.username=beer_service
spring.datasource.password=password
```

- Create `application-local-secure.properties` in Config Repo (Centralized Confi Repo)
- Encrpty Password using - `POST http://localhost:8888/encrypt`

```
spring.artemis.user=artemis
spring.artemis.password={cipher}<encrytped-password>

spring.datasource.username=beer_service
spring.datasource.password={cipher}<encrytped-password>
```

- Query Config server to retrieve all properties
  - GET `http://localhost:8888/beer-service/local-secure`
  - You will notice, the returned credentials are decrypted (because of `{cipher}`)

#### Configure microservices to use encryption on the client side

- In above case, the configuration server is handling Enryption and Decryption.
- To enable encryption/decryption at client side:
  - Disable decryption at Config side - `spring.cloud.config.server.encrypt.enabled: false`
  - Add security (encryption/decryption) jars on Client side
  ```
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-rsa</artifactId>
    </dependency>
  ```
  - Now if you hit config server - `http://localhost:8888/licensingservice/dev`, you should
    see crendetials sent are still encrypted, the onus is now on the client to decrypt it.

#### Closing thoughts

Application configuration management might seem like a mundane topic, but it’s of
critical importance in a cloud-based environment.

With a cloud-based model, the application configuration data should be segregated completely from the application, with the appropriate configuration data needs injected at runtime so that the same
server/application artifact are consistently promoted through all environments.

### Summary

- Spring Cloud configuration server allows you to set up application properties with
  environment specific values.

- Spring uses Spring profiles to launch a service to determine what environment properties
  are to be retrieved from the Spring Cloud Config service.

- Spring Cloud configuration service can use a file-based or Git-based application
  configuration repository to store application properties.

- Spring Cloud configuration service allows you to encrypt sensitive property files
  using symmetric and asymmetric encryption.

### KEY TERMS

- Instance needs to be launched with minimal human intervention
- Implement Configuration Management Solution
- Spring Cloud Configuration parent BOM (Bill of Materials)
- Spring Cloud Bootstrap class
- Spring Cloud Config Bootstrap class

- Spring Boot Actuator does offer a `@RefreshScope` annotation that will allow development
  team to access a /refresh endpoint

- Spring Cloud Bus
- Application Configuration Management
- Application Configuration Management is critically important in a cloud-based environment.
