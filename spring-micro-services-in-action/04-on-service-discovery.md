## On Service Discovery

- To Find a physical address of where a machine is located
- To locate the physical location of those resource.

- Service Discovery

  - Horizontal Scaling
  - Increase application Resiliency

- Service discovery is critical to microservice, cloud-based applications for two key
  reasons.

  - First, it offers the application team the ability to quickly horizontally scale up
    and down the number of service instances running in an environment.

  - The second benefit of service discovery is that it helps increase application resiliency.
    When a microservice instance becomes unhealthy or unavailable, most service
    discovery engines will remove that instance from its internal list of available services.

### Traditional way of locating a service

- A traditional service location resolution model using DNS and a load balancer

An application needs to invoke a service located in another part of the organization. It attempts to invoke the service by using a generic DNS name along with a path that uniquely represents the service that the application was trying to invoke. The DNS
name would resolve to a commercial load balancer, such as the popular F5 load balancer.

The load balancer, upon receiving the request from the service consumer, locates the
physical address entry in a routing table based on the path the user was trying to
access. This routing table entry contains a list of one or more servers hosting the service. The load balancer then picks one of the servers in the list and forwards the request onto that server.

Each instance of a service is deployed to one or more application servers. The
number of these application servers was often static (for example, the number of
application servers hosting a service didn’t go up and down) and persistent (for example, if a server running an application server crashed, it would be restored to the same state it was at the time of the crash, and would have the same IP and configuration that
it had previously.)

To achieve a form of high availability, a secondary load balancer is sitting idle and
pinging the primary load balancer to see if it’s alive. If it isn’t alive, the secondary load
balancer becomes active, taking over the IP address of the primary load balancer and
beginning serving requests.

While this type of model works well with applications running inside of the four
walls of a corporate data center and with a relatively small number of services running
on a group of static servers, it doesn’t work well for cloud-based microservice applications.

- Reasons for this include:

  - `Single point of failure` — While the load balancer can be made highly available, it’s
    a single point of failure for your entire infrastructure. If the load balancer goes
    down, every application relying on it goes down too. While you can make a load
    balancer highly available, load balancers tend to be centralized chokepoints
    within your application infrastructure.

  - `Limited horizontal scalability` — By centralizing your services into a single cluster of
    load balancers, you have limited ability to horizontally scale your load-balancing
    infrastructure across multiple servers. Many commercial load balancers are constrained by two things: their redundancy model and licensing costs. Most commercial load balancers use a hot-swap model for redundancy so you only have a
    single server to handle the load, while the secondary load balancer is there only
    for fail-over in the case of an outage of the primary load balancer. You are, in
    essence, constrained by your hardware. Second, commercial load balancers also
    have restrictive licensing models geared toward a fixed capacity rather than a
    more variable model.

  - `Statically managed` — Most traditional load balancers aren’t designed for rapid
    registration and de-registration of services. They use a centralized database to
    store the routes for rules and the only way to add new routes is often through
    the vendor’s proprietary API (Application Programming Interface).

  - `Complex` — Because a load balancer acts as a proxy to the services, service consumer requests have to have their requests
    mapped to the physical services. This translation layer often added a layer of complexity to your service infrastructure because the mapping rules for the service have to be defined and deployed by hand. In a traditional load balancer scenario, this registration of new service instances was done by hand and not at startup time of a new service instance.

- It does not scale effectively and isn't cost effective

### service discovery in the cloud

