## 01. Build a Spring Boot Web App

- Spring Framework is the most popular Java framework for building enterprise grade applications.
- Enterprise grade being highly scalable and reliable applications.
- Can be used for Traditional monolith applications
- Well suited for modern micro service based architectures
- Commonly used as the backend technology

### Introduction to Spring Framework

- Introduced by Rod Johnson in 2003
- Started as a simpler alternative to J2EE
- Rod Authored book called "Expert One on One J2EE without EJB
- EJB-Enterprise Java Beans, aka "XML Hell"
- Focuse on using POJOs to Simplify development
- March 2004 Spring Framework 1.0 Released
- Quickly adopted in the Java Community

### Spring Framework History

- 2007 - Spring Framework 2.5 Released with Annotation based configuration
- August 2009 - Spring Source Purchased by VMWare for $420 million
- December 2009 - Spring 3.0 Released with Java based configuration
- April 2013 - VMWare forms new company, Pivotal. All Spring applications moved to this company
- April 2014 - Spring Boot 1.0 Released
- July 2017 - Spring Framework 5 Released, Introduction of Reactive Programming
- Nov 2017 - Spring Boot 2.0 Released
- Fall of 2022 - Spring Framework 6.0 Released
  - Requires Java 17+
  - Spring Framework 5.x supports Java 8 - Java 17
- Fall of 2022 - Spring Boot 3.0 Released

#### Spring Framework v/s Spring Boot

- Spring Framework is a collection of framework libraries
  - Dependency Injection, Web, Transaction Management, etc.
- Spring Boot is automated tooling for Spring Framework Application

  - Think of it as a wrapped arround Spring
  - You can use Spring Framework without using Spring Boot
  - But you cannot use Spring Boot without Spring Framework

#### Spring Framework Major Components

- Data Access/Integration
  - JDBC, ORM, OXM, JMS, Transactions
- Web
  - WebSocket, Servlet, Web, Portlet
- AOP
- Aspects
- Messaging
- Core Container
  - Beans, Core, Contextm SpEL
- Test

#### Spring Boot Features

- Curated "Starter" Dependencies for common components
- Sensible "Auto-Configuration" based on Classes found on the Classpath
  - For example, will auto-configure an in memory database if H2 is on the class path
- Externalized Configuration via files and Environment variables
- Logging auto-configuration
- Performance Metrics
- Healthcheck endpoints
- Enhanced failure information

#### Spring Projects (Some Examples)

- Spring Data - Collection of Projects for persisting data to SQL and NoSQL Databases
- Spring Cloud - Tools for distributed systems
- Spring Security - Authentication and Authorization
- Spring Session - Distributed web application sessions
- Spring Integration - Enterprise Integration Patterns
- Spring Batch - Batch Processing
- Spring State Machine - Open Source State Machine

## 02. Application Overview

#### Spring Web App Overview

- Web based Application
- Browser communicates via HTTP on port 8080
  - Normal default is 8080
- Tomcat server listens for request on port 8080
- Tomcat routes requests to Spring Boot Application
- Spring Boot responds with HTML via HTTP

- Spring MVC
  - MVC Controllers
  - Spring Services
  - Spring Data JPA
  - JPA Entities

### Spring Initilizer

- https://start.spring.io/
- Enter the details

  - Group, Artifact, Name, Description, Package name, Packaging, Java Version, Spring Boot version
    Language, Build

- Add Dependencies

  - Spring Web
  - Spring Data JPA
  - H2 Database

- Click on Generate to Download the Project

### JPA Relationships

- create package `domain`

  - Author.java
  - Book.java

- Many to Many
  - `@ManyToMany(mappedBy="authors")`
  -
  ```
    @ManyToMany
    @JoinTable(name = "author_book", joinColumns=@JoinColumn(name="book_id"), inverseJoinColumns = @JoinColumn(name="author_id"))
  ```

One thing you typically want to do when you're working with JPA entities is to set an equals and hashCode
method because Hibernate is going to use this internally to determine object equity, and it's kind
of important to implement it because there's a couple of different strategies you can use.

### Spring Data Repositories

- create package `repositories`
  - AuthorRepository.java
  - BookRepository.java
