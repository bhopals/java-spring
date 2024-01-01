### java-spring

Spring Resources

#### Spring Intro

- Spring framework to create Java Enterprise Application. Spring came into being in 2003 as a response to the complexity of the early J2EE specifications:
  - Servlet API (JSR 340)
  - WebSocket API (JSR 356)
  - Concurrency Utilities (JSR 236)
  - JSON Binding API (JSR 367)
  - Bean Validation (JSR 303)
  - JPA (JSR 338)
  - JMS (JSR 914)

#### KEY TERMS

- You can switch persistence providers through configuration without changing your code.
- Many other Infrastructure concerns and integration with third-party APIs.
- Cassandra is a distributed, high-performance, always available, eventually consistent,
  partitioned-row-store, NoSQL database.
- non-blocking reactive code
- blocking imperative code

### Tips and Tricks for IntelliJ

#### Import multiple REPOS/PROJECTS under on workspace

- Create Workspace to bring in all the microservices Projects (Instead of Juggling b/w different windows)
- To do that:

  - Create an EMPTY Directory using Terminal
  - Open IntelliJ, create new Project -> select "Empty Project" ->
    select the above created directory -> finish
  - Now do `git clone` all the microservices projects that you intended to bring in.

  - Now Add those projects into the workspace as Module
    - File -> New -> Module -> Module from Existing Source -> Select Project that we have cloned (one by one)
    - Select "Import Module from externals" -> Select Maven -> Finish

##### Auto Import on Paste

- Preference -> Editor -> General -> Auto Import
  - 1. Java -> Insert Imports on Paste -> All
  - 2. Check `Add unambiguous imports on the fly`
  - 3. Check `Optimize imports on the fly (for current projects)`

### Reference

- Spring Projects - https://spring.io/projects

- Visual Studio IDE Extensions - https://spring.io/tools

  - https://marketplace.visualstudio.com/items?itemName=vmware.vscode-boot-dev-pack
  - https://marketplace.visualstudio.com/items?itemName=vmware.vscode-spring-boot
  - https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-spring-initializr
  - https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-spring-boot-dashboard

- Spring Framework Doc - https://docs.spring.io/spring-framework/reference/overview.html

- Design Patterns - https://stackify.com/service-locator-pattern/

- Udemy Course - https://www.udemy.com/course/spring-framework-5-beginner-to-guru/?LSNPUBID=JVFxdTr9V80&ranEAID=JVFxdTr9V80&ranMID=39197&ranSiteID=JVFxdTr9V80-Q3U09afakCGt0cOMJbfn1Q&utm_medium=udemyads&utm_source=aff-campaign

- Ways To Reduce JVM Docker Image Size

  - Use `Dive` to dive into the Docker Image
  - With `jlink`, we can create a custom Java runtime that includes only the necessary components, resulting in a smaller final image ( The Image size is going to be two times smaller than original size - with default java runtime)

  - Stripping Container Image Size, Even More, Using Java Dependency Analysis Tool (`Jdeps`). `Jdeps` understands static dependecies of your applications and libraries. - This is going
    to reduce the docker size by 30%.
  - Refer - https://dzone.com/articles/ways-to-reduce-jvm-docker-image-size

- More Resources
  - https://dzone.com/microservices
  - https://dzone.com/performance
  - https://dzone.com/security
  - https://dzone.com/containers
