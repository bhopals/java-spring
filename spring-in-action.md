## Spring In Action

### Spring Intro

- What is Spring?
  Any non-trivial application is composed of many components, each responsible for
  its own piece of the overall application functionality, coordinating with the other
  application elements to get the job done. When the application is run, those components somehow need to be created and introduced to each other.
  At its core, Spring offers a container, often referred to as the Spring application context, that creates and manages application components. These components, or beans,
  are wired together inside the Spring application context to make a complete application, much like bricks, mortar, timber, nails, plumbing, and wiring are bound together
  to make a house.

- The act of wiring beans together is based on a pattern known as dependency injection
  (DI). Rather than have components create and maintain the lifecycle of other beans
  that they depend on, a dependency-injected application relies on a separate entity
  (the container) to create and maintain all components and inject those into the beans
  that need them. This is done typically through constructor arguments or property
  accessor methods.

- The @Configuration annotation indicates to Spring that this is a configuration class
  that will provide beans to the Spring application context. The configuration’s class methods are annotated with @Bean, indicating that the objects they return should be added
  as beans in the application context (where, by default, their respective bean IDs will
  be the same as the names of the methods that define them).

- Java-based configuration offers several benefits over XML-based configuration,
  including greater type safety and improved refactorability

- Automatic configuration has its roots in the Spring techniques known as `autowiring`
  and `component scanning`. With component scanning, Spring can automatically discover
  components from an application’s classpath and create them as beans in the Spring
  application context. With autowiring, Spring automatically injects the components
  with the other beans that they depend on.

- Spring Boot is an extension
  of the Spring Framework that offers several productivity enhancements. The most
  well-known of these enhancements is `autoconfiguration`, where Spring Boot can make
  reasonable guesses of what components need to be configured and wired together,
  based on entries in the classpath, environment variables, and other factors

- Spring Boot autoconfiguration has dramatically reduced the amount of explicit
  configuration (whether with XML or Java) required to build an application.

- Benefits of Spring Boot, including starter dependencies and autoconfiguration.
- But in addition to starter dependencies and autoconfiguration,
  Spring Boot also offers a handful of other useful features:
  - The Actuator provides runtime insight into the inner workings of an application, including metrics, thread dump information, application health, and environment properties available to the application.
  - Flexible specification of environment properties.
  - Additional testing support on top of the testing assistance found in the core
    framework.

#### Initializing a Spring application

- The Spring Initializr is both a browser-based web application and a REST API,
  which can produce a skeleton Spring project structure that you can flesh out with
  whatever functionality you want.

```
@SpringBootApplication
public class TacoCloudApplication {
 public static void main(String[] args) {
 SpringApplication.run(TacoCloudApplication.class, args);
 }
}
```

- `@SpringBootApplication` is a composite application that combines three other
  annotations:

  - `@SpringBootConfiguration` — Designates this class as a configuration class.
    Although there’s not much configuration in the class yet, you can add Javabased Spring Framework configuration to this class if you need to. This annotation is, in fact, a specialized form of the @Configuration annotation.
  - `@EnableAutoConfiguration` — Enables Spring Boot automatic configuration.
    For now, know that this annotation tells Spring Boot to automatically configure any
    components that it thinks you’ll need.
  - `@ComponentScan` — Enables component scanning. This lets you declare other
    classes with annotations like @Component, @Controller, @Service, and others,
    to have Spring automatically discover them and register them as components in
    the Spring application context.

- Spring aims to make developer challenges easy, like creating web applications,
  working with databases, securing applications, and microservices.
- Spring Boot builds on top of Spring to make Spring even easier with simplified
  dependency management, automatic configuration, and runtime insights.
- Spring applications can be initialized using the Spring Initializr, which is webbased and supported
  natively in most Java development environments.
- The components, commonly referred to as beans, in a Spring application context can be declared explicitly with Java or XML, discovered by component scanning, or automatically configured with Spring Boot autoconfiguration.

### Spring Web

In a Spring web application, it’s a controller’s job to fetch and process data. And
it’s a view’s job to render that data into HTML that will be displayed in the browser.

