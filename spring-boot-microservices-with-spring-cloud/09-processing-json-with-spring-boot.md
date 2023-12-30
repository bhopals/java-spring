## Processing JSON with Spring Boot

### JavaScript Object Notation

- JSON is a lightweight data-interchange format.
- Based on the JavaScript Programming Language ECMA-262 - December 1999
- JSON is now independent of JavaScript
- Current Standard ECMA-404 - December 2017
- JSON can be read and created by all major programming languages
- JSON is an ideal data-interchange format because of its wide-spread adoption

### Processing JSON

- Serialization - is the process of converting a Java object to a JSON object
- De-serialization - is the process of converting a JSON object into a Java object
- The JSON is typically a string, possibly a buffer(web request) or a file from the file system
- In an HTTP GET action, Spring will read data into POJOs, serialize the POJO into a
  JSON string, then deliver the JSON payload to the client.
- In an HTTP POST action, the client will be using HTTP to post, Spring will read the body
  of the request and De-serialize the JSON payload into a Java POJO
- Can also be used for messaging

### Spring Boot JSON Libraries

- Spring Boot supports 3 JSON mapping libraries
  - Jackson - Spring Boot Default - most popular
  - GSON - Google’s library, very popular
  - JSON-B - JEE API Standard (JSR 367)
    - Providing an API standard for JSON binding
    - Adoption is unclear, momentum seems weak in the Java community
- Spring Boot will auto configure any of these 3 libraries

### Performance and Security

- Performance
  - Jackson tends to benchmark very well
  - GSON benchmarks well also, beating Jackson in some aspects
- Security
  - Keep an eye on security alerts, can be problematic
- Other
  - Jackson is popular - there is a lot of free help on the internet

### Common Jackson Annotations

- `@JsonProperty` - Allows you to set the property name
- `@JsonFormat` - Gives you control over how property is serialized. (Helpful for dates)
- `@JsonUnwrapped` - Allows you to flatten a property
- `@JsonView` - Allows you to configure virtual ‘views’ of objects
- `@JsonManagedReference`, `@JsonBackReference` - for mapping embedded items
- `@JsonIdentityInfo` - Allows you to specify a property to determine object identity
- `@JsonFilter` - Used to specify a programatic property filter

### Jackson Serialization Annotations

- `@JsonAnyGetter` - Takes a Map property and serializes the key to property names,
  values to values
- `@JsonGetter` - Allows you to identify a method as a ‘getter’, which will be serialized
  into a property
- `@JsonPropertyOrder` - Allows you to set the order of properties in the serialized
  JSON output
- `@JsonRawValue` - Will serialize the string value of the property as is. Warning - Can
  lead to invalid JSON.

- `@JsonValue` - Used to mark a method for the Serialization output. Will throw
  exception if more than one method is annotated. Warning: May produce invalid JSON.
- `@JsonRootName` - Creates a root element for the object.
- `@JsonSerialize` - Allows you to specify a custom serializer

### Jackson Deserialization Annotations

- `@JsonCreator` - Use identify the object constructor to use, and property binding
- `@JacksonInject` - Allows you to inject values into properties using deserialization
- `@JsonAnySetter` - Converts JSON object into a Java Map object, where property names
  are keys of the map.
- `@JsonSetter` - Identifies a Java method to use as a setter for the identified JSON property.
- `@JsonDeserialize` - Allows you to specify a custom deserializer

### Other Jackson Annotations

- `@JsonIgnoreProperties` - Class level annotation used to tell Jackson to ignore one
  or more property
- `@JsonIgnore` - Field level annotation use to tell Jackson to ignore a property
- `@JsonIgnoreType` - Class level annotation used to ignore class - typically used
  to ignore class, where the class is a property in other classes.
- `@JsonInclude` - Class level annotation used to control how null values are presented
  in serialization
- `@JsonAutoDetect` - Jackson will use reflection in serialization. This annotation allows
  you to tune which properties / methods are detected.

### Jackson and Spring Boot

- Spring Boot will like most things, auto configure Jackson with ‘sensible’ defaults
- The Jackson object mapper accepts a large variety of properties which can be
  used to customize its behavior
- The majority of configuration values can be adjusted via configuration properties
  in application.properties.
- If needed, you can override Spring Boot and provide a custom definition of the Jackson
  Object mapper via a Spring Bean.

### JSON Testing with Spring Boot

```
@JsonTest
class BeerDtoTest extends BaseTest{
    @Autowired
    ObjectMapper objectMapper;
    @Test
    void testSerializeDto() throws JsonProcessingException {
        BeerDto beerDto = getDto();
        String jsonString = objectMapper.writeValueAsString(beerDto);
        System.out.println(jsonString);
    }

    @Test
    void testDeserialize() throws IOException {
        String json = "{\"id\":\"639c00ff-9993-4ec1-af12-8277c85f93bb\",\"beerName\":\"BeerName\",\"beerStyle\":\"Ale\",\"upc\":123123123123,\"price\":12.99,\"createdDate\":\"2019-06-02T16:35:58.321001-04:00\",\"lastUpdatedDate\":\"2019-06-02T16:35:58.321872-04:00\"}";
        BeerDto dto = objectMapper.readValue(json, BeerDto.class);
        System.out.println(dto);
    }
}
```

