## Deploying your Microservices

- Deploying Microservices and related infrastructure

### The architecture of a build/deployment pipeline

- 1. A developer commits their code to the source code repository.
- 2. A build tool monitors the source control repository for changes and kicks off a build when a change is detected.
- 3. During the build, the application’s unit and integration tests are run and if everything passes, a deployable
     software artifact is created (a JAR, WAR, or EAR).
- 4. This JAR, WAR, or EAR might then be deployed to an application server running on a server (usually a development server).

### Your build and deployment pipeline in action

- 1. GitHub will be the source repository.
- 2. Travis CI will be used to build and deploy the EagleEye microservices
- 3. Maven with Spotify’s Docker plug-in will compile code, run tests, and create the executable artifact.
- 4. The machine image will be a Docker container.
- 5. The Docker container will be committed to a Docker Hub repo.
- 6. Python will be used to write the platform tests.
- 7. The Docker image will be deployed to an Amazon Elastic Container Service (ECS).

### SUMMARY

- The build and deployment pipeline is a critical part of delivering microservices. A well-functioning build and
  deployment pipeline should allow new features and bug fixes to be deployed in minutes.

- The build and deployment pipeline should be automated with no direct human interaction to deliver a service.
  Any manual part of the process represents an opportunity for variability and failure.

- The build and deployment pipeline automation does require a great deal of scripting and configuration to get right.
  The amount of work needed to build it shouldn’t be underestimated.

- The build and deployment pipeline should deliver an immutable virtual machine or container image. Once a server image has
  been created, it should never be modified.

- Environment-specific server configuration should be passed in as parameters at the time the server is set up.

### KEY TERMS

- Deployable software Artifact

- Github
- Travis CI
- Maven/Spotify Docker Plugin
- Docker
- Docker Hub
- Amazon’s EC2 Container Service (ECS)

- Build Deploy Engine
- Source Repository
- Automate Build and Deploy pipeline
- No direct human interaction to deploy a service
