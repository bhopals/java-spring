## Using Sagas with Spring

- More - https://microservices.io/patterns/data/saga.html

### The Problem with Transactions

- Saga solves the problem - How to implement transactions that span services? (2PC is not an option)
- Saga is an important cocept of how we coordinate between multiple microservices

#### Transactions

- A database transaction allows you to have sequence of steps
  - All steps must complete to be committed
  - Else, a rollback occurs returning the database to the original state
- The Order Allocation Scenario
  - Allocate Inventory - Updating Inventory and Order with Allocation
  - Works well within a monolith
  - Order and Inventory are two different Microservices / Databases
  - Breaks traditional transactions

#### Important Terms

- `Transaction` - A unit of work. One or more operations
  - Can be just one; can be hundreds or thousands.
- `Commit` - Indicates the end of the transaction and tells database to make changes permanent.
  - More efficient to do multiple operations in a transaction. There is a ‘cost’ with commits.
- `Rollback` - Revert all changes of the transaction
- `Save Point` - Programatic point you can set, which allows you to rollback to (ie rollback
  part of a transaction)

#### A.C.I.D. Transactions

- A.C.I.D. - Typically one database
  - Atomicity - All operations are completed successfully or database is returned to previous state.
  - Consistency - Operations do not violate system integrity constraints.
  - Isolated - Results are independent of concurrent transactions.
  - Durable - Results are made persistent in case of system failure (ie written to disk)
- Database handles all locking and coordination to guarantee transaction
- This is EXPENSIVE to-do - takes a lot of system resources

#### Distributed Transactions

- With Microservices, often multiple services are involved in what is considered a transaction
  - Order Allocation Example - Order Service, Inventory Service
- Java EE - Java Transaction API (JTA)
  - Enables distributed transactions for Java environments
  - Well supported by Spring
  - Transactions are managed across nodes by a Transaction Manager
  - Very Java Centric

#### Two Phase Commit - 2PC

- Happens in two phases - Voting and Commit
- Coordinator asks each node if proposed transaction is okay
  - If all respond `OKAY`
    - Commit message is sent
    - Each Node commits work and sends acknowledgement to coordinator
  - If any node responds `NOT OKAY`
    - Rollback message is sent
    - Each node rollsback and sends acknowledgement to coordinator

#### Problems with Two Phase Commit

- Problems with 2PC
  - Does not scale - expensive
  - Blocking Protocol - the various steps block and wait for other to complete
  - Performance is limited to the speed of the slowest node
  - Coordinator is a Single Point of Failure
  - Technology lock-in
    - Can be very difficult to mix technology stacks

#### Challenges with Microservices

- A transaction for a Microservice architecture will often span multiple microservices
- Each service should have its own database
  - Could be a mix of SQL and NoSQL databases
- Should be technology agnostic
  - Services can by in Java, .NET, Ruby, etc
- How to coordinate the ‘Transaction’ across multiple microservices???

### The Need for Sagas

#### The Microservice Death Star

- As the number of services grows, so does complexity
- Death Star - Examples
  - Netflix
  - Twitter
  - Uber

#### Challenges

- Business transactions often span multiple microservices
- ACID transactions are not an option between services
- Distributed Transactions / Two Phase Commits
  - Complex and do not scale
- Microservices should be technology agnostic
  - Making 2PC even more difficult to implement

#### CAP Theorem

- CAP - Consistency, Availability, Partition Tolerance
  - Consistency - Every read will have the most recent write
  - Availability - Each read will get a response, but without the guarantee data is most recent
    write
  - Partition Tolerance - System continues in lieu of communications errors or delays
- CAP Theorem - States a distributed system can only maintain two of three
  - Consistency, Availability, Partition Tolerance

#### BASE - An ACID Alternative

- BASE - Coined by Dan Pritchett of EBay in 2008
- BASE = Basically Available, Soft state, Eventually consistent
  - The opposite of ACID
- Basically Available - Build system to support partial failures
  - Lost of some functionality vs total system loss.
- Soft State - Transactions cascade across nodes, it can be inconsistent for a period of time
- Eventually Consistent - When processing is complete, system will be consistent

#### Introducing Sagas

- Concept introduced in 1987 by Gracia-Molina / Salem of Princeton University
- Originally was looking at Long Lived Transactions (LLTs) within a single database
  - LLTs hold on to database resources for an extended period of time
  - In 1987 computational resources were much more scarce
- Paper proposed rather than long complex processes to break up into smaller more atomic transactions
- Introduced the concept of compensating transactions to correct partial executions

#### Sagas

- Sagas are simply a series of steps to complete a business process
- Sagas coordinate the invocation of microservices via messages or requests
- Sagas become the transactional model
- Each step of the Saga can be considered a request
- Every step of the Saga has a compensating transaction (request)
  - Semantically undoes the effect of the request
  - Might not restore to the exact previous state - but effectively the same

#### Saga Steps

- Each step should be a message or event to be consumed by a microservice
- Steps are asynchronous
- Within a microservice, it’s normal to use traditional database transactions
- Each message (request) should be idempotent
  - Meaning if same message / event is sent there is no adverse effect on system state
- Each step has a compensating transaction to undo the actions

#### Compensating Transactions

- Compensating Transactions
  - Effectively become the ‘Feral Concurrency Control’
  - Are the mechanism used to maintain system integrity
  - Should also be Idempotent
  - Cannot abort - need to ensure proper execution
  - Not the same as a ‘rollback’ to the exact previous state
    - Implements business logic for a failure event

#### Sagas are ACD

