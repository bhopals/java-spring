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
    - Field-based Dependency Injection refers to the use of the `@Autowired` annotation on top
      of a field or property in a class.
  - Using private properties is EVIL
    - Spring can use reflection to set private properties
    - "Works" but is slow & make testing difficult
- By Setters - Area of much debate
  - Setter-based Dependency Injection refers to the use of the @Autowired annotation on top of
    the setter method of a class
- By Constructor - Most Preferred

  - Constructor-based Dependency Injection refers to the use of the `@Autowired` annotation on top of
    a class constructor.

- For Difference, Refer - https://javarevisited.blogspot.com/2012/11/difference-between-setter-injection-vs-constructor-injection-spring-framework.html#axzz8MZ7sqU93

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

#### Primary Beans

- In case of multiple Beans of the same type, we can instruct Spring to which one to use by
  annotation the Primary Bean with `@Primary` annotation

#### Using Qualifiers

- Control how Spring wiring things
- `@Qualifier('<probable-now-of-the-instance')`
- When we create Service, we can annotate that service with name

```
@Service('propertyGreetingService')
public class GreetingServicePropertyInjected implements GreetingService {}
// Where ever we want to inject the above service, we gonna use this name - `propertyGreetingService`
// `@Qualifier('propertyGreetingService')`
```

### Spring Profiles

- Profiles are a way that you can control what bean are wired into the context and which ones are not.
- `@Profile('<profile-name>')`
- To make a default profile, `@Profile({'<profile-name>', 'default'})`

### Spring Bean Life Cycle (Bean Creation)

- 1. Instantiate
- 2. Populate Properties
- 3. Call `setBeanName` of `BeanNameAware`
- 4. Call `setBeanFactory` of `BeanFactoryAware`
- 5. Call `setApplicationContext` of `ApplicationContextAware`
- 6. Pre Initialization Bean PostProcessors
- 7. Initializing Beans `afterPropertiesSet`
- 8. Custom Init Method
- 9. Post Initialization Bean PostProcessors
- 10. BEAN READY TO USE

### Spring Bean Life Cycle (Bean Termination)

- 1. Container Shutdown
- 2. `@PreDestroy` annotated method
- 3. Disposable Bean's `destroy()`

#### Callback Interfaces

- Spring has two interfaces you can implement for callback events
- InitializingBean.afterPropertiesSet()
  - Called after properties are set
- DisposableBean.destroy()
  - Called during bean destruction in shutdown

#### LifeCycle Annotations

- Spring has two annotations you can use to hook into bean life cycle
  - `@PostConstruct` annotated methods will be called after the bean has been constructed,
    but before its returned to the requesting object
  - `@PreDestroy` is called just before the bean is destroyed by the container

#### Bean Post Processors

- Gives you a means to tap into the Spring Context life cycle and interact with beans as they are processed
- Called for all beans in context
- Implement interface `BeanPostProcessor`
  - `postProcessBeforeInitialization` - Called before bean initialization method
  - `postProcessAfterInitialization` - Called after bean initialization

#### `Aware` Interfaces

- Spring has over 14`Aware` interfaces
- These are used to access the Spring Framework infrastructure
- There are largerly used within the framework
- Rarely used by Spring Developers
- Review extensions of the Aware Interface for current interfaces
- Examples
  - ApplicationContextAware, ApplicationEventPublisherAware, BeanClassLoaderAware, BeanFactoryAware
    BeanFactoryAware, BeanNameAware, BootstrapContextAware, LoadTimeWeaverAware, MessageSourceAware,
    NotificationPublisherAware, PortletConfigAware, PortletContextAware, ResourceLoaderAware,
    ServletConfigAware, ServletContextAware
