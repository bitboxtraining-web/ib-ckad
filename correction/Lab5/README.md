
# TP 5 : Volumes

## TP 5.1 : Shared volume

```shell
kubectl apply -f ~/workspaces/Lab5/pod--shared-emptyDir-vol--5.1.yml
#    pod/shared-vol-pod created
```

```shell
kubectl exec -it shared-vol-pod -c shell -- bash
> ls -l /data
#    total 0
#    -rw-r--r-- 1 root root 0 Feb 18 12:56 access.log
#    -rw-r--r-- 1 root root 0 Feb 18 12:56 error.log
> for i in $(seq 1 5); do curl -s localhost >/dev/null; done
> ls -l /data
#    total 4
#    -rw-r--r-- 1 root root 450 Feb 18 12:57 access.log
#    -rw-r--r-- 1 root root   0 Feb 18 12:56 error.log
```

## TP 5.2 : HostPath volume

### Initialisation

```shell
kubectl apply -f ~/workspaces/corrections/Lab5/pod--shared-hostPath.yml
#    pod/shared-hostpath-vol-pod created
```

### Check

```shell
kubectl exec shared-hostpath-vol-pod -c shell -- sh -c 'for i in $(seq 1 5); do curl -s localhost >/dev/null; done'
```

```shell
NODE_NAME=$(kubectl get pod shared-hostpath-vol-pod -o jsonpath='{.spec.nodeName}')
echo ${NODE_NAME}
#    minikube-m02

minikube ssh --node ${NODE_NAME} -- ls -l /mnt/sda1/data/var-log-nginx
#    total 4
#    -rw-r--r-- 1 root root 450 Feb 18 12:57 access.log
#    -rw-r--r-- 1 root root   0 Feb 18 12:57 error.log

minikube ssh --node ${NODE_NAME} -- cat /mnt/sda1/data/var-log-nginx/access.log
#    127.0.0.1 - - [18/Feb/2024:12:57:18 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.88.1" "-"
#    127.0.0.1 - - [18/Feb/2024:12:57:18 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.88.1" "-"
#    127.0.0.1 - - [18/Feb/2024:12:57:18 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.88.1" "-"
#    127.0.0.1 - - [18/Feb/2024:12:57:18 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.88.1" "-"
#    127.0.0.1 - - [18/Feb/2024:12:57:18 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.88.1" "-"
```

## TP 5.3 : PVC and PV

### Pre-requisites

```shell
minikube addons enable csi-hostpath-driver
#    💡  csi-hostpath-driver is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
#    You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
#    ❗  [WARNING] For full functionality, the 'csi-hostpath-driver' addon requires the 'volumesnapshots' addon to be enabled.
#
#    You can enable 'volumesnapshots' addon by running: 'minikube addons enable volumesnapshots'
#
#        ▪ Using image registry.k8s.io/sig-storage/csi-snapshotter:v6.1.0
#        ▪ Using image registry.k8s.io/sig-storage/csi-provisioner:v3.3.0
#        ▪ Using image registry.k8s.io/sig-storage/csi-attacher:v4.0.0
#        ▪ Using image registry.k8s.io/sig-storage/csi-external-health-monitor-controller:v0.7.0
#        ▪ Using image registry.k8s.io/sig-storage/csi-node-driver-registrar:v2.6.0
#        ▪ Using image registry.k8s.io/sig-storage/hostpathplugin:v1.9.0
#        ▪ Using image registry.k8s.io/sig-storage/livenessprobe:v2.8.0
#        ▪ Using image registry.k8s.io/sig-storage/csi-resizer:v1.6.0
#    🔎  Verifying csi-hostpath-driver addon...
#    🌟  The 'csi-hostpath-driver' addon is enabled

kubectl annotate storageclass standard storageclass.kubernetes.io/is-default-class-
#    storageclass.storage.k8s.io/standard annotated
kubectl annotate storageclass csi-hostpath-sc storageclass.kubernetes.io/is-default-class=true
#    storageclass.storage.k8s.io/csi-hostpath-sc annotated
```

### Pre-provisioning a PV

```shell
kubectl apply -f ~/workspaces/Lab5/pv--my-hostpath-pv--5.3.yml
#    persistentvolume/my-hostpath-pv created
```

```shell
kubectl get pv
#    NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
#    my-hostpath-pv   1Gi        RWO,ROX        Retain           Available                          <unset>                          3s
```

### Creating a PersistentVolumeClaim

```shell
kubectl apply -f ~/workspaces/Lab5/pvc--firstclaim--5.3.yml
#    persistentvolumeclaim/my-first-claim created
```

```shell
kubectl get pvc
#    NAME             STATUS   VOLUME           CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
#    my-first-claim   Bound    my-hostpath-pv   1Gi        RWO,ROX                       <unset>                 3s
```

```shell
kubectl get pv
#    NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
#    my-hostpath-pv   1Gi        RWO,ROX        Retain           Bound    default/my-first-claim                  <unset>                          49s
```

