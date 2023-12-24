## Spring RestTemplate with OAuth 2

### Maven Dependencies and Spring Configuration

- Add dependencies in `POM.XML`

```
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```

- Add configuration in `application.properties`

```
rest.template.rootUrl=http://localhost:8080
spring.security.oauth2.resourceserver.jwt.issuer-uri=http://localhost:9000
spring.security.oauth2.client.registration.springauth.client-id=messaging-client
spring.security.oauth2.client.registration.springauth.client-secret=secret
spring.security.oauth2.client.registration.springauth.scope[0]=message.read
spring.security.oauth2.client.registration.springauth.scope[1]=message.write
spring.security.oauth2.client.registration.springauth.authorization-grant-type=client_credentials
spring.security.oauth2.client.registration.springauth.provider=springauth
spring.security.oauth2.client.provider.springauth.authorization-uri=http://localhost:9000/auth2/authorize
spring.security.oauth2.client.provider.springauth.token-uri=http://localhost:9000/oauth2/token
```

- Modify Rest Template configuration

```
@Configuration
public class RestTemplateBuilderConfig {

    @Value("${rest.template.rootUrl}")
    String rootUrl;

    @Bean
    RestTemplateBuilder restTemplateBuilder(RestTemplateBuilderConfigurer configurer){
        assert rootUrl != null;
        return configurer.configure(new RestTemplateBuilder())
                .uriTemplateHandler(new DefaultUriBuilderFactory(rootUrl));
    }
}
```

### Spring Authorized Client Manager

- what this is going to do is handle the call to the authorization server for us so we don't have
  to hand code it.

- So this component is the component that we will be using in conjunction with the rest template builder.

```
private final ClientRegistrationRepository clientRegistrationRepository;
private final OAuth2AuthorizedClientService oAuth2AuthorizedClientService;

public RestTemplateBuilderConfig(ClientRegistrationRepository clientRegistrationRepository, OAuth2AuthorizedClientService oAuth2AuthorizedClientService) {
    this.clientRegistrationRepository = clientRegistrationRepository;
    this.oAuth2AuthorizedClientService = oAuth2AuthorizedClientService;
}

@Bean
OAuth2AuthorizedClientManager auth2AuthorizedClientManager(){
    var authorizedClientProvider = OAuth2AuthorizedClientProviderBuilder.builder()
            .clientCredentials()
            .build();

    var authorizedClientManager = new AuthorizedClientServiceOAuth2AuthorizedClientManager
            (clientRegistrationRepository, oAuth2AuthorizedClientService);
    authorizedClientManager.setAuthorizedClientProvider(authorizedClientProvider);
    return authorizedClientManager;
}
```

### Http Request Interceptor

- Configure an instance of a client HTTP request interceptor
- So basically we are going to configure the rest template to have an interceptor on it
  and to inspect it, see if it's got authorization.
- If it does not, it is going to add it for us in conjunction with the authentication manager.

- Create an Interceptor Class

```
@Component
public class OAuthClientInterceptor implements ClientHttpRequestInterceptor {
    private final OAuth2AuthorizedClientManager manager;
    private final Authentication principal;
    private final ClientRegistration clientRegistration;

    public OAuthClientInterceptor(OAuth2AuthorizedClientManager manager,  ClientRegistrationRepository clientRegistrationRepository) {
        this.manager = manager;
        this.principal = createPrincipal();
        this.clientRegistration = clientRegistrationRepository.findByRegistrationId("springauth");
    }

    @Override
    public ClientHttpResponse intercept(HttpRequest request, byte[] body, ClientHttpRequestExecution execution) throws IOException {
        OAuth2AuthorizeRequest oAuth2AuthorizeRequest = OAuth2AuthorizeRequest
                .withClientRegistrationId(clientRegistration.getRegistrationId())
                .principal(createPrincipal())
                .build();

        OAuth2AuthorizedClient client = manager.authorize(oAuth2AuthorizeRequest);

        if (isNull(client)) {
            throw new IllegalStateException("Missing credentials");
        }

        request.getHeaders().add(HttpHeaders.AUTHORIZATION,
                "Bearer " + client.getAccessToken().getTokenValue());

        return execution.execute(request, body);
    }

    private Authentication createPrincipal() {
        return new Authentication() {
            @Override
            public Collection<? extends GrantedAuthority> getAuthorities() {
                return Collections.emptySet();
            }

            @Override
            public Object getCredentials() {
                return null;
            }

            @Override
            public Object getDetails() {
                return null;
            }

            @Override
            public Object getPrincipal() {
                return this;
            }

            @Override
            public boolean isAuthenticated() {
                return false;
            }

            @Override
            public void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException {
            }

            @Override
            public String getName() {
                return clientRegistration.getClientId();
            }
        };
    }
}
```

- Add Interceptor to RestTemplate Builder

```
@Configuration
public class RestTemplateBuilderConfig {

    @Value("${rest.template.rootUrl}")
    String rootUrl;

    @Bean
    OAuth2AuthorizedClientManager auth2AuthorizedClientManager(ClientRegistrationRepository clientRegistrationRepository,
                                                               OAuth2AuthorizedClientService oAuth2AuthorizedClientService ){
        var authorizedClientProvider = OAuth2AuthorizedClientProviderBuilder.builder()
                .clientCredentials()
                .build();

        var authorizedClientManager = new AuthorizedClientServiceOAuth2AuthorizedClientManager
                (clientRegistrationRepository, oAuth2AuthorizedClientService);
        authorizedClientManager.setAuthorizedClientProvider(authorizedClientProvider);
        return authorizedClientManager;
    }

    @Bean
    RestTemplateBuilder restTemplateBuilder(RestTemplateBuilderConfigurer configurer,
                                            OAuthClientInterceptor interceptor){

        assert rootUrl != null;

        return configurer.configure(new RestTemplateBuilder())
                .additionalInterceptors(interceptor)
                .uriTemplateHandler(new DefaultUriBuilderFactory(rootUrl));
    }
}
```

### Mock OAuth2 Manager

- `@MockBean`

```
@MockBean
OAuth2AuthorizedClientManager manager;

@TestConfiguration
public static class TestConfig {

    @Bean
    ClientRegistrationRepository clientRegistrationRepository() {
        return new InMemoryClientRegistrationRepository(ClientRegistration
                .withRegistrationId("springauth")
                .authorizationGrantType(AuthorizationGrantType.CLIENT_CREDENTIALS)
                .clientId("test")
                .tokenUri("test")
                .build());
    }

    @Bean
    OAuth2AuthorizedClientService auth2AuthorizedClientService(ClientRegistrationRepository clientRegistrationRepository){
        return new InMemoryOAuth2AuthorizedClientService(clientRegistrationRepository);
    }

    @Bean
    OAuthClientInterceptor oAuthClientInterceptor(OAuth2AuthorizedClientManager manager, ClientRegistrationRepository clientRegistrationRepository){
        return new OAuthClientInterceptor(manager, clientRegistrationRepository);
    }
}

```

### KEY TERMS
