## Paging and Sorting

### What is Paging and Sorting?

- Paging and Sorting refer to techniques for dealing with lists of data
- Paging is also referred to pagination
- For example searching on the web:
  - First page is 25 results of thousands
  - Data on first page is ordered by “best search match”
- Clicking on next page says:

  - Give me the second page of the sorted data

- By default most SQL databases will return the full data set for a query
- Most GUI tools will set a default limit
- Large record sets can become problematic for performance and memory utilization
  - Large meaning hundreds of thousands or millions of records
- In SQL the limit clause is used to limit the number of records returned
- In SQL the offset clause is used to move the starting point of the record set
- Second page of 25 would be limit 25 offset 25

- By default most SQL databases will not sort data
- When no sort is specified the order returned is not guaranteed to be the same query to query
  - Often the order will be the same, but beware it can change!
- Sorting is controlled by via the SQL order by clause
- One or more columns can be specified with sort order (asc or desc)
- Default order is ascending
- Example: ORDER BY beerName ASC

### Paging and Sorting with Spring Data JPA

- Spring Data JPA provides robust support for paging and sorting
- Spring Data JPA does not have a default limit on records returned
- Spring Data JPA does not set a default sort
- Only limit is memory of the JVM
- Returning several million records in a JSON response will consume a lot of memory
  - Possibly crashing the application
- Best practice is to set default limits for the number of records returned

### Paging and Sorting with Spring

- Paging and Sorting in Spring has an API in the Spring Data Commons project
- This API is used to define Paging and Sorting in the Spring Data family of projects
- Each Spring Data project provides its own implementation of this API
- In this course we are using the JPA version
- Techniques learned with the JPA version will be portable to other platform versions of Spring Data
- Spring Data JPA abstracts the paging and sorting to the JPA Standard

### Paging and Sorting Core Components

- PageRequest - is a Java object used to describe the desired page of data
  - Includes the page number, page size and sort information
- Sort - Object describing how to sort the requested data
- Creating a PageRequest:

  - page number (zero based), and size (number of records)
    - Defaults to unsorted
  - page number, size, and sort

- Page - Is an interface describing the result of a paged query
  - content - list property containing requested page of data
  - Also has page number, size, and sort information
  - Has utility methods to get next and previous Pagable

### Paging and Sorting with Spring Data JPA

- Paging and Spring with Spring Data JPA is simple to implement
- Query methods in Spring Data JPA will accept paging request information
- Query methods in Spring Data JPA will return paging information
- To implement paging and sorting on Spring Data JPA query methods:
  - Add PageRequest as a parameter to the defined method
  - Use Page with generic of content type for the return type

### Example

`Page<Beer> findAllByBeerNameIsLikeIgnoreCase(String beerName, Pageable pageable);`

```
 @GetMapping(value = BEER_PATH)
    public Page<BeerDTO> listBeers(@RequestParam(required = false) String beerName,
                                   @RequestParam(required = false) BeerStyle beerStyle,
                                   @RequestParam(required = false) Boolean showInventory,
                                   @RequestParam(required = false) Integer pageNumber,
                                   @RequestParam(required = false) Integer pageSize){
        return beerService.listBeers(beerName, beerStyle, showInventory, pageNumber, pageSize);
    }

```

```
@Override
    public Page<BeerDTO> listBeers(String beerName, BeerStyle beerStyle, Boolean showInventory,
                                   Integer pageNumber, Integer pageSize) {

        PageRequest pageRequest = buildPageRequest(pageNumber, pageSize);

        Page<Beer> beerPage;

        if(StringUtils.hasText(beerName) && beerStyle == null) {
            beerPage = listBeersByName(beerName, pageRequest);
        } else if (!StringUtils.hasText(beerName) && beerStyle != null){
            beerPage = listBeersByStyle(beerStyle, pageRequest);
        } else if (StringUtils.hasText(beerName) && beerStyle != null){
            beerPage = listBeersByNameAndStyle(beerName, beerStyle, pageRequest);
        } else {
            beerPage = beerRepository.findAll(pageRequest);
        }

        if (showInventory != null && !showInventory) {
            beerPage.forEach(beer -> beer.setQuantityOnHand(null));
        }

        return beerPage.map(beerMapper::beerToBeerDto);

    }

    public PageRequest buildPageRequest(Integer pageNumber, Integer pageSize) {
        int queryPageNumber;
        int queryPageSize;

        if (pageNumber != null && pageNumber > 0) {
            queryPageNumber = pageNumber - 1;
        } else {
            queryPageNumber = DEFAULT_PAGE;
        }

        if (pageSize == null) {
            queryPageSize = DEFAULT_PAGE_SIZE;
        } else {
            if (pageSize > 1000) {
                queryPageSize = 1000;
            } else {
                queryPageSize = pageSize;
            }
        }

        Sort sort = Sort.by(Sort.Order.asc("beerName"));

        return PageRequest.of(queryPageNumber, queryPageSize, sort);
    }

    private Page<Beer> listBeersByNameAndStyle(String beerName, BeerStyle beerStyle, Pageable pageable) {
        return beerRepository.findAllByBeerNameIsLikeIgnoreCaseAndBeerStyle("%" + beerName + "%",
                beerStyle, pageable);
    }
```

### KEY TERMS

- Add `PageRequest` as a parameter to the defined method
- Use `Page` with generic of content type for the return type
- Different Persistence Implementations
- Spring Data JPA Abstracts the PAGING and SORTNG

  - `PageRequest` - `{pageNumber, size, sort}`

- PageRequest
- Pageable
