## Spring MVC Rest Services

### Rest Service Mappings

- `@RestController`
  `@Controller` is used to declare common web controllers which can return HTTP response but @RestController is used to create controllers for REST APIs which can return JSON. In Spring MVC, both `@Controller` and `@RestController` annotations are used to define web controllers as per MVC Design pattern

  `@RestController` = `@Controller` + `@ResponseBody`

- `@PathVariable`

```
/**
 * Created by jt, Spring Framework Guru.
 */
@Slf4j
@AllArgsConstructor
@RestController
@RequestMapping("/api/v1/beer")
public class BeerController {
    private final BeerService beerService;
    @RequestMapping(method = RequestMethod.GET)
    public List<Beer> listBeers(){
        return beerService.listBeers();
    }
    @RequestMapping(value = "{beerId}", method = RequestMethod.GET)
    public Beer getBeerById(@PathVariable("beerId") UUID beerId){
        log.debug("Get Beer by Id - in controller");
        return beerService.getBeerById(beerId);
    }
}
```

### Spring Boot Development Tools

- Add dependency in `pom.xml`

```
	<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
    </dependency>
```

- DevTools stands for Developer Tool. The aim of the module is to try and improve the development
  time while working with the Spring Boot application. Spring Boot DevTools pick up the changes and restart the application.
- Spring Boot Development Tools features

  - Property Defaults
  - Automatic Restart
  - LiveReload
    - The Spring Boot DevTools module includes an embedded server called LiveReload. It allows
      the application to automictically trigger a browser refresh whenever we make changes in the resources. It is also known as auto-refresh.
  - Remote Debug Tunneling
    - spring-boot-devtools provides out of the box remote debugging capabilities via HTTP, to have
      this feature it is required that spring-boot-devtools are packaged as part of the application. This can be achieved by disabling excludeDevtools configuration in the plugin in maven.
      `-Xdebug -Xrunjdwp:server=y,transport=dt_socket,suspend=n`
  - Remote Update and Restart

### HTTP Post with Spring MVC

- `@RequestMapping(method = RequestMethod.POST)` OR `@PostMapping`
- `@RequestBody`

  - Simply put, the @RequestBody annotation maps the HttpRequest body to a transfer or domain object, enabling automatic deserialization of the inbound HttpRequest body onto a Java object.

- `@ResponseBody`
  - The @ResponseBody annotation tells a controller that the object returned is automatically
    serialized into JSON and passed back into the HttpResponse object.

```
@PostMapping(value = "/content", produces = MediaType.APPLICATION_JSON_VALUE)
@ResponseBody
public ResponseTransfer postResponseJsonContent(
  @RequestBody LoginForm loginForm) {
    return new ResponseTransfer("JSON Content!");
}
```

### Set Header on HTTP Response

```
@PostMapping
//@RequestMapping(method = RequestMethod.POST)
public ResponseEntity handlePost(@RequestBody Beer beer){

    Beer savedBeer = beerService.saveNewBeer(beer);

    HttpHeaders headers = new HttpHeaders();
    headers.add("Location", "/api/v1/beer/" + savedBeer.getId().toString());

    return new ResponseEntity(headers, HttpStatus.CREATED);
}
```

### HTTP PUT with Spring MVC

@PutMapping annotation in String MVC framework is a powerful tool for handling HTTP PUT requests in your RESTful web services. It maps specific URLs to the handler method allowing you to receive and process the data submitted through PUT requests. The @PutMaping annotation is a Spring annotation that is used to map HTTP PUT requests onto specific handler methods. It is a shortcut for @RequestMapping annotation with method = RequestMethod.PUT attribute

- `@PutMapping`
- @PutMapping is a composed annotation that acts as a shortcut for @RequestMapping(method = RequestMethod.PUT).

```
@PutMapping("{beerId}")
public ResponseEntity updateById(@PathVariable("beerId")UUID beerId, @RequestBody Beer beer){
    beerService.updateBeerById(beerId, beer);
    return new ResponseEntity(HttpStatus.NO_CONTENT);
}
```

- `@DeleteMapping`

- `@PatchMapping` - Go and update non-null properties

- A patch operation allows you to update specific properties, while an update operation
  will update all properties.

### KEY TERMS

- Enabling automatic deserialization of the inbound HttpRequest body onto a java object
