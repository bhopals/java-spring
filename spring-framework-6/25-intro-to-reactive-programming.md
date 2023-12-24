## Introducton to Reactive Programming

### Reactive Manifesto - History

- Originally published in 2013
- Available at www.reactivemanifesto.org
- Authors: Jonas Bonér (Akka founder), Dave Farley, Roland Kuhn, and Martin Thompson
- The term “Reactive” is getting a bit overloaded in the IT community

### Reactive Manifesto

- Responsive
- Resilient
- Elastic
- Message Driven

### Reactive Manifesto Responsive

- The system responds in a timely manner.
- Responsiveness is the cornerstone of usability and utility.
- Responsiveness also means problems may be detected quickly and dealt with effectively.
- Responsive systems provide rapid and consistent response times.
- Consistent behavior simplifies error handling, builds end user confidence, and
  encourages further interaction.

### Reactive Manifesto Resilient

- System stays responsive in the face of failure.
- Resilience is achieved by replication, containment, isolation, and delegation.
- Failures are contained within each component.
- Parts of the system can fail, without compromising the system as a whole.
- Recovery of each component is delegated to another.
- High-availability is ensured by replication where necessary

### Reactive Manifesto Elastic

- The system stays responsive under varying workload.
- Reactive Systems can react to changes in the input rate by increasing or decreasing
  resources allocated to service inputs.
- Reactive Systems achieve elasticity in a cost effective way on commodity hardware and
  software platforms

### Reactive Manifesto Message Driven

- Reactive Systems rely on asynchronous message passing to establish a boundary between components.
- This ensures loose coupling, isolation, and location transparency.
- Message passing enables load management, elasticity, and flow control.
- Location transparent messaging makes management of failures possible
- Non-blocking communication allows recipients to only consume resources while
  active, leading to less system overhead.

### Reactive Programming

- Reactive Programming is a useful implementation technique.
- Reactive Programming focuses on non-blocking, asynchronous execution - a key
  characteristic of Reactive Systems.
- Reactive Programming is just one tool in building Reactive Systems.
- The Reactive Manifesto has a wholistic view of the system
- While Reactive Programming is more granular at the program level

### Reactive Programming Performance

- Reactive does NOT equal fast!!
- A typical CRUD type application will not see much, if any performance improvement
- Reactive can improve computing efficiency
- Best used for streaming type applications
- The immutable nature of Reactive applications can help with application quality

### Reactive Programming

- Reactive Programming is an asynchronous programming paradigm focused on streams of data.
- “Reactive programs also maintain a continuous interaction with their environment, but at a
  speed which is determined by the environment, not the program itself. Interactive programs
  work at their own pace and mostly deal with communication, while reactive programs only
  work in response to external demands and mostly deal with accurate interrupt handling.
  Real-time programs are usually reactive.” - Gerad Berry, French Computer Scientist

### Common Use Cases

- External Service Calls
- Highly Concurrent Message Consumers
- Spreadsheets
- Abstraction Over Asynchronous Processing
- Abstract whether or not your program is synchronous or asynchronou

### Features of Reactive Programming

- Data Streams
- Asynchronous
- Non-blocking
- Backpressure
- Failures as Messages

### Asynchronous

- Events are captured asynchronously.
- A function is defined to execute when an event is emitted.
- Another function is defined if an error is emitted.
- Another function is defined when complete is emitted.
- This can be a difficult paradigm to adjust to when first getting started!

### Non-Blocking

- The concept of using non-blocking is important.
- In Blocking, the code will stop and wait for more data (ie reading from disk, network, etc)
- Non-blocking in contrast, will process available data, ask to be notified when more is
  available, then continue.

### Back Pressure

- The ability of the subscriber to throttle data
- The asynchronous and non-blocking nature of Reactive design improves performance and memory usage.
- Project Reactor provides those capabilities to efficiently manage data streams.

- However, backpressure is a common problem in these kinds of applications

