## When bad things happen: client resiliency patterns with Spring Cloud and Netflix Hystrix

All systems, especially distributed systems, will experience failure. How we build our applications to respond to that
failure is a critical part of every software developer’s job. However, when it comes to building resilient systems, most software engineers only take into account the complete failure of a piece of infrastructure or a key service. They focus on building redundancy into each layer of their application using techniques such as clustering key servers, load balancing between services, and segregation of infrastructure into multiple locations.

When a service crashes, it’s easy to detect that it’s no longer there, and the application can route around it. However, when a service is running slow, detecting that poor performance and routing around it is extremely difficult because:

- 1.  Degradation of a service can start out as intermittent and build momentum
- 2.  Calls to remote services are usually synchronous and don’t cut short a long-running call
- 3.  Applications are often designed to deal with complete failures of remote resources, not partial degradations

What’s insidious about problems caused by poorly performing remote services is that they’re not only difficult to detect, but can trigger a cascading effect that can ripple throughout an entire application ecosystem. Without safeguards in place, a single
poorly performing service can quickly take down multiple applications. Cloud-based, microservice-based applications are particularly vulnerable to these types of outages because these applications are composed of a large number of fine-grained, distributed services with different pieces of infrastructure involved in completing a user’s transaction

### What are client-side resiliency patterns?

Client resiliency software patterns are focused on protecting a remote resource’s (another microservice call or database lookup) client from crashing when the remote resource is failing because that remote service is throwing errors or performing poorly. The goal of these patterns is to allow the client to “fail fast,” not consume valuable resources such as database connections and thread pools, and prevent the problem of the remote service from spreading “upstream” to consumers of the client.

There are four client resiliency patterns:

- 1. Client-side load balancing

  - The service client caches microservice endpoints retrieved during service discovery.

- 2. Circuit breakers

  - The circuit breaker pattern ensures that a service client does not repeatedly call a failing service.

- 3. Fallbacks

  - When a call does fail, fallback asks if there’s an alternative that can be executed.

- 4. Bulkheads

  - The bulkhead segregates different service calls on the service client to ensure a poor-behaving service does not use all
    the resources on the client.

- The four client resiliency patterns act as a protective buffer between a service consumer and the service.

#### Client-side load balancing

Client-side load balancing involves having the
client look up all of a service’s individual instances from a service discovery agent (like
Netflix Eureka) and then caching the physical location of said service instances.

Whenever a service consumer needs to call that service instance, the client-side load
balancer will return a location from the pool of service locations it’s maintaining.

Because the client-side load balancer sits between the service client and the service
consumer, the load balancer can detect if a service instance is throwing errors or
behaving poorly. If the client-side load balancer detects a problem, it can remove that
service instance from the pool of available service locations and prevent any future
service calls from hitting that service instance.

#### Circuit breaker

The circuit breaker pattern is a client resiliency pattern that’s modeled after an electrical circuit breaker. In an electrical system, a circuit breaker will detect if too much current is flowing through the wire. If the circuit breaker detects a problem,
it will break the connection with the rest of the electrical system and keep the downstream components from the being fried.

With a software circuit breaker, when a remote service is called, the circuit breaker will monitor the call. If the calls take too long, the circuit breaker will intercede and kill the call. In addition, the circuit breaker will monitor all calls to a remote resource and if enough calls fail, the circuit break implementation will pop, failing fast and preventing future calls to the failing remote resource.

- The key thing a circuit break patterns offers is the ability for remote calls to:
  - Fail Fast
  - Fail Gracefully
  - Recover Seamlessly

#### Fallback processing

With the fallback pattern, when a remote service call fails, rather than generating an exception, the service consumer will execute an alternative code path and try to carry out an action through another means. This usually involves looking for data from
another data source or queueing the user’s request for future processing. The user’s call will not be shown an exception indicating a problem, but they may be notified that their request will have to be fulfilled at a later date.

For instance, suppose you have an e-commerce site that monitors your user’s
behavior and tries to give them recommendations of other items they could buy. Typically, you might call a microservice to run an analysis of the user’s past behavior and return a list of recommendations tailored to that specific user. However, if the preference service fails, your fallback might be to retrieve a more general list of preferences that’s based off all user purchases and is much more generalized. This data might come from a completely different service and data source.

