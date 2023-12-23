## Spring RestTemplate

### Get Lists as JSON String

- `RestTemplateBuilder`
  - `getForEntity`
  - Returns `ResponseEntity<String>` ==> response.getBody()

```
/**
 * Created by jt, Spring Framework Guru.
 */
@RequiredArgsConstructor
@Service
public class BeerClientImpl implements BeerClient {
    private final RestTemplateBuilder restTemplateBuilder;
    @Override
    public Page<BeerDTO> listBeers() {
        RestTemplate restTemplate = restTemplateBuilder.build();
        ResponseEntity<String> stringResponse =
                restTemplate.getForEntity("http://localhost:8080/api/v1/beer", String.class);
        System.out.println(stringResponse.getBody());
        return null;
    }
}
```

### Get List as Java Map

```
 @Override
    public Page<BeerDTO> listBeers() {
        RestTemplate restTemplate = restTemplateBuilder.build();
        ResponseEntity<Map> mapResponse =
                restTemplate.getForEntity("http://localhost:8080/api/v1/beer", Map.class);
        System.out.println(stringResponse.getBody());

        return null;
    }
```

### Get List as Jackson Object

```
 @Override
    public Page<BeerDTO> listBeers() {
        RestTemplate restTemplate = restTemplateBuilder.build();
        ResponseEntity<JsonNode> jsonResponse =
                restTemplate.getForEntity("http://localhost:8080/api/v1/beer", JsonNode.class);

       jsonResponse.getBody().findPath("content")
                .elements().forEachRemaining(node -> {
                   System.out.println(node.get("beerName").asText());
                });


        return null;
    }
```

### Spring Pageable with Jackson

```
 @Override
    public Page<BeerDTO> listBeers() {
        RestTemplate restTemplate = restTemplateBuilder.build();

        ResponseEntity<BeerDTOPageImpl> stringResponse =
                restTemplate.getForEntity("http://localhost:8080/api/v1/beer", BeerDTOPageImpl.class);


        return null;
    }
```

```
@JsonIgnoreProperties(ignoreUnknown = true, value = "pageable")
public class BeerDTOPageImpl<BeerDTO> extends PageImpl<guru.springframework.spring6resttemplate.model.BeerDTO> {

    @JsonCreator(mode = JsonCreator.Mode.PROPERTIES)
    public BeerDTOPageImpl(@JsonProperty("content") List<guru.springframework.spring6resttemplate.model.BeerDTO> content,
                           @JsonProperty("number") int page,
                           @JsonProperty("size") int size,
                           @JsonProperty("totalElements") long total) {
        super(content, PageRequest.of(page, size), total);
    }

    public BeerDTOPageImpl(List<guru.springframework.spring6resttemplate.model.BeerDTO> content, Pageable pageable, long total) {
        super(content, pageable, total);
    }

    public BeerDTOPageImpl(List<guru.springframework.spring6resttemplate.model.BeerDTO> content) {
        super(content);
    }
}
```

### RestTemplateBuilder Configuration

- This will automatically override the default configuration of REST TEMPLATE

```
 @Bean
    RestTemplateBuilder restTemplateBuilder(RestTemplateBuilderConfigurer configurer){

        RestTemplateBuilder builder = configurer.configure(new RestTemplateBuilder());
        DefaultUriBuilderFactory uriBuilderFactory = new
                DefaultUriBuilderFactory("http://localhost:8080");

        return builder.uriTemplateHandler(uriBuilderFactory);
    }
```

### Externalize ROOT Url

- `application.properties`
  `rest.template.rootUrl=http://localhost:8080`

- RestTemplateBuilderConfig

```
 @Configuration
public class RestTemplateBuilderConfig {

    @Value("${rest.template.rootUrl}")
    String rootUrl;

    @Bean
    RestTemplateBuilder restTemplateBuilder(RestTemplateBuilderConfigurer configurer){

        assert rootUrl != null;

        RestTemplateBuilder builder = configurer.configure(new RestTemplateBuilder());
        DefaultUriBuilderFactory uriBuilderFactory = new
                DefaultUriBuilderFactory(rootUrl);

        return builder.uriTemplateHandler(uriBuilderFactory);
    }
}
```

### Uri Components Builder

- Use `UriComponentBuilder`

```
 RestTemplateBuilder builder = configurer.configure(new RestTemplateBuilder());
        DefaultUriBuilderFactory uriBuilderFactory = new
                DefaultUriBuilderFactory(rootUrl);
```

### HTTP Post with RestTemplate

- RestTemplate ==> `postForEntity`

```
 @Override
    public BeerDTO createBeer(BeerDTO newDto) {
        RestTemplate restTemplate = restTemplateBuilder.build();

        ResponseEntity<BeerDTO> response = restTemplate.postForEntity(GET_BEER_PATH, newDto, BeerDTO.class);

        return null;
    }
```

### HTTP Put with RestTemplate

```
@Override
    public BeerDTO updateBeer(BeerDTO beerDto) {
        RestTemplate restTemplate = restTemplateBuilder.build();
        restTemplate.put(GET_BEER_BY_ID_PATH, beerDto, beerDto.getId());
        return getBeerById(beerDto.getId());
    }
```

### HTTP Delete with RestTemplate

```
    @Override
    public void deleteBeer(UUID beerId) {
        RestTemplate restTemplate = restTemplateBuilder.build();
        restTemplate.delete(GET_BEER_BY_ID_PATH, beerId);
    }

```

### KEY TERMS
