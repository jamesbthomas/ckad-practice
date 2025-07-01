# General notes on Kubernetes architectural components

# Replication Controller
The 'brain' of Kubernetes that maintains High Availability
- balances load for multiple pods, regardless of the size of the pood
- stretches across multiple nodes
- older technology that is being replaced by a tech called a Replica Set

as with anything kubernetes, defined in a YAML with the four default root elements
- apiVersion: v1
- kind: ReplicationController
- spec.template - specifies the pod template to be used to create replicas; matches metadata and spec fields from the pod definition YAML
- spec.replicas - number of times the pod should be duplicated

created with the same kubectl create command as creating pods
getting pods will show all of the pods created, with name starting with the name of the replication controller

# Replica Set
Newer version of the HA tech for kubernetes
very similar to replication controller
- apiVersion: apps/v1 - if not apps, will return an error saying it doesn't understand replicaset
- kind: ReplicaSet
- spec.template - same as replication controller
- spec.replicas - same as replication controller
- spec.selector - identifies the pods that the replicaset should manage; allows replicaset to manage pods created outside of the definition file, so you could create pod1, then create replicaset1 to manage pod1 and the other pods in replicaset1
  - uses the `matchLabels` item, where child items are `type: <label name>`

still need to provide the template even if the ReplicaSet is managing existing pods; template tells it what to use to recreate a pod if one crashes

# Labels
provide easy filters to tell a ReplicaSet what to monitor
set at pod creation time in the yaml

# Deployments
Covers use cases like deploying multiple instances of applications in product environments, rolling updates, rollbacks, maintenance windows to collect multiple changes in one update
Deployment is one step above the ReplicaSet in the hierarchy
defined in a YAML file that looks exactly like the replicaset, except the kind root item which is set to `Deployment`

Creating a deployment automatically creates a ReplicaSet, which creates Pods based on the provided specification
- names will percolate down, so `deployment name-replicaset uuid-pod uuid`

# Namespaces
kind of like DNS zones; let's you have two things with the same name in the same kubernetes cluster but for different purposes or different uses
could have a DEV and PROD namespace with different policies, like DEV takes every update, but PROD only takes tagged versions

created via YAML files
- apiVersion: v1
- kind: Namespace
- metadata:
  - name: <namespace name>

there is a "default" namespace that everything uses unless explicitly identified via the `--namespace=<namespace>` option
can also set the namespace for the current context

## Resource Quotas
allow resource limiting within a namespace
defined via yaml files
- apiVersion: v1
- kind: ResourceQuota
- metadata:
  - name: <name>
  - namespace: <namespace to limit>
- spec:
  - hard - likely different variations that generates different behavior when approaching the limit
    - pods - max number of pods in the namespace
    - requests.cpu - lower limit that a pod is guaranteed to have at all times, measured in CPU units; i.e., a CPU unit is a single core, so setting this to "1" means that it will always have one core but it may get more
    - requests.memory - same but for memory, measured in GB
    - limits.cpu - upper limit on CPU usage, measured in CPU units
    - limits.memory - same but for memory, measured in GB; i.e., `10Gi` sets a 10GB limit

