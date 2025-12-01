
# Lab 7: Deployments

## Lab 7.1: Deployments

### Version1

```shell
kubectl apply -f ~/workspaces/Lab7/deploy--bitboxtraining-v1--7.1.yml --record
#    deployment.apps/bitboxtraining created
```

```shell
kubectl get deploy,rs,pod -l app=bitboxtraining
```




### Version 2

```shell
kubectl apply -f ~/workspaces/corrections/Lab7/deploy--bitboxtraining-v2.yml --record
#    deployment.apps/bitboxtraining configured
```
```shell
kubectl rollout status deployment bitboxtraining
#    Waiting for deployment "bitboxtraining" rollout to finish: 1 out of 3 new replicas have been updated...
#    Waiting for deployment "bitboxtraining" rollout to finish: 1 out of 3 new replicas have been updated...
#    Waiting for deployment "bitboxtraining" rollout to finish: 1 out of 3 new replicas have been updated...
#    Waiting for deployment "bitboxtraining" rollout to finish: 1 out of 3 new replicas have been updated...
#    Waiting for deployment "bitboxtraining" rollout to finish: 2 out of 3 new replicas have been updated...
#    Waiting for deployment "bitboxtraining" rollout to finish: 2 out of 3 new replicas have been updated...
#    Waiting for deployment "bitboxtraining" rollout to finish: 2 out of 3 new replicas have been updated...
#    Waiting for deployment "bitboxtraining" rollout to finish: 2 out of 3 new replicas have been updated...
#    Waiting for deployment "bitboxtraining" rollout to finish: 1 old replicas are pending termination...
#    Waiting for deployment "bitboxtraining" rollout to finish: 1 old replicas are pending termination...
#    Waiting for deployment "bitboxtraining" rollout to finish: 1 old replicas are pending termination...
#    deployment "bitboxtraining" successfully rolled out
```

```shell
kubectl get pod,svc,deploy,rs -l app=bitboxtraining
#    NAME                         READY   STATUS    RESTARTS   AGE
#    pod/bitboxtraining-dfdd57fdb-fp652   1/1     Running   0          60s
#    pod/bitboxtraining-dfdd57fdb-ftlcz   1/1     Running   0          45s
#    pod/bitboxtraining-dfdd57fdb-tz8gb   1/1     Running   0          29s
#
#    NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
#    service/bitboxtraining-svc   ClusterIP   10.103.212.4   <none>        80/TCP    4m1s
#
#    NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
#    deployment.apps/bitboxtraining   3/3     3            3           5m
#
#    NAME                                DESIRED   CURRENT   READY   AGE
#    replicaset.apps/bitboxtraining-55bc8cd87f   0         0         0       5m
#    replicaset.apps/bitboxtraining-dfdd57fdb    3         3         3       60s
```



```shell
kubectl rollout history deploy bitboxtraining
#    deployment.apps/bitboxtraining
#    REVISION  CHANGE-CAUSE
#    1         kubectl apply --filename=/home/ubuntu/workspaces/Lab7/deploy--bitboxtraining-v1--7.1.yml --record=true
#    2         kubectl apply --filename=/home/ubuntu/workspaces/corrections/Lab7/deploy--bitboxtraining-v2.yml --record=true
```
