## Spring Boot RestTemplate

- To interact with RestFul Web Services

### spring milestone

What is the use of spring milestones? - A milestone is a project management term.

When everyone is happy, a final version is released and the whole process begins again. In Spring land this process goes: Mx for a Milestone release, sequentially numbered. RCx for a Release Candidate, sequentially numbered.

### Http Clients

#### Communication Layers

- HTTP - Application Layer - ie how the client is communicating with the server
- TCP - Transmission Control Protocol - Transport Layer
  - How data is moved in packets between client and server
  - Server listens on a port (ie 80, 443), an ephemeral port is used on client
    to communicate back
  - Data is divided up into packets, transmitted, then re-assembled
- IP - Internet Protocol - Internet Layer
  - Specification of how packets are moved between hosts - just one packet

#### Java Input Output - IO

- Network communication in Java is done via java.io packages
- These are the low level libraries used to communicate with the host operating system
- TCP/IP connections are made via sockets
  - Light weight, but there is a cost to establish
- Early Java used one thread for each connection
  - Threads are much more costly
  - Modern OSs can support 100s of thousands of sockets, but only ~10,000 threads

#### Blocking and I/O

- Pre Java 1.4 threads would get blocked. One thread per connection.
  - Thread sleeps while IO completes
- Java 1.4 added non-blocking IO a.k.a. NIO - which allows for I/O without blocking the thread
  - Sets of sockets now can be used by a thread
- Java 1.7 added NIO.2 with asynchronous I/O
  - Networking tasks done completely in the background by the OS
- Non-Blocking is central to Reactive Programming

#### HTTP Client Performance

- Not uncommon for microservices to have many many client connections
- Non-blocking clients typically benchmark much higher than blocking clients
- Connection pooling can be used to avoid cost of thread creation & establishmnt of connections
  - Non-blocking and connection pooling can have a significant difference in
    the performance of your application
- As will all benchmarks - Your mileage may vary!!

#### Blocking Clients

- JDK - Java’s implementation
- Apache HTTP Client
- Jersey
- OkHttp - may be changing version 4 under development

### NIO (Non-blocking I/O) Clients

- Apache Async Client
- Jersey Async HTTP Client
- Netty - Used by Reactive Spring

#### HTTP/2

- HTTP/2 is more performant than HTTP 1.1
- HTTP/2 uses the TCP layer much more efficiently
  - Multiplex streams
  - Binary Protocols / Compression
  - Reduced Latency
  - Faster Encryption
- To the REST API Developer, Functionally the Same
  - Both server and client need to support HTTP/2

#### HTTP/2 HTTP Clients

- Java 9+
- Jetty
- Netty
- OkHttp
- Vert.x
- Firefly
- Apache 5.x (In beta as of August 2019)

### Apache HTTP Client Configuration

- Add Apache dependency in `POM.XML`

```
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpasyncclient</artifactId>
    <version>4.1.0</version>
</dependency>
// More - https://hc.apache.org/httpcomponents-asyncclient-4.1.x/index.html
```

- You can modify RestTemplate to use `apache http`, by implementing `RestTemplateCustomizer`
  `customize` method.

```
@Override
public void customize(RestTemplate restTemplate) {
    try {
        restTemplate.setRequestFactory(clientHttpRequestFactory());
    } catch (IOReactorException e) {
        e.printStackTrace();
    }
}
```

- BlockingIORestTemplateCustomizer

```
@Component
public class BlockingRestTemplateCustomizer implements RestTemplateCustomizer {

    public ClientHttpRequestFactory clientHttpRequestFactory(){
        PoolingHttpClientConnectionManager connectionManager = new PoolingHttpClientConnectionManager();
        connectionManager.setMaxTotal(100);
        connectionManager.setDefaultMaxPerRoute(20);

        RequestConfig requestConfig = RequestConfig
                .custom()
                .setConnectionRequestTimeout(3000)
                .setSocketTimeout(3000)
                .build();

        CloseableHttpClient httpClient = HttpClients
                .custom()
                .setConnectionManager(connectionManager)
                .setKeepAliveStrategy(new DefaultConnectionKeepAliveStrategy())
                .setDefaultRequestConfig(requestConfig)
                .build();

        return new HttpComponentsClientHttpRequestFactory(httpClient);
    }

    @Override
    public void customize(RestTemplate restTemplate) {
        restTemplate.setRequestFactory(this.clientHttpRequestFactory());
    }
}
```

- NonBlockingIORestTemplateCustomizer

```
@Component
public class NIORestTemplateCustomizer implements RestTemplateCustomizer {

    public ClientHttpRequestFactory clientHttpRequestFactory() throws IOReactorException {
        final DefaultConnectingIOReactor ioreactor = new DefaultConnectingIOReactor(IOReactorConfig.custom().
                setConnectTimeout(3000).
                setIoThreadCount(4).
                setSoTimeout(3000).
                build());

        final PoolingNHttpClientConnectionManager connectionManager = new PoolingNHttpClientConnectionManager(ioreactor);
        connectionManager.setDefaultMaxPerRoute(100);
        connectionManager.setMaxTotal(1000);

        CloseableHttpAsyncClient httpAsyncClient = HttpAsyncClients.custom()
                .setConnectionManager(connectionManager)
                .build();

        return new HttpComponentsAsyncClientHttpRequestFactory(httpAsyncClient);

    }

    @Override
    public void customize(RestTemplate restTemplate) {
        try {
            restTemplate.setRequestFactory(clientHttpRequestFactory());
        } catch (IOReactorException e) {
            e.printStackTrace();
        }
    }
}
```

- You can try out any of them to see the performance impact (At a time only one
  config can be active/passed)

### Apache Client Request Logging

- Request details logging
- Apache uses Request Interceptor

- dependency we already have added above(`org.apache.httpcomponents`) so gonna be config change
- Add the below property in `application.properties`
  `logging.level.org.apache.http=debug`

- Restart the application

### Externalize Properties

- Add properties in `application.properties`

```
sfg.maxtotalconnections=100
sfg.defaultmaxtotalconnections=20
sfg.connectionrequesttimeout=3000
sfg.sockettimeout=3000
```

- Modify Class to use `@Value` to map the properties

```
public BlockingRestTemplateCustomizer(
    @Value("${sfg.maxtotalconnections}") Integer maxTotalConnections,
    @Value("${sfg.defaultmaxtotalconnections}") Integer defaultMaxTotalConnetions,
    @Value("${sfg.connectionrequesttimeout}")Integer connectionRequestTimeout,
    @Value("${sfg.sockettimeout}")Integer socketTimeout) {
        this.maxTotalConnections = maxTotalConnections;
        this.defaultMaxTotalConnetions = defaultMaxTotalConnetions;
        this.connectionRequestTimeout = connectionRequestTimeout;
        this.socketTimeout = socketTimeout;
    }
```

### KEY TERMS

- Emphemeral Port
- Java IO Packages
- Java NIO Packages

- Request Interceptor
