## Service Routing with Spring Cloud and Zuul

In a distributed architecture like a microservices one, there will come a point where you’ll need to ensure that key behaviors such as security, logging, and tracking of users across multiple service calls occur.

Pushing the responsibilities to implement a cross-cutting concern like security down to the individual development teams greatly increases the odds that someone will not implement it properly or will forget to do it.

To solve this problem, you need to abstract these cross-cutting concerns into a service that can sit independently and act as a filter and router for all the microservice calls in your application. This cross-cutting concern is called a services gateway. Your service clients no longer directly call a service. Instead, all calls are routed through the
service gateway, which acts as a single Policy Enforcement Point (PEP), and are then
routed to a final destination.

Spring Cloud and Netflix’s Zuul to implement a services gateway. Zuul is Netflix’s open source services gateway implementation.
The service gateway has below responsibilities:

- 1. Put all service calls behind a single URL and map those calls using service discovery to their actual service instances
- 2. Inject correlation IDs into every service call flowing through the service gateway
- 3. Inject the correlation ID back from the HTTP response sent back from the client
- 4. Build a dynamic routing mechanism that will route specific individual organizations to a service instance endpoint
     that’s different than what everyone else is using

- Zuul at its heart is a reverse proxy

### What is a services gateway?

- Without a services gateway, the service client will call distinct endpoints for each service.

  - When a service client invokes a service directly, there’s no way you can easily implement cross-cutting concerns
    such as security or logging without having each service implement this logic directly in the service.

- The service gateway sits between the service client and the corresponding service instances. All service calls
  (both internal-facing and external) should flow through the service gateway.

  - The client invokes the service by calling the services gateway.
  - The services gateway pulls apart the URL being called and maps the path to a service sitting behind the services gateway

Because a service gateway sits between all calls from the client to the individual services, it also acts as a central Policy Enforcement Point (PEP) for service calls. The use of a centralized PEP means that cross-cutting service concerns can be implemented in a single place without the individual development teams having to implement these concerns. Examples of cross-cutting concerns that can be implemented in a service gateway include:

- 1. Static routing
- 2. Dynamic routing
- 3. Authentication and authorization
- 4. Metric collection and logging

In this case, a load balancer sitting in front of multiple service gateway instances is an appropriate design and ensures your service gateway implementation can scale. Having a load balancer sit in front of all your service instances isn’t a good idea because it becomes a bottleneck.

- Keep any code you write for your service gateway stateless.
- Keep the code you write for your service gateway light. The service gateway is the “chokepoint” for your service
  invocation. Complex code with multiple database calls can be the source of difficult-to-track-down performance problems in
  the service gateway.

### Introducing Spring Cloud and Netflix Zuul

Spring Cloud integrates with the Netflix open source project Zuul. Zuul is a services gateway that’s extremely easy to set up and use via Spring Cloud annotations. Zuul offers a number of capabilities, including:

- Mapping the routes for all the services in your application to a single URL
- Building filters that can inspect and act on the requests coming through the gateway

To get started with Zuul, you’re going to do three things:

- 1. Set up a Zuul Spring Boot project and configure the appropriate Maven dependences.
- 2. Modify your Spring Boot project with Spring Cloud annotations to tell it that it will be a Zuul service.
- 3. Configure Zuul to communicate with Eureka (optional).

#### 1. Set up a Zuul Spring Boot project and configure the appropriate Maven dependences.

```
<dependency>
 <groupId>org.springframework.cloud</groupId>
 <artifactId>spring-cloud-starter-zuul</artifactId>
</dependency>
```

#### 2. Modify your Spring Boot project with Spring Cloud annotations to tell it that it will be a Zuul service.

```
@SpringBootApplication
@EnableZuulProxy // Enables the service to be a Zuul server
public class ZuulServerApplication {
  public static void main(String[] args) {
    SpringApplication.run(ZuulServerApplication.class, args);
  }
}
```

#### 3. Configure Zuul to communicate with Eureka

The Zuul proxy server is designed by default to work on the Spring products. As such, Zuul will automatically use Eureka to look up services by their service IDs and then use Netflix Ribbon to do client-side load balancing of requests from within Zuul.

```
eureka.instance.preferIpAddress= true
eureka.client.registerWithEureka= true
eureka.client.fetchRegistry=true
eureka.client.serviceUrl.defaultZone="http://localhost:8761/eureka/"
```

### Configuring routes in Zuul

- A reverse proxy is a server that sits in front of one or more web servers, intercepting requests from clients.
  This is different from a forward proxy, where the proxy sits in front of the clients.

Zuul at its heart is a reverse proxy. A reverse proxy is an intermediate server that sits between the client trying to reach a resource and the resource itself. The client has no idea it’s even communicating to a server other than a proxy. The reverse proxy takes care of capturing the client’s request and then calls the remote resource on the client’s behalf.

