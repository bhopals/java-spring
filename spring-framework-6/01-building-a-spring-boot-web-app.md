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

### Application Overview

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

### KEY TERMS

- Spring 6.0
- Spring Boot 3.0
- Spring Framework is a collection of framework libraries(Dependency Injection, Web, Transaction Management)
- Spring Boot is automated tooling for Spring Framework Application(wrapper arround Spring)

- MVC Controllers
- Spring Services
- Spring Data JPA
- JPA Entities
