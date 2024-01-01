## Spring State Machine

### Introduction to Spring State Machine

- A state machine can be loosely defined as anything with a set of known states.

### What is a State Machine?

- A state machine can be loosely defined as anything with a set of known states.
- As a developer, you’ve already been writing state machines!
- A ‘state machine’ can be:
  - if - then - else
  - case statements
  - switch statements
- Each example reflects a ‘state’

### State Machine History

- The concept of a State Machine dates back to 1943 when McCulloch and Pitts wrote a paper referencing
  the concept of finite automata.
- Finite Automata consists of:
  - Finite set of states
  - Set of inputs
  - Initial State
  - Final State
  - Transition Function

### State Machine Use Cases

- Common use cases for State Machines
  - Message (Event) based applications - ie New Order, Pay Order, Ship Order
  - Events get published based on state changes
  - UI Applications With Actions triggered by Use - Caps Lock On, Caps Lock Off
  - Application behavior changes based on known states

### State Machine Terminology

- States - The specific state of the state machine. Finite and predetermined values.
  - Frequently defined in an enumeration
- Events - Something that happens to the system - may or may not change the state.
- Actions - The response of the State Machine to events. Can be changing variables, calling
  a method or changing to a different state
  - Transitions - Type of action which changes state
- Guards - Boolean conditions
- Extended State - State Machine variables (in addition to state)

### Why Use a State Machine?

- State Machines help define consistent behavior for a finite number of states
- Application logic is defined for specific states or state transitions
- Application logic becomes more modular and more precisely defined
- Long blocks of if, then, else if conditions are difficult to code, debug, and maintain
- Helps avoid spaghetti code for complex conditions

### Spring State Machine

- Spring State Machine (SSM) is a mature Spring Framework project
- Initially released in October 2015.
- Current version is 2.1.3
- 3.x is under development, which will introduce non-blocking, reactive types
- SSM has a robust set of features and is integrated with the Spring Framework

### Credit Card Payment State Machine Overview

- In Credit Card processing, with some vendors you have the option to pre-authorize a charge
- Example: Purchasing gas
  - Swipe card - pre-authorize charge occurs
  - Validates card, places hold on funds
  - Pump gas
  - Charge card with actual amount of sale

### Thinking in States

- The Credit Card Payment will have the following states:
  - New - new payment
  - Pre Authorized - Charge Pre Authorized with processor
  - Pre Authorized Error - Pre Authorization rejected by processor
  - Authorized - Charge approved by processor
  - Authorization Error - Charge rejected by processor

### Thinking in Events

- Credit Card Processing Events:
  - Pre Authorize Charge - Call processor for pre auth transaction
  - Pre Authorize Approved - Processor approved pre authorize
  - Pre Authorize Declined - Processor declined pre authorize
  - Authorize Charge - Call processor for charge authorization
  - Authorization Approved - Processor approved charge
  - Authorization Declined - Processor rejected charge

### Spring Boot Project Creating and Dependencies

- Create Project using Spring Boot Initilizer (Spring Boot 2.2.0.M4)

  - Spring Boot DevTool
  - Lombok
  - Spring Web Starter
  - Spring Data JPA
  - H2 Database

- Add Spring State machine dependency in `POM.XML`

```
<dependency>
    <groupId>org.springframework.statemachine</groupId>
    <artifactId>spring-statemachine-core</artifactId>
    <version>2.1.3.RELEASE</version>
</dependency>
```

#### State Machine Enumerations

- Create PaymentState

```
public enum PaymentState {
    NEW, PRE_AUTH, PRE_AUTH_ERROR, AUTH, AUTH_ERROR
}
```

- Create PaymentEvent

```
public enum PaymentEvent {
    PRE_AUTHORIZE, PRE_AUTH_APPROVED, PRE_AUTH_DECLINED, AUTHORIZE, AUTH_APPROVED, AUTH_DECLINED
}
```

#### Spring Data JPA Configuration

- Create Entity - `Payment`

```
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Entity
public class Payment {
    @Id
    @GeneratedValue
    private Long id;
    @Enumerated(EnumType.STRING)
    private PaymentState state;
    private BigDecimal amount;
}
```

#### State Configuration

- Add `StateMachingConfig.java` and override `configure` and annotate class with `@EnabledStateMachineFactory`
- The annotation is going to do is it tells Spring to go through this and it's going to scan this
  and by enabling the state machine factory.
- class `StateMachineConfig` should extend `StateMachineConfigurerAdapter<PaymentState, PaymentEvent>`

```
 @Override
    public void configure(StateMachineStateConfigurer<PaymentState, PaymentEvent> states) throws Exception {
        states.withStates()
                .initial(PaymentState.NEW)
                .states(EnumSet.allOf(PaymentState.class))
                .end(PaymentState.AUTH)
                .end(PaymentState.PRE_AUTH_ERROR)
                .end(PaymentState.AUTH_ERROR);
    }
```

#### Transition Configuration

