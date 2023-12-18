## Core Technologies

### The IoC Container

- Spring’s Inversion of Control (IoC) container

- Dependency injection (DI) is a specialized form of IoC, whereby objects define their dependencies
  (that is, the other objects they work with) only through constructor arguments, arguments to a factory method, or properties that are set on the object instance after it is constructed or returned from a factory method. The IoC container then injects those dependencies when it creates the bean. This process is fundamentally the inverse (hence the name, Inversion of Control) of the bean itself controlling the instantiation or location of its dependencies by using direct construction of classes or a mechanism such as the Service Locator pattern.

- The `org.springframework.beans` and `org.springframework.context` packages are the
  basis for Spring Framework’s IoC container.

- `BeanFactory` - The root interface for accessing a Spring bean container.

- The `BeanFactory` interface provides an advanced configuration mechanism capable of managing any type of object. `ApplicationContext` is a sub-interface of `BeanFactory`. It adds:

  - Easier integration with Spring’s AOP features
  - Message resource handling (for use in internationalization)
  - Event publication
  - Application-layer specific contexts such as the `WebApplicationContext` for use in web applications.

- In short, the `BeanFactory` provides the configuration framework and basic functionality
- The `ApplicationContext` adds more enterprise-specific functionality

- In Spring, the objects that form the backbone of your application and that are managed by
  the Spring IoC container are called beans.
- A bean is an object that is instantiated, assembled, and managed by a Spring IoC container.

- Otherwise, a bean is simply one of many objects in your application. Beans, and the dependencies among
  them, are reflected in the configuration metadata used by a container.

- Bean is a fundamental building block of a Spring application and represents a reusable component that
  can be wired together with other beans to create the application's functionality

- At its core, Spring offers a container, often referred to as the Spring application context,
  that creates and manages application components.

#### Container Overview

The `org.springframework.context.ApplicationContext` interface represents the Spring IoC container and is responsible for instantiating, configuring, and assembling the beans. The container gets its instructions on what objects to instantiate, configure, and assemble by reading configuration metadata. The configuration metadata is represented in XML, Java annotations, or Java code. It lets you express the objects that compose your application and the rich interdependencies between those objects.

- Configuration Metadata

  - Annotation-based configuration: define beans using annotation-based configuration metadata.

  - Java-based configuration: define beans external to your application classes by using Java rather
    than XML files. To use these features, see the @Configuration, @Bean, @Import, and @DependsOn annotations.

- Instantiating a Container
  - ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

### Bean Overview

A Spring IoC container manages one or more beans. These beans are created with the configuration metadata that you supply to the container (for example, in the form of XML <bean/> definitions).

### Dependency Injection

Dependency injection (DI) is a process whereby objects define their dependencies (that is, the other objects with which they work) only through constructor arguments, arguments to a factory method, or properties that are set on the object instance after it is constructed or returned from a factory method. The container then injects those dependencies when it creates the bean. This process is fundamentally the inverse (hence the name, Inversion of Control) of the bean itself controlling the instantiation or location of its dependencies on its own by using direct construction of classes or the Service Locator pattern.

- DI exists in two major variants:

  - Constructor-based dependency injection
  - Setter-based dependency injection.

- Constructor-based dependency injection
  - Constructor-based DI is accomplished by the container invoking a constructor with a number
    of arguments, each representing a dependency.
    ```
    <beans>
        <bean id="beanOne" class="x.y.ThingOne">
            <constructor-arg ref="beanTwo"/>
            <constructor-arg ref="beanThree"/>
        </bean>
        <bean id="beanTwo" class="x.y.ThingTwo"/>
        <bean id="beanThree" class="x.y.ThingThree"/>
    </beans>
    ```
- Setter-based dependency injection.
  - Setter-based DI is accomplished by the container calling setter methods on your beans after invoking a no-argument constructor or a no-argument static factory method to instantiate your bean.
  ```
  public class SimpleMovieLister {
        // the SimpleMovieLister has a dependency on the MovieFinder
        private MovieFinder movieFinder;
        // a setter method so that the Spring container can inject a MovieFinder
        public void setMovieFinder(MovieFinder movieFinder) {
            this.movieFinder = movieFinder;
        }
        // business logic that actually uses the injected MovieFinder is omitted...
    }
  ```

