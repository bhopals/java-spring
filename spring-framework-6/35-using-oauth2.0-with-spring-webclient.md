## Using OAuth 2.0 with Spring WebClient

- Add Dependency in `POM.XML`

```
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```

- Add properties in `application.properties`

```
webclient.rooturl=http://localhost:8080
spring.security.oauth2.client.provider.springauth.token-uri=http://localhost:9000/oauth2/token

spring.security.oauth2.client.registration.springauth.client-name=springauth
spring.security.oauth2.client.registration.springauth.client-id=messaging-client
spring.security.oauth2.client.registration.springauth.client-secret=secret
spring.security.oauth2.client.registration.springauth.scope[0]=message.read
spring.security.oauth2.client.registration.springauth.scope[1]=message.write
spring.security.oauth2.client.registration.springauth.authorization-grant-type=client_credentials
spring.security.oauth2.client.registration.springauth.provider=springauth
```

### Spring Security Configuration

```
/**
 * Created by jt, Spring Framework Guru.
 */
@Configuration
public class SpringSecurityConfig {
    @Bean
    public ReactiveOAuth2AuthorizedClientManager authorizedClientManager (
            ReactiveClientRegistrationRepository clientRegistrationRepository,
            ReactiveOAuth2AuthorizedClientService authorizedClientService
    ){
        ReactiveOAuth2AuthorizedClientProvider authorizedClientProvider =
                ReactiveOAuth2AuthorizedClientProviderBuilder.builder()
                        .clientCredentials()
                        .build();

        AuthorizedClientServiceReactiveOAuth2AuthorizedClientManager authorizedClientManager
                = new AuthorizedClientServiceReactiveOAuth2AuthorizedClientManager(
                        clientRegistrationRepository, authorizedClientService
        );

        authorizedClientManager.setAuthorizedClientProvider(authorizedClientProvider);

        return authorizedClientManager;
    }

}
```

### Web Client Filter Configuration

- Add WebClientConfig

```
/**
 * Created by jt, Spring Framework Guru.
 */
@Configuration
public class WebClientConfig implements WebClientCustomizer {

    private final String rootUrl;
    private final ReactiveOAuth2AuthorizedClientManager authorizedClientManager;

    public WebClientConfig(@Value("${webclient.rooturl}") String rootUrl,
                           ReactiveOAuth2AuthorizedClientManager authorizedClientManager) {
        this.rootUrl = rootUrl;
        this.authorizedClientManager = authorizedClientManager;
    }

    @Override
    public void customize(WebClient.Builder webClientBuilder) {
        ServerOAuth2AuthorizedClientExchangeFilterFunction oauth
                = new ServerOAuth2AuthorizedClientExchangeFilterFunction(authorizedClientManager);
        oauth.setDefaultClientRegistrationId("springauth");

        webClientBuilder.filter(oauth).baseUrl(rootUrl);
    }
}
```

- Call WebClient from Controller

```

    public BeerClientImpl(WebClient.Builder webClientBuilder) {
        this.webClient = webClientBuilder.build();
    }

    @Override
    public Mono<BeerDTO> patchBeer(BeerDTO dto) {
        return webClient.patch()
                .uri(uriBuilder -> uriBuilder.path(BEER_PATH_ID).build(dto.getId()))
                .body(Mono.just(dto), BeerDTO.class)
                .retrieve()
                .toBodilessEntity()
                .flatMap(voidResponseEntity -> getBeerById(dto.getId()));
    }
```

### KEY TERMS