#### Bulkheads

The bulkhead pattern is based on a concept from building ships. With a bulkhead
design, a ship is divided into completely segregated and watertight compartments
called bulkheads. Even if the ship’s hull is punctured, because the ship is divided into
watertight compartments (bulkheads), the bulkhead will keep the water confined to
the area of the ship where the puncture occurred and prevent the entire ship from
filling with water and sinking.

The same concept can be applied to a service that must interact with multiple
remote resources. By using the bulkhead pattern, you can break the calls to remote
resources into their own thread pools and reduce the risk that a problem with one
slow remote resource call will take down the entire application. The thread pools act
as the bulkheads for your service. Each remote resource is segregated and assigned to
the thread pool. If one service is responding slowly, the thread pool for that one type
of service call will become saturated and stop processing requests. Service calls to
other services won’t become saturated because they’re assigned to other thread pools.

### Why client resiliency matters

- An application is a graph of interconnected dependencies. If you don’t manage the remote
  calls between these, one poorly behaving remote resource can bring down all the services in the graph.

- The key thing a circuit break patterns offers is the ability for remote calls to:

  - Fail Fast
  - Fail Gracefully
  - Recover Seamlessly

- The circuit breaker will occasionally let calls through to a degraded service, and if those calls succeed enough times
  in a row, the circuit breaker will reset itself.

### Hystrix

Building implementations of the circuit breaker, fallback, and bulkhead patterns requires intimate knowledge of threads and
thread management.

We can use Spring Cloud and Netflix’s Hystrix library to provide you a battletested library that’s used daily in Netflix’s microservice architecture.

Below are the STEPs:

- 1. Include the Spring Cloud/Hystrix wrappers.(Add dependency in maven build file - pom.xml)

- 2. Spring Cloud/Hystrix annotations to wrapper remote calls with a circuit breaker pattern.

- 3. Customize the individual circuit breakers on a remote resource to use custom timeouts for each call made.

- 4. Implement a fallback strategy in the event a circuit breaker has to interrupt a call or the call fails.

- 5. Use individual thread pools in your service to isolate service calls and build bulkheads between different
     remote resources being called.

#### 1. Include the Spring Cloud/Hystrix wrappers.(Add dependency in maven build file - pom.xml)

```
<dependency>
 <groupId>org.springframework.cloud</groupId>
 <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
<dependency>
 <groupId>com.netflix.hystrix</groupId>
 <artifactId>hystrix-javanica</artifactId>
 <version>1.5.9</version>
</dependency>
```

#### 2. Spring Cloud/Hystrix annotations to wrapper remote calls with a circuit breaker pattern.

- `@EnableCircuitBreaker` annotation used to activate Hystrix in a service

```
@SpringBootApplication
@EnableEurekaClient
@EnableCircuitBreaker
public class Application {
    @LoadBalanced
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

#### 3. Implementing a circuit breaker using Hystrix

- Implementing Hystrix in two broad categories.

  - 1.  wrap (with `@HystrixCommand`) all calls to your database in services with a Hystrix circuit breaker.
  - 2.  wrap (with `@HystrixCommand`) the inter-service calls between services using Hystrix circuit breaker.

```
    // @HystrixCommand annotation is used to wrapper the getLicenseByOrg() method with a Hystrix circuit breaker.
    @HystrixCommand
    public List<License> getLicensesByOrg(String organizationId){
        return licenseRepository.findByOrganizationId(organizationId);
    }
