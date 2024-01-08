## Refactoring to Microservices

How do you migrate a legacy monolithic application to a microservice architecture?

- Strangler application
- Anti-corruption layer

### Strangler application

- Modernize an application by incrementally developing a new (strangler) application around the legacy application.
  So the monolith shrinks over time.

- The strangler application consists of two types of services. First, there are services that implement functionality
  that previously resided in the monolith. Second, there are services that implement new features. The latter are particularly useful since they demonstrate to the business the value of using microservices.

### Anti-corruption layer

- Define an anti-corruption layer, which translates between the two domain models.

#### Problem:

#### Forces:

### Solution:

### KEY TERMS
