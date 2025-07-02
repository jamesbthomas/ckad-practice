# Multi-Container Pods
Aligns with microservices concepts by developing and deploying two sets of code that work together directly
share the same lifecycle, storage, network component
## Declarative
just add a new element (with the `-`) to under spec.containers

# Design Patterns
## Sidecar
container that picks up logs and sends them to the central logging server
### InitContainers
new feature in K8s API V1.29 that ensures sidecar containers start before and end after the main container in a pod
example definition
```
spec:
  initContainers:
  - name: <sidecar name>
    image: <image ref>
    ...
  containers:
  - name: <main name>
    image: <image ref>
    ...
```
### Co-Located Containers
new term in v1.29 for sidecar containers that collaborate to achieve a common goal
defined the usual way in spec.containers, but different design pattern
## Adapter
container that formats logs from different applications into a standardized format, reducing compute load on the central server
## Ambassador
container that identifies which database to push data to at different stages of development
instead of changing the application code, Ambassador just proxies the connection to the correct database and abstracts the destination from the application