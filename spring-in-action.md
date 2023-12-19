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

### KEY TERMS

- At its core, Spring offers a container, often referred to as the `Spring application context`,
  that creates and manages application components.
