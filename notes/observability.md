# Observability

## Readiness Probes
pod lifecycle include status and conditions
- status describes where it is in its lifecycle
  - pending: pod hasn't been scheduled yet
  - ContainerCreating: pulling images and starting containers
  - running: all containers started and executing correctly
- conditions complement status via an array of true/false statements
  - PodScheduled: pod has been scheduled on a node
  - Initialized: pod has initialized and is creating containers
  - ContainersReady: all containers are ready
  - Ready: pod is completely ready

kubernetes assumes that containers are ready to provide services as soon as the container hits the ready state, even if the application in the pod isn't working
readiness probes fill that gap, providing an interface for the developer to tell Kubernetes when the application inside the pod is ready to go

configured in as a sub element to the container specification via the `readinessProbe` keyword
multiple tests available:
- httpGet - requests a web page to check for readiness
  - must include path and port options, path is relative and assumes localhost
- tcpSocket - attempts to connect to a port
  - must include port option, assumes connecting on localhost
- exec - runs a command or script that will return success when the application is ready
  - must include command sub element with array of command parameters

```
spec:
  containers:
    ...
    readinessProbe:
      httpGet:
        path: <relative url>
        port: <port>
      tcpSocket:
        port: <port>
      exec:
        command:
        - <command>
        - <arg1>
      initialDelaySeconds: <number of seconds to delay from pod ready to first probe>

      periodSecond: <number of seconds between probes>

      failureThreshold: <number of times to repeat the probe before failing, default = 3>
```
useful in multi-pod deployments so that adding a new pod to a live service is transparent until the application in the new replica pod is able to support users at the same level as the older pods
## Liveness Probes
similar to liveness probes, but they are continuous throughout the life of the container to make sure it doesn't freeze or hang and impact the service it provides
solves the problem where the application stops working, but the container stays alive; provides the interface for kubernetes to understand that the container needs to be restarted/rebuilt
similar probe types as readiness probes
- httpGet
- tcpSocket
- exec
```
spec:
  containers:
    livenessProbe:
      httpGet:
        ...
      tcpSocket:
        ..
      exec:
        command:
          ...
```
similar additional attributes for an initial delay, period, and failure threshold
## Container Logging
when working with pods, have to specify the pod and the container in the pod using the `kubectl logs -f` command
more advanced concepts in logging are covered in CKA
## Monitoring and Debugging Applications
no full-featured built in kubernetes monitoring solution
open source and proprietary tools include:
- metrics server
- prometheus
- elastic stack
- datadog
- dynatrace
### Heapster vs Metrics Server
Heapster was the original, but its deprecated now
Metrics Server is the slimmed down version of heapster
### Metrics Server
can only have one per cluster
stores metrics in memory, not in disk
need to use a more advanced solution for historical analysis
included in the minikube implementation by default
otherwise sourced from github directly
### Kubelet
kubernetes agent that receives instructions and runs containers on each node
includes the cAdvisor component that collects metrics from each node and exposes them through the API
