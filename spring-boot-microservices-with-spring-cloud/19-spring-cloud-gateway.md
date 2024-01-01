## Spring Cloud Gateway

### Gateway Responsibilities

- Routing / Dynamic Routing
- Security
- Rate Limiting
- Monitoring / Logging
- Blue / Green Deployments
- Caching
- Monolith Strangling

### Types of Gateways

- `Appliances / Hardware` - example: F5
- `SAAS` (Software As A Service) - AWS Elastic Load Balancer
- `Web Servers` - Configured as proxies
- `Developer Oriented` - Examples: Zuul (Netflix) or Spring Cloud Gateway
- `Others` - Not an exhaustive list, technology is evolving & overlapping
- Types can be combined

### History - Developer Oriented Gateways

- Netflix on June 12th, 2013 announced it was open sourcing Zuul
  - “Edge Service in the Cloud”
  - 1,000 different client types
  - 50,000+ requests per second

#### Problems with Zuul 1

- Used Java’s HTTP Servlet API
  - Blocking - Inefficient
  - Did not support HTTP 2
- September 2016 Netflix moved to Zuul 2
  - Non-Blocking - much more efficient
  - Support for HTTP 2
  - Announced they planned to Open Source Zuul 2

### Spring Cloud Gateway

- In 2017, the Spring Cloud team decided to not migrate Spring Cloud to Zuul 2
- Direction of Zuul 2 was unclear at the time
- While Netflix had open sourced Zuul 1, some components were still closed source
- Also when Spring 5 had recently gone GA, which included reactive support
- First milestone release in August, 2017
- 1.0 GA Release in November, 2017

- Features
  - Java 8+, Spring Framework 5, Spring Boot 2, Project Reactor
  - Non-blocking, HTTP 2 Support, Netty
  - Dynamic Routing
  - Route Mapping on HTTP Request attributes
  - Filters for HTTP Request and Response

### Spring Cloud Gateway Summary

- Spring Cloud Originally used Zuul 1
- Spring Cloud Gateway developed as a replacement to Zuul 1
- Zuul and Spring Cloud Gateway are not the only developer oriented gateways

### Spring Cloud Gateway Service Creation

- Use Spring Boot Initilizer

  - Spring Cloud Routing Gateway

- Add application name and port details in `application.properties`
  - `spring.application.name=sfg-brewery-gateway`
  - `server.port=9890`

#### Spring Cloud Gateway Route Configuration

- Create a Config Class `GoogleConfig` - An Example configuration for demonstrate

```
@Profile("google")
@Configuration
public class GoogleConfig {
    @Bean
    public RouteLocator googleRouteConfig(RouteLocatorBuilder builder){
        return builder.routes()
            .route(r -> r.path("/googlesearch2")
                    .filters(f -> f.rewritePath("/googlesearch2(?<segment>/?.*)", "/${segment}"))
            .uri("https://google.com")
            .id("google"))
            .build();
    }
}
```

### Beer Service Route Configuration

- Add `LocalHostRouteConfig.java`

```
@Profile("!local-discovery")
@Configuration
public class LocalHostRouteConfig {

    @Bean
    public RouteLocator localHostRoutes(RouteLocatorBuilder builder){
        return builder.routes()
                .route(r -> r.path("/api/v1/beer*", "/api/v1/beer/*", "/api/v1/beerUpc/*")
                        .uri("http://localhost:8080")
                        .id("beer-service"))
                .route(r -> r.path("/api/v1/customers/**")
                        .uri("http://localhost:8081")
                        .id("order-service"))
                .route(r -> r.path("/api/v1/beer/*/inventory")
                        .uri("http://localhost:8082")
                        .id("inventory-service"))
                .build();
    }
}

```

#### More on AntPathMatcher

- https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/util/AntPathMatcher.html

### KEY TERMS

- Event Queue
- Event Loop
- Network Call
- High degree of Flexibility
- Request / Response
- Callback

- Client
- Handler Mapping
  - Predicates
- Web Handler
  - Pre-Filters
  - Global Filters
  - Post-Filters
- Downstream Service

- Zuul - Ghostbuster’s Gatekeeper

- Re-write Path
- Manipulate Traffic going through the Gateway
- Spring Cloud Gateway uses Reactive Spring
