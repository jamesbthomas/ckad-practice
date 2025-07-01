# YAML

Notes on YAML files

## Structure

YAML is a structured mark-up language, much like XML. However, it relies on whitespace for parsing instead of strict open and close elements.

YAML files have the `.yml` file type, and every Kubernetes definition file has four root elements.
- `apiVersion` - specifies the version of the Kubernetes API to use to create the object
- `kind` - specifies what this YAML file creates
- `metadata` - an object or dictionary that allows you to name the pod and apply a label to it for easier management later on
- `spec` - 

The four root elements are required in every definition file.

### apiVersion
Possible values include v1, apps/v1, v1beta, etc.
### kind
Possible values include POD, Service, ReplicaSet, Deployment.
### metadata
Is an object as opposed to kind and apiVersion which are strings
Includes child items that must be indented under metadata - the number of spaces doesn't matter, but they must be the same.

for example, the below are all okay
```
metadata:
  name:
  labels:

metadata:
    name:
    labels:
```
but the below are not
```
metadata:
name:
labels:

metadata:
  name:
    labels:
```

Under `labels` you can define any combination of keys and values that will be valuable to identify the pod in the future.
### spec
The specification section, which provides additional information relevant to the object being created, and is different for each kind of object

For example, when creating a single pod, spec would look like this
```
spec:
  containers:
    - name: <what to name the container>
      image: <the name of the image to use in the container repository>
```
The containers label is a dictionary, so creating a pod with multiple containers just means adding another set of name and image tags. Be sure to include the `-`, which indicates that it's a new dictionary item and not a continuation of the previous one
