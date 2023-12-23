## Introduction to Spring Data REST

Spring data Rest is a way for us to expose our entities through RESTful Web services with absolutely minimal configuration.

- Example API - https://sfg-beer-works.github.io/brewery-api/#tag/Beer-Service/operation/listBeers
- Spring Data REST API - https://spring.io/projects/spring-data-rest/
- Spring Data JDBC and R2DBC - https://docs.spring.io/spring-data/relational/reference/#conditional.etag

### Add Spring Data REST Dependency

- Add this in POM.XML

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-rest</artifactId>
</dependency>
```

- Set Base Path in `application.properties`
  `spring.data.reset.base-path=/api/v1/`

- Customize URL Path

  - Annonate Repository - `@RepositoryRestResource(path='beer')`
  - https://docs.spring.io/spring-data/rest/reference/customizing/configuring-the-rest-url-path.html

- Spring Data follows HATEOS

- Version Property - ETag Header

  - The `ETag` Header provides a way to tag resources. This can pervent clients from overriding
    each other while also making it possible to reduce unnecessary calls.

    `@Version Long version; `

  - The @Version annotation (the JPA one in case you’re using Spring Data JPA, the Spring Data
    org.springframework.data.annotation.Version one for all other modules) flags this field as
    a version marker.

- Create / Update / Delete

  - You don't need to create a new resource/mapping. Just use the BASE URL (for create), and use POST
    instead of GET (all) to create

  - For Update, beer/id, instead of GET, use PUT with new Payload
  - For Delete, beer/id, instead of GET, use DELETE with new Payload

- Use Repository Methods
  - To search
    - Create a method `findBy<field-name>`
    - Use URL - `/api/v1/beer/serach/findBy<field-name>?<field-name>=<value>`

### KEY TERMS

- HATEOAS (Hypermedia as the Engine of Application State) is a constraint of the
  REST application architecture.
- HATEOAS keeps the REST style architecture unique from most other network application architectures.
- REST architectural style lets us use the hypermedia links in the API response contents. It allows
  the client to dynamically navigate to the appropriate resources by traversing the hypermedia links.