```

Now, with @HystrixCommand annotation in place, the licensing service will interrupt a call out to its database if the query takes too long.

The beauty of using method-level annotations for tagging calls with circuit-breaker
behavior is that it’s the same annotation whether you’re accessing a database or calling a microservice.

- Hystrix sits between each remote resource call and protects the client. It doesn’t
  matter if the remote resource call is a database call or a REST-based service call.

- Customizing the timeout on a circuit breaker
  `@HystrixCommand( commandProperties= {@HystrixProperty( name="execution.isolation.thread.timeoutInMilliseconds",value="12000")})`

#### 4. Implement a fallback strategy in the event a circuit breaker has to interrupt a call or the call fails.

Part of the beauty of the circuit breaker pattern is that because a “middle man” is between the consumer of a remote resource and the resource itself, you have an opportunity for the developer to intercept a service failure and choose an alternative
course of action to take.

- In Hystrix, this is known as a fallback strategy and is easily implemented

  - `@HystrixCommand(fallbackMethod = "<fallback-method-name>")`
  - `@HystrixCommand(fallbackMethod = "buildFallbackLicenseList")`

- This fallback method must reside in the same class as the original method that was protected by the `@HystrixCommand`.
- The fallback method must have the exact same method signature as the originating function as all of the parameters
  passed into the original method protected by the @HystrixCommand will be passed to the fallback.

#### 5. Use individual thread pools in your service to isolate service calls and build bulkheads

- Use individual thread pools in your service to isolate service calls and build bulkheads between
  different remote resources being called.

- All remote resource calls are in a single shared thread pool.

- A single slow-performing service can saturate the Hystrix thread pool and cause resource exhaustion in the Java
  container hosting the service.

Without using a bulkhead pattern, the default behavior for these calls is that the calls are executed using the same threads that are reserved for handling requests for the entire Java container. In high volumes, performance problems with one service out of many can result in all of the threads for the Java container being maxed out and waiting to process work, while new
requests for work back up. The Java container will eventually crash.

The bulkhead pattern segregates remote resource calls in their own thread pools so that a single misbehaving service
can be contained and not crash the container.

- Without Bulkhead - Hystrix uses One Thread pool for all Remote calls (Default)

  - Hystrix uses a thread pool to delegate all requests for remote services. By default, all Hystrix commands will share
    the same thread pool to process requests. This thread pool will have 10 threads in it to process remote service calls
    and those remote services calls could be anything, including REST-service invocations, database calls, and so on.
  - Default Hystrix thread pool shared across multiple resource types
  - The problem is if you have services that have far higher volumes or longer completion times then other services, you can
    end up introducing thread exhaustion into your Hystrix thread pools because one service ends up dominating all of the threads in the default thread pool.

- With Bulkhead
  - Fortunately, Hystrix provides an easy-to-use mechanism for creating bulkheads between different remote resource calls.
  - To implement segregated thread pools, you need to use additional attributes exposed through the @HystrixCommand annotation

```
// The `threadPoolProperties` attribute lets you define and customize the behavior of the threadPool.
// The `threadPoolKey` attribute defines the unique name of thread pool
// The `coreSize` attribute lets you define the maximum number of threads in the thread pool.
// The `maxQueueSize` lets you define a queue that sits in front of your thread pool and that can queue incoming requests
@HystrixCommand(fallbackMethod = "buildFallbackLicenseList", threadPoolKey = "licenseByOrgThreadPool",
 threadPoolProperties = {@HystrixProperty(name = "coreSize",value="30"), @HystrixProperty(name="maxQueueSize", value="10")})
