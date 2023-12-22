## MySQL with Spring Boot

- MySQL Installation - https://dev.mysql.com/doc/refman/8.0/en/installing.html

### About MySQL

- With over 100 million downloads, MySQL is the most popular database in history
- MySQL is a Relational Database System (aka RDMS)
- MySQL is owned by Oracle, but MySQL is open source and free to use
- Officially pronounced ‘My Ess Que Ell’

### MySQL History

- MySQL was created in 1995 by a Swedish company called MySQL AB
- Original developers included: Michael (Monty) Widenius, David Axmark, Allan Larsson
  - MySQL is named after Monty’s daughter ‘My’
- Under GPL, MySQL was open source in 2000
- MySQL had over 2 million active installations by the end of 2001

- In 2005, Oracle acquired Innobase, the company behind the storage backend of MySQL
- In 2006, MySQL had 8 million installations, 320 employees, across 25 countries
- Sun Microsystems bought MySQL in 2008 for $1 billion
- MySQL had become the choice database for large corporations, banks, and telecoms

- In 2010, after legal complications in the EU, Oracle’s acquisition of Sun Microsystems was finalized
  - This included the purchase of MySQL
- Michael (Monty) Widenius left Sun Microsystems and developed a fork of MySQL called MariaDB
  - Largely out of concern about the future of MySQL
  - The MariaDB API remains 100% compatible with MySQL

### MySQL Features

- MySQL is a Relational Database Management System
- “SQL” stands for Structured Query Language
  - MySQL supports the ANSI/ISO SQL standard
- MySQL is developed in C and C++, making it portable across many different platforms
- MySQL is very fast, stable and scalable.
- There are MySQL clients for all popular languages.

  - C, C++, Eiffel, Java, Perl, PHP, Python, Ruby, Tcl, and ODBC, JDBC, ADO.NET

- MySQL Features
  - Stored Procedures
  - Triggers
  - Cursors
  - Updated Views
  - Query Cache
  - Subselects
  - ACID Compliance
    - Atomicity - all or nothing
    - Consistency - transactions are valid to rules of the DB
    - Isolation - Results of transactions are as if they are done end to end
    - Durability - Once a transaction is committed, it remains so

### MySQL and Spring Boot

- Connectivity to MySQL is managed via a JDBC Driver
- JDBC - Java DataBase Connectivity
- All major Relational Databases have a database specific JDBC Driver
- MySQL was selected for this course due to its widespread popularity and its free to use
- Configuration steps for Spring Boot for other relational databases will be roughly the same
  - JDBC is a common API abstraction
  - Each will have vendor specific options

### Create MySQL Schema and User Accounts

```
DROP DATABASE IF EXISTS restdb;
DROP USER IF EXISTS `restadmin`@`%`;
CREATE DATABASE IF NOT EXISTS restdb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER IF NOT EXISTS `restadmin`@`%` IDENTIFIED WITH mysql_native_password BY 'password';
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, REFERENCES, INDEX, ALTER, EXECUTE, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, EVENT, TRIGGER ON `restdb`.* TO `restadmin`@`%`;
```

### Adding MySQL Dependencies

- Add `mysql-connector-java` and update `h2` to use `scope` - runtime

```
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
</dependency>
<dependency>
  <groupId>com.h2database</groupId>
  <artifactId>h2</artifactId>
  <scope>runtime</scope>
</dependency>

```

### Spring Boot MySql Profile

- Create a property file - `application-localmysql.properties`
- And what Spring Boot follows is if you do a properties file with Dash, whatever, that becomes a
  profile and that will become activated for that profile.

- Add details

```
spring.datasource.username=restadmin
spring.datasource.password=password
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/restdb?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
spring.jpa.database=mysql
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect
spring.jpa.hibernate.ddl-auto=create-drop

# Show SQL
spring.jpa.properties.hibernate.show_sql=true

# Format SQL
spring.jpa.properties.hibernate.format_sql=true

# Show Bind Values
logging.level.org.hibernate.orm.jdbc.bind=trace


```

### Console Logging of SQL Statements

```
# Show SQL
spring.jpa.properties.hibernate.show_sql=true

# Format SQL
spring.jpa.properties.hibernate.format_sql=true

# Show Bind Values (Be cautious while using it)
logging.level.org.hibernate.orm.jdbc.bind=trace
```

- Entity

```
/**
 * Created by jt, Spring Framework Guru.
 */
@Getter
@Setter
@Builder
@Entity
@NoArgsConstructor
@AllArgsConstructor
public class Beer {

    @Id
    @GeneratedValue(generator = "UUID")
    @GenericGenerator(name = "UUID", strategy = "org.hibernate.id.UUIDGenerator")
    @JdbcTypeCode(SqlTypes.CHAR)
    @Column(length = 36, columnDefinition = "varchar(36)", updatable = false, nullable = false)
    private UUID id;

    @Version
    private Integer version;

    @NotNull
    @NotBlank
    @Size(max = 50)
    @Column(length = 50)
    private String beerName;

    @NotNull
    private BeerStyle beerStyle;

    @NotNull
    @NotBlank
    @Size(max = 255)
    private String upc;
    private Integer quantityOnHand;

    @NotNull
    private BigDecimal price;
    private LocalDateTime createdDate;
    private LocalDateTime updateDate;
}
```

### Hikari DataSource Pool

- One of the things that Spring Boot does is configures a connection pool to the database.
- And there are several flavors out there, but `Hikari` is the probably the most preferred one.
- So what Hikari does is it creates a pool of connections to the database, establishes that
  network connection to the database
- Hikari is a JDBC DataSource implementation that provides a connection pooling mechanism.
  Compared to other implementations, it promises to be lightweight and better performing
- To enable it, add below in the configuration properties

```
# Hikari Data source pool configuration
spring.datasource.hikari.pool-name=RestDB-Pool
spring.datasource.hikari.maximum-pool-size=5
spring.datasource.hikari.data-source-properties.cachePrepStmts=true
spring.datasource.hikari.data-source-properties.prepStmtCacheSize=250
spring.datasource.hikari.data-source-properties.prepStmtCacheSqlLimit=2048
spring.datasource.hikari.data-source-properties.useServerPrepStmts=true
spring.datasource.hikari.data-source-properties.useLocalSessionState=true
spring.datasource.hikari.data-source-properties.rewriteBatchedStatements=true
spring.datasource.hikari.data-source-properties.cacheResultSetMetadata=true
spring.datasource.hikari.data-source-properties.cacheServerConfiguration=true
spring.datasource.hikari.data-source-properties.elideSetAutoCommits=true
spring.datasource.hikari.data-source-properties.maintainTimeStats=false
```

### Schema Script Generation

- One of the capabilities that Hibernate brings to the table is the ability to reflect upon your
  entities and create a database creation script.

- In `application.properties`

```
logging.level.guru.springframework=debug
#spring.jpa.properties.jakarta.persistence.schema-generation.scripts.action=drop-and-create
#spring.jpa.properties.jakarta.persistence.schema-generation.scripts.create-source=metadata
#spring.jpa.properties.jakarta.persistence.schema-generation.scripts.drop-target=drop-and-create.sql
#spring.jpa.properties.jakarta.persistence.schema-generation.scripts.create-target=drop-and-create.sql
```

### Spring Boot Database Initialization

### KEY TERMS
