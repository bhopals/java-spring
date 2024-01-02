## Securing Spring Cloud

- Security is a very large topic
- Larger than just a technology topic
- Securing applications involves technology solutions, adopting best practices,
  personal, and physical security
- Security often involves analyzing the risk vector then implementing mitigating actions

### Common Terminology

- `PII` - Personally Identifiable Information - name, address, email, tax ids, etc
- `Encryption at Rest` - Sensitive data needs to be encrypted when
  stored (database, filesystem, backup tapes, etc)
- `Encryption in Flight` - When transmitted, sensitive data needs to be
  encrypted - can be protocol (https, ssh, etc)
- `Segregation of Duties` - Avoid having powerful super users in organization
- `Processes and Controls` - Be able to document compliance (source control, issue management)

#### Security Audit Frameworks / Certifications

- `PCI-DSS` - Payment Card Industry Data Security Standard
  - Applicable if your organization processes credit / debit cards
- `SOX` - Sarbanes-Oxley
  - For US based publicly traded companies
- `HIPAA` - Health Insurance Portability and Accountability Act
  - US Based Medical Industry
- `SSAE 16` - Statement on Standards for Attestation Engagements (SSAE) No. 16
  - `CPA` - authoritative guidance for reporting on service organizations

### PCI DSS Requirements

- 1. Protect System with Firewalls
- 2. Configure Passwords and Settings - Don’t use defaults
- 3. Protect Stored Cardholder Data - Use Industry accepted algorithms, don’t roll your own!
- 4. Encrypt Transmission of Cardholder Data across open, public networks
- 5. Use and update anti-virus software
- 6. Regularly update and patch systems
- 7. Restrict access to card holder data by business need to know
- 8. Assign Unique Id to each person with computer access
- 9. Restrict physical access to workplace and cardholder data
- 10. Implement logging and log management
- 11. Conduct vulnerability scans and penetration tests
- 12. Documentation and risk assessments

### Other Best Practices

- Use OS Service Accounts for Applications
  - Service accounts should have minimal access needed
- Use database Service Accounts with minimal access
  - Application account should not have access to alter or drop database tables
- Use layers of network security to protect internal systems
  - ie - should not be able to reach database server from internet edge
  - VPCs, VPNs, multiple physical networks

### Configuration Property Encrypt / Decrypt

#### At Rest Encryption with Spring Cloud Config

- Spring Cloud Configuration supports property encryption and decryption
- Should be used for sensitive data such as passwords
- Java Cryptography Extension (JCE)
  - Prior to Java 8 u162 required additional setup
  - Older examples will refer to download and install of JCE
  - Included by default in Java 11+

#### Configuration

- Spring Cloud Configuration will store encrypted properties as:
  - {cipher}<your encrypted value here>
- When a Spring Cloud Config client requests an encrypted property the value is
  decrypted and presented to the client in the request
- Must set a symmetric key in property ‘encrypt.key’ - should prefer setting this
  as environment variable
- asymmetric (public / private) keys also supported - see documentation for details

#### Encryption / Decryption

- Spring Cloud Configuration provides endpoints for property encryption / decryption

  - POST /encrypt - will encrypt body of post
  - POST /decrypt - will decrypt body of post

- Configuration
  - Add encrypt key in bootstrap properties of Cloud Config project
    // should be env. variable
    `encrypt.key=MySuperSecretKey`
  - Encrpty - `POST http://localhost:8888/encrypt` - Add Body params that we want to Encrypt
  - Decrpty - `POST http://localhost:8888/decrypt` - Add Body params that we want to Decrypt

#### Encrypt Beer Service Passwords

- In application properties, we have credentials for JMS (artemis) and SQL

```
spring.artemis.user=artemis
spring.artemis.password=simetraehcapa

spring.datasource.username=beer_service
spring.datasource.password=password
```

- Create `application-local-secure.properties` in Config Repo (Centralized Confi Repo)
- Encrpty Password using - `POST http://localhost:8888/encrypt`

```
spring.artemis.user=artemis
spring.artemis.password={cipher}<encrytped-password>

spring.datasource.username=beer_service
spring.datasource.password={cipher}<encrytped-password>
```

- Query Config server to retrieve all properties
  - GET `http://localhost:8888/beer-service/local-secure`
  - You will notice, the returned credentials are decrypted (because of `{cipher}`)

### Secure Spring Cloud Config Server

- Using Basic Authentication
- Add Dependency in cloud config server `POM.XML`

