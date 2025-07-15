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
allow resource limiting within a namespace, defining max/min for all pods together in the namespace
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

# Resource Requests
used by the kube scheduler to find the right node to start a pod on
measured the same way as Resource Quotas, but specified differently in the yaml
- ResourceQuota is a kubernetes object that allows Resource Requests to be standardized across a deployment
despite the name, can only include upper limits
```
spec:
  containers:
    resources:
      requests:
        memory: "<memory request>" # measured in bytes
        cpu: <num> # measured in CPU units aka CPU cores
      limits:
        memory:
        cpu:
```

CPU is hard limited - pods can never use more CPU than they are assigned
Memory is soft limited - pods can use more memory than they are assigned, but will crash and run out of memory
requests = limits if requests are not specified
recommended approach is to use requests but not limits - make sure there's enough for everyone, but don't let anything go unused

## CPU units
as low as 0.1 - aka 100m or 100 milliunits
1 CPU unit is equivalent to 1 vCPU in AWS, 1 core in GCP, 1 core in Azure, 1 hyperthread (in other ecosystems)
## Memory units
measured in bytes, both gigi and giga
gigabytes - rounded, equal to 1 trillion bytes; indicated by the unit G
gibibytes - not rounded, equal to 1 trillion, 73 million 741 thousand eight hundred and 24 bytes; indicated by Gi
similar metrics for Mega/Mebibytes and Kilo/Kibibytes

# LimitRange
kubernetes object that works at the namespace level that provides default limits and default requests
limits are enforced when the pods are created - so if you change the limits have to redeploy the pods

```
apiVersion: v1
kind: LimitRange
metadata:
  name: <object name>
spec:
  limits:
  - default:
      cpu:
      memory:
    defaultRequest:
      cpu:
      memory:
    max:
      cpu:
      memory:
    min:
      cpu:
      memory:
    type: Container
```

# ConfigMaps
used to track Key Value Pairs across pods from a central location instead of in each pods definition file
can create imperatively or declaratively
## Imperative Definition
`kubectl create configmap <name of the config map> --from-literal=<name>=<value>`
repeat the --from-literal section multiple times if necessary
can also use `--from-file=<path>` to create it imperatively from a file
## Declarative Definition
uses a definition file iin YAML
- apiVersion: v1
- kind: ConfigMap
- metadata:
  - name: <name for the map>
- data - specifies all of the key value pairs you want added to the configmap
  - <name> : <value>

# Secrets
like config maps, but for data that you don't want in plaintext
- not a secure solution, but kubernetes handles secrets more securely than configmaps
doesn't actually secure the data, as it just base64 encodes it
other projects and services have tools to manage secrets more securely
- HashiCorp Vault or AWS Secrets Manager
- the Sealed Secrets project
- encrypting the filesystem and applying RBAC to the etcd data store (see kubernetes documentation for encryption at rest in etcd)
can be created imperatively or declaratively
when mounted as a volume, the key-value pairs are translated to file names as the key and contents as the value
anyone with access to create pods/deployments/ReplicaSets in a namespace can view the secrets
all values in a declaratively defined secret must be base64 encoded
## Imperative
same format, but instead of create configmap you use create secret generic
## Declarative
definition file with the same format, but instead of storing plaintext values you generate base64-encoded representations of the values
## Linux Encoding
`echo -n 'value' | base64 `
use the `--decode` option to the base64 module to decode the value

# Security Context
implements key security/privilege management features for containers, but in a way that takes advantage of centralized management and orchestration
can be set at the pod level
```
spec:
  securityContext:
    runAsUser: <uid to run as>
  containers:
    ...
```
or at the container level
```
spec:
  containers:
    - securityContext:
        runAsUser: <uid to run as>
        capabilities:
          add: ["<linux user capability>"]
      ...
```
The capabilities item is only supported at the container level,not at the pod level

# Service Accounts
accounts used by machines
creates a service account object, then creates a token and stores it in a secret object, then associates that secret object with the service account object
once created, you can copy the token out and use a third party application to access the kubernetes API
can also mount the secret directly to containers in the cluster

# Taints and Tolerations
used to specify how pods are applied to nodes
- taint: a repulsive characteristic set on nodes
- toleration: ability to ignore certain taints placed on pods
two part implementation
- applying taints to nodes prevents any hosts without the appropriate toleration from being scheduled on that node
- if no tolerations applied, no pods get put on tainted nodes
Doesn't ensure a pod will be placed on a specific node, just protects a node for a specific app.
## Tainting
imperative only
`kubectl taint nodes node-name key=value:taint-effect`
taint-effect tells the scheduler how to handle pods that aren't tolerant to that taint
- NoSchedule: prevents pods from being scheduled on the tainted host
- PreferNoSchedule: system will try to avoid scheduling on the tainted node, but not guaranteed
- NoExecute: new pods will not be scheduled, but existing pods will be evicted if they don't tolerate the taint
  - when re-scheduling in response to a NoExecute taint, pods are killed and then rescheduled if their pod definition requires it
## Tolerating
declarative only
new spec sub-element called tolerations, each item includes a key, operator, value, and effect
```
spec:
  tolerations:
  - key: "<name of the toleration, should match the taint command>"
    operator: "="
    value: "<value from the taint command>"
    effect: "<one of the effect values>"
```

quotes around all 4 values are required

# Node Affinity
inverse of taints and tolerations which protects nodes for certain pods, node affinity protects pods for certain nodes
more complex than node selectors, but also provides more powerful selections
## Pod Definition
still a spec sub-element
```
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: <label key>
            operator: <kind of operator>
            values:
            - <list of values>
```
multiple operators, but most common is "In"
- can also do "NotIn"
- "Exists" - doesn't need the values item
can add additional values to match multiple labels, include the `-` to denote that it's a new element in the array
## Types of Node Affinity
two types in use now, a third planned
- requiredDuringSchedulingIgnoredDuringExecution:
  - during scheduling, before pod starts, the pod can only be put on nodes with matching affinity. will not schedule pod without matching node affinity
  - ignored after scheduling, once the pod is running, so if the node affinity changes the pod will keep running
- preferredDuringSchedulingIgnoredDuringExecution
  - before pod starts, the pod will prefer to be put on matching affinity nodes, but will put the pod in execution on any node if it cant find a matching affinity
  - 
- (planned) requiredDuringSchedulingRequiredDuringExecution
  - requiredDuringExecution will evict pods on any node whenever affinity changes, even if the pod is already running
# Node Selectors
older technique replaced by node affinity
uses labels assigned to the nodes and in the pod definition to match pods to the nodes that should run them
have to label nodes first, then create the pod
only allows for simple matchings - i.e., can't do label1 OR label2, can't do NOT label3
## Label Node
`kubectl label nodes <node name> <label key>=<label value>`
key should be size, value can be whatever you want as long as it matches your pod definition
## Create Pod
another spec sub-element
```
spec:
  nodeSelector:
    size: <label value>
```
