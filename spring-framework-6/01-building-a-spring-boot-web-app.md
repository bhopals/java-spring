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

## 03. Dependency Injection with Spring

### Intro

- A fundamental tool in Spring Framework
- Allows you to inject dependencies into your application at runtime

### SOLID Principles of OOP

- SOLID principles are object-oriented design concepts relevant to software development.

- History of SOLID Principles of OOP

  - The SOLID Principles of OOP date back to March of 1955
  - The Principles are from Robert Martin (aka Uncle Bob)
  - Started as writing and blog posts
  - Later became the foundation of Martin's
    book: "Agile Software Development: Priciples, Patterns, and Practices"
  - Michael Father is credited with establishing the SOLID arconym

- SOLID Principles of OOP

  - S - Single Responsibility Principle
  - O - Open Closed Principle
  - L - Liskov Substitution Principle
  - I - Interface Segregation Principle
  - D - Dependency Inversion Principle

- The SOLID Principles of OOP will lead you to better quality code
- Your code will be more testable and easier to maintain
- A key theme is avoiding tight coupling in your code
- Be pragmatic when following SOLID
  - Every request path does need its own controller class

#### Single Responsibility Principle

- Each class should be responsible for a single part or functionality of the system.
- A class should only have one responsibility. Furthermore, it should only have one reason to change.

- Every Class should have a single responsibility
- There should never be more than one reason for a class to change
- Your classes should be small. No more than a screen full of code
- Avoid `god` classes
- Split Big classes into Smaller classes

- Example:

Every class in Java should have a single job to do. To be precise, there should only be one reason to change a class. Here’s an example of a Java class that does not follow the single responsibility principle (SRP):

```
public class Vehicle {
    public void printDetails() {}
    public double calculateValue() {}
    public void addVehicleToDB() {}
}
```

The Vehicle class has three separate responsibilities: reporting, calculation, and database. By applying SRP, we can separate the above class into three classes with separate responsibilities.

#### Open Closed Principle

- Software components should be open for extension, but not for modification.
- Classes should be open for extension but closed for modification. In doing so,
  we stop ourselves from modifying existing code and causing potential new bugs.
- one exception to the rule is when fixing bugs in existing code.
- Your classes should be open for extension
- But closed for modification
- You should be able to extend a class behaviour, without modifying it
- Use private variables with getters and setters - ONLY when you need them
- Use abstract base classes

- Example:
  Software entities (e.g., classes, modules, functions) should be open for an extension, but closed for modification.

Consider the below method of the class VehicleCalculations:

```
public class VehicleCalculations {
public double calculateValue(Vehicle v) {
if (v instanceof Car) {
return v.getValue() _ 0.8;
if (v instanceof Bike) {
return v.getValue() _ 0.5;

    }
}}
}
```

Suppose we now want to add another subclass called Truck. We would have to modify the above class by adding another if statement, which goes against the Open-Closed Principle.
A better approach would be for the subclasses Car and Truck to override the calculateValue method:

```
public class Vehicle {
    public double calculateValue() {...}
}
public class Car extends Vehicle {
    public double calculateValue() {
        return this.getValue() * 0.8;
    }
}
public class Truck extends Vehicle{
    public double calculateValue() {
        return this.getValue() * 0.9;
    }
}
```

Adding another Vehicle type is as simple as making another subclass and extending from the Vehicle class.

#### Liskov Substitution Principle

- Objects of a superclass should be replaceable with objects of its subclasses without breaking the system.
- If class A is a subtype of class B, we should be able to replace B with A without disrupting
  the behavior of our program.
- By Barbara Liskov, in 1988
- Objects in a Program would be replaceable with instances of their subtypes WITHOUT altering
  the correctness of the program
- Violations will oftern fail the "Is a" test
- A Square "Is a" Rectangle
- However, a Rectangle "Is Not" a Square

- Example:
  The Liskov Substitution Principle (LSP) applies to inheritance hierarchies such that derived classes must be completely substitutable for their base classes.

Consider a typical example of a Square derived class and Rectangle base class:

```
public class Rectangle {
    private double height;
    private double width;
    public void setHeight(double h) { height = h; }
    public void setWidht(double w) { width = w; }
    ...
}
public class Square extends Rectangle {
    public void setHeight(double h) {
        super.setHeight(h);
        super.setWidth(h);
    }
    public void setWidth(double w) {
        super.setHeight(w);
        super.setWidth(w);
    }
}

```

The above classes do not obey LSP because you cannot replace the Rectangle base class with its derived class Square. The Square class has extra constraints, i.e., the height and width must be the same. Therefore, substituting Rectangle with Square class may result in unexpected behavior

#### Interface Segregation Principle

- No client should be forced to depend on methods that it does not use.
- Larger interfaces should be split into smaller ones. By doing so, we can ensure that implementing
  classes only need to be concerned about the methods that are of interest to them.

- Make fine grained interfaces that are client specific
- Many client specific interfaces are better than one "General Purpose" interface
- Keep your components focused and minimize dependencies between them
- Notice relationship to the Single Responsibility Principle?
- ie avoid `god` interfaces

- Example:

