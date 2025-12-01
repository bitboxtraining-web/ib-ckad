# Lab 3: ReplicaSets

## Lab 3.1: ReplicaSets

### Clean up

```shell
kubectl delete pod --all
```

### First ReplicaSet

```shell
kubectl apply -f ~/workspaces/Lab3/rs--nginx-simple--3.1.yml
#    replicaset.apps/nginx created
```

```shell
kubectl get pod,rs --show-labels
```

```shell
kubectl delete $(kubectl get -o name pods -l app=nginx | head -1)
#    pod "nginx-8ss8q" deleted

kubectl get pod,rs --show-labels
```

We see that the deleted Pod has been replaced by a new one.

### ReplicaSet and Pod selector

```shell
kubectl apply -f ~/workspaces/Lab3/rs--nginx-cannot-create--3.1.yml
#    The ReplicaSet "frontend" is invalid: spec.template.metadata.labels: Invalid value: map[string]string{"app":"frontend", "level":"novice"}: `selector` does not match template `labels`
```

The ReplicaSet will create Pods with the label `level=novice` while it handles Pods with label `level=intermediate`

```shell
kubectl apply -f ~/workspaces/corrections/Lab3/rs--nginx-no-selector-fixed.yml
#    replicaset.apps/frontend created

kubectl get rs frontend
#    NAME       DESIRED   CURRENT   READY   AGE
#    frontend   2         2         2       47s
```

### ReplicaSet modifications

```shell
kubectl apply -f ~/workspaces/corrections/Lab3/rs--nginx-no-selector-avec-modifs.yml
#    replicaset.apps/frontend configured
```

```shell
kubectl get pods -l app=frontend
#    NAME             READY   STATUS    RESTARTS   AGE
#    frontend-4h6ql   1/1     Running   0          96s
#    frontend-cd4wt   1/1     Running   0          96s
```

```shell
kubectl label pods -l app=frontend level=advanced --overwrite
#    pod/frontend-4h6ql labeled
#    pod/frontend-cd4wt labeled
```

```shell
kubectl get pods -l app=frontend -L app,level
#    NAME             READY   STATUS    RESTARTS   AGE     APP        LEVEL
#    frontend-4h6ql   1/1     Running   0          2m15s   frontend   advanced
#    frontend-7tq7s   1/1     Running   0          14s     frontend   novice
#    frontend-b8vz5   1/1     Running   0          14s     frontend   novice
#    frontend-cd4wt   1/1     Running   0          2m15s   frontend   advanced
```

We notice that 2 new Pods frontend were created, because there were not enough Pods matching the ReplicaSet selector.

```shell
kubectl get pods -l app=frontend,level=novice -o json | jq .items[].spec.containers[0].image
#    "nginx:stable-alpine"
#    "nginx:stable-alpine"
```

### Scaling a ReplicaSet

```shell
kubectl scale --replicas=3 rs frontend
#    replicaset.apps/frontend scaled
```

```shell
kubectl apply -f ~/workspaces/corrections/Lab3/rs--nginx-no-selector-scaled.yml
#    replicaset.apps/frontend configured
```

```shell
kubectl describe $(kubectl get pod -l app=frontend,level=novice -o name | head -1) | grep 'Controlled By'
#    Controlled By:  ReplicaSet/frontend
```

```shell
kubectl get $(kubectl get pod -l app=frontend,level=novice -o name | head -1) -o json | jq .metadata.ownerReferences
#    [
#      {
#        "apiVersion": "apps/v1",
#        "blockOwnerDeletion": true,
#        "controller": true,
#        "kind": "ReplicaSet",
#        "name": "frontend",
#        "uid": "adf129f6-5318-4369-b293-b7fcc6c8c6e3"
#      }
#    ]
```

### ReplicaSet deletion

```shell
kubectl delete rs nginx
#    replicaset.apps "nginx" deleted
```

```shell
kubectl delete rs frontend --cascade=orphan
#    replicaset.apps "frontend" deleted
```

```shell
kubectl describe $(kubectl get pod -l app=frontend,level=novice -o name | head -1) | grep 'Controlled By'
#
```

```shell
kubectl get $(kubectl get pod -l app=frontend,level=novice -o name | head -1) -o json | jq .metadata.ownerReferences
#    null
```

### Migrate Pods from a ReplicaSet to another

There is no more ReplicaSet and we have orphaned Pods.

