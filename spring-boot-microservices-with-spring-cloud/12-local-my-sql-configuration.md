## Local MySQL Configuration

- H2 Database - https://www.h2database.com/html/features.html

### Spring Boot Datasource Configuration Overview

- Data Source Configuration tells Java how to connect to a JDBC complaint Data Source

#### What is Data Source Configuration?

- Data Source Configuration tells Java how to connect to a JDBC complaint Data Source
- JDBC is a standard for connecting to Relational Databases
- At a minimum you need:
  - JDBC URL
  - User Name
  - Password

#### Spring Boot Embedded Datasources

- If Embedded database is on classpath:
  - Spring will auto-configure embedded data source if not provided in config
    - H2, HSQL, and Derby are supported
- H2 is generally preferred for local development
- H2 supports compatibility modes for most major databases
- H2 has a DB Console application which can be used to browse database.
  - Configured automatically if Spring DevTools are on classpath

#### Spring Boot Datasources

- For permanent datasources you are responsible for providing data source configuration details
- When you provide data source connection details, an embedded database might not be started
- Typically you will be connecting to a local database, or a deployed database server
- A permanent data source can be:
  - Locally installed MySQL, Postgres for your personal development
  - A database running locally in a Docker Container
  - A deployed database on a server
  - A managed database - Amazon RDS

#### Datasource Configuration

- Typically managed via Spring Boot configuration options

  - application.properties
  - application-<profile-name>.properties
  - environment parameters
  - Spring Cloud Config

- use an account with minimal authority in the database

### Configure H2 MySql Compatibilty mode

- More - https://www.h2database.com/html/features.html

- To access h2 in-memory console, add `spring.h2.console.enabled=true` and restart application
- Once restarted, grab URL details from console logs and use it here - http://localhost:8080/h2-console

### Datasource Connections

- Establishing a Database Connection is an expensive operation as it involves:
  - Call out to Database Server to get authenticated
  - Database Server need to authenticate credentials
  - Establish a connection
  - Establish a session - ie allocate memory and resources

#### Datasource Optimizations

- Prepared Statements: SQL Statements with placeholders for variables
  - Saves server from having to parse and optimize execution plan
  - HUGE COST SAVINGS
  - Avoid SQL Injection attacks
- Optimizations within a single datasource connection:
  - Ability to cache prepared statements (may be at the server level too)
  - Use server side prepared statements
  - Statement Batching

#### Datasource Connection Pooling

- Spring Boot 1.x used Tomcat
- Spring Boot 2.x moved to HikariCP
  - HikariCP is very light weight
  - Very high performance!
- Hikari has a number of configuration options

#### Hackers Guide to Connection Pool Tuning

- Every RDMS will accept a max number of connections - each connection has a cost!
- If running multiple instances of your microservice, keep number of pool connections lower
  - If fewer, can go to a higher of connections per instance
- MySQL defaults to a limit 151 connections
  - Can be adjusted to much higher - depending on the hardware running MySQL
- Statement caching is good
  - BUT - does consume memory on the server
- Disabling autocommit can help improve performance

#### HickariCP Configuration for MySQL

- More on HickariCP - https://github.com/brettwooldridge/HikariCP
- More on MySQL Config - https://github.com/brettwooldridge/HikariCP/wiki/MySQL-Configuration

```
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

### KEY Terms

- JDBC Complaint Data source

- JDBC is a standard for connecting to Relational Databases
- Data Source Configuration tells Java how to connect to a JDBC complaint Data Source

- Use an account with minimal authority in the database

- Datasource Connection Pooling

- Connection Pooling

  - Spring Boot 1.x used Tomcat
  - Spring Boot 2.x moved to HikariCP

- Grabbing all Performance Benchmarks

- Connection cost - Memory on server, CPU Resources on Server, Time it takes to connect etc.

- Benchmarks
- Microbenchmarks
- Caching Prepared Statements