The capacity of the PVC is the one of the bound PV, i.e. 1Gi, even if only 500Mi were requested.

### Using the PVC

```shell
kubectl apply -f ~/workspaces/corrections/Lab5/pod--use-pvc.yml
#    pod/pod-pv created
```

### Check

```shell
kubectl exec pod-pv -c shell -- sh -c 'for i in $(seq 1 5); do curl -s localhost >/dev/null; done'
```

```shell
NODE_NAME=$(kubectl get pod pod-pv -o jsonpath='{.spec.nodeName}')
echo ${NODE_NAME}
#    minikube-m03

export PV_PATH=$(kubectl get pv my-hostpath-pv -o go-template --template '{{.spec.hostPath.path}}')
echo ${PV_PATH}
#    /data/pv/pv003

minikube ssh -n ${NODE_NAME} -- ls -l ${PV_PATH}
#    total 4
#    -rw-r--r-- 1 root root 450 Feb 18 13:21 access.log
#    -rw-r--r-- 1 root root   0 Feb 18 13:21 error.log
```

### Creating a second PVC

```shell
kubectl apply -f ~/workspaces/Lab5/pvc--second-claim--5.3.yml
#    persistentvolumeclaim/my-second-claim created
```

```shell
kubectl get pvc my-second-claim
#    NAME              STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
#    my-second-claim   Pending                                                     <unset>                 2
```

The PVC remains in __PENDING__ status because there is no PV available and it is not possible to create a PV dynamically with the empty StorageClass.

### Optional: Fix the situation

#### Option 1: Manually creating another PV

```shell
kubectl apply -f ~/workspaces/corrections/Lab5/pv--second-pv.yml
#    persistentvolume/second-pv created
```

#### Option 2: Use the default StorageClass

```shell
kubectl get storageclasses
#    NAME                        PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
#    csi-hostpath-sc (default)   hostpath.csi.k8s.io        Delete          Immediate           false                  8m25s
#    standard                    k8s.io/minikube-hostpath   Delete          Immediate           false                  3h51m
```

```shell
kubectl delete pvc my-second-claim
#    persistentvolumeclaim "my-second-claim" deleted

kubectl apply -f ~/workspaces/corrections/Lab5/pvc--second-claim-storageclass.yml
#    persistentvolumeclaim/my-second-claim created
```

```shell
kubectl get pvc my-second-claim
#    NAME              STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      VOLUMEATTRIBUTESCLASS   AGE
#    my-second-claim   Bound    pvc-a926ff60-cf4b-4252-8cf5-fb0db8c90653   500Mi      RWO            csi-hostpath-sc   <unset>                 12s

PV_NAME=$(kubectl get pvc my-second-claim --output jsonpath='{.spec.volumeName}')
echo ${PV_NAME}
#    pvc-a926ff60-cf4b-4252-8cf5-fb0db8c90653

kubectl get pv ${PV_NAME}
#    NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                     STORAGECLASS      VOLUMEATTRIBUTESCLASS   REASON   AGE
#    pvc-a926ff60-cf4b-4252-8cf5-fb0db8c90653   500Mi      RWO            Delete           Bound    default/my-second-claim   csi-hostpath-sc   <unset>                          41s

kubectl describe pv ${PV_NAME}
#    Name:              pvc-a926ff60-cf4b-4252-8cf5-fb0db8c90653
#    Labels:            <none>
#    Annotations:       pv.kubernetes.io/provisioned-by: hostpath.csi.k8s.io
#                       volume.kubernetes.io/provisioner-deletion-secret-name: 
#                       volume.kubernetes.io/provisioner-deletion-secret-namespace: 
#    Finalizers:        [kubernetes.io/pv-protection]
#    StorageClass:      csi-hostpath-sc
#    Status:            Bound
#    Claim:             default/my-second-claim
#    Reclaim Policy:    Delete
#    Access Modes:      RWO
#    VolumeMode:        Filesystem
#    Capacity:          500Mi
#    Node Affinity:
#      Required Terms:
#        Term 0:        topology.hostpath.csi/node in [minikube]
#    Message:
#    Source:
#        Type:              CSI (a Container Storage Interface (CSI) volume source)
#        Driver:            hostpath.csi.k8s.io
#        FSType:            
#        VolumeHandle:      e0497939-ce61-11ee-95f6-aad7033870d6
#        ReadOnly:          false
#        VolumeAttributes:      storage.kubernetes.io/csiProvisionerIdentity=1708262320356-8081-hostpath.csi.k8s.io-minikube
#    Events:                <none>
```

We see the creation of a dynamic PV with a __ReclaimPolicy__ `delete`.

```shell
kubectl delete pvc my-second-claim
#    persistentvolumeclaim "my-second-claim" deleted

kubectl get pv ${PV_NAME}
#    Error from server (NotFound): persistentvolumes "pvc-a926ff60-cf4b-4252-8cf5-fb0db8c90653" not found
```