#### Create Data/Domain/Model

```
//Using  'lombok' library - import lombok.Data;import lombok.RequiredArgsConstructor;
@Data
@RequiredArgsConstructor
public class Ingredient {
 private final String id;
 private final String name;
 private final Type type;
 public static enum Type {
 WRAP, PROTEIN, VEGGIES, CHEESE, SAUCE
 }
}
```

- @Data annotation at the class level is to generate all of those missing methods as well as
  a constructor that accepts all final properties as arguments

#### Creating a controller class

Controllers are the major players in Spring’s MVC framework. Their primary job is to
handle HTTP requests and either hand a request off to a view to render HTML
(browser-displayed) or write data directly to the body of a response (RESTful).

- Spring MVC request-mapping annotations

  - @RequestMapping General-purpose request handling
  - @GetMapping Handles HTTP GET requests
  - @PostMapping Handles HTTP POST requests
  - @PutMapping Handles HTTP PUT requests
  - @DeleteMapping Handles HTTP DELETE requests
  - @PatchMapping Handles HTTP PATCH requests

- To Register a Controller

```
@Configuration
public class WebConfig implements WebMvcConfigurer {
 @Override
 public void addViewControllers(ViewControllerRegistry registry) {
 registry.addViewController("/").setViewName("home");
 }
}
```

- Spring offers a powerful web framework called Spring MVC that can be used to
  develop the web frontend for a Spring application.

- Spring MVC is annotation-based, enabling the declaration of request-handling
  methods with annotations such as @RequestMapping, @GetMapping, and @PostMapping.

- Most request-handling methods conclude by returning the logical name of a
  view, such as a Thymeleaf template, to which the request (along with any model
  data) is forwarded.

- Spring MVC supports validation through the Java Bean Validation API and
  implementations of the Validation API such as Hibernate Validator.

- View controllers can be used to handle HTTP GET requests for which no
  model data or processing is required.

- In addition to Thymeleaf, Spring supports a variety of view options, including
  FreeMarker, Groovy Templates, and Mustache

### Spring Data

- Spring support for JDBC (Java Database Connectivity) to eliminate boilerplate code.
- Spring support for JPA (Java Persistence API), eliminating even more code.

- Spring’s JdbcTemplate greatly simplifies working with JDBC.
- PreparedStatementCreator and KeyHolder can be used together
  when you need to know the value of a database-generated ID.
- For easy execution of data inserts, use SimpleJdbcInsert.
- Spring Data JPA makes JPA persistence as easy as writing a repository interface

### Spring Security

- To Enable Spring Authentication

```
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {}
```

- Spring Security offers several options for configuring a user store, including
  these: (by overriding `configure` method)

  - In-Memory user store
  - JDBC-based user store
  - LDAP-backed user store (LDAP - Lightweight directory access protocol )
  - Custom User authentication - (User Details stored in DB)

- `AuthenticationManagerBuilder auth` to specify how users will be looked up during authentication.

#### LDAP

```
@Override
protected void configure(AuthenticationManagerBuilder auth)
 throws Exception {
 auth
 .ldapAuthentication()
 .userSearchBase("ou=people")
 .userSearchFilter("(uid={0})")
 .groupSearchBase("ou=groups")
 .groupSearchFilter("member={0}")
 .passwordCompare()
 .passwordEncoder(new BCryptPasswordEncoder())
 .passwordAttribute("passcode")
 .contextSource()
 .url("ldap://tacocloud.com:389/dc=tacocloud,dc=com");
}

```

- Spring Security autoconfiguration is a great way to get started with security, but most applications
  will need to explicitly configure security to meet their unique security requirements.
- User details can be managed in user stores backed by relational databases, LDAP, or
  completely custom implementations.
- Spring Security automatically protects against CSRF attacks.
- Information about the authenticated user can be obtained via the SecurityContext object (returned
  from SecurityContextHolder.getContext()) or injected into controllers using `@AuthenticationPrincipal`.

### KEY TERMS

- At its core, Spring offers a container, often referred to as the `Spring application context`,
  that creates and manages application components.
- persisting objects to a database
