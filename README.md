# Kubernetes Overview and Key Concepts

## Introduction to Docker and Kubernetes

As we know, Docker is used to containerize applications. While Docker makes running an application in any machine easy, it has limitations when it comes to scaling or managing multiple containers. For instance, if you want to create hundreds or thousands of containers, managing them manually with Docker is inefficient.

Kubernetes overcomes these limitations by allowing you to spin up as many containers as needed with a single command. Kubernetes can handle tasks such as:
- Spinning up thousands of containers with one command.
- Managing resource allocation, such as running a specific number of containers.
- Performing **A/B testing** by running a subset of containers for a new feature.

## Managing Multiple Applications

Consider a scenario where you have multiple projects:
- A **Payment service** in Python.
- A **Frontend** written in React.
- An **Authentication service** written in Java.
- A **Redis service** for caching.
- A **Database** for storing data.

Kubernetes allows you to define how many replicas of each service you want, and it automatically handles scaling and managing these services. These configurations are defined in **YAML files**, where you specify things like the number of replicas, configurations, and specifications for each service.

## Kubernetes Nodes and Cluster

Kubernetes allows you to create multiple nodes. Even if one node fails, other nodes can ensure your application is still accessible. Kubernetes also enables **load balancing** and **high availability** by distributing work among multiple nodes.

The **Control Plane** (formerly called the master node) manages the worker nodes. It decides:
- Which nodes to send traffic to.
- How many nodes to create.
- What happens when a node fails.

The **Control Plane** consists of the following components:
- **Kube-API Server**: Handles front-end APIs and services, including authentication and environment variables.
- **etcd**: Stores cluster data, such as worker node details.
- **Controller Manager**: Manages the state of the cluster, ensures nodes are running, and handles node failures.
- **Kube Scheduler**: Distributes work across containers and nodes.

Each **Worker Node** contains:
- **Kubelet**: Manages applications, ensures they are running, and reports their status.
- **Kube Proxy**: Handles communication between worker nodes and the control plane.
- **Container Runtime**: Runs containers on the worker node.

## Interacting with Kubernetes

Kubernetes provides a command-line interface (CLI) tool called `kubectl`. This tool allows you to:
- Spin up servers.
- Manage cluster changes.
- Scale applications with commands.

Here are some key `kubectl` commands:
- **kubectl version**: Displays the current version of `kubectl`.
- **kubectl --help**: Provides a list of available commands.
- **kubectl get nodes**: Lists all the nodes in the cluster.
- **kubectl get nodes -o wide**: Displays detailed information about the nodes, such as IP address, OS, and version.

### Pods in Kubernetes

In Kubernetes, **pods** are the smallest deployable units, which run one or more containers. Here's how they work:
- A **pod** can run multiple containers, such as your application and the environment required for it.
- You can scale pods by creating multiple replicas across different nodes.
- If one node fails, Kubernetes ensures the other replicas continue running.

An example use case is a **three-tier application** with services in Python, Java, and Angular. You can create separate pods for each service and scale them independently.

### YAML Files in Kubernetes

Kubernetes configurations are defined in **YAML files**. Here's a breakdown of a YAML file:
- **apiVersion**: Defines the API version for the configuration. For example, pods have `v1` as their version, while replicas may have `apps/v1`.
- **kind**: Specifies the type of resource (e.g., Pod, Service, ReplicaSet, Deployment).
- **metadata**: Contains information like the name and labels for the resource.
- **spec**: Defines the specification for the containers within the pod. This includes a list of containers, images, and their configurations.

```yaml
apiVersion: v1       # Defines the API version
kind: Pod            # Defines the kind as a Pod
metadata:
  name: my-app-pod   # Name of the Pod
  labels:            # Labels are key-value pairs
    app: my-app
    tier: backend
spec:
  containers:        # Define the containers to be run in this Pod
  - name: my-app-container
    image: nginx:latest    # Image to be used for this container
    ports:
    - containerPort: 80    # Port to expose from the container
```

#### Example Commands for Pods

Here are some useful `kubectl` commands for working with pods:
- **kubectl get pods**: Lists all the pods.
- **kubectl describe pod [pod-name]**: Provides detailed information about a specific pod.
- **kubectl run [pod-name] --image=[image-name]**: Runs a pod with a specified image.
- **kubectl get pod [pod-name] -o yaml**: Retrieves the YAML file for a specific pod.
- **kubectl create -f pod [yaml-file-name]**: Run a pod with a specified details in yaml file.

## YAML Indentation
**Important Note:** YAML files should use spaces instead of tabs for indentation, as tabs can cause issues with conversion to JSON.

## ReplicaSet in Kubernetes

### Why do we need a ReplicaSet?

In Kubernetes, a **ReplicaSet** ensures that a specified number of pod replicas are running at any given time. Let's imagine a scenario where we have a worker node with a single pod running. If that pod fails, access to the application will be lost. To maintain **high availability**, we can use a ReplicaSet, which automatically creates replicas of the pods, spreading them across multiple worker nodes to ensure continued service.

