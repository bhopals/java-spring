## Spring MVC Rest Docs

- To Generate API Documentation

- Spring REST Docs, It's actually a fairly clever tool. It hooks into your tests for
  your controllers. So by writing your tests, you also generate your API documentation and if something changes in your API, it can potentially break the test and also your documentation. So it's actually a pretty clever way to keep your API is accurate and reflecting what the controller is actually expecting.

- Spring REST Docs hooks into controller tests to generate documentation snippets
- The snippets are then assembled into final documentation via Asciidoctor

- Spring REST Docs - Code to ==> Document snippets
- Asciidoctor - Document snippets ==> Final Documentation

- Tests ==> SNIPPETS ==> AsciiDoctor ==> Final Documentation

### Spring REST Docs

- What is it? - A tool for generating API documentation
- Developed by Andy Wilkinson of Pivotal
- Spring REST Docs hooks into controller tests to generate documentation snippets
- The snippets are then assembled into final documentation via Asciidoctor
- Asciidoctor - Asciidoctor is a text processor for converting AsciiDoc documents into HTML, PDF and other formats.

- Test Clients Supported:

  - Spring MVC Test
  - WebTestClient (Webflux)
  - REST Assured

- Test Framework Supported

  - JUnit 5
  - JUnit 4
  - Spock
  - Test NG (additional configuration required)

- Default Snippets
  - curl-request
  - http-request
  - http-response
  - httpie-request
  - request-body
  - response-body

### Spring REST Doc Options

- Can Optionally use Markdown rather than Asciidoctor
- Maven and Gradle maybe used for the build process
- You can package the documentation to be served as static content via Spring Boot
- Extensive options for customizing Asciidotor output

### Third Party Extensions

- restdocs-wiremock - Auto generate WireMock Stubs
- restdocsext-jersey - Enable use of REST Docs with Jersey’s test framework
- spring-auto-restdocs - Use reflection to automatically document req and res parameters
- restdocs-api-spec - Generate OpenAPI 2 and OpenAPI 3 specifications

### Project Code Review

- Repo - https://github.com/bhopals/sfg-restdocs-example

#### Maven Configuration

- Add Spring Rest Docs dependency

```
<dependency>
    <groupId>org.springframework.restdocs</groupId>
    <artifactId>spring-restdocs-mockmvc</artifactId>
    <scope>test</scope>
</dependency>
```

- Add AsciiDoctor Maven Plugin

```
<plugin>
    <groupId>org.asciidoctor</groupId>
    <artifactId>asciidoctor-maven-plugin</artifactId>
    <version>2.2.2</version>
    <executions>
        <execution>
            <id>generate-docs</id>
            <phase>prepare-package</phase>
            <goals>
                <goal>process-asciidoc</goal>
            </goals>
            <configuration>
                <backend>html</backend>
                <doctype>book</doctype>
            </configuration>
        </execution>
    </executions>
    <dependencies>
        <dependency>
            <groupId>org.springframework.restdocs</groupId>
            <artifactId>spring-restdocs-asciidoctor</artifactId>
            <version>${spring-restdocs.version}</version>
        </dependency>
    </dependencies>
    </plugin>
```

- Create `src/main/asciiadoc` file named `index.adoc` with below default content

```
= SFG Brewery Order Service Docs
John Thompson;
:doctype: book
:icons: font
:source-highlighter: highlightjs

Sample application demonstrating how to use Spring REST Docs with JUnit 5.

`BeerOrderControllerTest` makes a call to a very simple service and produces three
documentation snippets.

One showing how to make a request using cURL:

include::{snippets}/orders/curl-request.adoc[]

One showing the HTTP request:

include::{snippets}/orders/http-request.adoc[]

And one showing the HTTP response:

include::{snippets}/orders/http-response.adoc[]

Response Body:
include::{snippets}/orders/response-body.adoc[]


Response Fields:
include::{snippets}/orders/response-fields.adoc[]
```

So the test documentation is gonna go out and generate snippets and then the asciidoctor
is gonna assemble things into a single document and then ultimately use this to produce an HTML document so it takes the many little pieces of code snippet and then assembles them into a single document.

#### Configure Spring Mock MVC Configuration

- Add required annotations on Test Controller - BeerControllerTest
  ```
  @ExtendWith(RestDocumentationExtension.class)
  @AutoConfigureRestDocs(uriScheme = "https", uriHost = "dev.springframework.guru", uriPort = 80)
  @WebMvcTest(BeerController.class)
  ```
- Add autowires `MockMvc`

  ```
  @Autowired
  MockMvc mockMvc;
  ```

- Add `doDoc` methods in Test cases

