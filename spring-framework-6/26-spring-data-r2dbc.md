## Spring Data R2DBC

- R2DBC - Reactive Relational Database Connectivity
- More - https://spring.io/projects/spring-data-r2dbc/

- The Spring Data R2DBC is a great framework to easily get started utilizing these reactive drivers.
  Once we have more persistence working, we are going to start looking at Spring Web Flux.
  Web Flux stands in parallel. You have spring MVC that uses the traditional Java servelet model, which is blocking, the APIs and that are blocking. They had completely redeveloped that in a reactive model and that's what Web Flux is built on top of. So we'll be exploring Web Flux and Web Flux client, test client, coming up in the next couple of sessions.

### Create Spring Boot Project

- Create new Project using Spring initializer
  - Spring Reactive Web
  - Spring Data R2DBC
  - H2 In-memory Database
  - Lombok
  - Validation

### Initializing Database

- Create database schema `schema.sql`

```
CREATE TABLE if NOT EXISTS beer
(
    id             integer NOT NULL PRIMARY KEY AUTO_INCREMENT,
    beer_name      varchar(255),
    beer_style     varchar (255),
    upc            varchar (25),
    quantity_on_hand integer,
    price          decimal,
    created_date   timestamp,
    last_modified_date timestamp
);
```

- Create DatabaseConfig Class (@Configuration Bean)

```
/**
 * Created by jt, Spring Framework Guru.
 */
@Configuration
public class DatabaseConfig {

    @Value("classpath:/schema.sql")
    Resource resource;

    @Bean
    ConnectionFactoryInitializer initializer(ConnectionFactory connectionFactory){
        ConnectionFactoryInitializer initializer = new ConnectionFactoryInitializer();
        initializer.setConnectionFactory(connectionFactory);
        initializer.setDatabasePopulator(new ResourceDatabasePopulator(resource));
        return initializer;
    }
}
```

- Update `application.properties` for logging
  `logging.level.org.springframework.r2dbc=trace`

### Create Spring Data R2DBC Repository

- All the repositories should extend `ReactiveCrudRepository<Beer, Integer>`

- Whenever we are using methods, need to use `subscribe` first
  `beerRepository.save(beer1).subscribe();`
  `beerRepository.count().subscribe();`

### Add Create and Update Date with Auditing

- To Enable Auditing, add `@EnableR2dbcAuditing` on `DatabaseConfig` Bean

- In test case, we can use `@Import` to import this Config class - `@Import(DatabaseConfig.class)`

### KEY TERMS

- Reactive Programming
- R2DBC - Reactive Relations Database Connectivity
