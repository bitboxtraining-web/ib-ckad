
# Lab 3: ReplicaSets

## Lab 3.1: ReplicaSets

### Clean up

```shell
kubectl delete pod --all
#    pod "light-sleeper" deleted
#    pod "liveness-http" deleted
#    pod "rp-check" deleted
#    pod "training-shell" deleted
#    pod "whoami" deleted
#    pod "whoami-and-clock" deleted
#    pod "whoami-and-clock-with-init" deleted
#    pod "yaml-pod" deleted
```

### First ReplicaSet

```shell
kubectl apply -f ~/workspaces/Lab3/rs--nginx-simple--3.1.yml
#    replicaset.apps/nginx created
```

```shell
kubectl get pod,rs --show-labels
#    NAME              READY   STATUS    RESTARTS   AGE   LABELS
#    pod/nginx-8ss8q   1/1     Running   0          11s   app=nginx,level=novice
#    pod/nginx-dlkhg   1/1     Running   0          11s   app=nginx,level=novice
#    pod/nginx-zw5vm   1/1     Running   0          11s   app=nginx,level=novice
#
#    NAME                    DESIRED   CURRENT   READY   AGE   LABELS
#    replicaset.apps/nginx   3         3         3       11s   <none>
```

```shell
kubectl delete $(kubectl get -o name pods -l app=nginx | head -1)
#    pod "nginx-8ss8q" deleted

kubectl get pod,rs --show-labels
#    NAME              READY   STATUS    RESTARTS   AGE   LABELS
#    pod/nginx-dlkhg   1/1     Running   0          41s   app=nginx,level=novice
#    pod/nginx-gdsbx   1/1     Running   0          11s   app=nginx,level=novice
#    pod/nginx-zw5vm   1/1     Running   0          41s   app=nginx,level=novice
#
#    NAME                    DESIRED   CURRENT   READY   AGE   LABELS
#    replicaset.apps/nginx   3         3         3       41s   <none>
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
#    [pod/ssd-monitor-fwsdg/main] SSD OK
#    [pod/ssd-monitor-fwsdg/main] SSD OK
#    [pod/ssd-monitor-fwsdg/main] SSD OK
#    [pod/ssd-monitor-fwsdg/main] SSD OK
#    [pod/ssd-monitor-fwsdg/main] SSD OK
#    [pod/ssd-monitor-fwsdg/main] SSD OK
#    [pod/ssd-monitor-fwsdg/main] SSD OK
#    [pod/ssd-monitor-fwsdg/main] SSD OK
#    [pod/ssd-monitor-67649/main] SSD OK
#    [pod/ssd-monitor-67649/main] SSD OK
#    [pod/ssd-monitor-67649/main] SSD OK
#    [pod/ssd-monitor-67649/main] SSD OK
#    [pod/ssd-monitor-67649/main] SSD OK
#    [pod/ssd-monitor-67649/main] SSD OK
#    [pod/ssd-monitor-67649/main] SSD OK
#    [pod/ssd-monitor-67649/main] SSD OK
#    [pod/ssd-monitor-fccdn/main] SSD OK
#    [pod/ssd-monitor-fccdn/main] SSD OK
#    [pod/ssd-monitor-fccdn/main] SSD OK
#    [pod/ssd-monitor-fccdn/main] SSD OK
#    [pod/ssd-monitor-fccdn/main] SSD OK
#    [pod/ssd-monitor-fccdn/main] SSD OK
#    [pod/ssd-monitor-fccdn/main] SSD OK
#    [pod/ssd-monitor-fccdn/main] SSD OK
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

## Lab 3.3: Jobs and CronJobs

### First Job

```shell
kubectl apply -f ~/workspaces/Lab3/job--compute-pi--3.3.yml
#    job.batch/pi created
```

```shell
kubectl logs $(kubectl get pods -l job-name=pi -o name)
#    3.14159265358979323846264338327950288419716939937510582097494459230781640628620899862803482534
#    2117067982148086513282306647093844609550582231725359408128481117450284102701938521105559644622
#    [...]
```

```shell
kubectl describe job pi
#    Name:             pi
#    Namespace:        default
#    Selector:         batch.kubernetes.io/controller-uid=26646ec1-2553-4829-bd14-17a068c33cf8
#    Labels:           batch.kubernetes.io/controller-uid=26646ec1-2553-4829-bd14-17a068c33cf8
#                      batch.kubernetes.io/job-name=pi
#                      controller-uid=26646ec1-2553-4829-bd14-17a068c33cf8
#                      job-name=pi
#    Annotations:      <none>
#    Parallelism:      1
#    Completions:      1
#    Completion Mode:  NonIndexed
#    Start Time:       Sun, 18 Feb 2024 11:00:55 +0000
#    Completed At:     Sun, 18 Feb 2024 11:01:50 +0000
#    Duration:         55s
#    Pods Statuses:    0 Active (0 Ready) / 1 Succeeded / 0 Failed
#    Pod Template:
#      Labels:  batch.kubernetes.io/controller-uid=26646ec1-2553-4829-bd14-17a068c33cf8
#               batch.kubernetes.io/job-name=pi
#               controller-uid=26646ec1-2553-4829-bd14-17a068c33cf8
#               job-name=pi
#      Containers:
#       pi:
#        Image:      perl:5.20-stretch
#        Port:       <none>
#        Host Port:  <none>
#        Command:
#          perl
#          -Mbignum=bpi
#          -wle
#          print bpi(2000)
#        Environment:  <none>
#        Mounts:       <none>
#      Volumes:        <none>
#    Events:
#      Type    Reason            Age   From            Message
#      ----    ------            ----  ----            -------
#      Normal  SuccessfulCreate  67s   job-controller  Created pod: pi-zc6vf
#      Normal  Completed         12s   job-controller  Job completed
```

### A bit of parallelism

```shell
kubectl apply -f ~/workspaces/corrections/Lab3/job--compute-pi-parallel.yml
#    The Job "pi" is invalid: spec.completions: Invalid value: 5: field is immutable

