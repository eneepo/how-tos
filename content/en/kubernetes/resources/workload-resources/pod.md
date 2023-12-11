---
title: "Pod"
linkTitle: "Pod"
description: "Pod"
date: 2023-12-10T15:28:38+01:00
categories: [CKAD,CKA]
tags: [pod,design-pattern]
weight: 1
---

A Pod is the smallest and simplest unit you can deploy. A Pod may contain one or more tightly coupled containers, sharing the same network namespace and having access to the same storage volumes. 

The containers inside a Pod share the same network IP address and port space, which means they can easily communicate with each other using "localhost." They also share the same lifecycle, so if the Pod is created, all containers inside it start running, and if the Pod is deleted, all the containers are terminated together.

## Example

```yaml
# kubectl run my-nginx --image=nginx --labels="app=my-app,env=prod" --port=80 --env="NGINX_PORT=80" --restart=Always

apiVersion: v1              # Specifies the Kubernetes API version being used for this resource.
kind: Pod                   # Indicates that this YAML defines a Kubernetes Pod resource.

metadata:                   # Metadata section contains information about the Pod.
  creationTimestamp: null     # The timestamp when this Pod was created, initially set to null.
  labels:                     # Labels are key-value pairs used to identify and categorize the Pod.
    app: my-app               # Label indicating the application name is 'my-app'.
    env: prod                 # Label indicating the environment is 'prod'.
  name: my-nginx              # The name assigned to this Pod.

spec:                       # The specification section defines the desired state of the Pod.
  containers:                 # List of containers running inside this Pod.
  - env:                      # Environment variables to be set within the container.
    - name: NGINX_PORT        # Name of the environment variable.
      value: "80"             # Value assigned to the NGINX_PORT environment variable.
    image: nginx              # The Docker image to be used for this container.
    name: my-nginx            # Name of this container within the Pod.
    ports:                    # List of ports to be exposed by this container.
    - containerPort: 80       # The container will listen on port 80.
    resources: {}             # Resource requests and limits can be specified here.
  dnsPolicy: ClusterFirst     # Defines the DNS resolution policy for this Pod.
  restartPolicy: Always       # Specifies the Pod's restart policy, which is set to "Always."

status: {}                    # The current status of the Pod (empty in this YAML as it's an initial state).
```

## CMD vs ENTRYPOINT
`CMD` commands are ignored by daemon when there are parameters stated within the docker run command while `ENTRYPOINT` instructions are not ignored but instead are appended as command line parameters by treating those as arguments of the command.

### `CMD`
Sets default parameters that can be overridden from the Docker command line interface (CLI) while running a docker container.

```dockerfile
FROM ubuntu
CMD ["echo", "Hello World"]
```

```bash
$ docker build -t cmd-instructions .
[+] Building 0.0s (5/5) FINISHED                           docker:desktop-linux

$ docker run cmd-instructions
Hello World

$ docker run cmd-instruction echo "Hola!"
Hola!

$ docker run cmd-instructions printenv
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=bd90afe95bd8
HOME=/root
```

### `ENTRYPOINT`
Sets default parameters that cannot be overridden while executing Docker containers with CLI parameters.

```dockerfile
FROM ubuntu
ENTRYPOINT ["echo", "Hello World"]
```

```bash
$ docker build -t entrypoint-instructions .
[+] Building 0.0s (5/5) FINISHED                           docker:desktop-linux

$ docker run entrypoint-instructions
Hello World

$ docker run entrypoint-instruction echo "Hola!"
Hello World echo Hola!

$ docker run entrypoint-instructions printenv
Hello World printenv

$ docker run --entrypoint printenv entrypoint-instructions
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=d21eb640262d
HOME=/root
```

### Using `CMD` and `ENTRYPOINT` together

```dockerfile
FROM ubuntu
ENTRYPOINT ["echo", "Hello"]
CMD ["World!"]
```

```bash
$ docker build -t entrypoint-cmd .
[+] Building 0.0s (5/5) FINISHED                           docker:desktop-linux

$ docker run entrypoint-cmd
Hello World!

$ docker run entrypoint-cmd Eneepo!
Hello Eneepo!

$ docker run entrypoint-instructions printenv
Hello World printenv

# Sleeps for 5 seconds. Equivalent of running `sleep 10`
$ docker run --entrypoint sleep entrypoint-cmd 5
```

### Equivalents in Kubernetes
In Kubernetes:
* Use `command` for `ENTRYPOINT`
* use `args` for `CMD`

```dockerfile
FROM Ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```

```shell
$ docker build ubuntu-sleeper .
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper
    command: ["sleep"]  
    args: ["10"]
```

Running the above pod is simillar to running:
```bash
$ docker run ubuntu-sleeper 10
```

#### Change `command` and `args` when creating pod

```bash
# Start the nginx pod using the default command, but use custom arguments (arg1 .. argN) for that command
# kubectl run nginx --image=nginx -- <arg1> <arg2> ... <argN>
$ kubectl run ubuntu-sleeper --image=ubuntu-sleeper-pod -- 5

# Start the nginx pod using a different command and custom arguments
# kubectl run nginx --image=nginx --command -- <cmd> <arg1> ... <argN>
$ kubectl run ubuntu-sleeper --image=ubuntu-sleeper-pod --command -- echo "Hello World!"
```