- Backpressure in Reactive Streams

  - Due to the non-blocking nature of Reactive Programming, the server doesn’t send the complete stream at once. It can push the data concurrently as soon as it is available. Thus, the client waits less time to receive and process the events. But, there are issues to overcome.

  - Backpressure in software systems is the capability to overload the traffic communication. In other words, emitters of information overwhelm consumers with data they are not able to process.

  - Eventually, people also apply this term as the mechanism to control and handle it. It is the protective actions taken by systems to control downstream forces.

#### What Is Backpressure?

In Reactive Streams, backpressure also defines how to regulate the transmission of stream elements. In other words, control how many elements the recipient can consume.

- Using Backpressure to Prevent Systemic Failures

### Controlling Backpressure

- We’ll focus on controlling the events emitted by the publisher. Basically, there are three strategies to follow:

- 1. Send new events only when the subscriber requests them. This is a pull strategy to gather
     elements at the emitter request

- 2. Limiting the number of events to receive at the client-side. Working as a limited push
     strategy the publisher only can send a maximum amount of items to the client at once

- 3. Canceling the data streaming when the consumer cannot process more events. In this case,
     the receiver can abort the transmission at any given time and subscribe to the stream later again

- Refer - https://www.baeldung.com/spring-webflux-backpressure

### Reactive Streams API

- Reactive Streams is a set of 4 interfaces which define the API:
  - Publisher
  - Processor
  - Subscriber
  - Subscription

### Spring MVC & Spring WebFlux

- Spring MVC
  - @Controller / @RequestMapping
  - spring-webmvc
  - Servlet API
  - Servlet Container
- Spring WebFlux
  - Router Functions
  - spring-webflux
  - HTTP / Reactive Streams
  - Tomcat, Jetty, Netty, Undertow

### Spring Reactive Types

- Two new reactive types are introduced with Spring Framework 5

  - Mono - is a publisher with zero or one elements in data stream.
  - Flux - is a publisher with zero or MANY elements in the data stream.

- A Mono is a reactive type that can emit at most one element, whereas a Flux can emit multiple elements.

- In other words, a Mono represents a stream of zero or one element, whereas a Flux represents a
  stream of zero to many elements.
- Both types implement the Reactive Streams Publisher interface

### Summary

- Reactive Programming focuses on processing streams of data.
- Traditional CRUD applications are still alive and well.
- Reactive does not equal fast - many applications will see little if any improvement
- Performance benefit is more apparent with a system under load (vs individual transactions)

### Create Spring Boot Project

- Use Spring Initializr
  - Lombok
  - Spring Reactive Web

### Implement Repository

- Person.java

```
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class Person {
    private Integer id;
    private String firstName;
    private String lastName;
}
```

- RepositoryImpl

```
/**
 * Created by jt, Spring Framework Guru.
 */
public class PersonRepositoryImpl implements PersonRepository {

    Person michael = Person.builder().id(1).firstName("Michael").lastName("Weston").build();
    Person fiona = Person.builder().id(2).firstName("Fiona").lastName("Glenanne").build();
    Person sam = Person.builder().id(3).firstName("Sam").lastName("Axe").build();
    Person jesse = Person.builder().id(4).firstName("Jesse").lastName("Porter").build();

    @Override
    public Mono<Person> getById(Integer id) {
        return Mono.just(michael);
    }

    @Override
    public Flux<Person> findAll() {
        return Flux.just(michael, fiona, sam, jesse);
    }
}
```

### Mono Operations

```
@Test
    void testMonoByIdBlock() {
        Mono<Person> personMono = personRepository.getById(1);

        Person person = personMono.block();

        System.out.println(
                person.toString()
        );
    }

    @Test
    void testGetByIdSubscriber() {
        Mono<Person> personMono = personRepository.getById(1);

        personMono.subscribe(person -> {
            System.out.println(person.toString());
        });
    }

    @Test
    void testMapOperation() {
        Mono<Person> personMono = personRepository.getById(1);

        personMono.map(person -> {
            return person.getFirstName();
        }).subscribe(firstName -> {
            System.out.println(firstName);
        });
    }
```

