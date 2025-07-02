# Services and Networking
Course section 7

# Services Basics
helps connect applications within the cluster or to users outside of the cluster
enables loose coupling between microservices
## Types
### NodePort
makes a pod-internal port accessible through a port on the node
three ports involved:
- target port: on the node, where the service forwards the request to
- port: on the service itself; functions like a virtual server inside the cluster
- node port: port on the node that gets forwarded to the target port
uses labels and selectors to match the service definition to the pods
when multiple pods are selected on the same node, will randomly load balance across all matching pods with session affinity
when multiple pods are selected on different nodes, the service object will stretch across all applicable nodes, and forward to the applicable pod on the node that was requested
```
apiVersion: v1
kind: Service
metadata:
  name: <service name>
spec:
  type: NodePort
  ports:
  - targetPort: <target port num>
    port: <service port num>
    nodePort: <node port num> ## must be greater than 30,000
  selector:
    <key>: <value>
    <key>: <value>
```
### ClusterIP
creates a virtual IP inside the cluster to enable communication between pods within the cluster
same as virtual/cluster IPs in conventional infrastructure
load balances randomly across all selected pods and nodes, same as the NodePort service
```
apiVersion: v1
kind: Service
metadata:
  name: <service name>
spec:
  type: ClusterIP ## also the default value for Service objects
  ports:
  - targetPort: <port on the pods>
    port: <port on the service>
  selector:
    <key>: <value>
    <key>: <value>
```

### LoadBalancer
provisions a load balancer in supported cloud providers
- not discussed in this class, since NodePort and ClusterIP do load balancing inherently

# Ingress Networking
layer7 load balancer that uses kubernetes primitives to solve the nested load balance problem of hosting multiple applications in the same cluster at the same IP/URL
still needs to be exposed externally to match the high external port to the ingress object
## Ingress Controller
the load balancer object
NOT deployed in kubernetes by default
can deploy any of a number of proxies, like nginx, contour, haproxy, traefik, istio, GCE
- nginx and GCE are supported and maintained by the kubernetes project
deployed like any other deployment with the nginx-ingress-controller image
have to provide args to the definition file to start the application and provide other nginx-relevant configurations
- under spec.template.spec.args
  - /nginx-ingress-controller - starts the controller
  - --configmap=$(POD_NAMESPACE)/nginx-configuration - matches a configmap object that includes other nginx-relevant configuration options; doesn't need to be filled in at first go, but needs to at least exist
- also need to provide environment variables under spec.template.spec for the name of the pod running the ingress controller and the namespace of the pod
- and ports at the same level for the ports that the controller use
key components:
- deployment, to create the pod
- service, to link the external system to the controller pod
- configmap, to provide additional parameters to the nginx application
- ServiceAccount, to access all of the objects
## Ingress Resources
the rules/routes that tell the controller what to do
still uses a definition file with the same root items
below routes everything to a single pod; there's no rules or special routing
``` 
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: <name for the ingress resource>
spec:
  backend:
    service:
      name: <name of the service object to forward to>
      port:
        number: <port to forward to>
```
below implements rules
rules cover a single domain and handle permutations of all URLs for that domain
can also have a catch-all domain name
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: <name of the service object>
spec:
  rules:
  - http:
      paths:
      - path: <relative url path>
        pathType: Prefix
        backend:
          service
            name: <service name>
            port:
              number: <port on the service>
```
catch-all domain is handled by the "default backend" property
there's one that's always set, but you have to create the service object to ensure it goes to an actual pod/page

below configures a single resource to match multiple domain names
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: <resource name>
spec:
  rules:
  - host: <host/domain name to match on for these rules>
    http:
      paths:
      - path:
        pathType: Prefix
        backend:
          service:
            name: <name of the service>
            port:
              number: <port num>
  - host: <second host>
  ...
```