## Readiness and LivenessProbe Probe
**ReadinessProbe:** is a Kubernetes feature that allows you to specify a readiness check for your containers within a Pod. It determines whether a container is ready to accept network traffic and serve requests. When a container is not ready, it is temporarily removed from the service's load balancing, ensuring that only healthy containers receive traffic. This is particularly useful during deployments or when containers require some time to initialize or become fully operational.

**LivenessProbe:** is another Kubernetes feature used to ensure the continuous health of a container within a Pod. It periodically checks whether a container is still running as expected. If the liveness probe fails (e.g., the application crashes or hangs), Kubernetes will restart the container automatically to maintain the desired state. Liveness probes help improve the reliability of your applications by preventing containers from entering a broken or non-responsive state.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod                   # The name of the Pod
spec:
  containers:
    - name: my-container         # The name of the container within the Pod
      image: nginx:latest        # The Docker image to use for this container (Nginx in this case)
      ports:
        - containerPort: 80      # Port to expose within the container
      livenessProbe:             # Liveness probe configuration
        httpGet:                 # Use an HTTP GET request for the liveness probe
          path: /                # The path to check for liveness (replace with your application's path)
          port: 80               # Port to access the path
        initialDelaySeconds: 15  # Delay before the first liveness probe is executed
        periodSeconds: 10        # How often to perform the liveness probe
      readinessProbe:            # Readiness probe configuration
        exec:                    # Use an exec command for the readiness probe
          command:               # The command to run for readiness
            - /bin/sh
            - -c
            - echo "Readiness check passed"  
        initialDelaySeconds: 5   # Delay before the first readiness probe is executed
        periodSeconds: 10        # How often to perform the readiness probe
```

## Multi-container Design Patterns

Multi-container design patterns are used to define how multiple containers within a pod interact and collaborate with each other to accomplish a specific task. These design patterns enable efficient communication and resource sharing between containers within a pod. Here are some commonly used multi-container design patterns in Kubernetes:

### Init Container Pattern
In this pattern, an init container is added to a pod to perform initialization tasks before the main container starts. Init containers are executed sequentially and can be used to set up prerequisites, populate shared volumes, or perform other one-time setup tasks.


An init container is a special type of container that runs and completes before the main application containers start. Init containers are designed to perform initialization tasks, such as setting up configuration files, populating a shared database, or fetching required resources, before the main containers are launched. They help ensure that the application containers start in a known and ready state.

Init containers run to completion before any of the application containers start, and they share the same pod and network namespace. The main containers in the pod do not start until all init containers have successfully completed their execution. If any init container fails, Kubernetes restarts the pod and retries running the init containers until they all succeed.

Init containers are defined in the pod specification, alongside the main containers, and they are executed in the order they are defined. Each init container runs one at a time, and only when the previous init container has successfully completed.

Here's an example of a pod specification with init containers:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: main-container
    image: my-main-image
    # ...
  initContainers:
  - name: init-container-1
    image: my-init-image-1
    # ...
  - name: init-container-2
    image: my-init-image-2
    # ...

```
{{< alert title="Note" >}}The following sections are not part of **CKAD** or **CKA** exams!{{< /alert >}}

### Sidecar Pattern
In this pattern, a sidecar container is added to a pod to enhance or extend the functionality of the main container. The sidecar container runs alongside the main container and shares the same lifecycle, network, and storage resources. It can perform tasks such as logging, monitoring, security, or proxying for the main container.

### Ambassador Pattern
The ambassador pattern is used to offload network-related concerns from the main container to a separate ambassador container within the same pod. The ambassador container acts as a proxy and handles tasks such as load balancing, SSL termination, or service discovery, allowing the main container to focus on its core functionality.

### Adapter Pattern
The adapter pattern involves using an adapter container to convert the output of one container into a format that another container can understand. This pattern is commonly used when integrating containers with different protocols or data formats, allowing them to communicate effectively.

### Sidecar Injector Pattern
The sidecar injector pattern is used to dynamically inject a sidecar container into a pod at runtime based on specific conditions or requirements. This pattern is often used for tasks such as injecting a container for logging or injecting a container with additional security features based on policies or configurations.

### Adapterless Pattern
The adapterless pattern is an alternative to the adapter pattern, where containers communicate directly with each other without the need for an intermediary adapter container. This pattern is suitable when containers can understand and work with each other's protocols or data formats without any conversion.

These are some of the commonly used multi-container design patterns in Kubernetes. Each pattern has its own benefits and use cases, allowing developers to design complex, scalable, and modular applications within Kubernetes clusters.

## Commands

```shell
# Create a resource from a file
kubectl create -f pod-definition.yaml

# Apply a configuration to a pod by file name
kubectl apply -f pod-definition.yaml

# Display one or many pods.
kubectl get pods

# Prints a table of the most important information about the specified resources.
kubectl describe pod myapp-pod

# Quickly generate a new pod definition 
kubectl run my-pod --image nginx --dry-run=client -o yaml > pod-definition.yaml

# Start the nginx pod using the default command, but use custom arguments (arg1 .. argN) for that command
kubectl run nginx --image=nginx -- <arg1> <arg2> ... <argN>

# Start the nginx pod using a different command and custom arguments
kubectl run nginx --image=nginx --command -- <cmd> <arg1> ... <argN>

# Generate a pod definition out of an exisiting pod
kubectl get pod mu-pod -o yaml > pod-definition-get.yaml
```