- To implement a robust-service discovery mechanism for cloudbased applications.
- The solution for a cloud-based microservice environment is to use a service-discovery mechanism that’s:

  - `Highly available` — Service discovery needs to be able to support a “hot” clustering environment where service lookups
    can be shared across multiple nodes in a service discovery cluster. If a node becomes unavailable, other nodes
    in the cluster should be able to take over.

  - `Peer-to-peer` — Each node in the service discovery cluster shares the state of a service instance.

  - `Load balanced`—Service discovery needs to dynamically load balance requests across all service instances to ensure that
    the service invocations are spread across all the service instances managed by it. In many ways, service discovery
    replaces the more static, manually managed load balancers used in many early web application implementations.

  - `Resilient` — The service discovery’s client should “cache” service information locally. Local caching allows for
    gradual degradation of the service discovery feature so that if service discovery service does become unavailable, applications can still function and locate the services based on the information maintained in its local cache.

  - `Fault-tolerant` — Service discovery needs to detect when a service instance isn’t healthy and remove the instance
    from the list of available services that can take client requests. It should detect these faults with services and
    take action without human intervention.

### The architecture of service discovery

- General concepts:

  - Service registration — How does a service register with the service discovery agent?
  - Client lookup of service address — What’s the means by which a service client looks up service information?
  - Information sharing — How is service information shared across nodes?
  - Health monitoring — How do services communicate their health back to the service discovery agent?

- Client applications never have direct knowledge of the IP address of a service. Instead they get it from
  a service discovery agent.

  - 1. A services location can be looked up by a logical name from the service discovery agent.

  - 2. When a service comes online it registers its IP address with a service discovery agent.

  - 3. Service discovery nodes share service instance health information among each other.

  - 4. Services send a heartbeat to the service discovery agent. If a service dies, the service discovery layer removes
       the IP of the “dead” instance

As service instances start up, they’ll register their physical location, path, and port
that they can be accessed by with one or more service discovery instances. While each
instance of a service will have a unique IP address and port, each service instance that
comes up will register under the same service ID. A service ID is nothing more than a
key that uniquely identifies a group of the same service instances.

A service will usually only register with one service discovery service instance. Most
service discovery implementations use a peer-to-peer model of data propagation
where the data around each service instance is communicated to all the other nodes
in the cluster.

Depending on the service discovery implementation, the propagation mechanism might use a hard-coded list of
services to propagate to or use a multi-casting protocol like the “gossip”2 or “infection-style”3
protocol to allow other nodes to “discover” changes in the cluster.

Finally, each service instance will push to or have pulled from its status by the service discovery service. Any services
failing to return a good health check will be removed from the pool of available service instances.

Once a service has registered with a service discovery service, it’s ready to be used by an application or service that
needs to use its capabilities. Different models exist for a client to “discover” a service. A client can rely solely on the
service discovery engine to resolve service locations each time a service is called. With this approach, the service
discovery engine will be invoked every time a call to a registered microservice instance is made. Unfortunately, this approach
is brittle because the service client is completely dependent on the service discovery engine to be running to find
and invoke a service.

A more robust approach is to use what’s called `client-side` load balancing.

- Client Side Load Balancing

  - 1. When a service client needs to call a service it will check a local cache for the service instance IPs.
       Load balancing between service instances will occur on the service.

  - 2. If the client finds a service IP in the cache, it will use it. Otherwise it goes to the service discovery.

  - 3. Periodically, the client-side cache will be refreshed with the service discovery layer.

- Client-side load balancing caches the location of the services so that the service client doesn’t have to contact
  service discovery on every call.

### Service discovery in action using Spring and Netflix Eureka

- Setting up Service Discovery Agent and then registering services with the agent.

- Spring Cloud and Netflix’s Eureka service discovery engine
- For the client-side load balancing, use Spring Cloud and Netflix’s Ribbon libraries.

- By implementing client-side caching and Eureka with the licensing and organization services, you can lessen the load on
  the Eureka servers and improve client stability if Eureka becomes unavailable.

- STEPS:

  - 1. As service instances start, they will register their IPs with Eureka.
  - 2. When the licensing service calls the organization service, it will use Ribbon to see if the organization service
       IPs are cached locally
  - 3. Periodically, Ribbon will refresh its cache of IP addresses.

#### Building your Spring Eureka Service

