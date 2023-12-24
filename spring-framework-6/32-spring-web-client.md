## Spring Web Client

- Web client is a replacement for Rest templates. So this is the new reactive way of working
  with Restful Services.

### Create new Spring Boot Project

- Create a new Spring Boot Project using Sping Boot Initializer

  - Lombok
  - Spring Reactive Web

- Add `awaitibility` dependency

### Get List as JSON String

```
@Service
public class BeerClientImpl implements BeerClient {

    private final WebClient webClient;

    public BeerClientImpl(WebClient.Builder webClientBuilder) {
        this.webClient = webClientBuilder.baseUrl("http://localhost:8080").build();
    }

    @Override
    public Flux<String> listBeer() {
        return webClient.get().uri("/api/v3/beer", String.class)
                .retrieve().bodyToFlux(String.class);
    }
}
```

### Get List as Java Map

```
@Service
public class BeerClientImpl implements BeerClient {

    public static final String BEER_PATH = "/api/v3/beer";
    private final WebClient webClient;

    public BeerClientImpl(WebClient.Builder webClientBuilder) {
        this.webClient = webClientBuilder.baseUrl("http://localhost:8080").build();
    }

    @Override
    public Flux<Map> listBeerMap() {
        return webClient.get().uri(BEER_PATH, Map.class)
                .retrieve().bodyToFlux(Map.class);
    }
}
```

### Get List as Jackson Object

- Using `jsonNode`

### Get List as Java POJOs

- Create DTOs

### Weclient Global Configuration

- Add `rootUrl` in `application.properties`
  `webclient.rooturl=http://localhost:8080`

- Add Config Class and annotate with `@Configuration`

```
/**
 * Created by jt, Spring Framework Guru.
 */
@Configuration
public class WebClientConfig implements WebClientCustomizer {

    private final String rootUrl;

    public WebClientConfig(@Value("${webclient.rooturl}") String rootUrl) {
        this.rootUrl = rootUrl;
    }

    @Override
    public void customize(WebClient.Builder webClientBuilder) {
        webClientBuilder.baseUrl(rootUrl);
    }
}
```

```
 public BeerClientImpl(WebClient.Builder webClientBuilder) {
        this.webClient = webClientBuilder.build();
    }

    @Override
    public Flux<BeerDTO> listBeerDtos() {
        return webClient.get().uri(BEER_PATH)
                .retrieve().bodyToFlux(BeerDTO.class);
    }
```

### Uri Components Builder

```
@Override
    public Mono<BeerDTO> getBeerById(String id) {
        return webClient.get()
                .uri(uriBuilder -> uriBuilder.path(BEER_PATH_ID)
                        .build(id))
                .retrieve()
                .bodyToMono(BeerDTO.class);
    }
@Override
public Flux<BeerDTO> listBeerDtos() {
    return webClient.get().uri(BEER_PATH)
            .retrieve().bodyToFlux(BeerDTO.class);
}
```

### Query Parameters

```
@Override
    public Flux<BeerDTO> getBeerByBeerStyle(String beerStyle) {
        return webClient.get().uri(uriBuilder -> uriBuilder
                        .path(BEER_PATH)
                        .queryParam("beerStyle", beerStyle).build())
                .retrieve()
                .bodyToFlux(BeerDTO.class);
    }
```

### HTTP Post with WebClient

```
@Override
    public Mono<BeerDTO> createBeer(BeerDTO beerDTO) {
        return webClient.post()
                .uri(BEER_PATH)
                .body(Mono.just(beerDTO), BeerDTO.class)
                .retrieve()
                .toBodilessEntity()
                .flatMap(voidResponseEntity -> Mono.just(voidResponseEntity
                        .getHeaders().get("Location").get(0)))
                .map(path -> path.split("/")[path.split("/").length -1])
                .flatMap(this::getBeerById);
    }

```

### HTTP Put with WebClient

```
 @Override
    public Mono<BeerDTO> updateBeer(BeerDTO beerDTO) {
        return webClient.put()
                .uri(uriBuilder -> uriBuilder.path(BEER_PATH_ID).build(beerDTO.getId()))
                .body(Mono.just(beerDTO), BeerDTO.class)
                .retrieve()
                .toBodilessEntity()
                .flatMap(voidResponseEntity -> getBeerById(beerDTO.getId()));
    }

```

### KEY TERMS
