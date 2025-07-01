# Kubectl Notes

## Full commands
### `kubectl create -f <name of yaml>.yml`
- creates a new pod based on the provided yaml file
- `-f` option specifies the yaml file to use

### `kubectl get pods`
- gets the name, status, and age of pods that kubectl is managing

### `kubectl delete deployment <name>`
- gets rid of a created pod (running or not) with the provided name
- name should match what's in get pods

### `kubectl describe pod <name>`
- provides a bunch of details on the pod, including the containers in the pod, their images, the node running the pod, log messages, etc

### `kubectl get pod <name> -o yaml <file name>`
- gets the configuration of the selected pod and writes it to a YAML file

### `kubectl edit pod <name>`
- used to modify properties of the pod, but only portions of the spec can be edited
- spec.containers[x].image - changes the image used for the selected container, index starts at 0
- spec.initContainers[x].image
- spec.activeDeadlineSeconds
- spec.tolerations
- spec.terminationGracePeriodSeconds

### `kubectl replace -f <file name>`
same as create, but doesn't error if there's a pod with the definition's name already running

### `kubectl scale --replicas=<num> -f <file name>`
increases the number of replicas that the ReplicaSet will have
- can replace file name with `replicaset <set name>` for the same results
- none of this changes the contents of the yaml; only changes the running config

### `kubectl config set-context $(kubectl config current-context) --namespace=<namespace>`
sets the context name space to the given namespace
allows the use of kubectl commands against a non-default namespace without the use of the --namespace flag

## Switches
### `--namespace=<namespace>`
- also `-ns=<namespace>`
specifies a namespace for the command
### `-o <format>`
specifies additional columns to include in the output
formats:
- json - generates the output as JSON
- name - or another column from the output, limits to only the selected column
- wide - prints additional plaintext information
- yaml - generates the output as YAML; useful for creating a yaml spec from a running container
### `--dry-run=<location>`
useful when dealing with declarative commands (where actions are based on a given file)
useful locations:
- client - does nothing with the action; checks the command and file locally and tells you if its right and if you can create the resource

