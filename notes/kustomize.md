# Kustomize

solves the problem of sending the same deployment to different environments
i.e., on local machine doing dev work, only 1 replica, but in prod you want more
could have a different YAML file for each environment - duplicating multiple config options, but changing the couple that need to be different in each environment
-  not optimal and difficult to scale, but otherwise its fine

underlying problem: how do we reuse existing configs and only change the parts that need to change?

builtin with kubectl
- may want to install directly since latest version may not be shipped
doesn't require templating languages
- can get hard to read
only uses plain yaml
- no special syntax or language
## Base Config
the lowest level config that is identical across all environments
default values, consistent config options, etc.
## Overlays
customizes behavior on a per-environment basis
only put the props/parameters that you want to change in the overlay definition file
overlays folder will include patch yaml files that apply patches based on the environment you're in
difference between a patch and an overlay is the `bases` root item
```
bases:
  - ../../base # relative path to the base directory which should have a kustomization.yaml
patch: |-
  ... # Patch data
```
can also add new configs in an overlay folder
- i.e., only deploy grafana in the production environment

## Folder Structure

k8s/
- base/ ## shared or default configs across all environments
  - kustomization.yaml
  - deployment.yaml  ## standard deployment definition
  - service.yaml  ## standard service definition
- overlays/  ## environment-specific configs
  - dev/
    - kustomization.yaml
    - config-map.yaml  ## standard config-map definition
  - prod/
    - kustomization.yaml
    - config-map.yaml

# Kustomize vs Helm
both modify definition files with environment-specific values

## Helm - more complex but more features
uses a Go-like template to assign variables to properties, and a separate file to define the values
different value definition, different environment, different end config

example folder structure
k8s/
- environments/
  - values.dev.yaml
  - values.prod.yaml
- templates/
  - deployment.yaml

pros:
- package manager in addition to configuration customization
- additional functionality like conditions, loops, functions, hooks
cons:
- not a standard part of kubernetes; have to install separately
- templates get complex because they're all variables
- not valid YAML

## Kustomize - easer and simpler

# Kustomization.yaml
have to create the file directly and have to name it kustomization.yaml

two sections:
- resources: list of all resources (yaml files) that kustomize will manage
- customizations or transformations that you want to change

example
```
apiVersion: kustomize.config.k8s.io/v1beta1  ## not required
kind: Kustomization  ## not required
resources:
  deployment-1.yaml
  service-1.yaml

commonLabels: ## can replace with other transformations, this one applies a value to all managed resources
  name: value
```

## Transformers
lots of built ins, and the ability to make your own transformers
transformers are applied at the folder level of the kustomization.yaml that includes them
### Common Transformers
applies a common configuration to all governed resources
looks like they all operate on the metadata section of a definition
- `commonLabels`: applies labels to metadata item for all objects
- `namePrefix` or `nameSuffix`: add a prefix or suffix to all names, just a plain append and prepend
- `namespace`: adds a common namespace to all resources
- `commonAnnotations`: adds an annotation to all resources; adds an annotation item under metadata
### Image Transformers
modifies an image used by a deployment or container
have to specify the name of the image to replace, and the newName to put in
example
```
images:
- name: <image to remove>
  newName: <image to add>
```
this is specifically the image name, not the container name
- `newName`: replaces the image name
- `newTag`: adds a tag to the image name, formed as `<name>:<tag>`
can use both together to replace the name and the tag 

## Building
use the `kustomize build <directory>` command, where the directory includes your definition files and the kustomization.yaml
imports the resources it is supposed to manage and applies the transformations
dumps the final configuration out to the terminal
- example above applies the name: value label to the metadata item
will put all resources together in the same yaml-formatted outputs, separated by a bunch of `-`

does NOT apply or create, just spits it to the terminal

## Output
have to redirect `kustomize build` to kubectl in order to get the configs applied
as simple as `kustomize build <directory> | kubectl apply -f -`
- pipes kustomize build to stdin for kubectl
- the `-` after `-f` tells kubectl to read from stdin

can also apply directly by pointing kubectl apply to the directory that has the kustomization.yaml in it, except us `-k` instead of `-f`

deleting is similar to applying, just replace apply with delete

## Multiple Directories
can list resources using relative paths and keep the kustomization.yaml at the root
can also nest kustomization.yaml files
- root kustomization.yaml specifies directories
- subordinate kustomization.yaml specifies the resources

## Patches
more targeted than transformers
includes three parameters
- operation type: add/remove/replace, more infrequent options
- target: what resource to apply it to
  - can match on Kind, apiVersion, name, namespace, or label or annotation selectors
- value: what value should be put in (as a result of add/replace operations)

example
```
patches:
  - target:
      kind: <what kind of object>
      name: <what the name of the object is>
      ...
    patch: |-  # called an inline patch
      - op: <operation>
        path: <hack-delimited path to the item to replace, i.e., /metadata/name>
        value: <value to leave behind>
```

### JSON 6902 vs Strategic Merge
6902 patch is the example above
- have to include the target and patch details
Strategic Merge patch includes defining valid YAML and only including the items that need to be changed

strategic merge example
```
patches:
- patch: |-
    apiVersion: <>
    kind: <>
    metadata:
      <>: <>
    spec:
      <>: <>
```
no issues using either one, just personal preference
when deleting via strategic merge formatting, have to include the `$patch: delete` directive at the same level as what you want to delete

### Inline vs Separate File
Inline (indicated by `|-`) is put in the kustomization.yaml
separate file you create another yaml file with your patches and reference the path in the kustomization.yaml

``` kustomization.yaml for 6902
patches:
- path: <path to patch.yaml>
  target:
    <>: <>
    <>: <>
```

``` patch.yaml for 6902
- op: <>
  path: <>
  value: <>
```

``` kustomization.yaml for strategic merge
patches:
- <path to patch.yaml>
```

``` patch.yaml for strategic merge
apiVersion: <>
kind: <>
metadata:
  <>: <>
spec:
  <>: <>
```

# Components
provide the ability to define reusable pieces of configuration logic (resources and patches) that can be used in multiple overlays
- i.e., you want the same features created in prod and staging, but not in dev and testing
Components stored in another folder along side the base yaml
i.e.:
```
/
 - base/
 - Components/
    <yaml files for components>
 - overlays/
```

in the appropriate overlays, have to import the components you need included
```
bases:
  - ...

components:
  - <relative path to component>
```
