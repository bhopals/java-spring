## OpenAPI with Spring Boot

- OpenAPI is a specification standard to describe RESTful APIs

### What is OpenAPI?

- OpenAPI is a specification standard to describe RESTful APIs
- OpenAPI is a structured document written in JSON or YAML which adheres to the specification schema
- OpenAPI is used to:
  - Define API Endpoints
  - Authentication Methods
  - Parameters
  - Expected request and response bodies

### OpenAPI History

- OpenAPI’s origin starts with the Swagger Specification in 2010
- In 2015, Swagger was acquired by SmartBear Software who renamed Swagger with version 2.0 to
  OpenAPI and donated it to the Linux Foundation
- SmartBear formed an organization called the OpenAPI Initiative which includes member
  companies Apigee, IBM, Capital One, Google, Microsoft, PayPal and Intuit
- In February 2021 OpenAPI released version 3.1.0, which included alignment with JSON
  schema and Webhook support
- Often abbreviated as OA3 (OpenAPI 3.x)

### OpenAPI Today

- Today OpenAPI is has wide adoption in the industry
- OpenAPI has become the de facto standard for defining APIs OpenAPI is the source for rich user documentation
- OpenAPI Generator can be used to generate client and server stubs for all popular programming languages
- OpenAPI has a very large and robust open source community

### Code First Development With OpenAPI

- Most programming languages have libraries to generate the OpenAPI specification from the source code
- Good for initial spec generation for existing projects
- Specification will always match what the source code has defined
- Source code can be decorated to improve completeness of generated specification

### Problems with Code First Development

- Generated OA3 Specification is typically minimal
- Source code decorations to improve generated code creates clutter in source code
- Breaking API changes are hard to detect because spec always matches source code
- Lack of single source of truth for what the API should do
  - Is it the source code?
  - A Jira ticket, an email, a wiki page???

### Specification First Development

- In Spec First development, the OA3 Specification is created first
- The OA3 Spec becomes the contract to define what is expected of the API
  - Single source of truth to define the API
- Enables parallel development - the producer and consumer can each develop to the spec
- The produced specification is much richer in content than a generated specification

### Best Practices

- Write OpenAPI specification first
- Provide rich meta-data in specification
- Use JSON schema features to define expected request and response payloads
- Define properties that are required
- Define constraints - ie min/max size, date formats, etc
- Use OpenAPI Generator to generate client, server stubs, and data models
- Use tools to verify adherence to specification in integration tests

### OpenAPI for Spring Boot Development

- OpenAPI Specification Generation (Code First)
- Spring-Fox - Use for OpenAPI 2 / Swagger
  - WARNING: Project seems no longer active, last release in 2020
  - Does not support OA3
- Springdoc - Use for OpenAPI 3
  - Project is active and part of OpenAPI Collective
  - Use Springdoc v2 for Spring Framework 6 support

### OpenAPI for Testing

- Atlassian Swagger Request Validator - This is a rich library for testing HTTP Requests and
  Responses against an OpenAPI Specification
- Validates your code matches the OpenAPI Specification
- Spring MVC / Mock MVC supported (Spring Framework 6 not supported, yet)
- Spring WebFlux not supported
- Validates Pact Consumer Driven Contracts
- Validates RestAssured Tests
- Validates WireMock interactions

### OpenAPI for Code Generation

- OpenAPI Code Generation plugins are available for both Maven and Gradle
- Very useful for consuming 3rd party APIs with OpenAPI Specification
  - Generated client code can be a separate project, or in a submodule
  - When you have a quality OpenAPI specification, generated client code is usable
- Can be used to generate server stubs - good starting point
- Can be used to generate request / response POJOs
  - Often useful to publish POJOs in jar to share with other projects

### Spring Doc Maven Dependencies

- Refer - https://springdoc.org/
- Spring Doc - library to generate OpenAPI documentation
- `springdoc-openapi` java library helps to automate the generation of API documentation using
  spring boot projects.

- SETUP
  - Add `springdoc-openapi-starter-webmvc-ui` dependency
  ```
  <dependency>
      <groupId>org.springdoc</groupId>
      <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
      <version>2.3.0</version>
   </dependency>
  ```
  - Add `springdoc-openapi-starter-common` dependency
  ```
  <dependency>
      <groupId>org.springdoc</groupId>
      <artifactId>springdoc-openapi-starter-common</artifactId>
      <version>2.3.0</version>
   </dependency>
  ```

### Springdoc Spring Security Configuration

- Add configuration for the new endpoints we are going to expose for openAPI.