```
// POM.XML
<dependencies>
 <dependency>
 <groupId>org.springframework.cloud</groupId>
 <artifactId>spring-cloud-starter-eureka-server</artifactId>
 </dependency>
 </dependencies>
```

```
// Spring Boot Starter Class
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
 public static void main(String[] args) {
 SpringApplication.run(EurekaServerApplication.class, args);
 }
}
```

```
// application.properties
server.port: 8761
eureka.client.registerWithEureka: false
eureka.client.fetchRegistry: false
eureka.client.server.waitTimeInMsWhenSyncEmpty: 5
```

With that, Spring-based Eureka server is up and running.

#### Registering services with Spring Eureka

```
// POM.XML
<dependency>
 <groupId>org.springframework.cloud</groupId>
 <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
```

```
spring.application.name= organizationservice
spring.profiles.active=default
spring.cloud.config.enabled=true
eureka.instance.preferIpAddress= true
eureka.client.registerWithEureka= true
eureka.client.fetchRegistry= true
eureka.client.serviceUrl.defaultZon= http://localhost:8761/eureka/
```

Every service registered with Eureka will have two components associated with it: the `application ID` and the `instance ID`.

- Why prefer IP address?
  By default, Eureka will try to register the services that contact it by hostname. This
  works well in a server-based environment where a service is assigned a DNS-backed
  host name. However, in a container-based deployment (for example, Docker), containers will be started with randomly generated hostnames and no DNS entries for the containers.
  If you don’t set the eureka.instance.preferIpAddress to true, your client
  applications won’t properly resolve the location of the hostnames because there will
  be no DNS entry for that container. Setting the preferIpAddress attribute will
  inform the Eureka service that client wants to be advertised by IP address.
  Personally, we always set this attribute to true. Cloud-based microservices are supposed to be ephemeral and stateless. They can be started up and shut down at will.
  IP addresses are more appropriate for these types of services.

#### Using service discovery to look up a service

The libraries we’ll explore include

- Spring Discovery client
- Spring Discovery client enabled RestTemplate
- Netflix Feign client

```
@RequestMapping(value="/{licenseId}/{clientType}",
method = RequestMethod.GET)
public License getLicensesWithClient(
 @PathVariable("organizationId") String organizationId,
 @PathVariable("licenseId") String licenseId,
 @PathVariable("clientType") String clientType) {
 return licenseService.getLicense(organizationId,
 licenseId, clientType);
}
```

In this code, the clientType parameter passed on the route will drive the type of client we’re going to use in the code examples. The specific types you can pass in on this route include:

- 1.  Discovery—Uses the discovery client and a standard Spring RestTemplate class to invoke the organization service
- 2.  Rest—Uses an enhanced Spring RestTemplate to invoke the Ribbon-based service
- 3.  Feign—Uses Netflix’s Feign client library to invoke a service via Ribbon

##### 01. Looking up service instances with Spring DiscoveryClient

- The Spring DiscoveryClient offers the lowest level of access to Ribbon and the services registered within it.
- Using the DiscoveryClient, you can query for all the services registered with the ribbon client and their corresponding URLs.

// Setting up the bootstrap class to use the Eureka Discovery Client

```
@SpringBootApplication
// Activates the Spring DiscoveryClient for use
@EnableDiscoveryClient
public class Application {
 public static void main(String[] args) {
 SpringApplication.run(Application.class, args);
 }
}
```

The `@EnableDiscoveryClient` annotation is the trigger for Spring Cloud to enable the application to use the DiscoveryClient and Ribbon libraries.

// Using the DiscoveryClient to look up information

