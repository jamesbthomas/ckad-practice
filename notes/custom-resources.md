# Custom Resources

## Resources
basically any kind of object in kubernetes, including deployments, replicasets, etc.
## Controllers
the other half of the resource-controller paradigm that makes kubernetes work
uses a "monitor" to watch the resource and ensure it's performing in line with it's specifications
watches etcd to keep an eye on the current state of the resource
generally written in Go and is part of the kubernetes source code
## ETCD
the key-value data store in the middle that lets resources report their status and the controllers monitor their resources using a standardized API and format

## Custom Resource
extensible component of kubernetes that allows us to define our own thing for Kubernetes to orchestrate
have to define the resource and controller for it to work - resource is the what, the controller is the what to do about it
### Custom Resource Definition
can make basically whatever you want in the instance definition yaml, but first have to register a new value for the "kind" field
CRD registers the new "kind" value
```
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: <name of the resource>
spec:
  scope: <Namespaced or not>
  groups: <group name> ## API group to put this in
  names:
    kind: <> # goes into the kind field in a definition file
    singular: <> # used to display resource type when using kubectl commands
    plural: <> # whats used by the ap resources command
    shortNames:
    - <short>
  versions:
  - name: v1 ## Follow API lifecycle versioning
    served: <true | false> ## defines whether or not it's offered by the API server
    storage: <true | false> ## only one version can be true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              <prop name>:
                type: <prop type>
              <prop name>:
                type: <prop type>
                ## different types have different additional parameters to add, like integer has minimum and maximum        
```

just creates and stores the resource in etcd; doesn't do anything with it
### Controller
watches the resources and performs specified actions based on the data in etcd
example/starter for building a custom controller written in Go at https://github.com/kubernetes/sample-controller
once you have your logic, you can package it into a container and then deploy it as a pod for your cluster
default controllers all run inside the existing kubernetes pods; custom ones need to have their own pods created

# Operator Framework
mechanism for deploying CRDs and controllers as a single object
etcd is an example of an operator in this framework
- CRD that defines the cluster, a backup, and a restore object
- controllers that add data to the cluster once initiated, backup data, and restore from backup
- all under a single operator
operatorhub.io - public repository for operators for popular applications