```
@Configuration
public class SpringSecConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests()
                .requestMatchers("/v3/api-docs**", "/swagger-ui/**",  "/swagger-ui.html")
                .permitAll()
                .anyRequest().authenticated()
                .and().oauth2ResourceServer().jwt();
        return http.build();
    }
}
```

- Run the application, and go to - `http:8081/v3/api-docs`

  - It will return the JSON. Copy this json and go to `https://editor.swagger.io/` and paste the JSON here

- For UI, and go to - `http:8081/swagger-ui/index.html`
  - It will open the Swagger UI page with all the UI components loaded

### Save OpenAPI Specification in Maven Build

- Refer - https://springdoc.org/#maven-plugin
- Generate OpenAPI specification using the spring boot maven plug in with the spring dock maven plug in

- Add execution goal
- Add swagger apiDocs url
- Add output file

- Complete `pom.xml`

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.0.2</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>guru.springframework</groupId>
    <artifactId>spring-6-rest-mvc</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring-6-rest-mvc</name>
    <description>spring-6-rest-mvc</description>
    <properties>
        <java.version>17</java.version>
        <org.mapstruct.version>1.5.2.Final</org.mapstruct.version>
        <spring-boot.run.profiles>localmysql</spring-boot.run.profiles>
        <jwt-uri>http://example.com:9090</jwt-uri>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.flywaydb</groupId>
            <artifactId>flyway-mysql</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct</artifactId>
            <version>${org.mapstruct.version}</version>
        </dependency>
        <dependency>
            <groupId>com.opencsv</groupId>
            <artifactId>opencsv</artifactId>
            <version>5.7.0</version>
        </dependency>
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
            <version>2.0.2</version>
        </dependency>
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-common</artifactId>
            <version>2.0.2</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <executions>
                    <execution>
                        <id>springdoc</id>
                        <goals>
                            <goal>start</goal>
                            <goal>stop</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>build-info</id>
                        <goals>
                            <goal>build-info</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <jvmArguments>-Dspring.application.admin.enabled=true</jvmArguments>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springdoc</groupId>
                <artifactId>springdoc-openapi-maven-plugin</artifactId>
                <version>1.4</version>
                <configuration>
                    <apiDocsUrl>http://localhost:8081/v3/api-docs.yaml</apiDocsUrl>
                    <outputFileName>oa3.yml</outputFileName>
                </configuration>
                <executions>
                    <execution>
                        <id>integration-test</id>
                        <goals>
                            <goal>generate</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.10.1</version>
                <configuration>
                    <source>17</source>
                    <target>17</target>
                    <annotationProcessorPaths>
                        <path>
                            <groupId>org.mapstruct</groupId>
                            <artifactId>mapstruct-processor</artifactId>
                            <version>${org.mapstruct.version}</version>
                        </path>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                            <version>${lombok.version}</version>
                        </path>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok-mapstruct-binding</artifactId>
                            <version>0.2.0</version>
                        </path>
                    </annotationProcessorPaths>
                    <compilerArgs>
                        <compilerArg>-Amapstruct.defaultComponentModel=spring</compilerArg>
                    </compilerArgs>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### Best Practices

- Use Spec First Development
- Use OpenAPI Code Generator to generate client when available
- Use OpenAPI to Code Generator publish and share model when possible
- Use Swagger Request Validator when possible
  - Verify Spec in integration tests, Pact Contracts or with RestAssured
  - Spring 6 Support expected in Q1 of 2023

### KEY TERMS

- In 2015, Swagger was acquired by SmartBear Software who renamed Swagger with version 2.0 to
  OpenAPI and donated it to the Linux Foundation

- OpenAPI is a specification standard to describe RESTful APIs
- OpenAPI has become the de facto standard for defining APIs
- Often abbreviated as OA3 (OpenAPI 3.x)
- Spring-Fox - Use for OpenAPI 2 / Swagger
- Springdoc - Use for OpenAPI 3

- Atlassian (at·la·see·uhn)

- Atlassian Swagger Request Validator - This is a rich library for testing HTTP Requests and
  Responses against an OpenAPI Specification

- OpenAPI Code Generation plugins are available for both Maven and Gradle
- Use OpenAPI Code Generator to generate client when available

- `Spring-Fox` - Use for OpenAPI 2 / Swagger
- `Springdoc` - Use for OpenAPI 3 (OA3)
- `Atlassian Swagger Request Validator`
- `OA3` (Open API 3.x)
- OpenAPI Code Generation plugins are available for both Maven and Gradle

- API Producer
- API Consumer
- API Contract
- Coding to Specification provided
- Provide rich meta-data in specification
- Use OpenAPI Generator to generate `client`, `server stubs`, and `data models`
- Verify adherence to specification in integration tests

- Open API Documentation
