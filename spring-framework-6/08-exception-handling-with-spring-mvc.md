### Exception Handling with Spring MVC

### Standard Spring MVC Exceptions

- Spring MVC does support a number of standard exceptions
- Standard Exceptions are handled by the `DefaultHandlerExceptionResolver`
- The DefaultHandlerExceptionResolver sets the appropriate HTTP status code
  - BUT does not write content to the body of the response
- Spring MVC does have robust support for customizing error responses

### Spring Standard Exceptions

- BindException - 400 Bad Request
- ConversionNotSupportedException - 500 Internal Server Error
- HttpMediaTypeNotAcceptableException - 406 Not Acceptable
- HttpMediaTypeNotSupportedException - 415 Unsupported Media Type
- HttpMessageNotReadableException - 400 Bad Request
- HttpMessageNotWritableException - 500 Internal Server Error
- HttpRequestMethodNotSupportedException - 405 Method Not Allowed
- MethodArgumentNotValidException - 400 Bad Request
- MissingServletRequestParameterException - 400 Bad Request
- MissingServletRequestPartException - 400 Bad Request
- NoSuchRequestHandlingMethodException - 404 Not Found
- TypeMismatchException - 400 Bad Request

### Spring MVC Exception Handling

- `@ExceptionHandler` on controller method to handle specific Exception types
- `@ReponseStatus` - annotation for custom exception classes to set desired HTTP status
- Implement `AbstractHandlerException` Resolver - full control over response (including body)
- `@ControllerAdvice` - used to implement a global exception handler
- `ResponseStatusException.class` - (Spring 5+) Exception which can be thrown which
  allows setting the HTTP status and message in the constructor

### Spring Boot ErrorController

- Provides Whitelabel Error Page for HTML requests, or JSON response for RESTful requests
- Properties:
  - server.error.include-binding-errors - default: never
  - server.error.include-exception - default: false
  - server.error.include-message - default: never
  - server.error.include-stacktrace - default: never
  - server.error.path - default: /error
  - server.error.whitelabel.enabled - default: true

### Spring Boot BasicErrorController

- Spring Boot includes a BasicError Controller
- This class can be extended for additional error response customization
- Allows for wide support of the needs of various clients and situations
- Rarely used, but important to know it is available for use when needed

### Using Exception Handler

```
 @ExceptionHandler(NotFoundException.class)
public ResponseEntity handleNotFoundException(){
    return ResponseEntity.notFound().build();
}
```

### Control Advice

- To set up a Global Exception Handler

```
/**
 * Created by jt, Spring Framework Guru.
 * This exception handler will be used acroos all controller for 'NotFoundException' exception
 *
 */
@ControllerAdvice
public class ExceptionController {
    @ExceptionHandler(NotFoundException.class)
    public ResponseEntity handleNotFoundException(){
        return ResponseEntity.notFound().build();
    }
}
```

### Using ResponseStatus Annotation

```
/**
 * Created by jt, Spring Framework Guru.
 */
@ResponseStatus(value = HttpStatus.NOT_FOUND, reason = "Value Not Found")
public class NotFoundException extends RuntimeException {
    public NotFoundException() {
    }

    public NotFoundException(String message) {
        super(message);
    }

    public NotFoundException(String message, Throwable cause) {
        super(message, cause);
    }

    public NotFoundException(Throwable cause) {
        super(cause);
    }

    public NotFoundException(String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace) {
        super(message, cause, enableSuppression, writableStackTrace);
    }
}


@Test
void getBeerByIdNotFound() throws Exception {
    given(beerService.getBeerById(any(UUID.class))).willThrow(NotFoundException.class);
    mockMvc.perform(get(BeerController.BEER_PATH_ID, UUID.randomUUID()))
            .andExpect(status().isNotFound());
}
```

### Using Java Optional

```
// Service
@Override
public Optional<Beer> getBeerById(UUID id) {
    log.debug("Get Beer by Id - in service. Id: " + id.toString());
    return Optional.of(beerMap.get(id));
}

// Controller
 @GetMapping(value = BEER_PATH_ID)
public Beer getBeerById(@PathVariable("beerId") UUID beerId){
    log.debug("Get Beer by Id - in controller");
    return beerService.getBeerById(beerId).orElseThrow(NotFoundException::new);
}

// Junit Test
@Test
void getBeerByIdNotFound() throws Exception {

    given(beerService.getBeerById(any(UUID.class))).willReturn(Optional.empty());

    mockMvc.perform(get(BeerController.BEER_PATH_ID, UUID.randomUUID()))
            .andExpect(status().isNotFound());
}

```

### KEY TERMS

- Standard Exceptions are handled by the `DefaultHandlerExceptionResolver`
- `@ControllerAdvice` - used to implement a global exception handler
- Implement `AbstractHandlerException` Resolver - full control over response (including body)
- `@ExceptionHandler` on controller method to handle specific Exception types
- `ResponseStatusException.class` - (Spring 5+) Exception which can be thrown which
  allows setting the HTTP status and message in the constructor
