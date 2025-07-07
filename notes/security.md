# Security

# Authentication
end users is all handled by the applications running inside the pods, so focus is admins, developers, and service accounts
kubernetes can't manage admins/developers; they rely on third-party tools
service accounts can be maintained in k8s
## User accounts
everything goes through the kube-apiserver which authenticates the request before performing the service
### Auth Mechanisms
- Static password file: i.e., csv file that has the password, username, and userid. optionally fourth column for groups
  - added to the kube-apiserver via the `--basic-auth-file=<file name>` command option
- Static token file: instead of passwords, has tokens (similar to authorization bearer tokens)
- Certificates
- Identity services (like LDAP)

# KUBECONFIG
configuration file that includes command options that can be added en masse to a kubectl command
by default, every kubectl command will look for $HOME/.kube/config and add that to every command
useful for environments with multiple clusters, or that are using cert based authentication so you don't have to type everything out every time
## Format
three sections:
- Clusters: dev/prod, different providers, etc.
  - specifies the server/cluster to use
- Contexts: assign user accounts to clusters
- Users: user accounts that have access to each cluster
  - provides the keys, certificates, root CA, etc.

doesn't create any new users or grant privileges to the cluster; just tells kubectl which account to use when going to each cluster
all root elements are array-formatted
don't have to create this, just leave it as is and kubectl will read it an invocation time

current-context specifies the default context to use
specify non-default kubeconfig with `kubectl config view --kubeconfig=<config file>`
change current context with `kubectl config use-context <context name>`
```
apiVersion: v1
kind: Config
current-context: <context name>
clusters:
- name: <cluster name>
  cluster:
    certificate-authority: <ca.crt file> ##optionally replace with certificate-authority-data and base64 encoded cert data
    server: https://<cluster kube-apiserver>
contexts:
- name: <username>@<cluster name>
  context:
    cluster: <cluster name>
    user: <user name>
    namespace: <namespace> ##optional, sets the default namespace for a specific context
users:
- name: <user name>
  user:
    client-certificate: <user cert file>
    client-key: <user private key>
```

# API Groups
api is split into several groups based on purpose:
- /version: checks the version of kubernetes running on the cluster
- /metrics: provides point-in-time performance metrics
- /healthz: point-in-time health metrics
- /logs: integration with third-party logging
- /api: core; includes all core functionality exists, like pods, namespaces, PVs, PVCs, configmaps, etc.
- /apis: named; more organized, apps, extensions, networking.k8s.io, /storage.k8s.io, /authentication.k8s.io
  - future state for the api

API documentation at kubernetes.io/docs/reference/generate/kubernetes-api will call out the API group for each function at the top of the page
Can also view locally by curling the cluster directly on port 6443 with no additional paths; returns complete list of paths
- need to specify the cred/cert to use to access the cluster using curl, or stand up the proxy service `kubectl proxy` and access it at port 8001 and it will put your kubeconfig on all commands
### structure
Group (apis) > Group (/apps) > /v1 > Resource (/deployments) > Verb (get)

# Authorization
determines what a user can do after they've gained access
## Mechanisms
### Node-based authorization
kubelet acts as the API server on the node and reports node status to the Kube API server
- handled by the Node Authorizer
### Attribute-based authorization (ABAC)
not really ABAC, more of a per-user auth mechanism
defined in a JSON-formatted policy file like:
```
{"kind": "Policy",
"spec": {
    "user": "<user name>",
    "namespace": "<namespace name or *>",
    "resource": "<resource name, like pods>",
    "apiGroup": "<api group name, or *>"
}}
```
different policy file for each user
have to manually edit file and restart kube-apiserver
### Role-based authorization (RBAC)
maps users to preset roles, and preset roles to permissions
roles made declaratively 
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: <role name>
  namespace: <restrict role to a specific ns> ## Optional
rules:
- apiGroups: ["<api group name>"] ## leave blank for core groups
  resources: ["<resource 1>","<resource 2>"] ## types of resources like pods
  resourceNames: ["<resource name 1>","<resource name 2>"] ## names of specific resources, like the name of a deployment in the cluster
  verbs: ["<verb 1>","<verb 2>","<verb 3>"]
## can repeat these three sub elements for multiple combinations
```
roles linked to users declaratively via RoleBindings
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: <binding name>
  namespace: <restrict role to a specific ns> ## Optional
subjects:
- kind: User
  name: <user name>
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: <role name>
  apiGroup: rbac.authorization.k8s.io
```
### Webhook
used to offload authorization decisions to a third party, like Open Policy Agent
kube api provides user details and access requested, third party returns a decision
### AlwaysAllow
performs no checks and lets all requests go through
### AlwaysDeny
performs no checks and lets no requests go through
## Configuration
default is AlwaysAllow
specify which mechanisms to use as options in the kube-apiserver command as comma-separated list
request is authorized in specified order, so Node,RBAC, Webhook goes:
- Node tries to authorize you, you're not configured so it says no
- Node forwards to RBAC module, which sees you're configured and says yes and calls it a day
failed checks forward, passed checks exit
AlwaysAllow/Deny useful for fail open and explicit deny all

## Roles vs Cluster Roles
Roles are normally scoped by namespace
Cluster Roles are used to manage cluster-wide resources, like nodes which can't be scoped down to a single name space
- nodes, PVs, namespaces, clusterroles and their bindings, etc.

still made declaratively, like so
the role:
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: <role name>
rules:
- apiGroups: [""]
  resources: [""] ## must be cluster-scoped
  verbs: [""]
```
and the binding:
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: <binding name>
subjects:
- kind: User
  name: <user name>
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: <role name>
  apiGroup: rbac.authorization.k8s.io
```
you can create clusterroles with namespace-scopes resources if you want to give a role access to a resource across all namespaces

# Admission Controllers
next step after Authorization
allows you to inspect definition files to validate configuration, change requests, and perform other actions
pre-built:
- AlwaysPullImages: ensures every pod is created from a non-cached image
- DefaultStorageClass: observes creation of PVCs and adds a default storage class if not set
- EventRateLimit: limits the number of requests the API Server can handle at a time to prevent flooding
- NamespaceExists: rejects requests to non-existent namespaces
- NamespaceAutoProvision: not enabled by default, but automatically creates a namespace if it doesn't exist

to add new ones, update the command flag for `enable-admission-plugins` via /etc/kubernetes/manifests/kube-apiserver.yaml
- similar for disabling
## Validating Controllers
type of admission controller that performs a check/validation and returns a result
generally invoked second, after mutating controllers so that any changes can also be validated
## Mutating Controllers
type of admission controller that checks something, and changes the operation based on pre-determined conditions
generally invoked first, before validating controllers
## Custom Controllers
leverage the MutatingAdmissionWebHook and ValidatingAdmissionWebHook controllers
they run last of all of the predefined controllers
passes an AdmissionReview object which includes details on the object, user, and operation they are trying to perform
the webhook server returns an AdmissionReview object which states if the operation is allowed or not
### Deploy a webhook server
first step, basic API server
must accept the mutate and validate APIs and response with an expected JSON object
### Configure the webhook object


```
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration ## or MutatingWebhook...
metadata:
  name: <webhook name>
webhooks:
- name: <webhook name>
  clientConfig:
    url: <url path> ## If deployed as an external service
    service: ## if deployed on the same kubernetes cluster
      namespace: <namespace>
      name: <service name>
    caBundle: "<base64 ca bundle>"
  rules:
  - apiGroups: [""]
    apiVersions: [""]
    operations: [""]
    resources: [""]
    scope: "<Namespaced or Cluster>"
```