- A - Atomic
  - All transactions are executed or compensated
- C - Consistency
  - Referential integrity within a service by the local database
  - Referential integrity across services by the application - ‘Feral Concurrency Control’
- D - Durability
  - Persisted by database of each microservice

#### Sagas & Eventually Consistent

- BASE = Basically Available, Soft state, Eventually consistent
- During execution of the Saga system is in the ‘Soft State’
- Eventually consistent - meaning system will be consistent at the conclusion of the Saga
  - Consistency achieved via normal completion of the Saga
  - In the event of an error, consistency is achieved via compensating transactions

#### Sagas Summary

- Saga Definition - “A long, involved story, account, or series of incidents”
- Microservices by nature run in a distributed environment, across many computers
- Sagas are used to address the challenges faced when operating in a distributed environment
- Sagas are a tool used to coordinate a series of steps for a business transaction across multiple services
- Sagas not only prescribe the series of necessary steps, they also maintain system integrity

### Saga Coordination

- Saga Definition - “A long, involved story, account, or series of incidents”
  - Saga - Series of Steps
  - With compensating transactions
- How to define a saga in a distributed environment?
  - Two Primary Approaches for Saga Coordination
    - Choreography - Distributed decision making. Each actor decides next steps.
    - Orchestration - Centralized decision making. Central component decides next steps.

#### Choreography Coordination

- Distributed Decision Making - each actor determines next steps
- Benefits
  - Simple, loosely coupled
  - Good for simpler Sagas
- Problems
  - Cyclic dependencies
  - Harder to understand - logic is spread out
  - Components are more complex

#### Choreography Implementation

- Choreography Coordination typically implemented using events
- Each actor emits an event for the next step in the Saga
- Requires each actor to have logic about the Saga
- Each actor needs to know how to perform a compensating transaction
  - Thus each actor has more coupling to other system components

#### Orchestration Based Coordination

- Centralized Decision Making - step components do not decide next steps
- Benefits
  - Logic is centralized and easier to understand
  - Reduced coupling, better separation of concerns
- Problems
  - Risk of over centralization - need to maintain focus on separation of concerns

#### Orchestration Implementation

- Orchestration Coordination has a central component directing other actors
- Central component maintains state for the Saga
  - State Machine
  - Saga Execution Coordinator (aka SEC)
  - Event Sourcing
- Must take responsibility for the completion of the Saga
  - ie persist state to DB, use persistent message queues, etc

#### Conclusion

- Which to Use?
  - Choreography Coordination for smaller simple Sagas
  - Orchestration Coordination for larger more complex Sagas
- How to Implement?
  - Typically custom solution - wide variety of implementations
  - Open Source / Commercial solutions are emerging
    - Still fairly early and are maturing

### Order Allocation Saga

- Order Allocation is the process of assigning inventory to an order
- Order Allocation Steps:
  - Validate Order
  - Allocate Inventory
  - Update Order with result of Allocation
  - Order Delivered

#### Order Allocation in Beer Services

- Order Allocation Steps:
  - Validate Order - Call Beer Service, validate beers ordered
  - Allocate Inventory - Inventory Service Checks for Available Inventory
  - Update Order with Result of Allocation - Receive Allocation Action
  - Order Delivered - Gone from system

#### Order Cancelation

- Order Cancelation Can Happen Until Delivery
- Order Cancelation Steps:
  - Update Order Status to Canceled
  - If Allocated, Release Inventory

#### Orchestration Saga

- Need Saga Coordinator for Order Allocation
- Will sequence steps of Microservices to perform Order Allocation
- Accommodate ‘happy path’ of Order Allocation
- Apply compensating transactions if there are errors
- Handle Order Cancelation

#### Saga Execution Coordinator

- Saga Execution Coordinator

  - Implement using Spring State Machine

    - Events - VALIDATE_ORDER, VALIDATION_PASSED, VALIDATION_FAILED,
      ALLOCATION_SUCCESS, ALLOCATION_NO_INVENTORY, ALLOCATION_FAILED,
      BEER_ORDER_PICKED_UP, CANCEL_ORDER

    - States - NEW, VALIDATED, VALIDATION_EXCEPTION, ALLOCATED, ALLOCATION_ERROR,
      PENDING_INVENTORY,PICKED_UP, DELIVERED, DELIVERY_EXCEPTION, CANCELED

### Refactoring Model To Common Package

- Model - Objects exposed as JSON
- Problem: in Spring, serialization / deserialization expects same package / object name.
  - Could address w/additional Jackson configuration
- Solution move model objects to package - ‘guru.sfg.brewery.model’
  - All services are related under the ‘Brewery’ set of services
- Problem Lombok Builders with parent classes problematic
  - Solution: flatten classes

### KEY TERMS

- We can utilize State Machine to coordinate more and more complex transactions
- Long live running transactions with Saga
- Compensating Transactions
- Distributed Transactions
- Technology Agnostic
- Business Transactions
- ACID transaction
- Two Phase Commit
- Distributed System CAP theorem
- CAP - Consistency, Availability, Partition Tolerance
- CAP Theorem - States a distributed system can only maintain two of three
- BASE - Basically Available, Soft state, Eventually consistent
- LLT - Long Lived Transactions
- More Atomic Transcations

- Semantically undoes the effect of the request
- Sagas are ACD (Atomic, Consistency, Durability)

- Distributed Decision Making (Choreography)
- Centralized Decision Making (Orchestration)
- Choreography Coordination
- Orchestration Based Coordination

- State Machine
- Saga Execution Coordinator
- Event Sourcing
