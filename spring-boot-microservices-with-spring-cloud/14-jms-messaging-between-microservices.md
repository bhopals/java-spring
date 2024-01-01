## JMS Messaging Between Microservices

### Tips and Tricks IntelliJ - Auto Import on Paste

- Preference -> Editor -> General -> Auto Import
  - 1. Java -> Insert Imports on Paste -> All
  - 2. Check `Add unambiguous imports on the fly`
  - 3. Check `Optimize imports on the fly (for current projects)`

### Create new Beer Event Objects

- Add BeerEvent

```
//Parent Event
@Data
@AllArgsConstructor
@Builder
@NoArgsConstructor
public class BeerEvent implements Serializable {
    static final long serialVersionUID = -5781515597148163111L;
    private BeerDto beerDto;
}
```

- Add Brew Beer Event

```
@NoArgsConstructor
public class BrewBeerEvent extends BeerEvent {
    public BrewBeerEvent(BeerDto beerDto) {
        super(beerDto);
    }
}
```

- Add Inventory Event

```
@NoArgsConstructor
public class NewInventoryEvent extends BeerEvent {
    public NewInventoryEvent(BeerDto beerDto) {
        super(beerDto);
    }
}
```

### Create Brewing Service

```
@Slf4j
@Service
@RequiredArgsConstructor
public class BrewingService {
    private final BeerRepository beerRepository;
    private final BeerInventoryService beerInventoryService;
    private final JmsTemplate jmsTemplate;
    private final BeerMapper beerMapper;
    @Scheduled(fixedRate = 5000) //every 5 seconds
    public void checkForLowInventory(){
        List<Beer> beers = beerRepository.findAll();
        beers.forEach(beer -> {
            Integer invQOH = beerInventoryService.getOnhandInventory(beer.getId());
            log.debug("Checking Inventory for: " + beer.getBeerName() + " / " + beer.getId());
            log.debug("Min Onhand is: " + beer.getMinOnHand());
            log.debug("Inventory is: "  + invQOH);

            if(beer.getMinOnHand() >= invQOH){
                jmsTemplate.convertAndSend(JmsConfig.BREWING_REQUEST_QUEUE, new BrewBeerEvent(beerMapper.beerToBeerDto(beer)));
            }
        });
    }
}
```

### Create Brew Beer Listener

```
@Service
@RequiredArgsConstructor
@Slf4j
public class BrewBeerListener {
    private final BeerRepository beerRepository;
    private final JmsTemplate jmsTemplate;
    @Transactional
    @JmsListener(destination = JmsConfig.BREWING_REQUEST_QUEUE)
    public void listen(BrewBeerEvent event){
        BeerDto beerDto = event.getBeerDto();
        Beer beer = beerRepository.getOne(beerDto.getId());
        beerDto.setQuantityOnHand(beer.getQuantityToBrew());
        NewInventoryEvent newInventoryEvent = new NewInventoryEvent(beerDto);
        log.debug("Brewed beer " + beer.getMinOnHand() + " : QOH: " + beerDto.getQuantityOnHand());
        jmsTemplate.convertAndSend(JmsConfig.NEW_INVENTORY_QUEUE, newInventoryEvent);
    }
}
```

### Jackson JMS Configuration

```
@Configuration
public class JmsConfig {

    public static final String BREWING_REQUEST_QUEUE = "brewing-request";
    public static final String NEW_INVENTORY_QUEUE = "new-inventory";
    public static final String VALIDATE_ORDER_QUEUE = "validate-order";
    public static final String VALIDATE_ORDER_RESPONSE_QUEUE = "validate-order-response";

    @Bean // Serialize message content to json using TextMessage
    public MessageConverter jacksonJmsMessageConverter(ObjectMapper objectMapper) {
        MappingJackson2MessageConverter converter = new MappingJackson2MessageConverter();
        converter.setTargetType(MessageType.TEXT);
        converter.setTypeIdPropertyName("_type");
        converter.setObjectMapper(objectMapper);
        return converter;
    }
}
```

### Tips and Tricks - Logging Level

- Add properties in `application.properties`
  - `logging.level.guru.springframework=debug`

### KEY TERMS

- Spring stereotype
- More process control
- Processes between microservices
