## Service Discovery with Eureka

- Use Open Feign to utilize Eureka Service Discovery and the client side load balancing.

### OpenFeign Client

- Add dependency of openFeign client in Beer Service

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

- Enable FeignClient by adding annotation `@EnableFeignClients` on Main Class

```
@EnableFeignClients
@SpringBootApplication
public class MsscBeerServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(MsscBeerServiceApplication.class, args);
    }
}
```

- Create FeignClient interface

```
@FeignClient(name = "inventory-service", fallback = InventoryServiceFeignClientFailover.class, configuration = FeignClientConfig.class)
public interface InventoryServiceFeignClient {
    @RequestMapping(method = RequestMethod.GET, value = BeerInventoryServiceRestTemplateImpl.INVENTORY_PATH)
    ResponseEntity<List<BeerInventoryDto>> getOnhandInventory(@PathVariable UUID beerId);
}
```

- Use FeignClient to retrived Data

```
ResponseEntity<List<BeerInventoryDto>> responseEntity = inventoryServiceFeignClient.getOnhandInventory(beerId);
```

Here our feign client is going out and getting the inventory service from Eureka and registering it.

### Configure Gateway for Service Discovery

We want to do is do service lookups to find the services and the gateway, so our gateway is now utilizing Eureka

- Add Eureka Client dependency

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

- Add profile `local-discovery` and disable client registration with Eureka as Gateway should not need to.
  `eureka.client.register-with-eureka=false`

- Add Load Balacer configration in Cloud Gateway repo

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

### KEY TERMS

- OpenFeign Client
