## Spring MockMVC Test with Mockito

#### Testing Terminology

- Unit Tests / Unit Testing - Code written to test code under test
- Designed to test specific sections of code
- Percentage of lines of code tested is code coverage
- Ideal coverage is in the 70-80% range
- Should be ‘unity’ and execute very fast
- Should have no external dependencies
- ie no database, no Spring context, etc

#### Types of Tests

- Integration Tests - Designed to test behaviors between objects and parts of the overall system
  - Much larger scope
  - Can include the Spring Context, database, and message brokers
  - Will run much slower than unit tests
- Functional Tests - Typically means you are testing the running application
  - Application is live, likely deployed in a known environment
  - Functional touch points are tested - (i.e. Using a web driver, calling web
    services, sending / receiving messages, etc)

#### Testing Hierarchy

- UNIT TESTS ==> INTEGRATION TESTS ==> FUNCTIONAL TESTS (Order - Bottom to Top)

- All three types of tests play important roles for software quality
- The majority of tests should be UNIT Tests
  - The foundation of your testing strategy
  - Small, fast, light weight tests
  - Very detailed and specific
- INTEGRATION Tests should be next largest category
- FUNCTIONAL Tests are smallest and least detailed of the categories.

#### Testing Spring MVC Controllers is Tricky

- Spring MVC Controllers are tricky to test property
- Controllers have a high degree of integration with the Spring MVC Framework
- Request path and HTTP Method decides which method to invoke
- Path variables are parsed from path
- JSON is bound to POJOs
- Response is expressed as a HTTP Response
- JUnit tests are NOT sufficient to test the framework interaction

- The reason this is tricky because Framework is doing a lots of work with Request/Response.
- Hence, it is tough to test Framework interaction

#### Testing Spring MVC Controllers is Tricky

- `Spring Mock MVC` is a testing environment for the testing of Spring MVC controllers
  - Provides mocks of the Servlet runtime environment
    - HTPP Request / Response, Dispatcher Servlet, etc
    - Simulates the execution of controller as if it was running under Spring within Tomcat
- Can be run with or without the Spring Context
- True unit test when run without the Spring Context
- Technically an Integration Test when used in conjunction with Spring Context

#### Spring Boot Test Splices

- Spring Boot supports a concept of what is called Test Splices
- Test Splices bring up a targeted segment of the Auto-Configured Spring Boot Environment
  - ie - Just the Database components; or just the web components
  - User defined Spring beans typically are NOT initialized
- @WebMvcTest - is a Spring Boot test splice which creates a MockMVC environment for
  the controller (or controllers) under test
  - Dependencies of controllers are NOT included

#### Using Mocks

- Controller dependencies must be added to the Spring Context in the test environment
- Dependencies can be any proper implementation
  - Example of why we code to an interface, any implementation will work
  - We could easily use the hash map implementation we’ve been using in the course
- For testing, it is common to use mock objects
- Mocks allow you to supply a specific response for a given input
  - ie - when method abcd is called, return foo…

#### What is Mockito?

- Mockito is the most popular mocking framework for testing Java
- Mocks (aka Test Doubles) are alternate implementations of objects to replace real objects in tests
- Works well with Dependency Injection
- For the class under test, injected dependencies can be mocks

#### Types of Mocks (aka Test Doubles)

- Dummy - Object used just to get the code to compile
- Fake - An object that has an implementation, but not production ready
- Stub - An object with pre-defined answers to method calls
- Mock - An object with pre-defined answers to method calls, and has expectations
  of executions. Can throw an exception if an unexpected invocation is detected
- Spy - In Mockito Spies are Mock like wrappers around the actual object

#### Important Terminology

- Verify - Used to verify number of times a mocked method has been called
- Argument Matcher - Matches arguments passed to Mocked Method & will allow or disallow
- Argument Captor - Captures arguments passed to a Mocked Method
  - Allows you to perform assertions of what was passed in to method

#### Testing Controllers with Mocks

- `Argument captors` can be used to verify request data is properly being parsed
  and passed to service layer
- `Verify` interactions can be used Mocked object was called
- `Mock` return values supply data back to controller
  - ie - object returned when getById is called on service
- `Mocks` can also be instructed to throw exceptions to test exception handling

#### Testing Controllers with MockMVC & Mockito

JUnit Test <==> MockMVC <==> Controller Under Test <==> Mockito

The request will come in, will initiate in the unit test via mock MVC. The controller will handle the request, ultimately calling the Mockito mock via the injected service that we'll be setting up. Mockito will provide the response back that will return to controller and then Mock MVC will capture the response that was generated by the controller and Spring framework. And then finally the JUnit test that is providing the test framework that to actually perform the execution and capture the test results.

- Example

```
@WebMvcTest(BeerController.class)
class BeerControllerTest {

    @Autowired
    MockMvc mockMvc;

    @MockBean
    BeerService beerService;

    @Test
    void getBeerById() throws Exception {
        mockMvc.perform(get("/api/v1/beer/" + UUID.randomUUID())
            .accept(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk());
    }
}
```

#### Using JSON Matchers

- Using `jsonPath` library to match field values of data received from the API.
- https://github.com/json-path/JsonPath

```
package guru.springframework.spring6restmvc.controller;

import guru.springframework.spring6restmvc.model.Beer;
import guru.springframework.spring6restmvc.services.BeerService;
import guru.springframework.spring6restmvc.services.BeerServiceImpl;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.hamcrest.core.Is.is;
import static org.mockito.BDDMockito.given;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(BeerController.class)
class BeerControllerTest {

    @Autowired
    MockMvc mockMvc;

    @MockBean
    BeerService beerService;

    BeerServiceImpl beerServiceImpl = new BeerServiceImpl();

    @Test
    void getBeerById() throws Exception {
        Beer testBeer = beerServiceImpl.listBeers().get(0);

        given(beerService.getBeerById(testBeer.getId())).willReturn(testBeer);

        mockMvc.perform(get("/api/v1/beer/" + testBeer.getId())
                .accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(content().contentType(MediaType.APPLICATION_JSON))
                .andExpect(jsonPath("$.id", is(testBeer.getId().toString())))
                .andExpect(jsonPath("$.beerName", is(testBeer.getBeerName())));
    }
}
```

