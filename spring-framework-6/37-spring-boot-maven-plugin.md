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

### KEY TERMS
