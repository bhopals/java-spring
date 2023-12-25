## Spring Boot Maven Plugin

### A Build Lifecycle is Made Up of Phases

Each of these build lifecycles is defined by a different list of build phases, wherein a build phase represents a stage in the lifecycle.

For example, the default lifecycle comprises of the following phases (for a complete list of the lifecycle phases, refer to the Lifecycle Reference):

- `clean` - clean the traget repo
- `validate` - validate the project is correct and all necessary information is available
- `compile` - compile the source code of the project
- `test` - test the compiled source code using a suitable unit testing framework. These tests
  should not require the code be packaged or deployed
- `package` - take the compiled code and package it in its distributable format, such as a JAR.
- `verify` - run any checks on results of integration tests to ensure quality criteria are met
- `install` - install the package into the local repository, for use as a dependency in other projects locally
  `deploy` - done in the build environment, copies the final package to the remote repository for sharing with other developers and projects.

So if you run `package`, it is going to run `validate, compile, test, package`, and
if you run `deploy`, it is going to run `validate, compile, test, package, verify, install, deploy`,

### Maven from the Command Line

- Run command

  - `./mvnw clean`
  - `mvn clean package`

- To run any `.jar` file, `java -jar <jar-name>.jar`

- To run project with active profile `.jar` file,
  - `java -jar -Dspring.profiles.active=localmysql <jar-name>.jar`
  - `java -jar -Dspring.profiles.active=localmysql spring-6-rest-mvc-0.0.1-SNAPSHOT.jar`
  - Basically, the above command will run the whole Spring Application using JAR with just one command.
  - Once it is running, you can use all the endpoints

### Spring Boot Repackage to Executable Jar

- Refer - https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/
- In our application, we have been using this goal - `spring-boot:repackage`

- Below are the goals that has been used by Spring Boot to Run/Start/Stop the Spring Application
  spring-boot:run

Run an application in place.

- spring-boot:run - Run an application in place.

- spring-boot:start - Start a spring application. Contrary to the run goal, this does not block and allows other goals to operate on the application. This goal is typically used in integration test scenario where the application is started before a test suite and stopped after.

- spring-boot:stop - Stop an application that has been started by the "start" goal. Typically invoked once a test suite has completed.

We can customize and add profiles in any of the goal (ex. Run database when `run`)
`<spring-boot.run-profiles>`

- To run it from command prompt, use this command
  `./mvnw spring-boot:run`

### Resource Filtering

- Externalize a property from `application.properties`, so it can setup using MVN

  - `spring.security.oauth2.resourceserver.jwt.issue-uri=http://localhost:9000`

- Update the properties to use `@<name>@`
  - `spring.security.oauth2.resourceserver.jwt.issue-uri=@jwt-uri@`
- Setup value in POM.XML
  - `<properties><jwt.uri>http://example.com</jwt.uri></properties>`
  - The value will be replaced when you run the application

### Maven Build Info

- `spring-boot:build-info`
- It gathers information about build and creates `build-info.properties` with details
  in `/target/META-INF/build-info.properties`

- To inject any phase/goal as part of existing build, you need to add in `execution`

  ```
  <plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <executions>
        <execution>
            <goals>
                <goal>build-info</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <excludes>
            <exclude>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
            </exclude>
        </excludes>
    </configuration>
  </plugin>
  ```

- The FULL `POM.XML`

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
                        <goals>
                            <goal>build-info</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
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

### KEY TERMS
