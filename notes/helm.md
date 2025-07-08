# Helm
kubernetes is good at managing complex infrastructures, but humans aren't always that good
setting up a single application requires a large number of objects, like a deployment for the pods, a PV and PVC for the persistent storage, a service to make it available and a secret to store the admin password, at a minimum
thats a bunch of different yaml files and kubectl apply to start
writing a single yaml file doesn't solve the problem either; organization would be tough in a 25-page text file

Helm changes the paradigm - kubernetes doesn't care about the app, it just makes objects
Helm is sometimes called a "package manager" for kubernetes because it looks at all of those objects as part of the same group
- we tell it what settings we want, and it does all of the underlying yaml files on its own
- one command to install, one command to upgrade, one to rollback, and one to remove
- one file to manage, and helm extrapolates out to the appropriate kubernetes primitives

# Installing Helm
## Prerequisites
must have a functioning kubernetes cluster
must have kubectl working locally with appropriate configs and secrets to manage the cluster
## Sources
available on snap, apt-get, pkg, based systems

# Concepts
## Templates
first step in the transition to helm; replace values with variables
identified by `{{ .Values.<variable name>}}`
values.yaml stores the variables like:
```
<variable name>: <variable value>
```
## Charts
key helm concept
A chart is the combination of templates and variables
also includes a chart.yaml file that includes details on the chart itself, like keywords, descriptions, versions, maintainers, etc
artifacthub.io is a public repo for helm charts, so you can use someone else's chart or build your own
- can also search via command line with `helm search hub <application name>`
- add new repos with `helm repo add <name> <url>`
- then replace `hub` with the name

### Installing
uses `helm install <release name> <chart name>`
- release name is a name that you decide; you can deploy the same chart multiple times with a different release name and then you get multiple instances of the same application


# Command References

## `helm pull --untar <chart name>`
pulls the chart from the remote repo and unpacks it, but doesn't install it