public List<License> getLicensesByOrg(String organizationId){
    return licenseRepository.findByOrganizationId(organizationId);
}
```

### Getting beyond the basics; fine-tuning Hystrix

Remember, Hystrix does more than time out long-running calls. Hystrix will also monitor the number of times a call
fails and if enough calls fail, Hystrix will automatically prevent future calls from reaching the service by failing the call before the requests ever hit the remote resource.

```
@HystrixCommand(
 fallbackMethod = "buildFallbackLicenseList",
 threadPoolKey = "licenseByOrgThreadPool",
 threadPoolProperties ={
    @HystrixProperty(name = "coreSize",value="30"),
    @HystrixProperty(name="maxQueueSize"value="10"),
 },
 commandPoolProperties ={
    @HystrixProperty(name="circuitBreaker.requestVolumeThreshold", value="10"),
    @HystrixProperty(name="circuitBreaker.errorThresholdPercentage", value="75"),
    @HystrixProperty(name="circuitBreaker.sleepWindowInMilliseconds", value="7000"),
    @HystrixProperty(name="metrics.rollingStats.timeInMilliseconds", value="15000")
    @HystrixProperty(name="metrics.rollingStats.numBuckets", value="5")
})
```

### The HystrixConcurrencyStrategy in action

- Hystrix, by default, will not propagate the parent thread’s context to threads managed by a Hystrix command

- Hystrix allows you to define a custom concurrency strategy that will wrap your Hystrix calls and allows you to inject
  any additional parent thread context into the threads managed by the Hystrix command. To implement a custom
- HystrixConcurrencyStrategy you need to carry out three actions:
  - 1. Define your custom Hystrix Concurrency Strategy class
  - 2. Define a Java Callable class to inject the UserContext into the Hystrix Command
  - 3. Configure Spring Cloud to use your custom Hystrix Concurrency Strategy

### SUMMARY

- When designing highly distributed applications such as a microservice-based application, client resiliency must be
  taken into account.

- Outright failures of a service (for example, the server crashes) are easy to detect and deal with.

- A single poorly performing service can trigger a cascading effect of resource exhaustion as threads in the calling client
  are blocked waiting for a service to complete.

- Three core client resiliency patterns are the circuit-breaker pattern, the fallback pattern, and the bulkhead pattern.

- The circuit breaker pattern seeks to kill slow-running and degraded system calls so that the calls fail fast and prevent
  resource exhaustion.

- The fallback pattern allows you as the developer to define alternative code paths in the event that a remote service
  call fails or the circuit breaker for the call fails.

- The bulk head pattern segregates remote resource calls away from each other, isolating calls to a remote service
  into their own thread pool. If one set of service calls is failing, its failures shouldn’t be allowed to eat up all
  the resources in the application container.

- Spring Cloud and the Netflix Hystrix libraries provide implementations for the circuit breaker, fallback, and bulkhead patterns.

- The Hystrix libraries are highly configurable and can be set at global, class, and thread pool levels.

- Hystrix supports two isolation models: THREAD and SEMAPHORE.

- Hystrix’s default isolation model, THREAD, completely isolates a Hystrix protected call, but doesn’t propagate the
  parent thread’s context to the Hystrix managed thread.

- Hystrix’s other isolation model, SEMAPHORE, doesn’t use a separate thread to make a Hystrix call. While this is more
  efficient, it also exposes the service to unpredictable behavior if Hystrix interrupts the call.

- Hystrix does allow you to inject the parent thread context into a Hystrix managed Thread through a
  custom HystrixConcurrencyStrategy implementation.

### KEY TERMS

- Distributed Systems
- Remote Services
- Remote resourse service
- Remote resource client
- Microservice Consumer

- Failure is a critical part
- Building resilient systems

- Focus on building redundancy into each layer of the application using techniques such as

  - Clustering key servers
  - Load balancing between services
  - Segregation of Infrastructure into multiple locations

- Clustering key servers
- Load balancing between services
- Segregation of Infrastructure into multiple locations

- Call to remote services
- Partial degradation
- Deal with Complete failures of remote resources

- Resource Exhaustion
- Thread Pool or Database Connection maxes out
- Poorly performing remote services

- Applications are composed of a large number of fine-grained distributed services
- fine-grained distributed services

- Client resiliency software patterns
- Allow the client to `fail-fast`

- Microservice Consumer / Service Consumer
- Microservice Instance / Service Instance

- Implementing a Service-based architecture
- Database connections in the service container's connection pools become exhausted
- Battle-tested library

- Client-side Load Balancing
- Circuit Breaker
- Fallback
- Bulkhead

- An application is a GRAPH OF INTERCONNECTED SERVICES

- Fail Fast
- Fail Gracefully
- Recover Seamlessly

- Remote Resources
- Database calls
- Inter-service calls

- Fallbacks are a mechanism to provide a course of action when a resource has timed out or failed.

- Hystric-wrapped resource call
- Hystric worker Thread

- Saturate the Thread Pool

- Resources Segregation
- Segregates remote resource calls in their own thread pools
