## Spring Data JPA with Spring MVC

- Map Struct - Java bean mappings, the easy way!
- More - https://mapstruct.org/ || https://mapstruct.org/documentation/reference-guide/

- Spring Data JPA is an abstraction built upon JPA.
- JPA is a standard API, standard Java persistence API, Hibernate is the implementation.
- Spring Data JPA is the go to toolset for persisting data to a relational database.

- Using Java Optional is generally considered a best practice since it indicates the return value
  may be null and reduces null type checking.
- Using optional also helps reduce unintentional Null pointer errors at runtime.

### Data Transfer Objects

- DTOs - Data Transfer Objects
- DTOs are simple Java POJOs
- DTOs are data structures, generally should NOT have behavior
- DTOs are objects used to transfer data between producers and consumers
- Controller models are typically DTOs

### DTOs and Entities

- Spring MVC Controller - DTOs
- Service - DTOs
- Repository - Entities

### Why Not Entities?

- Database Entities are also POJOs, why can’t we use those?
- For simple applications you can
- Spring Data REST exposes database entities directly
- Database entities can “leak” data to client tier
- As applications become more complex, having the separation becomes more important
- The needs of the consumers are different than the needs of persistence
- DTOs can be optimized for JSON serialization and deserialization

### Type Conversions

- Type Conversions are often done within methods
- Best practice is to use dedicated converters
  - Single Responsibility Principle
- Spring Framework provides an Interface called “Converter” with generics
  - Can be used with conjunction with Conversion service
- MapStruct is a code generator which automates generation of type converters

### MapStruct

- MapStruct is a code generator
- You provide the interface, MapStruct generates the implementation
- Works like Lombok via annotation processing during code compile
- Has good Spring integration - can generate Spring Converters or Spring Components
- We will use Mapstruct Components for injection into services
- Refer - https://mapstruct.org/documentation/reference-guide/

### Spring Data JPA Denependencies

- Add Data JPA Starter Dependecies (JPA and H2 in-memory database)

```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
  <groupId>com.h2database</groupId>
  <artifactId>sh2</artifactId>
</dependency>

```

### Create Entities and Repository

- Create Entities
  - `Beer`

```
@Getter
@Setter
@Builder
@Entity
@NoArgsConstructor
@AllArgsConstructor
public class Beer {

   @Id
   @GeneratedValue(generator = "UUID")
   @GenericGenerator(name = "UUID", strategy = "org.hibernate.id.UUIDGenerator")
   @Column(length = 36, columnDefinition = "varchar", updatable = false, nullable = false)
   private UUID id;

   @Version
   private Integer version;
   private String beerName;
   private BeerStyle beerStyle;
   private String upc;
   private Integer quantityOnHand;
   private BigDecimal price;
   private LocalDateTime createdDate;
   private LocalDateTime updateDate;
}
```

- Create Repositories
  - Create interface and extends `JpaRepository<Beer, UUID>`

```
public interface BeerRepository extends JpaRepository<Beer, UUID> {}
```

### SpringBoot JPA Test Suite

- Create Test class with `@DataJpaTest`

```
@DataJpaTest
class BeerRepositoryTest {

    @Autowired
    BeerRepository beerRepository;

    @Test
    void testSaveBeer() {
        Beer savedBeer = beerRepository.save(Beer.builder()
                        .beerName("My Beer")
                .build());

        assertThat(savedBeer).isNotNull();
        assertThat(savedBeer.getId()).isNotNull();
    }
}
```

### MapStruct Dependencies and Configuration

- Add MapStruct Version
  - `<org.mapstruct.version>1.5.2.Final</org.mapstruct.version>`
- Add Dependency in POM.XML

```
<dependency>
  <groupId>org.mapstruct</groupId>
  <artifactId>mapstruct</artifactId>
  <version>${org.mapstruct.version}</version>
</dependency>
```

- Update Maven compiler Plugin details in POM.XML

```
<plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.10.1</version>
                <configuration>
                    <source>17</source>
                    <target>17</target>
                    <annotationProcessorPaths>
                        <path>
                            <groupId>org.mapstruct</groupId>
                            <artifactId>mapstruct-processor</artifactId>
                            <version>${org.mapstruct.version}</version>
                        </path>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                            <version>${lombok.version}</version>
                        </path>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok-mapstruct-binding</artifactId>
                            <version>0.2.0</version>
                        </path>
                    </annotationProcessorPaths>
                    <compilerArgs>
                        <compilerArg>-Amapstruct.defaultComponentModel=spring</compilerArg>
                    </compilerArgs>
                </configuration>
            </plugin>
```

### MapStruct Mappers

- Create a package `mapper`
- Create an Interface with both from and to methods

```
@Mapper
public interface BeerMapper {
    Beer beerDtoToBeer(BeerDTO dto);
    BeerDTO beerToBeerDto(Beer beer);

}

```

### JPA Update If Item not Found

- Background - If we look at the BeerController, we moved for the getBeerById, we are throwing
  the NotFoundException inside the controller. So we kind of said that we want to have that logic for throwing the exception inside of our controller and not in the service layer.

- Use AtomicReference

```
@Override
    public Optional<BeerDTO> updateBeerById(UUID beerId, BeerDTO beer) {
        AtomicReference<Optional<BeerDTO>> atomicReference = new AtomicReference<>();

        beerRepository.findById(beerId).ifPresentOrElse(foundBeer -> {
            foundBeer.setBeerName(beer.getBeerName());
            foundBeer.setBeerStyle(beer.getBeerStyle());
            foundBeer.setUpc(beer.getUpc());
            foundBeer.setPrice(beer.getPrice());
            atomicReference.set(Optional.of(beerMapper
                    .beerToBeerDto(beerRepository.save(foundBeer))));
        }, () -> {
            atomicReference.set(Optional.empty());
        });

        return atomicReference.get();
    }

```

### KEY TERMS

- DTOs - Data Transfer Objects
- Spring Data JPA is an abstraction built upon JPA.
- JPA is a standard API, standard Java persistence API, Hibernate is the implementation.
- MapStruct Works like Lombok via annotation processing during code compile
- Annotation processing during code compile

- Using Java Optional is generally considered a best practice since it indicates the return value
  may be null and reduces null type checking.
- Using optional also helps reduce unintentional Null pointer errors at runtime.
