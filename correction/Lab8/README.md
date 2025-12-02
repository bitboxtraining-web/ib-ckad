
# Lab 8: StatefulSets

## Lab 8.1: StatefulSets

```shell
kubectl apply -f ~/workspaces/Lab8/statefulset--db-cluster--8.1.yml
#    service/db-cluster-headless-svc created
#    statefulset.apps/db-cluster created
```

```shell
kubectl get pod,svc,sts,pvc -l app=db-cluster
#    NAME               READY   STATUS    RESTARTS   AGE
#    pod/db-cluster-0   1/1     Running   0          29s
#    pod/db-cluster-1   1/1     Running   0          18s
#
#    NAME                              TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
#    service/db-cluster-headless-svc   ClusterIP   None         <none>        8080/TCP   29s
#
#    NAME                          READY   AGE
#    statefulset.apps/db-cluster   2/2     29s
#
#    NAME                                      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      VOLUMEATTRIBUTESCLASS   AGE
#    persistentvolumeclaim/data-db-cluster-0   Bound    pvc-364457fe-4155-4aef-bf25-c0385c80c945   50Mi       RWO            csi-hostpath-sc   <unset>                 29s
#    persistentvolumeclaim/data-db-cluster-1   Bound    pvc-bfa61691-8f9c-4027-960b-5e988879eca1   50Mi       RWO            csi-hostpath-sc   <unset>                 18s
```

```shell
kubectl exec gateway -- curl -s db-cluster-0.db-cluster-headless-svc.default.svc.cluster.local:8080
#    DB cluster node: db-cluster-0
```

```shell
kubectl exec db-cluster-1 -- touch /data/i-was-here
```

```shell
kubectl scale sts db-cluster --replicas=1
#    statefulset.apps/db-cluster scaled
```

```shell
kubectl get pod db-cluster-1
#    NAME           READY   STATUS        RESTARTS   AGE
#    db-cluster-1   1/1     Terminating   0          11m

kubectl get pvc data-db-cluster-1
#    NAME                STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      VOLUMEATTRIBUTESCLASS   AGE
#    data-db-cluster-1   Bound    pvc-bfa61691-8f9c-4027-960b-5e988879eca1   50Mi       RWO            csi-hostpath-sc   <unset>                 5m31s
```

```shell
kubectl scale sts db-cluster --replicas=2
#    statefulset.apps/db-cluster scaled
```

```shell
kubectl get pod db-cluster-1
#    NAME           READY   STATUS    RESTARTS   AGE
#    db-cluster-1   1/1     Running   0          23s
```

```shell
kubectl exec db-cluster-1 -- ls -l /data/
#    total 0
#    -rw-r--r-- 1 root root 0 Jul  2 15:25 i-was-here
```

```shell
kubectl delete sts db-cluster
#    statefulset.apps "db-cluster" deleted
```

```shell
kubectl get pvc -l app=db-cluster
#    NAME                STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      VOLUMEATTRIBUTESCLASS   AGE
#    data-db-cluster-0   Bound    pvc-364457fe-4155-4aef-bf25-c0385c80c945   50Mi       RWO            csi-hostpath-sc   <unset>                 6m26s
#    data-db-cluster-1   Bound    pvc-bfa61691-8f9c-4027-960b-5e988879eca1   50Mi       RWO            csi-hostpath-sc   <unset>                 6m15s
```

```shell
kubectl delete pvc -l app=db-cluster
#    persistentvolumeclaim "data-db-cluster-0" deleted
#    persistentvolumeclaim "data-db-cluster-1" deleted
```