### Flux Operations

```
class PersonRepositoryImplTest {

    PersonRepository personRepository = new PersonRepositoryImpl();

    @Test
    void testMonoByIdBlock() {
        Mono<Person> personMono = personRepository.getById(1);

        Person person = personMono.block();

        System.out.println(
                person.toString()
        );
    }

    @Test
    void testGetByIdSubscriber() {
        Mono<Person> personMono = personRepository.getById(1);

        personMono.subscribe(person -> {
            System.out.println(person.toString());
        });
    }

    @Test
    void testMapOperation() {
        Mono<Person> personMono = personRepository.getById(1);

        personMono.map(Person::getFirstName).subscribe(firstName -> {
            System.out.println(firstName);
        });
    }

    @Test
    void testFluxBlockFirst() {
        Flux<Person> personFlux = personRepository.findAll();

        Person person = personFlux.blockFirst();

        System.out.println(person.toString());
    }

    @Test
    void testFluxSubscriber() {
        Flux<Person> personFlux = personRepository.findAll();

        personFlux.subscribe(person -> {
            System.out.println(person.toString());
        });
    }

    @Test
    void testFluxMap() {
        Flux<Person> personFlux = personRepository.findAll();

        personFlux.map(Person::getFirstName).subscribe(fistName -> System.out.println(fistName));
    }

    @Test
    void testFluxToList() {
        Flux<Person> personFlux = personRepository.findAll();

        Mono<List<Person>> listMono = personFlux.collectList();

        listMono.subscribe(list -> {
            list.forEach(person -> System.out.println(person.getFirstName()));
        });
     }
}
```

### Filter Flux Objects

```
 @Test
    void testFilterOnName() {
        personRepository.findAll()
                .filter(person -> person.getFirstName().equals("Fiona"))
                .subscribe(person -> System.out.println(person.getFirstName()));
    }

```

### Error Handling

```
@Test
    void testFindPersonByIdNotFound() {
        Flux<Person> personFlux = personRepository.findAll();

        final Integer id = 8;

        Mono<Person> personMono = personFlux.filter(person -> person.getId() == id).single()
                .doOnError(throwable -> {
                    System.out.println("Error occurred in flux");
                    System.out.println(throwable.toString());
                });

        personMono.subscribe(person -> {
            System.out.println(person.toString());
        }, throwable -> {
            System.out.println("Error occurred in the mono");
            System.out.println(throwable.toString());
        });
    }
```

### Step Verifier

- To test Reactive Flux and Mono

### Mono v/s Flux

- Mono and Flux are both implementations of the Publisher interface

- Mono
  Mono is a special type of Publisher. A Mono object represents a single or empty value. This means it can emit only one value at most for the onNext() request and then terminates with the onComplete() signal. In case of failure, it only emits a single onError() signal.

- Flux
  Flux is a standard Publisher that represents 0 to N asynchronous sequence values. This means that it can emit 0 to many values, possibly infinite values for onNext() requests, and then terminates with either a completion or an error signal.

- Refer - https://www.baeldung.com/java-reactor-flux-vs-mono

### KEY TERMS

- Reactive System
- Building Reactive System

- The system stays responsive under varying workload.
- Reactive Systems achieve elasticity in a cost effective way
- High-availability is ensured by replication where necessary
- Resilient - System stays responsive in the face of failure.
- Reactive Systems rely on asynchronous message passing to establish a boundary between components.
- This ensures loose coupling, isolation, and location transparency.

- Reactive Programming focuses on non-blocking, asynchronous execution

- Abstraction Over Asynchronous Processing
- Highly Concurrent Message Consumers

- Backpressure
- Failures as Messages

- Functions are defined when a success, an error, and a complete events are emitted
- Goal is to create a standard for asynchronous stream processing with non-blocking back pressure
- 2 new reactive types introduced - Mono, Flux

- In reactive programing, we need to have back pressure enabled, so always use `subscribe`
  and then further operations on the stream object.

- A Flux will contain zero or many elements, while a Mono will contain zero or one element.
