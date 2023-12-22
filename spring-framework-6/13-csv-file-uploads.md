## CSV File Uploads

- OpenCSV - https://opencsv.sourceforge.net/
- Opencsv is an easy-to-use CSV (comma-separated values) parser library for Java. It was developed because all the CSV parsers at the time didn’t have commercial-friendly licenses. Java 8 is currently the minimum supported version.

### Add Dependency

- Add dependencies in `POM.XML`

```
<dependency>
    <groupId>com.opencsv</groupId>
    <artifactId>5.7.0</artifactId>
</dependency>
```

### Mapping with OpenCSV

- Create a Model Class

```
/**
 * Created by jt, Spring Framework Guru.
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
public class BeerCSVRecord {

    @CsvBindByName
    private Integer row;

    @CsvBindByName(column = "count.x")
    private Integer count;

    @CsvBindByName
    private String abv;

    @CsvBindByName
    private String ibu;

    @CsvBindByName
    private Integer id;

    @CsvBindByName
    private String beer;

    @CsvBindByName
    private String style;

    @CsvBindByName(column = "brewery_id")
    private Integer breweryId;

    @CsvBindByName
    private Float ounces;

    @CsvBindByName
    private String style2;

    @CsvBindByName(column = "count.y")
    private String count_y;

    @CsvBindByName
    private String city;

    @CsvBindByName
    private String state;

    @CsvBindByName
    private String label;

}
```

### CSV Parse Service

```

/**
 * Created by jt, Spring Framework Guru.
 */
public class BeerCsvServiceImpl implements BeerCsvService {
    @Override
    public List<BeerCSVRecord> convertCSV(File csvFile) {

        try {
            List<BeerCSVRecord> beerCSVRecords = new CsvToBeanBuilder<BeerCSVRecord>(new FileReader(csvFile))
                    .withType(BeerCSVRecord.class)
                    .build().parse();
            return beerCSVRecords;
        } catch (FileNotFoundException e) {
            throw new RuntimeException(e);
        }
    }
}
```

### CSV Data to Database

```
private void loadCsvData() throws FileNotFoundException {
        if (beerRepository.count() < 10){
            File file = ResourceUtils.getFile("classpath:csvdata/beers.csv");

            List<BeerCSVRecord> recs = beerCsvService.convertCSV(file);

            recs.forEach(beerCSVRecord -> {
                BeerStyle beerStyle = switch (beerCSVRecord.getStyle()) {
                    case "American Pale Lager" -> BeerStyle.LAGER;
                    case "American Pale Ale (APA)", "American Black Ale", "Belgian Dark Ale", "American Blonde Ale" ->
                            BeerStyle.ALE;
                    case "American IPA", "American Double / Imperial IPA", "Belgian IPA" -> BeerStyle.IPA;
                    case "American Porter" -> BeerStyle.PORTER;
                    case "Oatmeal Stout", "American Stout" -> BeerStyle.STOUT;
                    case "Saison / Farmhouse Ale" -> BeerStyle.SAISON;
                    case "Fruit / Vegetable Beer", "Winter Warmer", "Berliner Weissbier" -> BeerStyle.WHEAT;
                    case "English Pale Ale" -> BeerStyle.PALE_ALE;
                    default -> BeerStyle.PILSNER;
                };

                beerRepository.save(Beer.builder()
                                .beerName(StringUtils.abbreviate(beerCSVRecord.getBeer(), 50))
                                .beerStyle(beerStyle)
                                .price(BigDecimal.TEN)
                                .upc(beerCSVRecord.getRow().toString())
                                .quantityOnHand(beerCSVRecord.getCount())
                        .build());
            });
        }
    }
```

### Hibernate Create and Update TimeStamp

- Annotate fields with `@CreationTimeStamp` and `@UpdateTimeStamp`

### More Details

Refer - https://opencsv.sourceforge.net/#general

For reading, create a bean to harbor the information you want to read, annotate the bean fields with the opencsv annotations, then do this:

```
List<MyBean> beans = new CsvToBeanBuilder(FileReader("yourfile.csv"))
       .withType(Visitors.class).build().parse();
```

For writing, create a bean to harbor the information you want to write, annotate the bean fields with the opencsv annotations, then do this:

```
   // List<MyBean> beans comes from somewhere earlier in your code.
     Writer writer = new FileWriter("yourfile.csv");
     StatefulBeanToCsv beanToCsv = new StatefulBeanToCsvBuilder(writer).build();
     beanToCsv.write(beans);
     writer.close();

```

### KEY TERMS
