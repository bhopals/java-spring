## Spring Security Basic Auth

### Add Spring Security Dependencies

- Add dependency in `POM.XML`

```
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-security</artifactId>
</dependency>

<dependency>
<groupId>org.springframework.security</groupId>
<artifactId>spring-security-test</artifactId>
</dependency>
```

- To Test

```
MvcResult mvcResult = mockMvc.perform(post(BeerController.BEER_PATH)
                        .with(httpBasic("user1", "password")))
```

### Customizing User Name and Password

- Add below in `application.properties`
  `spring.security.user.name=user1`
  `spring.security.user.password=password`

### Spring Security Config - Disable CSRF

- Add Security Configuration Class

```
/**
 * Created by jt, Spring Framework Guru.
 */
@Configuration
public class SpringSecConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.csrf().ignoringRequestMatchers("/api/**");
        return http.build();
    }

}
```

### HTTP Basic with RestTemplate

### KEY TERMS
