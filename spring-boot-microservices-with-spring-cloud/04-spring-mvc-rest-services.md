## Spring MVC Rest Services

- https://sfg-beer-works.github.io/brewery-api/#tag/Beer-Service

### Introducing SFG Beer Works

#### SFG Beer Works

- SFG Beer Works is a set of Spring Applications to showcase microservices and Spring Cloud
- Theme of the services is beer distribution

- Beer Consumer - Creates demand for beer

  - Name - `Beer Svc`
  - Tech Stack - Kotlin, WebClient

- Pub - Provides beer to Beer Consumer, reorders beer from the Distributor

  - Name - `Pub`
  - Tech Stack - WebFlux.fn, Sring Data Mongo, MongoDB

- Distributor - re-stocks beer to Pubs, orders beer from Brewery

  - Name - `Distributor`
  - Tech Stack - WebFlux, Spring Data Mongo, MongoDB

- Brewery - Brews beer, supplies beer to distributor
  - Name - `Brewery`
  - Tech Stack - Spring MVC, Spring Data JPA, Hibernate

### Project initialize (mssc-brewery)

- Create project using Spring Initializer

  - Spring Web
  - Lombok
  - DevTools
  - Actuator

- Enable `Annotation Processor` in IntelliJ

  - Preferences -> Build, Execution, Deployment -> Compiler -> Annotation Processors
    - Check `Enable Annotation Processor`
  - Intellij Plugins - Maven, Lombok, Delombok, Axis Tcp Monitor OR Tcp Tunneling

- Repo - https://github.com/bhopals/mssc-brewery

#### Axis TCPMon

- TCPMon is a utility that allows the messages to be viewed and resent. It is very much
  useful as a debug tool.

- The most common usage of TCPMon is as an intermediary, which monitors the communication between the client (front end) and the back-end server. That is, the messages sent from the client are received by the intermediary instead of the back-end server.

### Spring Boot Development Tools

#### Developer Tools

- Added to Project via artifact ‘spring-boot-devtools’
- Developer Tools are automatically disabled when running a packaged application (ie java -jar)
- By default, not included in repackaged archives

#### Developer Tools Features

- Automatic Restart

  - Triggers a restart of the Spring Context when classes change
  - Uses two classloaders. One for your application, one for project jar dependencies
  - Restarts are very fast, since only your project classes are bring loaded

- Eclipse:
  - Restart is triggered with save (which by default will compile the class, which
    triggers the restart)
- IntelliJ:
  - By default you need to select ‘Build / Make Project’
  - There is an advanced setting you can change to make this more seamless

#### Template Caching

- By default templates are cached for performance
- But Caching will require a container restart to refresh the cache
- Developer Tools will disable template caching so the restart is not required to see changes

#### LiveReload

- LiveReload is a tech. to automatically trigger a browser refresh when resources are changed
- Spring Boot Developer Tools includes a LiveReload server
- Browser plugins are available for a free download at livereload.com

### API Versioning

- Versioning your APIs is considered a best practice
- Example “/api/v1/beer” - “v1” is the API version
- API Versioning allows you to evolve the API without breaking existing API consumers
- Typical lifespan:
  - v1 - first release
  - v2 - second release, notify consumers v1 version is deprecated
  - v3 - remove v1 (optional), notify consumers v2 is deprecated

#### Semantic Versioning 2.0.0

- See website - https://semver.org
- Version - MAJOR.MINOR.PATCH

  - MAJOR - version for major incompatible API changes - aka breaking changes
  - MINOR - new functionality - backwards compatible changes
  - PATCH - backwards compatible bug fixes

- API URLs typically only use MAJOR versions
  - Can optionally use MINOR and PATCH
  - /v1 or /v1.1

#### Non-Breaking Changes

- Non-Breaking changes may be performed under MINOR or PATCH versions
- Examples:
  - New optional parameter
  - New response fields
  - New service (endpoint)
  - Bug fixes - behavior change, NOT change to API itself

#### Breaking Changes

- Breaking Changes should be done under a MAJOR version
- Examples:
  - New required parameter
  - Removal of existing parameter
  - Removal of response value
  - Parameter name change or type
  - Deprecation of a service

### Semflow: The SemVer Workflow

- A Git Workflow for Version Management: RESTful APIs and beyond
- Keeping branch names `v1`, `v2`, `v3` for each version
- A kind of branching strategies
- Refer - https://github.com/lyndseypadget/semflow

### New Microservice - Beer Service - LC Beer Service

- Create New Spring Project using Spring Initilizer
  - Web
  - Actuator
  - DevTools
  - Lombok

### KEY TERMS

- Semantic Versioning
- Incompatible API changes - aka breaking changes
- Backward Compatible changes
- Versioning allows you to evolve the API without breaking existing API consumers.
