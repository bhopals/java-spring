## OpenAPI validation with RestAssured

- Refer - https://bitbucket.org/atlassian/swagger-request-validator

- Add maven dependency for Rest Assured.

```
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>rest-assured</artifactId>
    <scope>test</scope>
</dependency>
```

- Add Swagger validator for REST Assured

```
 <dependency>
    <groupId>com.atlassian.oai</groupId>
    <artifactId>swagger-request-validator-restassured</artifactId>
    <version>2.32.0</version>
    <scope>test</scope>
</dependency>
```

### RestAssured Test

- RestAssured libary - Testing and validating REST services in Java

```
/**
 * Created by jt, Spring Framework Guru.
 */
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class BeerControllerRestAssuredTest {

    @LocalServerPort
    Integer localPort;

    @BeforeEach
    void setUp() {
        RestAssured.baseURI = "http://localhost";
        RestAssured.port = localPort;
    }

    @Test
    void testListBeers() {
        given().contentType(ContentType.JSON)
                .when()
                .get("/api/v1/beer")
                .then()
                .assertThat().statusCode(200);
    }
}
```

### Spring Security Configuration

- We can disable Spring Security for Rest assured
- To disable that, we need to add `@Profile(!test)` on top of `SpringSecConfig`.
- Use `test` profile for Test configuration - `@ActiveProfiles('test')`

```
//SpringSecConfig.java
/**
 * Created by jt, Spring Framework Guru.
 */
@Profile("!test")
@Configuration
public class SpringSecConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests()
                .requestMatchers("/v3/api-docs**", "/swagger-ui/**",  "/swagger-ui.html")
                .permitAll()
                .anyRequest().authenticated()
                .and().oauth2ResourceServer().jwt();

        return http.build();
    }

}
```

```
// BeerControllerRestAssuredTest.java
/**
 * Created by jt, Spring Framework Guru.
 */
@ActiveProfiles("test")
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Import(BeerControllerRestAssuredTest.TestConfig.class)
@ComponentScan(basePackages = "guru.springframework.spring6restmvc")
public class BeerControllerRestAssuredTest {

    @Configuration
    public static class TestConfig {
        @Bean
        public SecurityFilterChain filterChain(HttpSecurity http) throws Exception{
            http.authorizeHttpRequests().anyRequest().permitAll();

            return http.build();
        }
    }

    @LocalServerPort
    Integer localPort;

    @BeforeEach
    void setUp() {
        RestAssured.baseURI = "http://localhost";
        RestAssured.port = localPort;
    }

    @Test
    void testListBeers() {
        given().contentType(ContentType.JSON)
                .when()
                .get("/api/v1/beer")
                .then()
                .assertThat().statusCode(200);
    }
}
```

### Configure Swagger Request Validator

- Copy the generated `openApi.yml` file (Generated using plugin), to `test/resources` dir
- Add openAPI validator in Test file `BeerControllerRestAssuredTest.java`
- Add `filter` to configure our test to use the swagger request validator filter.
  And what that filter is going to do that is going to inspect the request and response that it's going

```
@ActiveProfiles("test")
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Import(BeerControllerRestAssuredTest.TestConfig.class)
@ComponentScan(basePackages = "guru.springframework.spring6restmvc")
public class BeerControllerRestAssuredTest {

    OpenApiValidationFilter filter = new OpenApiValidationFilter(OpenApiInteractionValidator
            .createForSpecificationUrl("oa3.yml")
            .build());

    @Configuration
    public static class TestConfig {
        @Bean
        public SecurityFilterChain filterChain(HttpSecurity http) throws Exception{
            http.authorizeHttpRequests().anyRequest().permitAll();

            return http.build();
        }
    }

    @LocalServerPort
    Integer localPort;

    @BeforeEach
    void setUp() {
        RestAssured.baseURI = "http://localhost";
        RestAssured.port = localPort;
    }

    @Test
    void testListBeers() {
        given().contentType(ContentType.JSON)
                .when()
                .filter(filter)
                .get("/api/v1/beer")
                .then()
                .assertThat().statusCode(200);
    }
}
```

### Whitelist Validation Rules

- We can whitelist some errors that we get in Request validators (if something obvious), to get our test pass.

```
 OpenApiValidationFilter filter = new OpenApiValidationFilter(OpenApiInteractionValidator
            .createForSpecificationUrl("oa3.yml")
            .withWhitelist(ValidationErrorsWhitelist.create()
                    .withRule("Ignore date format",
                    messageHasKey("validation.response.body.schema.format.date-time")))
            .build());
```

### KEY TERMS

- Big proponents of doing Spec first type Development
- Also known as the `spec-first` approach
- RestAssured libary - Testing and validating REST services in Java
