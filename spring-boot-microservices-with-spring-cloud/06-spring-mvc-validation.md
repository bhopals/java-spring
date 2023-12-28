## Spring MVC Validation

### Bean Validation Implementation

- Add dependency in `POM.XML`

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

#### Spring Boot Class Validations

- Add validator annotations on DTOs and ENTITIES

- Add `@Validated` annotation in Method param
  `public ResponseEntity handleUpdate(@PathVariable("beerId") UUID beerId, @Validated @RequestBody BeerDTO beerDto)`

- Add `ExceptionHandler` in controller

```
@ExceptionHandler(ConstraintViolationException.class)
public ResponseEntity<List> validationErrorHandler(ConstraintViolationException e){
    List<String> errors = new ArrayList<>(e.getConstraintViolations().size());

    e.getConstraintViolations().forEach(constraintViolation -> {
        errors.add(constraintViolation.getPropertyPath() + " : " + constraintViolation.getMessage());
    });

    return new ResponseEntity<>(errors, HttpStatus.BAD_REQUEST);
}
```

### Spring Boot Method Validations

- To add validations in method params, add validation annotation before method params:
  `public ResponseEntity handleUpdate(@NoNull @PathVariable('beerId') UUID beerId)`

- Add `@Validated` annotation on the Class

- Add `ExceptionHandler` in controller

```
@ExceptionHandler(ConstraintViolationException.class)
public ResponseEntity<List> validationErrorHandler(ConstraintViolationException e){
    List<String> errors = new ArrayList<>(e.getConstraintViolations().size());

    e.getConstraintViolations().forEach(constraintViolation -> {
        errors.add(constraintViolation.getPropertyPath() + " : " + constraintViolation.getMessage());
    });

    return new ResponseEntity<>(errors, HttpStatus.BAD_REQUEST);
}
```

#### Spring MVC Controller Advice

- To great global exception handler using `@ControllerAdvice` annotation

### KEY TERMS
