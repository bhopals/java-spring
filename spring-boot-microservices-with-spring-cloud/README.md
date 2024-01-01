## Spring Boot Microservices with Spring Cloud

### Twelve Factors

- I. Codebase - One codebase tracked in revision control, many deploys
- II. Dependencies - Explicitly declare and isolate dependencies
- III. Config - Store config in the environment
- IV. Backing services - Treat backing services as attached resources
- V. Build, release, run - Strictly separate build and run stages
- VI. Processes - Execute the app as one or more stateless processes
- VII. Port binding - Export services via port binding
- VIII. Concurrency - Scale out via the process model
- IX. Disposability - Maximize robustness with fast startup and graceful shutdown
- X. Dev/prod parity - Keep development, staging, and production as similar as possible
- XI. Logs - Treat logs as event streams
- XII. Admin processes - Run admin/management tasks as one-off processes

### Tips and Tricks for IntelliJ

- Create Workspace to bring in all the microservices Projects (Instead of Juggling b/w different windows)
- To do that:

  - Create an EMPTY Directory using Terminal
  - Open IntelliJ, create new Project -> select "Empty Project" ->
    select the above created directory -> finish
  - Now do `git clone` all the microservices projects that you intended to bring in.

  - Now Add those projects into the workspace as Module
    - File -> New -> Module -> Module from Existing Source -> Select Project that we have cloned (one by one)
    - Select "Import Module from externals" -> Select Maven -> Finish

### Reference

- Repo - https://sfg-beer-works.github.io/brewery-api/#tag/Beer-Service

- https://12factor.net/

- Differenece between OpenAPI 2.0 and OpenAPI 3.0

  - https://blog.stoplight.io/difference-between-open-v2-v3-v31

- https://microservices.io/
- https://microservices.io/patterns/data/saga.html