- List Beers

```
 @Test
    void testListBeers() throws Exception {
        given(beerService.listBeers()).willReturn(beerServiceImpl.listBeers());

        mockMvc.perform(get("/api/v1/beer")
                .accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(content().contentType(MediaType.APPLICATION_JSON))
                .andExpect(jsonPath("$.length()", is(3)));
    }
```

#### Create JSON Using Jackson

- To create Dummy JSON objects - using `ObjectMapper`

```
    @Autowired
    ObjectMapper objectMapper;

    @MockBean
    BeerService beerService;

    BeerServiceImpl beerServiceImpl = new BeerServiceImpl();

    @Test
    void testCreateNewBeer() throws JsonProcessingException {
        Beer beer = beerServiceImpl.listBeers().get(0);
        System.out.println(objectMapper.writeValueAsString(beer));

    }
```

#### MockMVC Test Create Beer

```
@Test
    void testCreateNewBeer() throws Exception {
        Beer beer = beerServiceImpl.listBeers().get(0);
        beer.setVersion(null);
        beer.setId(null);

        ////import static org.mockito.ArgumentMatchers.any;
        //// import static org.mockito.BDDMockito.given;
        given(beerService.saveNewBeer(any(Beer.class))).willReturn(beerServiceImpl.listBeers().get(1));

        mockMvc.perform(post("/api/v1/beer")
                .accept(MediaType.APPLICATION_JSON)
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(beer)))
                .andExpect(status().isCreated())
                .andExpect(header().exists("Location"));
    }

```

#### MockMVC Test Update Beer

```
@Test
void testUpdateBeer() throws Exception {
    Beer beer = beerServiceImpl.listBeers().get(0);
    mockMvc.perform(put("/api/v1/beer/" + beer.getId())
            .accept(MediaType.APPLICATION_JSON)
            .contentType(MediaType.APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(beer)))
            .andExpect(status().isNoContent());
    verify(beerService).updateBeerById(any(UUID.class), any(Beer.class));
}
```

#### MockMVC Test Delete Beer

```
@Test
void testDeleteBeer() throws Exception {
    Beer beer = beerServiceImpl.listBeers().get(0);
    mockMvc.perform(delete("/api/v1/beer/" + beer.getId())
            .accept(MediaType.APPLICATION_JSON))
            .andExpect(status().isNoContent());

    ArgumentCaptor<UUID> uuidArgumentCaptor = ArgumentCaptor.forClass(UUID.class);
    verify(beerService).deleteById(uuidArgumentCaptor.capture());
    assertThat(beer.getId()).isEqualTo(uuidArgumentCaptor.getValue());
}
```

#### MockMVC Test Patch Beer

```
 @Test
void testPatchBeer() throws Exception {
    Beer beer = beerServiceImpl.listBeers().get(0);

    Map<String, Object> beerMap = new HashMap<>();
    beerMap.put("beerName", "New Name");

    mockMvc.perform(patch("/api/v1/beer/" + beer.getId())
            .contentType(MediaType.APPLICATION_JSON)
            .accept(MediaType.APPLICATION_JSON)
                    .content(objectMapper.writeValueAsString(beerMap)))
            .andExpect(status().isNoContent());
    verify(beerService).patchBeerById(uuidArgumentCaptor.capture(), beerArgumentCaptor.capture());
    assertThat(beer.getId()).isEqualTo(uuidArgumentCaptor.getValue());
    assertThat(beerMap.get("beerName")).isEqualTo(beerArgumentCaptor.getValue().getBeerName());
}

```

#### DRY - Don’t Repeat Yourself

- DRY - is closely related to the Single Responsibility Principle
- Define things one time, avoid duplication
- Leads to higher quality and cleaner code
- Changes happen in one place, not many
- Avoids inconsistencies in programming logic
- Code maintenance and refactoring becomes easier

### DRY - Violation

- Controller path URLs are defined multiple times
  - Defined in controllers
  - Also repeated in JUnit tests
  - A change would require a lot of refactoring
- Solutions:
  - Externalize to property and set at runtime (usually not necessary)
  - Define as constant and reuse - Simple and efficient

### KEY TERMS

- Unit Tests
- Integration Tests
- Functional Tests
- Integration Tests - Designed to test behaviors between objects and parts of the overall system
- Functional Tests - Typically means you are testing the running application
- JNUIT Tests are NOT Sufficient to test the framework interaction
- `Spring Mock MVC` is a testing environment for the testing of Spring MVC controllers
- True unit test when run without the Spring Context
- Technically an Integration Test when used in conjunction with Spring Context
- Spring Boot supports a concept of what is called Test Splices
- Test Splices bring up a targeted segment of the Auto-Configured Spring Boot Environment
- Target segment of auto-configured Spring Boot Environment
- `@WebMvcTest` is a Spring Boot test splice which creates a MockMVC environment for the controller
- `Mockito` is the most popular mocking framework for testing Java
- Mocks (aka Test Doubles)
  - Dummy, Fake, Stub, Mock, Spy
- Spy - Wrapper arounf the actual objects
- Argument Captor
- Arguement Matcher

- Spring MockMVC allows you to test the controller interactions in a servlet context without
  the application running in a application server.
- Verify any value being passed using Mockito Mock with an argument captor.
