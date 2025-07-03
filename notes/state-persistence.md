# State Persistence

# Volumes
## In Docker
containers meant to be transient; called upon when required and destroyed once finished
same for the data inside the container
persisting data means mounting a volume to the container and saving the data to that location
container gets deleted, data remains
## In K8s
same paradigm; containers meant to be transient

in a pod definition - great for single nodes
```
...
spec:
  containers:
  - ...
    volumeMounts:
    - mountPath: <path inside the container to mount>
      name: <name of the volume, should match the spec in the volumes item>
  volumes:
  - name: <name of the volume>
    hostPath:
      path: <path on the node to mount the volume to>
      type: <Directory|>
```

in a multinode cluster, generally need an external service to maintain the volume across multiple nodes
otherwise, everyone will write to their node and wont be able to see data from other containers on other nodes
see below for an example configuring an elastic block storage service in AWS
```
...
spec:
  ...
  volumes:
  - name: <name>
    awsElasticBlockStore:
      volumeID: <volume identifier>
      fsType: <file system type, like ext4>
```

# Persistent Volumes
persistent volumes allow for centralized storage
- cluster-wide pool of storage volumes to be used by pods deployed in the cluster
- users select storage from the pool using persistent volume claims

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: <volume name>
spec:
  accessModes:
  - <ReadWriteOnce|ReadOnlyMany|ReadWriteMany>
  capacity:
    storage: <amount of storage to retain for the volume, measured in gigibytes>
  hostPath: ## tells kubernetes to make storage on the local node part of this persistent volume
  ## can replace with a supported external service for multi-node persistent volumes
    path: <path on the node>
  persistentVolumeReclaimPolicy: <Retain|Delete|Recycle>
  
```

## Claims
claims are how the storage becomes available to nodes
Persistent Volume Claims (PVCs) are created by the user, while PVs are created by the administrator
- every claim is bound to a single volume with sufficient capacity
- filters on capacity, access modes, volume modes, storage class, and selectors
- claims may select a larger volume if it's the best match, but claims and volumes are 1:1 so nobody else can use that volume

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: <name>
spec:
  accessModes:
  - <list of access modes requested>
  resources:
    requests:
       storage: <amount of storage request, gigi or megibytes>
```

when claims are deleted, the default behavior is to retain the volume until deleted by an administrator
- can also set it to delete the volume so the storage will be available on the end storage device
- or recycle: data in the volume is scrubbed and the volume is made available again

claims are configured as objects, and then identified in the pod definition file, like so
```
...
spec:
  ...
  volumes:
  - name: <volume name>
    persistentVolumeClaim:
      claimName: <name of the pvc object>
```
this can also be added to ReplicaSets or Deployments in a similar manner
test