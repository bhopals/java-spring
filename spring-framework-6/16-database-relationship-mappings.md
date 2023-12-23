## Database Relationship Mappings

### Relational Databases

- JPA is designed to work with relational databases
- Prior to relational databases, data was often stored in flat files which typically were just text files
- Flat files do not have structure or formal rules governing data in them
- E. F. Cobb of IBM originally coined the term “Relational Database” in 1970
- The relational database introduced the concept of storing data in tables and columns
- Complex data can be represented in tables which have relationships
- For example, orders have order lines which have products - 3 tables of data with relationships

### Database Relationships

- One to One - Both tables have only one record on each side of the relationship
  - Like an extension of the data row
- One to Many - The primary table has one record that relates to zero/many records in the related table
  - An object with a list property
- Many to Many - Each record in both tables may be related to zero/many records in the related table
  - Two lists, related to each other

### Database Constraints

- Best practice is to use database constraints to enforce relationships
- One to One - Both tables can share the primary key value, or one table can have
  its own primary key and unique key on id column of related table
- One to Many - The related table has column for primary key of primary
  table, with foreign key constraint
- Many to Many - Join table is used with composite primary key consisting of the primary
  key values of related tables, with foreign key constraints

### Relationship Direction

- Bi-Directional - Relationship can be accessed from either side of the relationship
  - Example OrderHeader and OrderLine - likely needed from either side
- Uni-Directional - Relationship can be access from either side of the relationship
  - Example OrderLine and Product - unlikely you will need to access Order Lines from Product
  - The Product entity does not have a reference to OrderLine

### Cascade Operations

- Hibernate has the ability to Cascade persistence operations
- Example - A delete of just Order Header would fail on foreign key constraints to OrderLine
  and OrderApproval
  - Explicitly, you would need to perform deletes of the child records first
  - Optionally, Hibernate can be configured to delete OrderLines and OrderApproval before
    deleting the OrderHeader
- Use with caution - you would not wish to delete Product records on delete for OrderLine

### Foreign Key Declaration

- JPA does have a `@ForeignKey` annotation
- This is meta-data information only.
- Hibernate will reference this for schema generation only
- It is not enforced nor generated if missing
- When using schema migration tools like Liquibase or Flyway it is not needed

### One to Many (Bidirectional)

- Setting Bidirectional Mapping between Customer and BeerOrder Object.

- One `Customer` ==> Many `BeerOrder`

- In Customer, add `mappedBy` of target Entity with `@OneToMany`
  ```
  @OneToMany(mappedBy="customer")
  private Set<BeerOrder> beerOrders;
  ```
- In BeerOrder, add `@ManyToOne`

  ```
  @ManyToOne
  private Customer customer;
  ```

- Use `saveAndFlush` method to persist objects (especially when you have BI-Directional Relationship)
- `saveAndFlush` may cause performance degradation

- `@Builder.default` - Lombok annotation to use default initialization

### Many to Many

- Create a third table that is going to store both side PK ids and manage relationship

- Table Details

  - `beer`
  - `category`
  - `beer_category`

- Entity Settings

  - `Category` Entity

    ```
    @ManyToMany
    @JoinTable(name="beer_category",
      joinColumns = @JoinColumn(name="category_id"),
      inverseJoinColumns = @JoinColumn(name="beer_id"))
    private Set<Beer> beers;
    ```

  - `Beer` Entity

  ```
  @ManyToMany
  @JoinTable(name="beer_category",
      joinColumns = @JoinColumn(name="beer_id"),
      inverseJoinColumns = @JoinColumn(name="category_id"))
  private Set<Category> categories
  ```

### One to One Bi-Directional

- Table Details

  - `beer_order`
  - `beer_order_shipment`

- Entity Details

  - `BeerOrder`

  ```
  @OneToOne
  private BeerOrderShipment beerOrderShipment;
  ```

  - `BeerOrderShipment`

  ```
  @OneToOne
  private BeerOrder beerOrder;
  ```

### Helper Method

- A trick to Init/reflect entity details (Instead of using `saveAndFlush`), we can use Setter Method
  ```
  // BeerOrder.java
  public void setCustomer(Customer customer) {
        this.customer = customer;
        customer.getBeerOrders().add(this);
    }
  ```
- Also default set of List

```
 @Builder.Default
 private Set<Category> categories = new HashSet<>();
```

### Hibernate Transient v/s Detached

- Transient :In this state, an instance is not associated with any persistence context.
- Detached :This is a state for an instance which was previously associated with a persistence
  context and has been currently closed.

- Transient objects do not have association with the databases and session objects. They are simple objects and not persisted to the database. Once the last reference is lost, that means the object itself is lost. And of course, garbage collected. The commits and rollbacks will have no effects on these objects. They can become into persistent objects through the save method calls of Session object.

- The detached object have corresponding entries in the database. These are persistent and not connected to the Session object. These objects have the synchronized data with the database when the session was closed. Since then, the change may be done in the database which makes this object stale. The detached object can be reattached after certain time to another object in order to become persistent again.

### JPA Specific Cascade Types

- ALL - propagates all operations
- PERSIST - Will also save child objects (transient instances)
- MERGE - Merge copies the state of a given object to the persistent object
  - MERGE includes child entities
- REMOVE - Cascades delete operations to child objects
- REFRESH - Cascades refresh operations to child objects
- DETACH - Detaches child objects from persistence context

### Hibernate Specific Cascade Types

- DELETE - Same as JPA REMOVE
- SAVE_UPDATE - Cascades Hibernate Save and Update operations to child objects
- REPLICATE - Replicates child objects to second data source
- LOCK - Reattaches entity and children to persistence context - without refresh

### KEY TERMS

- ERD stands for `Entity Relationship Diagram`
- A foreign key constraint
- May cause performance degradation