#### Constructor-based v/s setter-based DI?

Since you can mix constructor-based and setter-based DI, it is a good rule of thumb to use constructors for mandatory dependencies and setter methods or configuration methods for optional dependencies. Note that use of the @Autowired annotation on a setter method can be used to make the property be a required dependency; however, constructor injection with programmatic validation of arguments is preferable.

The Spring team generally advocates constructor injection, as it lets you implement application components as immutable objects and ensures that required dependencies are not null. Furthermore, constructor-injected components are always returned to the client (calling) code in a fully initialized state. As a side note, a large number of constructor arguments is a bad code smell, implying that the class likely has too many responsibilities and should be refactored to better address proper separation of concerns.

Setter injection should primarily only be used for optional dependencies that can be assigned reasonable default values within the class. Otherwise, not-null checks must be performed everywhere the code uses the dependency. One benefit of setter injection is that setter methods make objects of that class amenable to reconfiguration or re-injection later. Management through JMX MBeans is therefore a compelling use case for setter injection.

Use the DI style that makes the most sense for a particular class. Sometimes, when dealing with third-party classes for which you do not have the source, the choice is made for you. For example, if a third-party class does not expose any setter methods, then constructor injection may be the only available form of DI.

#### Dependency Resolution Process

The container performs bean dependency resolution as follows:

- The ApplicationContext is created and initialized with configuration metadata that describes
  all the beans. Configuration metadata can be specified by XML, Java code, or annotations.

- For each bean, its dependencies are expressed in the form of properties, constructor arguments,
  or arguments to the static-factory method (if you use that instead of a normal constructor). These dependencies are provided to the bean, when the bean is actually created.

- Each property or constructor argument is an actual definition of the value to set, or a
  reference to another bean in the container.

- Each property or constructor argument that is a value is converted from its specified format
  to the actual type of that property or constructor argument. By default, Spring can convert a
  value supplied in string format to all built-in types, such as int, long, String, boolean, and so forth.

#### XML Shortcut with the p-namespace

The p-namespace lets you use the bean element’s attributes (instead of nested <property/> elements) to describe your property values collaborating beans, or both.

#### XML Shortcut with the c-namespace

Similar to the XML Shortcut with the p-namespace, the c-namespace, introduced in Spring 3.1, allows inlined attributes for configuring the constructor arguments rather then nested constructor-arg elements.

#### Lazy-initialized Beans

By default, ApplicationContext implementations eagerly create and configure all singleton beans as part of the initialization process. Generally, this pre-instantiation is desirable, because errors in the configuration or surrounding environment are discovered immediately, as opposed to hours or even days later. When this behavior is not desirable, you can prevent pre-instantiation of a singleton bean by marking the bean definition as being lazy-initialized. A lazy-initialized bean tells the IoC container to create a bean instance when it is first requested, rather than at startup.
`<bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>`
`<bean name="not.lazy" class="com.something.AnotherBean"/>`

#### Autowiring Collaborators

The Spring container can autowire relationships between collaborating beans. You can let Spring resolve collaborators (other beans) automatically for your bean by inspecting the contents of the ApplicationContext

- On a per-bean basis, you can exclude a bean from autowiring. In Spring’s XML format, set the autowire-candidate attribute of the <bean/> element to false

#### Bean Scopes

When you create a bean definition, you create a recipe for creating actual instances of the class defined by that bean definition.
The following table describes the supported scopes:

- singleton - (Default) Scopes a single bean definition to a single object instance for each Spring IoC container.

  - Only one shared instance of a singleton bean is managed, and all requests for beans with an ID or IDs that match that bean definition result in that one specific bean instance being returned by the Spring container.

- prototype - Scopes a single bean definition to any number of object instances.

  - The non-singleton prototype scope of bean deployment results in the creation of a new bean instance every time a request for that specific bean is made.

- request - Scopes a single bean definition to the lifecycle of a single HTTP request. That is, each HTTP request has its own instance of a bean created off the back of a single bean definition. Only valid in the context of a web-aware Spring ApplicationContext.