- create interface and `extends` it by `CrudRepository<T, ID>`
  `BookRepository extends CrudRepository<Book, Long>`

So with these defined what's going to occur when we run this application, Spring Data JPA is going
to provide us the implementation so we do not write the implementation of the repository that is going
to be generated for us at runtime by Spring Data JPA.

### Initializing Data with Spring

To Initialize data at the start of the application in our "H2" database

- create package `bootstrap`
- create class `Bootstrap`, annotate it with `@Component`, and implement interface
  `org.springframework.boot.CommandLineRunner`

### Introduction to H2 Database Console

- Memory database running in Spring Application Context
- To see data, we need to enable an utility that is produced by H2, which is web-based console
  that we can utilize to view the data

- To enable it, go to `application.properties`, Add `spring.h2.console.enabled=true`
- Re-Run the spring application
- Grab database connection URL details from the console
- Go to - `localhost:8080/h2-console`
- Enter JDBC Url details (`jdbc:h2:mem:<4f2ab983-8c47-457b-a147-1e7bf4008cff>`), click on Connect Button
- You should be redirected to Web based SQL console and see the data

### Introduction of Spring MVC

- MVC stands for Model View Controller, so it's a programming paradigm as to how you develop applications.
- MVC is a common design pattern for GUI and Web Applications
- MVC

  - Model - Simple POJO with collection of Properties
  - View - Data as requested by the client. Implemented with JSP, Thymeleaf, Jackson, etc.
  - Controller - Java class implemented to handle request mapping.

- Spring MVC
  - Client Request ==> Dispatcher Servlet ==> Controller (Returns Model - POJO) ==> Service ==> Spring Data JPA
  - refer attached - `01-spring-mvc.png`

### Create Service Layer

- Create interface `BookService` and Implementation Calss (`BookServiceImpl`)
- Annotate Impl class with `@Service` annotation
- `@Service` is a Spring Stereotype saying that this is a service
- `@Service` register the class as Spring Component
- Add method implementation

```
    @Service
    public class BookServiceImpl implements BookService {
        private final BookRepository bookRepository;
        public BookServiceImpl(BookRepository bookRepository) {
            this.bookRepository = bookRepository;
        }
        @Override
        public Iterable<Book> findAll() {
            return bookRepository.findAll();
        }
    }
```

### Configuring Spring Controllers

- Create Controller Calss (`Controller`)
- Annotate Impl class with `@Controller` annotation

```
@Controller
public class BookController {

	private final BookService bookService;

	public BookController(BookService bookService) {
		this.bookService = bookService;
	}

	@RequestMapping("/books")
	public String getBooks(Model model) {
		model.addAttribute("books", bookService.findAll());
		return "books";
	}

}
```

### Thymeleaf Templates

- Add `thymeleaf` dependeny in `pom.xml`

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

- Add `books.html` under /resources/
- Restart the Spring Boot Server
- Go to `http://localhost:8080/books` to get the list of the books

### KEY TERMS

- Spring 6.0
- Spring Boot 3.0
- Spring Framework is a collection of framework libraries(Dependency Injection, Web, Transaction Management)
- Spring Boot is automated tooling for Spring Framework Application(wrapper arround Spring)

- MVC Controllers
- Spring Services
- Spring Data JPA
- JPA Entities

- Spring Boot JPA is a Java specification for managing relational data in Java applications.
- It allows us to access and persist data between Java object/ class and relational database.
- JPA follows Object-Relation Mapping (ORM).
- Object-Relational metadata
- The API itself, defined in the persistence package
- The Java Persistence API
- It is a powerful repository and custom object-mapping abstraction.
- It supports for cross-store persistence.
- Java Persistence API
- Vendor JPA Implementation
- ORM Vendor
- The ORM layer exists between the application and the database

### KEY TERMS

- SOLID Principles of OOP
  - S - Single Responsibility Principle
  - O - Open Closed Principle
  - L - Liskov Substitution Principle
  - I - Interface Segregation Principle
  - D - Dependency Inversion Principle
- Software entities (e.g., classes, modules, functions)
- REST, or REpresentational State Transfer, is an architectural style for providing standards between
  computer systems on the web, making it easier for systems to communicate with each other
