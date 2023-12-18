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
