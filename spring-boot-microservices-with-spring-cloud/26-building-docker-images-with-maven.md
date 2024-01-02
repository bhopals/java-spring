## Building Docker Images with Maven

- Docker Layer
  - A Docker layer is effectively a TAR archive with a hash. Main thing to remember it is read only
    file system slice

### Docker Host Considerations

- Running multiple Docker containers on a host can be resource intensive
- Can also consume a lot of disk space on host system
- Recommendations:
  - Use ‘slim’ base images - you don’t need a full featured linux OS for Java runtime
  - Be aware of build layers as you build images
    - A host system only needs one copy of a layer

#### Which Base Image?

- Highly debated and opinionated topic in Java community
- We will be using OpenJDK Slim
- Opinionated options are from Fabric8 are a good choice
- Azul Systems publishes curated JVM Docker images, good choice for commercial applications
- For security and compliance, companies might want to build their own base image

#### Building Docker Images with Maven

- Maven can be configured to build and work with Docker images
- Capability is done with Maven plugins
- Several very good options available
- Will be using Fabric8’s Maven Docker Plugin - (pronounced fabricate)
  - Very versatile plugin w/rich capabilities - only using for build
  - Fabric8 is a DevOps platform for Kubernetes and Openshift - worth becoming more familiar with

#### Docker Integration

- The Maven plugins work with Docker installed on your system
  - Generally, will auto detect the Docker daemon
  - This can be different depending on your operating system
- If Fabric8 cannot connect to Docker you may need to configure the plugin
  - Under the Maven POM properties element:
    - Set property “docker.host” for your operating system

#### Building Docker Images with Maven (Our App)

- For microservices using common BOM
  - Fabric8 is configured in parent
  - Each service will need a Dockerfile in /src/main/docker
- For microservices NOT using common BOM
  - Fabric8 will need to be configured in Build element of Maven
  - Each service will need a Dockerfile in /src/main/docker

#### Spring Boot Layered Builds

- Layered builds is a new feature with Spring Boot 2.3.0
  - Detailed blog post in course resources
- We will be configuring our builds to perform layered builds
- Services using BOM need to use 1.0.17 or higher
- Services not using BOM need to use Spring Boot 2.3.0 or higher

#### Publishing to Docker Hub

- If you’ve created a Docker hub account and wish to publish images to your own account:
  - Configure server credentials in settings.xml (User home dir/.m2)
  - In servers element, add server with id of ‘docker.io'
  - Add your username and password to respective elements

### Configure Docker Build

- Open `Beer Service`
- Add Fabric8 plugin configuration in `POM.XML`
  - More - https://maven.fabric8.io/
- Add Spring Boot maven configuration in `POM.XML for Docker Layers

- Create Docker file `main/Dockerfile`

```
FROM openjdk:11-jre-slim as builder
WORKDIR application
ADD maven/${project.build.finalName}.jar ${project.build.finalName}.jar
RUN java -Djarmode=layertools -jar ${project.build.finalName}.jar extract

FROM openjdk:11-jre-slim
LABEL PROJECT_NAME=${project.artifactId} \
      PROJECT=${project.id}

EXPOSE 8080

WORKDIR application
COPY --from=builder application/dependencies/ ./
COPY --from=builder application/spring-boot-loader/ ./
COPY --from=builder application/snapshot-dependencies/ ./
COPY --from=builder application/application/ ./
ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom", "org.springframework.boot.loader.JarLauncher"]
```

- Run Docker Command to build image - ``
- More Details - https://springframework.guru/why-you-should-be-using-spring-boot-docker-layers/

### Push Images to Docker Hub

- `mvn clean package docker:build docker:push`

### KEY TERMS

- Use `slim` Base Image

- Fabric8

- Docker Layer
- Docker File
- Layered Images