In the case of a microservices architecture, Zuul (your reverse proxy) takes a microservice call from a client and forwards it
onto the downstream service. The service client thinks it’s only communicating with Zuul. For Zuul to communicate with
the downstream clients, Zuul has to know how to map the incoming call to a downstream route. Zuul has several mechanisms to do this, including:

- 1. Automated mapping of routes via service discovery
- 2. Manual mapping of routes using service discovery
- 3. Manual mapping of routes using static URLs

#### 1. Automated mapping routes via service discovery

All route mappings for Zuul are done by defining the routes in the zuulsvr/src/main/ resources/application.yml file. However, Zuul can automatically route requests based on their service IDs with zero configuration. If you don’t specify any routes, Zuul will
automatically use the Eureka service ID of the service being called and map it to a downstream service instance. For instance, if you wanted to call your organizationservice and used automated routing via Zuul, you would have your client call the Zuul service instance, using the following URL as the endpoint: `http://localhost:5555/organizationservice/v1/organizations/e254f8c-c442-4ebea82a-e2fc1d1ff78a`

Your Zuul server is accessed via `http://localhost:5555` The service you’re trying (organizationservice) to invoke is represented by the first part of the endpoint path in the service.

If you want to see the routes being managed by the Zuul server, you can access the routes via the /routes endpoint (`http://localhost:5555/routes`) on the Zuul server. This will return a listing of all the mappings on your service.

#### 2. Mapping routes manually using service discovery

Zuul allows you to be more fine-grained by allowing you to explicitly define route mappings rather than relying solely on the automated routes created with the service’s Eureka service ID.

- Have the flexibility to customize the URL's
- Add prefixes - `/api/`

```
zuul.ignored-services= "*"
zuul.prefix= "/api"
zuul.routes.organizationservice= "/organization/**"
zuul.routes.licensingservice= "/licensing/**"
```

#### 3. Manual mapping of routes using static URLs

Zuul can be used to route services that aren’t managed by Eureka. In these cases, Zuul can be set up to directly route to a statically defined URL.

```
zuul.routes.licensestatic.path= "/licensestatic/**"
zuul.routes.licensestatic.url= "http://licenseservice-static:8081"
```

#### Dynamically reload route configuration

Just commit the changes to GitHub. Zuul exposes a POST-based endpoint route `/refresh` that will cause it to reload its route configuration. Once this `/refresh` is hit, if you then hit the `/routes` endpoint, you’ll see that the two new
routes are exposed (The changes we made).

#### Zuul and service timeouts

Zuul uses Netflix’s Hystrix and Ribbon libraries to help prevent long-running service calls from impacting the performance of the services gateway. By default, Zuul will terminate and return an HTTP 500 error for any call that takes longer than one second to
process a request. (This is the Hystrix default.)

### The real power of Zuul: filters

While being able to proxy all requests through the Zuul gateway does allow you to simplify your service invocations, the real power of Zuul comes into play when you want to write custom logic that will be applied against all the service calls flowing through
the gateway. Most often this custom logic is used to enforce a consistent set of application policies like security, logging, and tracking against all the services.

These application policies are considered cross-cutting concerns because you want
them to be applied to all the services in your application without having to modify
each service to implement them.

- Zuul supports three types of filters:

  - 1. Pre-filters - Authentication
    - `TrackingFilter`
  - 2. Post-filters - Log response back from target service
    - `ResponseFilter`
  - 3. Route-filters - Add Dynamic routing / Intercept the call
    - `SpecialRouteFilter`

- Invocation STEPS:

  - 1. Service client calls the service through Zuul
  - 2. `Pre-route filters` are executed as the incoming request comes into Zuul
  - 3. `Route filters` allow you to override Zuul’s default routing logic and will route a user to where they need to go.
  - 4. A route filter may dynamically route services outside Zuul.
  - 5. In the end, Zuul will determine the target route and send the request onto its target destination.
  - 6. After the target service is invoked, the response back from it will flow back through any Zuul `post filter`.

### Building your first Zuul pre-filter generating correlation IDs

- All Zuul filters must extend the `ZuulFilter` class and override four methods:
  - `filterType()`, `filterOrder()`, `shouldFilter()`, and `run()`.