```
@Override
public void configure(StateMachineTransitionConfigurer<PaymentState, PaymentEvent> transitions) throws Exception {
    transitions.withExternal().source(PaymentState.NEW).target(PaymentState.NEW).event(PaymentEvent.PRE_AUTHORIZE)
                .action(preAuthAction).guard(paymentIdGuard)
            .and()
            .withExternal().source(PaymentState.NEW).target(PaymentState.PRE_AUTH).event(PaymentEvent.PRE_AUTH_APPROVED)
                .action(preAuthApprovedAction)
            .and()
            .withExternal().source(PaymentState.NEW).target(PaymentState.PRE_AUTH_ERROR).event(PaymentEvent.PRE_AUTH_DECLINED)
                .action(preAuthDeclinedAction)
            //preauth to auth
            .and()
            .withExternal().source(PaymentState.PRE_AUTH).target(PaymentState.PRE_AUTH).event(PaymentEvent.AUTHORIZE)
                .action(authAction)
            .and()
            .withExternal().source(PaymentState.PRE_AUTH).target(PaymentState.AUTH).event(PaymentEvent.AUTH_APPROVED)
                .action(authApprovedAction)
            .and()
            .withExternal().source(PaymentState.PRE_AUTH).target(PaymentState.AUTH_ERROR).event(PaymentEvent.AUTH_DECLINED)
                .action(authDeclinedAction);
}
```

#### Logging Configuration using State Change Listeners

```
@Override
    public void configure(StateMachineConfigurationConfigurer<PaymentState, PaymentEvent> config) throws Exception {
        StateMachineListenerAdapter<PaymentState, PaymentEvent> adapter = new StateMachineListenerAdapter<>(){
            @Override
            public void stateChanged(State<PaymentState, PaymentEvent> from, State<PaymentState, PaymentEvent> to) {
                log.info(String.format("stateChanged(from: %s, to: %s)", from, to));
            }
        };

        config.withConfiguration()
                .listener(adapter);
    }
```

#### Initilize State Machine from Database

```
private StateMachine<PaymentState, PaymentEvent> build(Long paymentId){
    Payment payment = paymentRepository.getOne(paymentId);
    StateMachine<PaymentState, PaymentEvent> sm = stateMachineFactory.getStateMachine(Long.toString(payment.getId()));
    sm.stop();
    sm.getStateMachineAccessor()
            .doWithAllRegions(sma -> {
                sma.addStateMachineInterceptor(paymentStateChangeInterceptor);
                sma.resetStateMachine(new DefaultStateMachineContext<>(payment.getState(), null, null, null));
            });
    sm.start();
    return sm;
}
```

#### State Change Interceptor

- Intercept the change in the State, and persist the current state into the DB

```
@RequiredArgsConstructor
@Component
public class PaymentStateChangeInterceptor extends StateMachineInterceptorAdapter<PaymentState, PaymentEvent> {
    private final PaymentRepository paymentRepository;
    @Override
    public void preStateChange(State<PaymentState, PaymentEvent> state, Message<PaymentEvent> message,
                               Transition<PaymentState, PaymentEvent> transition, StateMachine<PaymentState, PaymentEvent> stateMachine) {
        Optional.ofNullable(message).ifPresent(msg -> {
            Optional.ofNullable(Long.class.cast(msg.getHeaders().getOrDefault(PaymentServiceImpl.PAYMENT_ID_HEADER, -1L)))
                    .ifPresent(paymentId -> {
                        Payment payment = paymentRepository.getOne(paymentId);
                        payment.setState(state.getId());
                        paymentRepository.save(payment);
                    });
        });
    }
}
```

#### State Machine Action

- Action setups - preAuthAction, postAuthAction, preAuthDeclineAction etc.

#### State Machine Guards

- A way to add into the state machine the ability to approve an action.

```
    public Guard<PaymentState, PaymentEvent> paymentIdGuard(){
        return context -> {
            return context.getMessageHeader(PaymentServiceImpl.PAYMENT_ID_HEADER) != null;
        };
    }
```

### Microservice Design Pattern - Event Sourcing

- Event sourcing is a persistence pattern, ensuring that every data change is recorded as an immutable event.
- With event sourcing, data changes are getting saved in domain events. These events are held in
  the chronological order of occurrence. These events can be replayed to get to the current state of the data.

- More - https://microservices.io/patterns/data/event-sourcing.html

- Event Sourcing v/s Event Driven
  - `Event sourcing` is a persistence pattern, ensuring that every data change is recorded as an immutable event. These events serve as the single source of truth for the system's state. With `event-driven` architecture, these events are passed on to various parts of the system in a loosely coupled manner.

### KEY TERMS

- Allow reliability and stability in your application
- Help define consistent behaviour for a finite number of states
- Configure State Maching
- Spring Boot State Machine
- Initial State
- Terminal State

- Persistance Pattern
- Single Source of truth for the System's state
- Replaying the events can re-establish the state
- Traditional distributed transaction (2PC)
- 2PC - 2 Phase commit
