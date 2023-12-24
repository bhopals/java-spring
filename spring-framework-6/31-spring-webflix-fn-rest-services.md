## Spring WebFlux.fn Rest Services

Spring WebFlux includes WebFlux.fn, a lightweight functional programming model in which functions are used to route and handle requests and contracts are designed for immutability. It is an alternative to the annotation-based programming model but otherwise runs on the same Reactive Core foundation.

- Docs - https://docs.spring.io/spring-framework/reference/web/webflux-functional.html

```
RouterFunction<ServerResponse> route = route()
	.GET("/person/{id}", accept(APPLICATION_JSON), handler::getPerson)
	.GET("/person", accept(APPLICATION_JSON), handler::listPeople)
	.POST("/person", handler::createPerson)
	.build();
```

- Create Route Config - `BeerRouteConfig`

```
/**
 * Created by jt, Spring Framework Guru.
 */
@Configuration
@RequiredArgsConstructor
public class BeerRouterConfig {
    public static final String BEER_PATH = "/api/v3/beer";
    public static final String BEER_PATH_ID = BEER_PATH + "/{beerId}";
    private final BeerHandler handler;
    @Bean
    public RouterFunction<ServerResponse> beerRoutes(){
        return route()
                .GET(BEER_PATH, accept(APPLICATION_JSON), handler::listBeers)
                .GET(BEER_PATH_ID, accept(APPLICATION_JSON), handler::getBeerById)
                .build();
    }
}


//Handler
@Component
@RequiredArgsConstructor
public class BeerHandler {
    private final BeerService beerService;

    public Mono<ServerResponse> getBeerById(ServerRequest request){
        return ServerResponse
                .ok()
                .body(beerService.getById(request.pathVariable("beerId")), BeerDTO.class);
    }

    public Mono<ServerResponse> listBeers(ServerRequest request){
        return ServerResponse.ok()
                .body(beerService.listBeers(), BeerDTO.class);
    }
}
```

- Validate - `doOnNext` method

### KEY TERMS
