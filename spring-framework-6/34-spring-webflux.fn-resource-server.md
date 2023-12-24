## Spring WebFlux.fn Resource Server

- Add dependency in `POM.XML`

```
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

- Add properties in `application.properties`

```
#logging.level.org.springframework.r2dbc=trace
spring.security.oauth2.resourceserver.jwt.issuer-uri=http://localhost:9000
```

- Add Spring Security Configuration

```
/**
 * Created by jt, Spring Framework Guru.
 */
@Configuration
@EnableWebFluxSecurity
public class SecurityConfig {

    @Bean
    SecurityWebFilterChain securityWebFilterChain(ServerHttpSecurity http){
        http.csrf().disable()
                .authorizeExchange()
                .anyExchange().authenticated()
                .and()
                .oauth2ResourceServer().jwt();

        return http.build();
    }

}
```

- Add OAuth2 Client dependency (For Testing Client)

```
<dependency>
<groupId>org.springframework.security</groupId>
<artifactId>spring-security-oauth2-client</artifactId>
<scope>test</scope>
</dependency>
```

```
@Test
    void testPatchIdNotFound() {
        webTestClient
                .mutateWith(mockOAuth2Login())
                .patch()
                .uri(BeerController.BEER_PATH_ID, 999)
                .body(Mono.just(BeerRepositoryTest.getTestBeer()), BeerDTO.class)
                .exchange()
                .expectStatus().isNotFound();
    }

    @Test
    void testDeleteNotFound() {
        webTestClient.mutateWith(mockOAuth2Login())
                .delete()
                .uri(BeerController.BEER_PATH_ID, 999)
                .exchange()
                .expectStatus().isNotFound();
    }

    @Test
    @Order(999)
    void testDeleteBeer() {
        webTestClient.mutateWith(mockOAuth2Login())
                .delete()
                .uri(BeerController.BEER_PATH_ID, 1)
                .exchange()
                .expectStatus()
                .isNoContent();
    }
```