### Jackson Property Naming Strategies

- Example - Enable Snake Case on all the properties

- 1. create a properties file `application-snake.properties`

  - Whenever we create a property files with `application-<name>.properties`, that `<name>`
    becomes a PROFILE.

    - `test/resources/application-snake.properties`
      - `spring.jackson.property-naming-strategy=SNAKE_CASE`
    - `test/resources/application-kebak.properties`
      - `spring.jackson.property-naming-strategy=KEBAB_CASE`

- Annotate the Class with Profile - `@ActiveProfiles("kebab")`

```
@ActiveProfiles("kebab")
@JsonTest
public class BeerDtoKebabTest extends BaseTest {

    @Test
    void testKabab() throws JsonProcessingException {
        BeerDto dto = getDto();

        String json = objectMapper.writeValueAsString(dto);

        System.out.println(json);
        // beerName ==> beer-name
    }
}
```

- Annotate the Class with Profile - `@ActiveProfiles("snake")`

```
@ActiveProfiles("snake")
@JsonTest
public class BeerDtoSnakeTest extends BaseTest {

    @Test
    void testSnake() throws JsonProcessingException {
        BeerDto dto = getDto();

        String json = objectMapper.writeValueAsString(dto);

        System.out.println(json);
        // beerName ==> beer_name
    }
}
```

### Setting Property Names with Jackson

- Annotate Class property with `@JsonProperty("beerId") private UUID id;`

- Using @JsonFormat with Jackson

  ```
  @JsonFormat(shape= JsonFormat.Shape.STRING)
  private BigDecimal price;

  @JsonFormat(pattern="yyyy-MM-dd'T'HH:mm:ssZ", shape=JsonFormat.Shape.STRING)
  private OffsetDateTime createdDate;
  ```

- Custom Serializer with Jackson

  - Create a class that extends `JsonSerializer<TypeOfProperty>` - override `serialize`

  ```
  public class LocalDateSerializer extends JsonSerializer<LocalDate> {
        @Override
        public void serialize(LocalDate value, JsonGenerator jsonGenerator, SerializerProvider serializers) throws IOException {
            jsonGenerator.writeObject(value.format(DateTimeFormatter.BASIC_ISO_DATE));
        }
    }
  ```

  - Annotate the property with the above Class

  ```
  @JsonSerialize(using = LocalDateSerializer.class)
  private LocalDate myLocalDate;
  ```

- Custom Deserializer with Jackson

  - Create a class that extends `StdDeserializer<TypeOfProperty>` - override `deserialize`

    ```
    public class LocalDateDeserializer extends StdDeserializer<LocalDate> {
        public LocalDateDeserializer() {
            super(LocalDate.class);
        }
        @Override
        public LocalDate deserialize(JsonParser p, DeserializationContext ctxt) throws IOException, JsonProcessingException {
            return LocalDate.parse(p.readValueAs(String.class), DateTimeFormatter.BASIC_ISO_DATE);
        }
    }
    ```

    - Annotate the property with the above Class

    ```
    @JsonDeserialize(using = LocalDateDeserializer.class)
    private LocalDate myLocalDate;
    ```

  - Both Serialize/Deserilize on the same property

  ```
    @JsonSerialize(using = LocalDateSerializer.class)
    @JsonDeserialize(using = LocalDateDeserializer.class)
    private LocalDate myLocalDate;
  ```

### Jackson JSON Creator

Jackson JSON Creator for a paged object. So sometimes you need to consume a paging resource from Spring Framework. if you're talking to another service, and then bringing that JSON in,
so that paged object.

We are giving Jackson the capability to talk to another Spring Framework service where
it has a page property and then bind to a BeerPageList.

```
public class BeerPagedList extends PageImpl<BeerDto> implements Serializable {

    static final long serialVersionUID = 1114715135625836949L;

    @JsonCreator(mode = JsonCreator.Mode.PROPERTIES)
    public BeerPagedList(@JsonProperty("content") List<BeerDto> content,
                         @JsonProperty("number") int number,
                         @JsonProperty("size") int size,
                         @JsonProperty("totalElements") Long totalElements,
                         @JsonProperty("pageable") JsonNode pageable,
                         @JsonProperty("last") boolean last,
                         @JsonProperty("totalPages") int totalPages,
                         @JsonProperty("sort") JsonNode sort,
                         @JsonProperty("first") boolean first,
                         @JsonProperty("numberOfElements") int numberOfElements) {

        super(content, PageRequest.of(number, size), totalElements);
    }

    public BeerPagedList(List<BeerDto> content, Pageable pageable, long total) {
        super(content, pageable, total);
    }

    public BeerPagedList(List<BeerDto> content) {
        super(content);
    }
}
```

### KEY TERMS

- JSON is an ideal data-interchange format because of its wide-spread adoption
- Ideal Data-interchange Format
- Jackson uses reflection to serialize Java objects
