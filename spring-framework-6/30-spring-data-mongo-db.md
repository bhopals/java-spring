## Spring Data MongoDB

- Docs - https://docs.spring.io/spring-data/mongodb/reference/#mongodb.repositories.queries

### Stack YML (MongoDB)

- Running MongoDB in Docker

```
# Use root/example as user/password credentials
version: '3.1'
services:
  mongo:
    image: mongo
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example
    ports:
    - "27017:27017"

  mongo-express:
    image: mongo-express
    restart: always
    ports:
      - 8081:8081
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: root
      ME_CONFIG_MONGODB_ADMINPASSWORD: example
      ME_CONFIG_MONGODB_URL: mongodb://root:example@mongo:27017/
```

- User the command `docker-compose -f stack.yml` up from same directory as file. The Docker compose file will expose a database browser on http://localhost:8081.

- Use `Studio 3T` for web ui client

### Create New Spring Boot Project

- Create new project using Spring Initializer

  - Lombok
  - Spring Reactive Web
  - Spring Data Reactive MongoDB
  - Validation

- Add `MapStruct` dependency with version details
- Add details in Maven Plugin section to annonatation processing

  - MapStruct
  - Lombok

- Also Add `compilerArg` for `MapStruct`

- Below will be the complete `pom.xml`

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
    <artifactId>reactive-mongo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring-6-reactive-mongo</name>
    <description>spring-6-reactive-mongo</description>
    <properties>
        <java.version>17</java.version>
        <org.mapstruct.version>1.5.2.Final</org.mapstruct.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-mongodb-reactive</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webflux</artifactId>
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
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>io.projectreactor</groupId>
            <artifactId>reactor-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
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

### How to use MapStruct

- Add/Update in POM.XML

  - Add `MapStruct` dependency and version details in POM.XML
  - Add config in `<plugin>` section for annotation parsing
  - Add `compilerArgs` to `<compilerArg>-Amapstruct.defaultComponentModel=spring</compilerArg>`

- Create an interface Mapper - `BeerMapper`
- Annotate the interface with `@Mapper`
- Create two methods

  - `BeerDTO beerToBeerDto(Beer beer)`
  - `Beer beerDTOToBeer(BeerDTO beerDTO)`

- call the mapper method
  `.map(beerMapper::beerToBeerDto);`

### Add Mongo Database and Client Configuration

- Add a configuration class with `@Configuration` annotation - MongoConfig

```
/**
 * Created by jt, Spring Framework Guru.
 */
@Configuration
public class MongoConfig extends AbstractReactiveMongoConfiguration {

    @Bean
    public MongoClient mongoClient() {
        return MongoClients.create();
    }

    @Override
    protected String getDatabaseName() {
        return "sfg";
    }

    @Override
    protected void configureClientSettings(MongoClientSettings.Builder builder) {
        builder.credential(MongoCredential.createCredential("root",
                "admin", "example".toCharArray()))
                .applyToClusterSettings(settings -> {
                    settings.hosts((singletonList(
                            new ServerAddress("127.0.0.1", 27017)
                    )));
                });
    }
}
```

### Using Awaitibility

- Awaitibility library - A Java DSL for synchronizing asynchronous operations
- Awaitility is a small Java DSL for synchronizing (waiting for) asynchronous operations.
- It makes it easy to test asynchronous code.
- DSL - Domain Specific Language
- More - https://github.com/awaitility/awaitility/wiki/Getting_started

```
@Test
void saveBeer() {
    AtomicBoolean atomicBoolean = new AtomicBoolean(false);
    Mono<BeerDTO> savedMono = beerService.saveBeer(Mono.just(beerDTO));
    savedMono.subscribe(savedDto -> {
        System.out.println(savedDto.getId());
        atomicBoolean.set(true);
    });
    await().untilTrue(atomicBoolean);
}
```

### KEY TERMS

- Awaitibility library - A Java DSL for synchronizing asynchronous operations
