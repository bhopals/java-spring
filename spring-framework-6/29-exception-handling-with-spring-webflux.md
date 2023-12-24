## Exception Handling with Spring WebFlux

```
@PutMapping(BEER_PATH_ID)
    Mono<ResponseEntity<Void>> updateExistingBeer(@PathVariable("beerId") Integer beerId,
                                               @Validated @RequestBody BeerDTO beerDTO){
        return beerService.updateBeer(beerId, beerDTO)
                .switchIfEmpty(Mono.error(new ResponseStatusException(HttpStatus.NOT_FOUND)))
                .map(savedDto -> ResponseEntity.noContent().build());
    }



@GetMapping(BEER_PATH_ID)
    Mono<BeerDTO> getBeerById(@PathVariable("beerId") Integer beerId){
        return beerService.getBeerById(beerId)
                .switchIfEmpty(Mono.error(new ResponseStatusException(HttpStatus.NOT_FOUND)));
    }
```

### KEY TERMS

- `.switchIfEmpty`
- `Mono.error`