kubectl delete job pi
#    job.batch "pi" deleted

kubectl apply -f ~/workspaces/corrections/Lab3/job--compute-pi-parallel.yml
#    job.batch/pi created
```

### CronJob (Optional)

```shell
kubectl apply -f ~/workspaces/Lab3/cron--hello-from-k8s-cluster--3.3.yml
#    cronjob.batch/hello created
```

```shell
kubectl get cronjob
#    NAME    SCHEDULE      TIMEZONE   SUSPEND   ACTIVE   LAST SCHEDULE   AGE
#    hello   */1 * * * *   <none>     False     0        <none>          7s
```

```shell
kubectl get jobs
#    NAME             COMPLETIONS   DURATION   AGE
#    hello-28470904   1/1           4s         62s
#    hello-28470905   0/1           2s         2s
```

```shell
kubectl get cronjob
#    NAME    SCHEDULE      TIMEZONE   SUSPEND   ACTIVE   LAST SCHEDULE   AGE
#    hello   */1 * * * *   <none>     False     0        9s              2m53s

kubectl describe cronjob hello
#    Name:                          hello
#    Namespace:                     default
#    Labels:                        <none>
#    Annotations:                   <none>
#    Schedule:                      */1 * * * *
#    Concurrency Policy:            Allow
#    Suspend:                       False
#    Successful Job History Limit:  3
#    Failed Job History Limit:      1
#    Starting Deadline Seconds:     <unset>
#    Selector:                      <unset>
#    Parallelism:                   <unset>
#    Completions:                   <unset>
#    Pod Template:
#      Labels:  <none>
#      Containers:
#       hello:
#        Image:      debian:11-slim
#        Port:       <none>
#        Host Port:  <none>
#        Args:
#          bash
#          -c
#          date; echo Hello from the Kubernetes cluster
#        Environment:     <none>
#        Mounts:          <none>
#      Volumes:           <none>
#    Last Schedule Time:  Sun, 18 Feb 2024 11:06:00 +0000
#    Active Jobs:         hello-28470906
#    Events:
#      Type    Reason            Age   From                Message
#      ----    ------            ----  ----                -------
#      Normal  SuccessfulCreate  2m3s  cronjob-controller  Created job hello-28470904
#      Normal  SawCompletedJob   119s  cronjob-controller  Saw completed job: hello-28470904, status: Complete
#      Normal  SuccessfulCreate  63s   cronjob-controller  Created job hello-28470905
#      Normal  SawCompletedJob   60s   cronjob-controller  Saw completed job: hello-28470905, status: Complete
#      Normal  SuccessfulCreate  3s    cronjob-controller  Created job hello-28470906
```

```shell
kubectl patch cronjobs hello -p '{"spec" : {"suspend" : true }}'
#    cronjob.batch/hello patched
```