```shell
kubectl get pods -l app=frontend -L app,level
#    NAME             READY   STATUS    RESTARTS   AGE     APP        LEVEL
#    frontend-4h6ql   1/1     Running   0          3m42s   frontend   advanced
#    frontend-7tq7s   1/1     Running   0          101s    frontend   novice
#    frontend-b8vz5   1/1     Running   0          101s    frontend   novice
#    frontend-cd4wt   1/1     Running   0          3m42s   frontend   advanced
#    frontend-k8q5p   1/1     Running   0          49s     frontend   novice
#    frontend-x48wr   1/1     Running   0          55s     frontend   novice
```

```shell
kubectl get rs
#    No resources found in default namespace.
```

Creation of a new ReplicaSet :

```shell
kubectl apply -f ~/workspaces/corrections/Lab3/rs--frontend-rebirth.yml
#    replicaset.apps/frontend-rebirth created
```

```shell
kubectl get rs frontend-rebirth
#    NAME               DESIRED   CURRENT   READY   AGE
#    frontend-rebirth   2         2         2       4s
```

We notice that no new Pods were created and that the ReplicaSet even deleted Pods in excess.

```shell
kubectl get pod -l app=frontend,level=novice
#    NAME             READY   STATUS    RESTARTS   AGE
#    frontend-7tq7s   1/1     Running   0          2m20s
#    frontend-x48wr   1/1     Running   0          94s

kubectl describe $(kubectl get pod -l app=frontend,level=novice -o name | head -1) | grep 'Controlled By'
#    Controlled By:  ReplicaSet/frontend-rebirth

kubectl get $(kubectl get pod -l app=frontend,level=novice -o name | head -1) -o json | jq .metadata.ownerReferences
#    [
#      {
#        "apiVersion": "apps/v1",
#        "blockOwnerDeletion": true,
#        "controller": true,
#        "kind": "ReplicaSet",
#        "name": "frontend-rebirth",
#        "uid": "256dcd10-2295-423f-b289-39462d4cdba9"
#      }
#    ]
```
## Lab 3.2: DaemonSets

### Setting up the label on the nodes

```shell
kubectl label nodes --all disk=ssd
#   node/minikube labeled
#   node/minikube-m02 labeled
#   node/minikube-m03 labeled
```

### First DaemonSet

```shell
kubectl apply -f ~/workspaces/Lab3/ds-ssd-tp-3.2.yml
#    daemonset.apps/ssd-monitor created
```

```shell
kubectl get ds
#    NAME          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
#    ssd-monitor   3         3         3       3            3           disk=ssd        4s
```

```shell
kubectl get pods -l app=ssd-monitor -o wide
#    NAME                READY   STATUS    RESTARTS   AGE   IP            NODE           NOMINATED NODE   READINESS GATES
#    ssd-monitor-67649   1/1     Running   0          20s   10.244.1.12   minikube-m02   <none>           <none>
#    ssd-monitor-fccdn   1/1     Running   0          20s   10.244.2.14   minikube-m03   <none>           <none>
#    ssd-monitor-fwsdg   1/1     Running   0          20s   10.244.0.7    minikube       <none>           <none>
```

We notice that the DaemonSet has 3 Pods, one for each node.

```shell
kubectl logs -l app=ssd-monitor --prefix --tail 10
```

```shell
kubectl delete $(kubectl get pods -l app=ssd-monitor -o name | head -1) --now
#    pod "ssd-monitor-67649" deleted

kubectl get pods -l app=ssd-monitor
#    NAME                READY   STATUS    RESTARTS   AGE
#    ssd-monitor-fccdn   1/1     Running   0          70s
#    ssd-monitor-fwsdg   1/1     Running   0          70s
#    ssd-monitor-rdbg8   1/1     Running   0          10s
```

### Changing the Node label

```shell
kubectl label $(kubectl get nodes -l disk -o name | head -1) disk=hdd --overwrite
#    node/minikube labeled
```

```shell
kubectl get ds
#    NAME          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
#    ssd-monitor   2         2         2       2            2           disk=ssd        87s

kubectl get pods -l app=ssd-monitor
#    NAME                READY   STATUS    RESTARTS   AGE
#    ssd-monitor-fccdn   1/1     Running   0          108s
#    ssd-monitor-rdbg8   1/1     Running   0          48s
```

The Pod is removed from the node that doesn't match the label.