```
 <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

- Add user credential details in Cloud Config application properties

```
spring.security.user.name=MyUserName
spring.security.user.password=MySecretPassword
```

- Add same user credential details in Micro Service bootstrap properties

```
spring.cloud.config.fail-fast=true
spring.cloud.config.username=MyUserName
spring.cloud.config.password=MySecretPassword
```

Now when we make a call from Service to Cloud Config, it automatically uses above
credentials and validate against it.

### Use Spring security to secure Eureka Server

Add HTTP Basic Auth Using Spring Security

- Add Dependency in cloud config server `POM.XML`

```
 <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

- Add user credential details in Eureka application properties

```
spring.security.user.name=netflix
spring.security.user.password=eureka
```

- Add Spring Security Configuration

```
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
            .authorizeRequests()
            .anyRequest().authenticated()
            .and()
            .httpBasic();
    }
}
```

- Restart the application, and go to `http://localhost:8761`
- Enter the above credentials to login

- To enable cloud config to talk to Eureka, add eureka details in Config server
  bootstrap properties (So cloud config can authenticate and Register itself in Eureka) -
  `eureka.client.service-url.defaultZone=http://netflix:eureka@localhost:8761/eureka`

### Secure Inventory Service using Spring Security (HTTP Basic)

- Add Dependency in cloud config server `POM.XML`

```
 <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

- Add security Config

```
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
            .authorizeRequests()
            .anyRequest().authenticated()
            .and()
            .httpBasic();
    }
}
```

- Add user credential details in application properties

```
spring.security.user.name=good
spring.security.user.password=beer
```

### Configuer RESTTemplate for HTTP Basic Authentication

- Add username and password in application properties - Beer Service

```
sfg.brewery.inventory-user=good
sfg.brewery.inventory-password=beer
```

- Inject credentials into RestTemplate for HTTP Basic Authentication

```
@ConfigurationProperties(prefix = "sfg.brewery", ignoreUnknownFields = true)
@Component
public class BeerInventoryServiceRestTemplateImpl implements BeerInventoryService {

public BeerInventoryServiceRestTemplateImpl(RestTemplateBuilder restTemplateBuilder,
                                                @Value("${sfg.brewery.inventory-user}") String inventoryUser,
                                                @Value("${sfg.brewery.inventory-password}")String inventoryPassword) {
        this.restTemplate = restTemplateBuilder
                .basicAuthentication(inventoryUser, inventoryPassword)
                .build();
    }
}
```

### Configuer Feign Client for HTTP Basic Authentication

- Add username and password in application properties (if not already been) - Beer Service

```
sfg.brewery.inventory-user=good
sfg.brewery.inventory-password=beer
```

- Inject credentials into RestTemplate for HTTP Basic Authentication

```
@Configuration
public class FeignClientConfig {

    @Bean
    public BasicAuthRequestInterceptor basicAuthRequestInterceptor( @Value("${sfg.brewery.inventory-user}") String inventoryUser,
                                                                    @Value("${sfg.brewery.inventory-password}")String inventoryPassword){
        return new BasicAuthRequestInterceptor(inventoryUser, inventoryPassword);
    }
}

```

### Feign v/s RestTemplate

RestTemplate is used for making the synchronous call. When using RestTemplate, the URL parameter is constructed programmatically, and data is sent across to the other service. In more complex scenarios, we will have to get to the details of the HTTP APIs provided by RestTemplate or even to APIs at a much lower level.

Feign is a Spring Cloud Netflix library for providing a higher level of abstraction over REST-based service calls. Spring Cloud Feign works on a declarative principle. When using Feign, we write declarative REST service interfaces at the client, and use those interfaces to program the client. The developer need not worry about the implementation

More - https://stackoverflow.com/questions/46884362/what-are-the-advantages-and-disadvantages-of-using-feign-over-resttemplate

#### Security Retrospective

- Problems with HTTP Basic Authentication
  - Eureka Client Sending id/password free text
  - Other clients send in Header with base64 encoding
    - Easily converted to plain text
  - Sent in each request - thus more exposure to attack
  - HTTPS can be used to mitigate security risk with HTTP Basic

### Encryption in Flight

- HTTPS for RESTFul services
  - Should be a `must` for public networks
  - Should be highly considered for internal networks
    - There is a CPU cost to using HTTPS - but less of concern than it was in the past
- Use encrypted connections for network communication to databases and message brokers
  - Typically is not, options vary per implementation

### Encryption at Rest

- Consider using encrypted file systems
- Databases and message brokers have encryption options for data stored
- Don't overlook backups. Backup files should also be encrypted

### KEY TERMS

- mitigate threats

- PII (Personally Identifiable Information)
- Encryption at Rest
- Encryption at Flight
- Segregation of Duties
- Process and Controls

- Securing application involves
  - technology solutions, adopting best practices, personal, and physical security
- Implement mitigating actions

- JCE (Java Cryptography Extension)
- Network tunnel to the client using HTTPs