```
import com.netflix.zuul.ZuulFilter;
import org.springframework.beans.factory.annotation.Autowired;
//Removed other imports for conciseness
@Component
public class TrackingFilter extends ZuulFilter{
    private static final int FILTER_ORDER = 1;
    private static final boolean SHOULD_FILTER=true;
    private static final Logger logger = LoggerFactory.getLogger(TrackingFilter.class);

    @Autowired
    FilterUtils filterUtils;

    @Override
    public String filterType() {
        return FilterUtils.PRE_FILTER_TYPE;
    }

    @Override
    public int filterOrder() {
        return FILTER_ORDER;
    }

    public boolean shouldFilter() {
        return SHOULD_FILTER;
    }

    private boolean isCorrelationIdPresent(){
        if (filterUtils.getCorrelationId() !=null){
            return true;
        }
            return false;
    }

    private String generateCorrelationId(){
        return java.util.UUID.randomUUID().toString();
    }

    public Object run() {
        if (isCorrelationIdPresent()) {
            logger.debug("tmx-correlation-id found in tracking filter: {}.", filterUtils.getCorrelationId());
        }
        else{
            filterUtils.setCorrelationId(generateCorrelationId());
            logger.debug("tmx-correlation-id generatedin tracking filter: {}.", filterUtils.getCorrelationId());
        }

        RequestContext ctx = RequestContext.getCurrentContext();
        logger.debug("Processing incoming request for {}.",
        ctx.getRequest().getRequestURI());
        return null;
    }
}
```

#### Using the correlation ID in your service calls

- The correlation-ID is readily accessible to the microservice that’s being invoked
- Any downstream service calls the microservice might make also propagate the correlation-ID on to the downstream call

- STEPS:
  - 1.  The licensing service is invoked via a route in Zuul.
  - 2.  The UserContextFilter will retrieve the correlation ID out of the HTTP header and store it in the UserContext object.
  - 3.  The business logic in the service has access to any values retrieved in the UserContext.
  - 4.  The UserContextInterceptor ensures that all outbound REST calls have the correlation ID from the UserContext in them.

#### Building a Post filter receiving Correlation IDs

- `ResponseFilter`

```
@Component
public class ResponseFilter extends ZuulFilter{
    private static final int FILTER_ORDER=1;
    private static final boolean SHOULD_FILTER=true;
    private static final Logger logger =LoggerFactory .getLogger(ResponseFilter.class);

    @Autowired
    FilterUtils filterUtils;

    @Override
    public String filterType() {
        return FilterUtils.POST_FILTER_TYPE;
    }

    @Override
    public int filterOrder() {
        return FILTER_ORDER;
    }

    @Override
    public boolean shouldFilter() {
        return SHOULD_FILTER;
    }

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        logger.debug("Adding the correlation id to the outbound headers. {}", filterUtils.getCorrelationId());
        ctx.getResponse().addHeader(FilterUtils.CORRELATION_ID,filterUtils.getCorrelationId());
        logger.debug("Completing outgoing request for {}.",ctx.getRequest().getRequestURI());
        return null;
    }
}
```

#### Building a dynamic route filter

- `SpecialRoutesFilter`

- STEPS:
  - 0. Service client calls thservice through Zuul
  - 1. SpecialRoutesFilter retrieves the service ID.
  - 2. SpecialRoutes service checks if there’s a new alternate endpoint service, and the percentage of calls (weight number)
       to be sent to new versus old service
  - 3.  SpecialRoutesFilter generates random number and checks against weight number to determine routing.
  - 4.  If request was routed to new alternate service endpoint, Zuul still routes response back through
        any pre-defined post filters.

### SUMMARY

- Spring Cloud makes it trivial to build a services gateway.

- The Zuul services gateway integrates with Netflix’s Eureka server and can automatically map services registered with Eureka
  to a Zuul route.

- Zuul can prefix all routes being managed, so you can easily prefix your routes with something like `/api`.

- Using Zuul, you can manually define route mappings. These route mappingsare manually defined in the applications
  configuration files.

- By using Spring Cloud Config server, you can dynamically reload the route mappings without having to restart the Zuul server.

- You can customize Zuul’s Hystrix and Ribbon timeouts at global and individual service levels.

- Zuul allows you to implement custom business logic through Zuul filters. Zuul has three types of filters:

  - `pre-`, `post`, and `routing` Zuul filters.

- Zuul `pre-filters` can be used to generate a correlation ID that can be injected into every service flowing through Zuul.

- A Zuul `post filter` can inject a correlation ID into every HTTP service response back to a service client.

- A custom Zuul `route filter` can perform dynamic routing based on a Eureka service ID to do A/B testing between
  different versions of the same service.

### KEY TERMS

- Consistently enforced across all of your services
- Abstract these cross-cutting concerns
- Service Clients
- Service Gateway
- Policy Enforcement Point (PEP)
- Act as a Single Policy Enforcement Point
- Use Spring Cloud and Netflix’s Zuul to implement a services gateway.

- Reverser Proxy
  - Shield the servers
- Forward Proxy

  - Shiled Web clients

- Communicate with the down-stream clients
- Map incoming call to down-stream roure
- Eurkea Service ID

- Fine-grained control
- Cross-cutting concerns

- Service Client
- Zuul services gateway
  - Edge Server
- Filters
  - Pre-filter
  - Post-filter
  - Route-filter
- Target Service

- Downstream service calls

- Zuul pre-filters can be used to generate a correlation ID that can be injected into every service flowing through Zuul.
- A Zuul post filter can inject a correlation ID into every HTTP service response back to a service client.