```
@Component
public class OrganizationDiscoveryClient {

    // DiscoveryClient is auto-injected into the class.
    @Autowired
    private DiscoveryClient discoveryClient;

    public Organization getOrganization(String organizationId) {
        RestTemplate restTemplate = new RestTemplate();

        // Gets a list of all the instances of organization services
        List<ServiceInstance> instances = discoveryClient.getInstances("organizationservice");

        if (instances.size()==0) return null;

        String serviceUri = String.format("%s/v1/organizations/%s", instances.get(0).getUri().toString(), organizationId);

        // Uses a standard Spring REST Template class to call the service
        ResponseEntity< Organization > restExchange = restTemplate.exchange(serviceUri, HttpMethod.GET, null, Organization.class, organizationId); // Retrieves the service endpoint we are going to call

        return restExchange.getBody();
    }
}
```

The first item of interest in the code is the DiscoveryClient. This is the class you’ll use to interact with Ribbon.
To retrieve all instances of the organization services registered with Eureka, you can use the getInstances() method, passing in the key of service you’re looking for, to retrieve a list of ServiceInstance objects.

With The DiscoveryClient, You aren’t taking advantage of Ribbon’s client side load-balancing—By calling the DiscoveryClient directly, you get back a list of services, but it becomes your responsibility
to choose which service instances returned you’re going to invoke.

You’re doing too much work—Right now, you have to build the URL that’s going to be
used to call your service. It’s a small thing, but every piece of code that you can avoid
writing is one less piece of code that you have to debug

Observant Spring developers might have noticed that you’re directly instantiating the RestTemplate class in the code. This is antithetical to normal Spring REST invocations, as normally you’d have the Spring Framework inject the RestTemplate the class
using it via the @Autowired annotation.

In summary, there are better mechanisms for calling a Ribbon-backed service.

##### 02. Invoking services with Ribbon-aware Spring RestTemplate

- How to use a RestTemplate that’s Ribbonaware.
- To use a Ribbon-aware RestTemplate class, you need to define a RestTemplate bean construction method with a Spring
  Cloud annotation called `@LoadBalanced`.

```
package com.thoughtmechanix.licenses;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;
@SpringBootApplication
public class Application {

    // The @LoadBalanced annotation tells Spring Cloud to create a Ribbon backed RestTemplate class.
    @LoadBalanced
    @Bean
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

Using the Ribbon-backed RestTemplate class pretty much behaves like a standard Spring RestTemplate class, except for one small difference in how the URL for target service is defined. Rather than using the physical location of the service in the
RestTemplate call, you’re going to build the target URL using the Eureka service ID of the service you want to call.

```
/*Package and import definitions left off for conciseness*/
@Component
public class OrganizationRestTemplateClient {
    @Autowired
    RestTemplate restTemplate;

    public Organization getOrganization(String organizationId){
        ResponseEntity<Organization> restExchange =restTemplate.exchange("http://organizationservice/v1/organizations/{organizationId}", HttpMethod.GET, null, Organization.class, organizationId);

    // When using a Ribbon-back RestTemplate, you build the target URL with the Eureka service ID.
    return restExchange.getBody();
 }
}
```

The Ribbon-enabled RestTemplate will parse the URL passed into it and use whatever is passed in as the server name as the key to query Ribbon for an instance of a service. The actual service location and port are completely abstracted from the developer.

##### 03. Invoking services with Netflix Feign client

An alternative to the Spring Ribbon-enabled RestTemplate class is Netflix’s Feign client library. The Feign library takes a different approach to calling a REST service by having the developer first define a Java interface and then annotating that interface with Spring Cloud annotations to map what Eureka-based service Ribbon will invoke.

The Spring Cloud framework will dynamically generate a proxy class that will be used
to invoke the targeted REST service. There’s no code being written for calling the service other than an interface definition.

```
// Enabling the Spring Cloud/Netflix Feign client in the licensing service
@SpringBootApplication
// The @EnableFeignClients annotation is needed to use the FeignClient in your code.
@EnableFeignClients
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

// Defining a Feign interface for calling the organization service