```
mockMvc.perform(get("/api/v1/beer/{beerId}", UUID.randomUUID().toString())
                .param("iscold", "yes")
                .accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andDo(document("v1/beer-get",
                        pathParameters (
                                parameterWithName("beerId").description("UUID of desired beer to get.")
                        ),
                        requestParameters(
                                parameterWithName("iscold").description("Is Beer Cold Query param")
                        ),
                        responseFields(
                                fieldWithPath("id").description("Id of Beer"),
                                fieldWithPath("version").description("Version number"),
                                fieldWithPath("createdDate").description("Date Created"),
                                fieldWithPath("lastModifiedDate").description("Date Updated"),
                                fieldWithPath("beerName").description("Beer Name"),
                                fieldWithPath("beerStyle").description("Beer Style"),
                                fieldWithPath("upc").description("UPC of Beer"),
                                fieldWithPath("price").description("Price"),
                                fieldWithPath("quantityOnHand").description("Quantity On hand")
                        )));
```

#### Documenting Validation Constraints

- Add a new file in `test/resources/org/springframework/restdocs/templates` with name
  `request.fields.snippet`

```
|===
|Path|Type|Description|Constraints

{{#fields}}
|{{path}}
|{{type}}
|{{description}}
|{{constraints}}

{{/fields}}
|===
```

- Create a inner class in `BeerControllerTest`

```
private static class ConstrainedFields {

        private final ConstraintDescriptions constraintDescriptions;

        ConstrainedFields(Class<?> input) {
            this.constraintDescriptions = new ConstraintDescriptions(input);
        }

        private FieldDescriptor withPath(String path) {
            return fieldWithPath(path).attributes(key("constraints").value(StringUtils
                    .collectionToDelimitedString(this.constraintDescriptions
                            .descriptionsForProperty(path), ". ")));
        }
    }
```

- Create an instance of the above class by passing DTO.class
  `ConstrainedFields fields = new ConstrainedFields(BeerDto.class);`

- Use instance `fields.withPath` method

```
mockMvc.perform(post("/api/v1/beer/")
    .contentType(MediaType.APPLICATION_JSON)
    .content(beerDtoJson))
    .andExpect(status().isCreated())
    .andDo(document("v1/beer-new",
            requestFields(
                    fields.withPath("id").ignored(),
                    fields.withPath("version").ignored(),
                    fields.withPath("createdDate").ignored(),
                    fields.withPath("lastModifiedDate").ignored(),
                    fields.withPath("beerName").description("Name of the beer"),
                    fields.withPath("beerStyle").description("Style of Beer"),
                    fields.withPath("upc").description("Beer UPC").attributes(),
                    fields.withPath("price").description("Beer Price"),
                    fields.withPath("quantityOnHand").ignored()
            )));
```

- Now Run the Test case, and it should generate snippets under
  `target/generated-snippets/v1/beer/` => \*.adoc files

#### URI Customization

- Use this Annotation on Controller Test Class
  `@AutoConfigureRestDocs(uriScheme = "https", uriHost = "dev.springframework.guru", uriPort = 80)`

#### Documentation Generation

- Go to `target/generated-snippets/v1/beer/` => \*.adoc files and copy details from
  those fileds to `src/main/asciidoc/index.adoc` (main) file

- Now go to `target/generated-docs/index.html`, and open this is the browser
  to see the generated documention

#### Serving Doc with Spring Boot

- Publish Rest Docs with Spring Boot
- We have below two files generated

  - `target/classes/guru/application.properties`
  - `target/classes/generated-docs/index.html`

- Now, we need to copy `index.html` into `target/classes/guru/` dir
- We can either do it manually, OR we can update mvn resoure plugin to copy this for us

```
 <plugin>
                <artifactId>maven-resources-plugin</artifactId>
                <version>2.7</version>
                <executions>
                    <execution>
                        <id>copy-resources</id>
                        <phase>prepare-package</phase>
                        <goals>
                            <goal>copy-resources</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>
                                ${project.build.outputDirectory}/static/docs
                            </outputDirectory>
                            <resources>
                                <resource>
                                    <directory>
                                        ${project.build.directory}/generated-docs
                                    </directory>
                                </resource>
                            </resources>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
```

### KEY TERMS

- Hooks into controller tests to generate documentation snippets
- The snippets are then assembled into final documentation via Asciidoctor

- Spring Data Docs
- Asciidoctor - Asciidoctor is a text processor for converting AsciiDoc documents into HTML, PDF and other formats.

- Differenece between OpenAPI 2.0 and OpenAPI 3.0

  - https://blog.stoplight.io/difference-between-open-v2-v3-v31

- Spring REST Docs - Code to ==> Document snippets
- Asciidoctor - Document snippets ==> Final Documentation