- session - Scopes a single bean definition to the lifecycle of an HTTP Session. Only valid in the context of a web-aware Spring ApplicationContext.

- application - Scopes a single bean definition to the lifecycle of a ServletContext. Only valid in the context of a web-aware Spring ApplicationContext.

- websocket - Scopes a single bean definition to the lifecycle of a WebSocket. Only valid in the context of a web-aware Spring ApplicationContext.

#### Request, Session, Application, and WebSocket Scopes

The `request`, `session`, `application`, and `websocket` scopes are available only if you use a web-aware Spring ApplicationContext implementation (such as XmlWebApplicationContext). If you use these scopes with regular Spring IoC containers, such as the ClassPathXmlApplicationContext, an IllegalStateException that complains about an unknown bean scope is thrown.

##### Initial Web Configuration

- To support the scoping of beans at the `request`, `session`, `application`, and `websocket` levels (web-scoped beans), some minor initial configuration is required before you define your beans. (This initial setup is not required for the standard scopes: singleton and prototype.)

- If you access scoped beans within Spring Web MVC, in effect, within a request that is processed by the Spring `DispatcherServlet`, no special setup is necessary. DispatcherServlet already exposes all relevant state.

- If you use a Servlet web container, with requests processed outside of Spring’s DispatcherServlet (for example, when using JSF), you need to register the org.springframework.web.context.request.`RequestContextListener` `ServletRequestListener`

- `DispatcherServlet`, `RequestContextListener`, and `RequestContextFilter` all do exactly the same thing, namely bind the HTTP request object to the Thread that is servicing that request. This makes beans that are request- and session-scoped available further down the call chain.

#### Customizing the Nature of a Bean

The Spring Framework provides a number of interfaces you can use to customize the nature of a bean. This section groups them as follows:

- Lifecycle Callbacks
- ApplicationContextAware and BeanNameAware
- Other Aware Interfaces

##### Lifecycle Callbacks

The JSR-250 @PostConstruct and @PreDestroy annotations are generally considered best practice for receiving lifecycle callbacks in a modern Spring application. Using these annotations means that your beans are not coupled to Spring-specific interfaces. For details, see Using @PostConstruct and @PreDestroy.

#### Initialization Callbacks

he org.springframework.beans.factory.InitializingBean interface lets a bean perform initialization work after the container has set all necessary properties on the bean. The `InitializingBean` interface specifies a single method:

`void afterPropertiesSet() throws Exception;`

Use ` @PostConstruct` instead

#### Destruction Callbacks

Implementing the org.springframework.beans.factory.DisposableBean interface lets a bean get a callback when the container that contains it is destroyed. The `DisposableBean` interface specifies a single method:

`void destroy() throws Exception;`

- Use ` @PreDestroy` instead

#### Annotation-based Container Configuration

The <context:annotation-config/> element implicitly registers the following post-processors:

- ConfigurationClassPostProcessor
- AutowiredAnnotationBeanPostProcessor
- CommonAnnotationBeanPostProcessor
- PersistenceAnnotationBeanPostProcessor
- EventListenerMethodProcessor

## KEY TERMS

- IoC (Inversion of Control) Principle
- DI (Dependency Injection) is a specialized form of IoC
- It is responsibilty of IoC Container to inject those dependencies when it creates the BEAN
- Service Locator Pattern
- The `BeanFactory` provides the configuration framework and basic functionality
- The `ApplicationContext` adds more enterprise-specific functionality
- Bean is a Fundamental building block of a Spring Application
- It Represents a reusable component that can be wired together with other beans
- At its core, Spring offers a container, often referred to as the Spring application context,
  that creates and manages application components

- The ApplicationContext interface is responsible for instantiating, managing and configuring the beans.

- Beans are created using configuration metadata that you supply
- Reference Bean
- Collaborator Bean
- The p-namespace lets you use the bean element’s attributes (instead of nested <property/> elements)
  to describe your property values collaborating beans, or both.
- web-aware Spring ApplicationContext
- Web-scoped beans - `request`, `sesion`, `websocket`
- scoped beans
