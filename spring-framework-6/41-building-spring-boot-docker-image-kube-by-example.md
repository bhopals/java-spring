## Building Spring Boot Docker Image - Kube By Example - Extra

- A Docker image is a collection of layers.
- Each layer is an immutable TAR archive with a hash code generated from the file.
- When you build a Docker image, each command which add files, will result in a layer being created.

- Building Images for Spring Boot

### Introduction

- Taking a Docker Build file and Getting our Spring boot deployable jar into that Image, and Running it.

- Resources:
  - API Docs - https://sfg-beer-works.github.io/brewery-api/#tag/Beer-Service
  - Rest Project - https://github.com/springframeworkguru/kbe-rest-brewery

### Creating Docker File

- Setup a Basic Docker file for our Spring Project

- Create a Docker file `Dockerfile` in `main/java/dockerbase/`

```
FROM openjdk:11-jre-slim
ENV JAVA_OPTS " -Xms512m -Xmx512m -Djava.security.egd=file:/dev/./urandom"
WORKDIR application
COPY ../../../target/kbe-rest-brewery-0.0.1-SNAPSHOT.jar ./
ENTRYPOINT ["java", "-jar", "kbe-rest-brewery-0.0.1-SNAPSHOT.jar"]
```

So at the top we are saying the Docker image that we want to pull from the JRE slim, we are setting
environment variable for Java ops, we're just setting some memory limits and then the security that
helps with our application startup time, we are telling it the working directory called application,
we are copying from our local file system.

So we are doing a relative path there to target to the generated jar file.
We are copying that into our Docker image and then we are setting the entry point and this is the command
that is going to be running on startup.

### Building and Running Docker Image

- `docker ps` - List docker containers

- Build Docker Image
  - `docker build -f ./src/main/dockerBase/Dockerfile -t kbe-rest .`
- Run Docker Image
  - `docker run -p 8080:8080 kbe-rest`
  - Running this make the application up and running inside the docker container

### Building Layered Image

- Docker Images works on Layers

- For Spring, we are going to break our applicaton into LAYER
- Example, All those common dependencies and the stuff that does not change very much,
  we are going to put that in one layer

- Stuff that is changing in our application is going to be much smaller, so we are going to put that
  into another layer.

- We are going to learn how to use Spring Boot Layers, and how to configure our Docker Build file
  to to ahead and utilize these layers.

- It will make our life easier when you are planning to deploy our changes in containerized environments.

### Overview of Maven Configuration

- Refer - https://springframework.guru/why-you-should-be-using-spring-boot-docker-layers/

- Spring Boot Docker Layers allows you to separate your dependencies and application class files into
  different layers.

- This allows your dependency layers to be re-used when possible, significantly reducing the
  size of new releases.

- Enable Packaging Layer (`<layers>`) in Spring Boot Maven Plugin

```
<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
      <configuration>
        <layers>
          <enabled>true</enabled>
          <includeLayerTools>true</includeLayerTools>
        </layers>
      </configuration>
    </plugin>
  </plugins>
</build>
```

### Multi-Stage Docker File

- Create a file `Dockerfile` in `/src/main/docker`

```
FROM openjdk:11-jre-slim as builder
WORKDIR application
ADD target/kbe-rest-brewery-0.0.1-SNAPSHOT.jar ./
RUN java -Djarmode=layertools -jar kbe-rest-brewery-0.0.1-SNAPSHOT.jar extract

FROM openjdk:11-jre-slim

WORKDIR application
COPY --from=builder application/dependencies/ ./
COPY --from=builder application/spring-boot-loader/ ./
COPY --from=builder application/snapshot-dependencies/ ./
COPY --from=builder application/application/ ./
ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom", "org.springframework.boot.loader.JarLauncher"]
```

This is going to be a multi stage build, meaning that we will have two Docker images start up.
So one will build and we are going to be using that to run a command to extract out the layers.
And then the second one is going to be the actual Docker file that we're using for our image, and we'll
take the layers from the first one and copy those over.

- Docker build

  - `docker build -f ./src/main/docker/Dockerfile -t kbe-rest .`

- Docker History
  - `docker history kbe-rest`

### Introduction - Building Docker Images with Maven

- Updating hardcoded values with build evn. variables

  - `ADD maven/${project.build.finalName}.jar ./`
  - `RUN java -Djarmode=layertools -jar ${project.build.finalName}/.jar extract`

- Same in `POM.XML`

### Introduction Docker Maven

- Refer

  - https://hub.docker.com/_/maven (old)
  - https://github.com/eclipse/jkube (new)
  - https://github.com/fabric8io/docker-maven-plugin
  - https://dmp.fabric8.io/#authentication

- Add plugin in `POM.xml`

```
<plugin>
<groupId>io.fabric8</groupId>
<artifactId>docker-maven-plugin</artifactId>
<version>0.35.0</version>
<configuration>
    <verbose>true</verbose>
    <images>
    <image>
        <name>kbe-rest</name>
        <build>
          <assembly><descriptor>artifact</descriptor></assembly>
          <dockerFile>Dockerfile</dockerFile>
          <tags><tag>latest</tag></tags>
        </build>
    </image>
    </iamges>
</configuration>
</plugin>
```

- Update Docker file to use `maven` instead of `target`

  - `ADD target/kbe-rest-brewery-0.0.1-SNAPSHOT.jar ./`

- Run Docker docker:build
  `docker docker:build`

### Use Properties in Build and Docker file

- `ADD maven/${project.build.finalName}.jar ./`
- `RUN java -Djarmode=layertools -jar ${project.build.finalName}/.jar extract`

- Same in `POM.XML`
  - `<tags><tag>${project.version}</tag></tags>`
  - `<name>springframeworkguru/${project.artifactId}</name>`

### Publish to Docker Hub

- Define Docker Authentication credentials

  - https://dmp.fabric8.io/#authentication

- There are six different locations searched for credentials. In order, these are:

  - Providing system properties docker.username and docker.password from the outside.
  - Providing system properties registry.username and registry.password from the outside.
  - Using a <authConfig> section in the plugin configuration with <username> and <password> elements.
  - Using OpenShift configuration in ~/.config/kube
  - Using a <server> configuration in ~/.m2/settings.xml
  - Login into a registry with docker login (credentials in a credential helper or in ~/.docker/config.json)

- Setup any of the above in your project configuration
- Build, Package, and Publish (push) - Pushing the docker file up to the registry
  - `mvn clean package docker:build docker:push`

### KEY TERMS

- Spring Boot Docker Layers
