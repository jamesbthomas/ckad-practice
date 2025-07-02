# Section 6 - Pod Design

# Labels and Selectors
## Labels
apply user-defined properties to pods
specified in the `metadata` section of a definition file under the `labels` sub element and key-value pairs
## Selectors
provides the logic specification that allows users to filter on defined properties
built with `<key>=<value>` prompts and the --selector flag in imperative commands
defined declaratively similar to ReplicaSets
```
spec:
  selector:
    matchLabels"
      <key>: <value>
```
## Use
`--selector` flag can filter any kubectl command to only match the provided labels

# Updates and Rollbacks in Deployments
Rollout - first time you create a deployment
- creates deployment revision1
when application container is upgraded, triggers and update
- creates deployment revision2
`kubectl rollout status <deployment name>`- shows current rollout status
`kubectl rollout history <deployment name>` - shows all revisions of select deployment
## Deployment strategy
### Recreate
traditional approach would be to delete all active instances and then recreate them with the newer version
- creates outage, and known as the recreate strategy; not default
### RollingUpdate
modern approach is to take containers down and recreate them one by one
- no outages, and known as the rolling update; default if not specified
in both, the deployment creates a new replicaset and deletes the old ones
### Blue/Green
can't be explicitly configured,  but still good to know
involves standing up both side-by-side, with testing on the new/green version and traffic going to the old/blue version
once tests are passed, traffic swaps over to green and blue gets deleted
#### Implementation
using kubernetes primitives
essentially setting up two deployments, one with a version=1 tag and one with a version=2 tag
the service uses a selector, and changing the selector to version=2 switches everything to the green deployment
### Canary
cant be explicitly configured
involves standing up a small deployment with the new version (the canary), that services some traffic in line with the old version
once confident that all tests pass, then the rest of the deployment can be updated
#### Implementation
using kubernetes primitives
two deployments
- primary with tag version=1
- canary with tag version=2
only want to route a small percentage of traffic to the canary deployment
1) create a common label (i.e., app=frontend) and update the selector to use it
- routes traffic to both, but routes to them equally
2) reduce the number of pods in the deployment to the absolute minimum
- service routes traffic equally across all *pods*, not all deployments, so a total of 6 pods will see ~17% of traffic to each pod
once tests are done and system is working, update the primary deployment and delete the canary deployment
other mechanisms, namely service meshes, provide more granular control of traffic routing that's not reliant on using a large number of containers
## Updating deployment
doesn't matter what you're doing on the underside, this is all in the deployment definition file
update the yaml, then use `kubectl apply -f <definition file>` to trigger a rollout
can also define imperatively via things like `kubectl set image <deployment name> <container name>=<container image>`
- creates differences between running application and definition baselines
## Rollback
reverses the upgrade; deletes new replicaset and recreates old replicaset
`kubectl rollout undo <deployment name>`

# Jobs
another kind of workload, similar to web services, databases, etc.
jobs are workloads designed to perform a task and then close, as opposed to offering a service over a longer period of time
when done in kubernetes, it will continuously restart the pod until it hits a specific threshold because its designed to make pods highly available using the default `restartPolicy: Always`
- if set to `Never` or `OnFailure`, will not restart a container that finishes its task
## Job Object
similar to replicaset, but creates enough containers until a task is complete instead of until a set limit is reached
```
apiVersion: batch/v1
kind: Job
metadata:
  name: <job object name>
spec:
  completions: <number of pods that need to end successfully>
  parallelism: <number of pods that can run at the same time>
  template:
    <pod definition for the job>
    spec:
      ...
      restartPolicy: <Never|Always|OnFailure>
```

## CronJobs
same name as crontab, same function as crontab
schedules Job objects
```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: <name>
spec:
  schedule: "<same schedule string as crontab>"
  jobTemplate:
    spec:
       <same spec as the Job object, including completions, parallelism, template, etc.>
```