#### Key Points:
- ReplicaSets ensure high availability by maintaining a specified number of replicas.
- If a pod fails, the ReplicaSet automatically replaces it with a new one.
- ReplicaSets can distribute pod replicas across multiple worker nodes for better load balancing.
- You can't move a running pod to a different node manually. If needed, the pod must be deleted and recreated by the ReplicaSet.
- Traffic distribution is handled by the Kubernetes control plane.

### YAML Structure for a ReplicaSet

The basic structure of a ReplicaSet YAML file consists of four main parts:
1. **apiVersion**: Defines the API version.
2. **kind**: Specifies the type of resource (ReplicaSet in this case).
3. **metadata**: Contains the name and labels for the ReplicaSet.
4. **spec**: Defines the number of replicas, selector, and pod template.

#### Example ReplicaSet YAML File

```yaml
apiVersion: apps/v1            # API version for ReplicaSet
kind: ReplicaSet               # Specifies ReplicaSet
metadata:
  name: my-app-replicaset      # Name of the ReplicaSet
  labels:
    app: my-app                # Labels to categorize the ReplicaSet
spec:
  replicas: 3                  # Number of pod replicas to maintain
  selector:                    # Selector to match the pod labels
    matchLabels:
      app: my-app
  template:                    # Pod template definition
    metadata:
      labels:
        app: my-app            # Label for the pods created by this ReplicaSet
    spec:
      containers:
      - name: my-app-container
        image: nginx:latest    # Container image
        ports:
        - containerPort: 80    # Port exposed by the container
```

### Commands to Manage ReplicaSets

1. **Create a ReplicaSet:** `kubectl apply -f replicaset.yaml`

2. **Get a list of ReplicaSets:** `kubectl get replicaset`

3. **View the pods created by the ReplicaSet:** `kubectl get pods`

4. **Scale up or scale down the replicas without editing aml file:** `kubectl scale replicaset <replicaset name> --replicas=<number of replica>` 

5. **Update the number of replicas:** Modify the replicas field in the YAML file, then run

`kubectl replace -f replicaset.yaml`


## Deployment in Kubernets

A deployment in Kubernetes is responsible for managing application updates, scaling, and ensuring that the application is always available. You can update pods sequentially, rolling out new changes while ensuring high availability.

### Creating a Deployment

To create a deployment, we need to define a YAML configuration file similar to the one for a replica set. However, in a deployment YAML file, we need to change two things:

- **API Version:** `apps/v1`
- **Kind:** `Deplyment`

 Here’s an example of a deployment YAML file:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: example-app
  template:
    metadata:
      labels:
        app: example-app
    spec:
      containers:
      - name: example-container
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

### Commands to Manage Deployment

1. **Create Deployment:** `kubectl create -f deployment-definition.yaml`
2. **Get List of Deployments:** `kubectl get deployments`
3. **Update Deployment**
     - Update the image of your deployment by modifying the YAML file and applying the changes: `kubectl apply -f deployment-definition.yaml`
     - Alternatively, without modifying the YAML file: `kubectl set image deployment/example-deployment example-container=nginx:1.16.0`
4. **Rollback to Previous Version:** `kubectl rollout undo deployment/example-deployment`
5. **Pause and Resume Deployment** `kubectl rollout pause deployment/example-deployment`
6. **To resume** `kubectl rollout resume deployment/example-deployment`

### Key Concepts of Deployment

- **Rolling Updates :** Update pods sequentially to ensure high availability. This is useful when you need to release features without downtime.
- **Pause and Resume :** First Pause all the pods and update the pods with new changes then resume the pods.

## Kubernetes Services

Services in Kubernetes help connect different pods, allowing them to communicate. A service provides an abstraction to expose pods as a network service.

### Types of Services

1. **ClusterIP**
    - Default type.
    - Used for internal communication between pods (e.g., Flask to Redis).
    - Not exposed to the outside world.

2. **NodePort**
     - Exposes the service on each node’s IP at a static port.
     - Can be accessed externally via NodeIP:NodePort.
     - Used to expose a web application, e.g., an Angular app.
  
3. **LoadBalancer**   
    - Exposes the service externally and automatically integrates with a cloud provider's load balancer.
    - Distributes the load between multiple pods.

#### ClusterIP Service Example

When creating a Redis service (ClusterIP), it allows internal communication within the cluster, for example, a Python Flask pod connecting to a Redis pod.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-service
spec:
  type: ClusterIP
  ports:
    - port: 6379
      targetPort: 6379
  selector:
    app: redis
```

#### NodePort Service Example

NodePort exposes a service to the external world using a specific port. Here’s an example of an Angular app being exposed:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: angular-service
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30001
  selector:
    app: angular-app
```

#### LoadBalancer Service Example

If you want to distribute load across multiple nodes or integrate cloud provider load balancers:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-loadbalancer-service
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 80
  selector:
    app: my-app
```


### Service Selector and Load Balancing

The selector in the service definition ensures that traffic is routed to pods matching the label. If you have multiple pods with the same label, the service automatically balances traffic between them. Kubernetes uses a random algorithm to distribute traffic, but you can customize this behavior.


  