The Interface Segregation Principle (ISP) states that clients should not be forced to depend upon interface members they do not use. In other words, do not force any client to implement an interface that is irrelevant to them.

Suppose there’s an interface for vehicle and a Bike class:

```
public interface Vehicle {
    public void drive();
    public void stop();
    public void refuel();
    public void openDoors();
}
public class Bike implements Vehicle {

    // Can be implemented
    public void drive() {...}
    public void stop() {...}
    public void refuel() {...}

    // Can not be implemented
    public void openDoors() {...}
}
```

As you can see, it does not make sense for a Bike class to implement the openDoors() method as a bike does not have any doors! To fix this, ISP proposes that the interfaces be broken down into multiple, small cohesive interfaces so that no class is forced to implement any interface, and therefore methods, that it does not need.

#### Dependency Inversion Principle

- High-level modules should not depend on low-level modules, both should depend on abstractions.
- The principle of dependency inversion refers to the decoupling of software modules. This way,
  instead of high-level modules depending on low-level modules, both will depend on abstractions.

- Abstractions should not depend upon details
- Details should depend upon abstractions
- Important that higher level and lower level objects depend on the same abstracion interacton
- This is not the same as Dependency Injection - Which is how objects obtain dependent objects
- Abstraction should work together and should not be decoupled

- Example:
  The Dependency Inversion Principle (DIP) states that we should depend on abstractions (interfaces and abstract classes) instead of concrete implementations (classes). The abstractions should not depend on details; instead, the details should depend on abstractions

Consider the example below. We have a Car class that depends on the concrete Engine class; therefore, it is not obeying DIP.

```
public class Car {
    private Engine engine;
    public Car(Engine e) {
        engine = e;
    }
    public void start() {
        engine.start();
    }
}
public class Engine {
   public void start() {...}
}
```

The code will work, for now, but what if we wanted to add another engine type, let’s say a diesel engine? This will require refactoring the Car class.
However, we can solve this by introducing a layer of abstraction. Instead of Car depending directly on Engine, let’s add an interface:

```
public interface Engine {
    public void start();
}
```

Now we can connect any type of Engine that implements the Engine interface to the Car class:

```
public class Car {
    private Engine engine;
    public Car(Engine e) {
        engine = e;
    }
    public void start() {
        engine.start();
    }
}
public class PetrolEngine implements Engine {
   public void start() {...}
}
public class DieselEngine implements Engine {
   public void start() {...}
}
```

### Spring Context

```
@Controller
public class MyController {
	public static String sayHello() {
		System.out.println("I am in MyController");
		return "Hello MyController";
	}
}


@SpringBootApplication
public class Spring6WebappApplication {
	public static void main(String[] args) {
		ApplicationContext ctx = SpringApplication.run(Spring6WebappApplication.class, args);
		MyController ctrl = ctx.getBean(MyController.class);
		System.out.println("In Main Method");
		System.out.println(ctrl.sayHello());
	}
}
```

So we can see, Spring Application Context has access to all the beans created in Spring Context.

### Basics of Dependency Injection

- Dependency Injection is where a needed dependeny is injected by another object.
- Can be at instantiation via constructor, or after via setter
- The class being injected has no responsibility in instantiting the object being injected.
- Some say you avoid declaring objects using `new`
  - Not 100% correct
  - Be pragmatic in what is and is not being managed in the Spring Context

#### Types of Dependency Injections

- By Class properties - Least preferred
  - Can be public or private properties
  - Using private properties is EVIL
    - Spring can use reflection to set private properties
    - "Works" but is slow & make testing difficult
- By Setters - Area of much debate
- By Constructor - Most Preferred

#### Dependency Injections (DI) in Concrete Classes v/s Interfaces

- DI can be done with Concrete Classes or with Interfaces
- Generally DI with Concrete Classes should be avoided
- DI via Interfaces is highly preferred
  - Allows runtime to decide implementation to Inject
  - Follows Interface segregation Principle of SOLID
  - Also, makes your code more testable - mocking becomes trivial

#### Inversion of Control (IoC)

- Inversion of Control - aka IoC
- Is a technique to allow dependencies to be injected at runtime
- Dependencies are not predetermined
- Allows the framework to compose the application by controlling which implementation is injected
  - Example - H2 in memory data source or MySQL data source

#### IoC v/s Dependency Injection (DI)

- IoC and DI are easily confused
- DI refers much to the composition of your classes
  - ie - You compose your classes with DI in mind
  - You might write code to `inject` a dependency
- IoC is the runtime environment of your code
  - Control of dependency injection is inverted to the framework
  - Spring is in control of the injection of dependencies

#### Best Practices with Dependency Injection

- Favor using Constructor Injection over Setter Injection
- Use final properties for injected components
  - Declare property `private final` and initialize in the constructor
- Whenever practical, code to an interface

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

- SOLID Principles of OOP
  - S - Single Responsibility Principle
  - O - Open Closed Principle
  - L - Liskov Substitution Principle
  - I - Interface Segregation Principle
  - D - Dependency Inversion Principle
- Software entities (e.g., classes, modules, functions)

```

```
