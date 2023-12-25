## Kube By Example - Spring Boot on Kubernetes

- Take a Spring Boot Application and Run it in Kubernetes

- So there's a number of things that we need to do when we're running spring applications.
  Obviously, we want to be exposing ports, running it in Docker, getting a containerised, bringing
  it up in Kubernetes, deploying it in Kubernetes, so that there's a lot that we need to do, like environment

### Setup

- Kubernetes
  - You will need access to a Kubernetes cluster
  - Online you can use OpenShift Playground for free
  - Locally you can run a single node cluster
    - Docker Desktop (easiest if you have Docker Desktop installed)
    - minikube (VM based)
    - kind (Docker based)
  - kubectl configured to connect to your cluster option

### Enable Kubernetes in Docker Desktop

- Docker Desktop Enable kubernetes Settings

  - Install Docker Desktop, Open it
  - Click on "kubernetes", select "Enable kubernetes"
  - Click on Apply and Restart

- Example commands

  - `docker ps`
  - `kubectl get nodes`
  - `kubectl config get-context`

- Enable Docker Desktop context
  - `kubectl config use-context docker-desktop`

### Introduction to Deploying on Kubernetes

#### Create Deployment

- A deployment is responsible for keeping a set of pods running.
- Deployments ensure the desired state of your application, handling the lifecycle and scaling

- Go to docker, where we have generated docker image (in last tutorail)

- `kubectl get all`

- Create Deployment YML using `kubectl`

  - `kubectl create deployment kbe-rest --image springframeworkguru/kbe-rest-brewery --dry-run=client -o=yaml >> deployment.yml`

- `ls`
- `kubectl apply -f deployment.yml` - To creat a pod
- `kubectl get all`

#### Create Service

- A service is responsible for enabling network access to a set of pods.
- Services act as the gateway for accessing your application, enabling seamless communication
  and load balancing.

- `kubectl create service clusterip kbe-rest --tcp=8080:8080 --dry-run=client -o=yaml >> service.yml`
- `ls`
- `kubectl apply -f service.yml` - To creat a pod
- `kubectl get all`

### Port Forwarding

- To access our services
- `ipconfig getifaddr en0`

- `kubectl port-forward service/kbe-rest 8080:8080` (mostly when you are working on local)

- `curl localhost:8080/actuator/health`

### Terminating Services and Deployments

- `kubectl get all`
- `kubectl delete service kbe-rest`
- `kubectl delete deployment kbe-rest`
- `kubectl get all`

#### Exposing Services

- We are doing that by changing spec type to `NodePort` instead of `ClusterIp`
- `vi service.yml`
- Update type to `NodePort` instead of `ClusterIp`

- `kubectl apply -f deployment.yml`

- `kubectl get all`

`ClusterIP` is used for Pod-to-Pod communication within the same Kubernetes cluster. In contrast, `NodePort` and `LoadBalancer` Services are used for communication between applications within the cluster and external clients outside the cluster.

#### Accessing Logs

- Accesing Container Logs

- Docker Logs

  - `docker ps` - Grab the id
  - `docker logs <pid>`
  - `docker -f logs <pid>` (Tailing the logs)

- Kubernete Cluster Logs
  - `kubectl get all` - List all the pods - grab the name
  - `kubectl logs <name-of-the-pod>` (Screen full logs)
  - `kubectl logs --tail=20 <name-of-the-pod>` (20 lines)
  - `kubectl logs -f <name-of-the-pod>` (Tailing the logs)

#### Setting Environment Variables

- Update `application.properties`

- `kubectl get all` - List all the pods - grab the name
- `kubectl logs <name-of-the-pod>` (Screen full logs)

- To update log level by overriding `application.properties` property

  - In `application.properties`
    `logging.level.guru.springframework.sfgrestbrewery=debug`
  - `vi deployment.yml`
  - Add property under `spec container image env`
    - `name:LOGGING_LEVEL_GURU_SPRINGFRAMEWORK_SFGRESTBREWERY`
    - `value: "info"`
  - Save `deployment.yml`

- Reapply the change

  - `kubectl apply -y deployment.yml`

- `kubectl get all`

#### Readiness Probe

- Readiness Probe basically tells Kubernetes that the container is ready to start accepting traffic.

- `vi deployment.yml`
- Add properties under `spec container image env`
  - `name:MANAGEMENT_ENDPOINT_HEALTH_PROBES_ENABLED`
  - `value: "true"`
  - `name:MANAGEMENT_HEALTH_READINESSTATE_ENABLED`
  - `value: "true"`
- Add properties under `spec container image`
  - `readinessProbe`
  - `httpGet:`
  - `  port:8080`
  - `  path: /actuator/health/readiness`
- Save the change

- Reapply the changes
  - `kubectl apply -f deployment`

#### Liveness Probe

- The readiness probe basically tells Kubernetes when the application has started up and is ready to
  start accepting traffic
- A Liveness Probe is a health check saying that, hey, I'm alive and I can continue

- `vi deployment.yml`
- Add properties under `spec container image env`
  - `name:MANAGEMENT_ENDPOINT_LIVENESS_PROBES_ENABLED`
  - `value: "true"`
- Add properties under `spec container image`

  - `livenessProbe`
  - `httpGet:`
  - `  port:8080`
  - `  path: /actuator/health/liveness`

- Save the change

- Reapply the changes
  - `kubectl apply -f deployment`

### Graceful Shutdown

Another feature that is available in Spring Boot 2.3 or higher is the ability to handle a graceful shutdown.
So what we don't want Kubernetes to do is terminate our application while it's still processing stuff.
So we want to give that a little bit of time to complete properly.

`vi deployment.yml`

- Add properties under `spec container image env`
  - `name:SERVER_SHUTDOWN`
  - `value: "graceful"`
- Add properties under `spec container image`

  - `lifecycle`
  - `preStop:`
  - `  exec:`
  - `     command: ["sh", "-c", "sleep 10"]`

- Save the change

- Reapply the changes
  - `kubectl apply -f deployment`

### KEY TERMS

- Readiness Probe