```
/*Package and import left off for conciseness*/
// Identify your service to Feign using the FeignClient Annotation.
@FeignClient("organizationservice")
public interface OrganizationFeignClient {
    @RequestMapping(method= RequestMethod.GET, value="/v1/organizations/{organizationId}", consumes="application/json") Organization getOrganization(@PathVariable("organizationId") String organizationId);
}
```

To use the OrganizationFeignClient class, all you need to do is autowire and use it. The Feign Client code will take care of all the coding work for you.

#### Summary

- The service discovery pattern is used to abstract away the physical location of services.

- A service discovery engine such as Eureka can seamlessly add and remove service instances from an environment without the
  service clients being impacted.

- Client-side load balancing can provide an extra level of performance and resiliency by caching the physical location
  of a service on the client making the service call.

- Eureka is a Netflix project that when used with Spring Cloud, is easy to set up
  and configure.

- You used three different mechanisms in Spring Cloud, Netflix Eureka, and Netflix Ribbon to invoke a service.
  These mechanisms included:

  – Using a Spring Cloud service DiscoveryClient
  – Using Spring Cloud and Ribbon-backed RestTemplate
  – Using Spring Cloud and Netflix’s Feign client

### KEY TERMS

- Monolithics
- Single Tenant
- Vertical Scaling
- Horizontal Scaling
- Increase application Resiliency

- F5 Load Balancer

- Load Balancer
- Service Consumer
- Routing Table
- Service Instances
- Dynamically load balance

- Hosting servers
- Limited horizontal scalability
- Centralized Chokepoints
- Load balancers tend to be centralized chokepoints within your application infrastructure
- Single clusters of Load Balancer
- Load blancing your infrastructure across multiple servers
- Commercial Load balancer
- Hot-swap model
- redundency model and licensing costs
- Most commercial load balancers use a hot-swap model for redundancy
- Primary and Secondary Load Balancer
- Vendor's proprietary API
- Load Balancer acts as a Proxy to the services
- Translation layer often adds a layer of complexity to your service infrastructure
- Mapping rules for the services needs to be defined

- Inbound (ingress) Port
- Outbound (egress) Port
- A load balancer can lock down inbound (ingress) and outbound (egress) port access to all the servers sitting behind it.
- Concept of least network access

- It does not scale effectively and isn't cost effective
- Centralizing SSL termination and managing service port security
- Implement a robust-service discovery mecchanism for cloud-based applications

- Each node in a service discovery cluster
- Dynamically load balance requests across all service instances
- Gradual degradation of Service Discovery (Using client side caching)
- Service Discovery Cluster

- Problem with Traditional Load Balancer

  - Single Point of Failure
  - Limited Horizental Scalability
  - Statically managed
  - Complex

- Cloud-based service discovery Benefits

  - Highly Available
  - Peer-to-peer
  - Load Balanced
  - Resilient
  - Fault-tolerant

- Concepts:

  - Service Registration
  - Client lookup of service address
  - Information Sharing
  - Health Monitoring

- Client Applications
- Service Discovery Layer (with Service Discover Nodes)
- Service Instances
- Service Client
- Service Discovery Agent
- Service Location
- Service Discovery Agent
- Service Discover Node
- Service Instance Health Information
- Logical Name of the Service
- Resolve service location

- peer-to-peer model of data propagation
- Multi casing protocol
- Allow other nodes to discover changes in the cluster
- A client can rely solely on the service discovery engine to resolve service locations each time a service is called.

- Client-side cache/load balancing
- Service discovery layer

- Ensures Service calls are spread across multiple service instances

- Netflix’s Ribbon - Client Load Balancing library
- Netflix Eureak - Service discovery engine

- From the lowest level of Abstraction to the highest level

- Spring Discovery Client
- Spring Discovery Agent

- Antithetical (Contrary to)
- Ribbon Aware RestTemplate
- Ribbon backed RestTemplate class
- The Spring Cloud Framework will dynamically generate a proxy class that will be used to invoke the targeted REST Service.
