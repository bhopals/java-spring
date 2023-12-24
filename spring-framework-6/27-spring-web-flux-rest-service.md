## Spring WebFlux Rest Service

### Create WebFlux Controller

```
/**
 * Created by jt, Spring Framework Guru.
 */
@RestController
public class BeerController {
    public static final String BEER_PATH = "/api/v2/beer";
    @GetMapping(BEER_PATH)
    Flux<BeerDTO> listBeers(){
        return Flux.just(BeerDTO.builder().id(1).build(),
                BeerDTO.builder().id(2).build());
    }
}
```

```
@Mapper
public interface BeerMapper {
    Beer beerDtoToBeer(BeerDTO dto);
    BeerDTO beerToBeerDto(Beer beer);
}
```

```
@Override
    public Mono<BeerDTO> updateBeer(Integer beerId, BeerDTO beerDTO) {
        return beerRepository.findById(beerId)
                .map(foundBeer -> {
                    //update properties
                    foundBeer.setBeerName(beerDTO.getBeerName());
                    foundBeer.setBeerStyle(beerDTO.getBeerStyle());
                    foundBeer.setPrice(beerDTO.getPrice());
                    foundBeer.setUpc(beerDTO.getUpc());
                    foundBeer.setQuantityOnHand(beerDTO.getQuantityOnHand());

                    return foundBeer;
                }).flatMap(beerRepository::save)
                .map(beerMapper::beerToBeerDto);
    }
```

### KEY TERMS

- MapStruct - For Mapping DTOs to Entities and vice-versa by creating Mapper Classes
- Performing type conversions is a common task in Java development.
- MapStruct is a tool to assist with type conversions and help improve your productivity.
- WebFlux means - All the REPOS, Service, Controllers are going to return either `Flux` or `Mono`
