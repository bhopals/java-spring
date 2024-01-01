## Circuit Breaker Pattern

- The Circuit Breaker Pattern is a simple concept which allows you to recover from errors
- If a service is unavailable or has unrecoverable errors, via the Circuit Breaker Pattern
  you can specify an alternative action.
- For example, we wish to always have inventory for potential orders.
  - If the inventory service is down, we can provide a fallback service to respond with inventory

### Spring Cloud Circuit Breaker

- Spring Cloud Circuit Breaker is a project which provides abstractions across several
  circuit breaker implementations
- Thus your source code is not tied to a specific implementation (like SLF4J)
- Supported:
  - Netflix Hystrix
  - Resilience4J
  - Sentinel
  - Spring Retry
- Spring Cloud Gateway supports Netflix Hystrix and Resilience4J
- Gateway Filters are used on top the the Spring Cloud Circuit Breaker APIs
- Netflix has placed Hystrix into maintenance mode, Spring suggests using Resilience4J

### Setup Steps

- Create Inventory Failover service
- Configure Spring Cloud Gateway to use circuit breaker for failover
- Configure Feign to use Circuit Breaker

### Resilience4j Failover for Spring Cloud Gateway

- Add resilience4j dependency in Spring Cloud Gateway project's POM.XML

```
 <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
</dependency>
```

- Add routing config for `inventory-failover` service

```
.route(r -> r.path("/inventory-failover/**")
                        .uri("lb://inventory-failover")
                        .id("inventory-failover-service"))
```

- Add service name and port details in `Inventory Failover` service configuration

  - `spring.application.name=inventory-failover`
  - `server.port=8083`

- Add Circuit breaker configuration in Routing

```
@Profile("local-discovery")
@Configuration
public class LoadBalancedRoutesConfig {
    @Bean
    public RouteLocator loadBalancedRoutes(RouteLocatorBuilder builder){
        return builder.routes()
                .route(r -> r.path("/api/v1/beer*", "/api/v1/beer/*", "/api/v1/beerUpc/*")
                        .uri("lb://beer-service")
                        .id("beer-service"))
                .route(r -> r.path("/api/v1/customers/**")
                        .uri("lb://order-service")
                        .id("order-service"))
                .route(r -> r.path("/api/v1/beer/*/inventory")
                        .filters(f -> f.circuitBreaker(c -> c.setName("inventoryCB")
                                        .setFallbackUri("forward:/inventory-failover")
                                        .setRouteId("inv-failover")
                                    ))
                        .uri("lb://inventory-service")
                        .id("inventory-service"))
                .route(r -> r.path("/inventory-failover/**")
                        .uri("lb://inventory-failover")
                        .id("inventory-failover-service"))
                .build();
    }
}
```

### Use Hystric Circuit Breaker with Feign Client

- Enable Feign Client - `application-loca-discovery.properties`
  `feign.hystrix.enabled=true`

- Specify Fallback class
  `@FeignClient(name = "inventory-service", fallback = InventoryServiceFeignClientFailover.class, configuration = FeignClientConfig.class)`

- Provoide implementation for Failover client

```
@RequiredArgsConstructor
@Component
public class InventoryServiceFeignClientFailover implements InventoryServiceFeignClient {
    private final InventoryFailoverFeignClient failoverFeignClient;
    @Override
    public ResponseEntity<List<BeerInventoryDto>> getOnhandInventory(UUID beerId) {
        return failoverFeignClient.getOnhandInventory();
    }
}
```

### KEY TERMS

- Netflix Hystrix - Circuit Breaker (In maintenance mode now)
- Resilience4j with Feign Client (Preferred)

- Spring Cloud Circuit Breaker
