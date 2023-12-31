## Enterprise Dependency Management

### Maven Bill of Materials

- Bill of Materials
  - Originally a manufacturing term
  - Like a recipe for building widgets
  - An engine will need 8 cylinders, 8 spark plugs, 16 valves, 1 fuel body, etc
- A Maven BOM steals from this concept to serve a Bill of Materials for your services

#### Spring Parent POM

- Very similar to a Maven BOM
- Spring is already providing us a parent POM which provides us a set of curated dependencies
  - Also common set of properties is inherited
- Differences from BOM:
  - Does not set common dependencies
  - Does not set common plugins
  - Does not set common plugin configuration

#### BOM Configuration

- Set common Maven Properties
- Set common Maven Plugins and configuration
- Set dependency versions
- Set common dependencies
- Set common build profiles
- Set just about any inheritable property which is common

### Maven BOM Creation

- Create a new Maven Project
- Update `POM.XML` to use pom Packaging
  `<packaging>pom</packaging>`

- Add `spring-boot` as a Parent POM

```
 <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.5.RELEASE</version>
        <relativePath /> <!-- lookup parent from repository -->
    </parent>
```

- Add `artifact` details for the current pom (so the child POM can use it to Inherit it)

```
    <groupId>guru.springframework</groupId>
    <artifactId>mssc-brewery-bom</artifactId>
    <version>1.0-SNAPSHOT</version>
```

- To see the build POM, we can (in IntelliJ)

  - Right click -> Maven -> Show Effective POM

#### Maven BOM Setting Common Properties

- Add `<properties>` section for any property set(Version details)

```
<properties>
      <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>11</java.version>
        <maven.compiler.source>${java.version}</maven.compiler.source>
        <maven.compiler.target>${java.version}</maven.compiler.target>
        <org.mapstruct.version>1.3.0.Final</org.mapstruct.version>
        <jaxb.version>2.3.0</jaxb.version>
        <awaitility.version>3.1.6</awaitility.version>
</properties>
```

#### Maven BOM Dependency Management

- `dependencyManagement` v/s `dependencies`
  - The `dependencyManagement` allows you to set up dependencies that can be inherited from this POM downstream.
  - It allows us to centralize the specific dependencies that we want included.

```
<dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>javax.xml.bind</groupId>
                <artifactId>jaxb-api</artifactId>
                <version>${jaxb.version}</version>
            </dependency>
            <dependency>
                <groupId>com.sun.xml.bind</groupId>
                <artifactId>jaxb-core</artifactId>
                <version>${jaxb.version}</version>
            </dependency>
            <dependency>
                <groupId>com.sun.xml.bind</groupId>
                <artifactId>jaxb-impl</artifactId>
                <version>${jaxb.version}</version>
            </dependency>
            <dependency>
                <groupId>org.mapstruct</groupId>
                <artifactId>mapstruct</artifactId>
                <version>${org.mapstruct.version}</version>
            </dependency>
            <dependency>
                <groupId>org.awaitility</groupId>
                <artifactId>awaitility</artifactId>
                <version>${awaitility.version}</version>
                <scope>test</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

- Basically, what we're doing is declaring common dependencies for all of our projects in a single spot
  inside of our our Maven BOM here.

- Difference b/w `dependencies` and `dependencyManagement` Tags

  - `dependencyManagement`

    - This tag consists of a dependencies tag which itself might contain multiple dependency tags. Each dependency is supposed to have at least three main tags: groupId, artifactId, and version. Let’s see an example:

    ```
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.apache.commons</groupId>
                <artifactId>commons-lang3</artifactId>
                <version>3.13.0</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
    ```

    - The above code just declares the new artifact commons-lang3, but it doesn’t really add it to the project dependency resource list.
    - `dependencyManagement` is just a declaration, and it does not really add a dependency.

  - `dependencies`
    - This tag contains a list of dependency tags. Each dependency is supposed to have at least two main
      tags, which are groupId and artifactId.
      Let’s see a quick example:
      ```
      <dependencies>
            <dependency>
                <groupId>org.apache.commons</groupId>
                <artifactId>commons-lang3</artifactId>
            </dependency>
        </dependencies>
      ```
      - The version and scope tags can be inherited implicitly if we have used the dependencyManagement tag before in the POM file:

- More - https://www.baeldung.com/maven-dependencymanagement-vs-dependencies-tags
- A very common use case for this feature is a multi-module project.

#### Maven BOM Common Dependencies

- To add dependencies to the Project.
- All the common dependencies of our microservices.

```
 <dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

#### Maven BOM Common Build Plugins

- All the plugins (lombok, maptsruct) that should be shared across child projects

```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
        ....
    </plugins>
</build>
```

- More - https://github.com/bhopals/mssc-brewery-bom/blob/main/pom.xml

#### Maven Enforcer Build Plugins

- Add `maven-enforce-plugin` in Plugins List.
- This is a rule from `Apache`
- It is gonna enforce a set of rules.
- Common Rules, you can fail your build on if not met, for example:

  - requireMavenVersion
  - requireJavaVersion
  - requireReleaseDeps
  - Any PCI compliance RULE.

- For setting up baselines for some commonality among multiple projects as far as versioning and controls that.

```
<plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-enforcer-plugin</artifactId>
                <executions>
                    <execution>
                        <id>enforce-versions</id>
                        <goals>
                            <goal>enforce</goal>
                        </goals>
                        <configuration>
                            <rules>
                                <requireMavenVersion>
                                    <version>3.6.0</version>
                                </requireMavenVersion>
                                <requireJavaVersion>
                                    <version>11</version>
                                </requireJavaVersion>
                                <requireReleaseDeps>
                                    <onlyWhenRelease>true</onlyWhenRelease>
                                    <message>Release builds must not have on snapshot dependencies</message>
                                </requireReleaseDeps>
                            </rules>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
```

#### Update child projects (Microservices to use Parent POM)

```
<parent>
        <groupId>com.github.sfg-beer-works</groupId>
        <artifactId>sfg-brewery-bom</artifactId>
        <version>1.0.17</version>
    </parent>
```

- If we have multiple microservicse that use different data source (DB),
  in that case, instead of just having one parent POM, we can build a hierarchy. For example:
  - Super PARENT BOM (Inherits Spring BOOT) => Parent1 BOM with Mongo ==> Service POM (Microservice1)
  - Super PARENT BOM (Inherits Spring BOOT) => Parent2 BOM with SQL ==> Service POM (Microservice2)
  - Super PARENT BOM (Inherits Spring BOOT) => Parent3 BOM with Queue ==> Service POM (Microservice3)

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

### KEY TERMS

- Maven Bill of Materials
- Set of Curated Dependencies
- Multi Module Projects

- Maven Central Repo
- Bill of Materials for Project
