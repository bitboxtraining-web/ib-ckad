
# Lab 9: Helm

## Lab 9.1: Helm

```shell
kubectl delete deploy,sts,ds,rs,svc --all
kubectl delete pods --all
kubectl delete pvc,pv --all
for node in $(kubectl get nodes -o name); do
  minikube ssh --node ${node} -- docker image prune --all --force
done
```

```shell
helm version
#    version.BuildInfo{Version:"v3.14.4", GitCommit:"81c902a123462fd4052bc5e9aa9c513c4c8fc142", GitTreeState:"clean", GoVersion:"go1.21.9"}
```

```shell
kubectl create namespace helm-dockercoins
#    namespace/helm-dockercoins created
kubectl ns helm-dockercoins
#    Context "minikube" modified.
#    Active namespace is "helm-dockercoins".
```

```shell
cd ~/workspaces/Lab9/dockercoins

cat values.yaml
#    # Default values for dockercoins.
#    # This is a YAML-formatted file.
#    # Declare variables to be passed into your templates.
#
#    replicaCount:
#      hasher: 1
#      worker: 1
#
#    image:
#      default:
#        version: "1.0"
#      rng:
#        repository: training/dockercoins-rng
#      hasher:
#        repository: training/dockercoins-hasher
#    [...]
```

```shell
helm dependency update
#    Getting updates for unmanaged Helm repositories...
#    ...Successfully got an update from the "https://charts.bitnami.com/bitnami/" chart repository
#    Saving 1 charts
#    Downloading redis from repo https://charts.bitnami.com/bitnami/
#    Deleting outdated charts
```

```shell
public_ip=${PUBLIC_IP}  # or $(minikube ip)
echo "public_ip: ${public_ip}"
#    public_ip: 51.44.6.238

helm install dockercoins ./ \
  --set ingress.webui.host=dockercoins.${public_ip}.sslip.io
#    NAME: dockercoins
#    LAST DEPLOYED: Sun Feb 18 14:37:38 2024
#    NAMESPACE: helm-dockercoins
#    STATUS: deployed
#    REVISION: 1
#    TEST SUITE: None
#    NOTES:
#    1. Open the application by going to this URL: http://dockercoins.51.44.6.238.sslip.io
```

```shell
kubectl get pod,svc,ds,deploy,rs,sts,pvc,pv
#    NAME                                                  READY   STATUS    RESTARTS   AGE
#    pod/dockercoins-hasher-dockercoins-754494669f-wkpxf   1/1     Running   0          85s
#    pod/dockercoins-rng-dockercoins-7t64w                 1/1     Running   0          85s
#    pod/dockercoins-rng-dockercoins-8zfcb                 1/1     Running   0          85s
#    pod/dockercoins-rng-dockercoins-pmnwb                 1/1     Running   0          85s
#    pod/dockercoins-webui-dockercoins-86b9bcd747-ncrz5    1/1     Running   0          85s
#    pod/dockercoins-worker-dockercoins-5c58cfd45d-d72d9   1/1     Running   0          85s
#    pod/redis-node-0                                      2/2     Running   0          85s
#
#    NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE
#    service/hasher           ClusterIP   10.101.5.212     <none>        80/TCP               85s
#    service/redis            ClusterIP   10.108.254.79    <none>        6379/TCP,26379/TCP   85s
#    service/redis-headless   ClusterIP   None             <none>        6379/TCP,26379/TCP   85s
#    service/rng              ClusterIP   10.101.219.207   <none>        80/TCP               85s
#    service/webui            ClusterIP   10.105.80.35     <none>        80/TCP               85s
#
#    NAME                                         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
#    daemonset.apps/dockercoins-rng-dockercoins   3         3         3       3            3           <none>          85s
#
#    NAME                                             READY   UP-TO-DATE   AVAILABLE   AGE
#    deployment.apps/dockercoins-hasher-dockercoins   1/1     1            1           85s
#    deployment.apps/dockercoins-webui-dockercoins    1/1     1            1           85s
#    deployment.apps/dockercoins-worker-dockercoins   1/1     1            1           85s
#
#    NAME                                                        DESIRED   CURRENT   READY   AGE
#    replicaset.apps/dockercoins-hasher-dockercoins-754494669f   1         1         1       85s
#    replicaset.apps/dockercoins-webui-dockercoins-86b9bcd747    1         1         1       85s
#    replicaset.apps/dockercoins-worker-dockercoins-5c58cfd45d   1         1         1       85s
#
#    NAME                          READY   AGE
#    statefulset.apps/redis-node   1/1     85s
#
#    NAME                                            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      VOLUMEATTRIBUTESCLASS   AGE
#    persistentvolumeclaim/redis-data-redis-node-0   Bound    pvc-25e6829e-6013-4817-8c95-a6ccd8b1a4d0   8Gi        RWO            csi-hostpath-sc   <unset>                 85s
#
#    NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                      STORAGECLASS      VOLUMEATTRIBUTESCLASS   REASON   AGE
#    persistentvolume/pvc-25e6829e-6013-4817-8c95-a6ccd8b1a4d0   8Gi        RWO            Delete           Bound    helm-dockercoins/redis-data-redis-node-0   csi-hostpath-sc   <unset>                          79s
```

```shell
helm list
#    NAME            NAMESPACE               REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
#    dockercoins     helm-dockercoins        1               2024-02-18 14:40:55.020157836 +0000 UTC deployed        dockercoins-0.1.0       1.0.0
```

```shell
helm upgrade dockercoins ./ \
  --reuse-values \
  --set replicaCount.worker=2
#    Release "dockercoins" has been upgraded. Happy Helming!
#    NAME: dockercoins
#    LAST DEPLOYED: Sun Feb 18 14:43:18 2024
#    NAMESPACE: helm-dockercoins
#    STATUS: deployed
#    REVISION: 2
#    TEST SUITE: None
#    NOTES:
#    1. Open the application by going to this URL: http://dockercoins.51.44.6.238.sslip.io
```

```shell
helm upgrade dockercoins ./ \
  --reuse-values \
  --set image.worker.version=1.1
#    Release "dockercoins" has been upgraded. Happy Helming!
#    NAME: dockercoins
#    LAST DEPLOYED: Sun Feb 18 14:43:44 2024
#    NAMESPACE: helm-dockercoins
#    STATUS: deployed
#    REVISION: 3
#    TEST SUITE: None
#    NOTES:
#    1. Open the application by going to this URL: http://dockercoins.51.44.6.238.sslip.io
```

```shell
kubectl get pods,rs,ds,deploy,svc,ing -o wide
#    NAME                                                  READY   STATUS        RESTARTS   AGE     IP            NODE           NOMINATED NODE   READINESS GATES
#    pod/dockercoins-hasher-dockercoins-754494669f-wkpxf   1/1     Running       0          3m24s   10.244.1.9    minikube-m02   <none>           <none>
#    pod/dockercoins-rng-dockercoins-7t64w                 1/1     Running       0          3m24s   10.244.2.8    minikube-m03   <none>           <none>
#    pod/dockercoins-rng-dockercoins-8zfcb                 1/1     Running       0          3m24s   10.244.0.8    minikube       <none>           <none>
#    pod/dockercoins-rng-dockercoins-pmnwb                 1/1     Running       0          3m24s   10.244.1.10   minikube-m02   <none>           <none>
#    pod/dockercoins-webui-dockercoins-86b9bcd747-ncrz5    1/1     Running       0          3m24s   10.244.1.8    minikube-m02   <none>           <none>
#    pod/dockercoins-worker-dockercoins-5c58cfd45d-d72d9   1/1     Terminating   0          3m24s   10.244.1.7    minikube-m02   <none>           <none>
#    pod/dockercoins-worker-dockercoins-5c58cfd45d-x2xth   1/1     Terminating   0          60s     10.244.2.10   minikube-m03   <none>           <none>
#    pod/dockercoins-worker-dockercoins-7c4c96bdd9-gtpmh   1/1     Running       0          26s     10.244.2.11   minikube-m03   <none>           <none>
#    pod/dockercoins-worker-dockercoins-7c4c96bdd9-lgd88   1/1     Running       0          34s     10.244.1.11   minikube-m02   <none>           <none>
#    pod/redis-node-0                                      2/2     Running       0          3m24s   10.244.2.9    minikube-m03   <none>           <none>
#
#    NAME                                                        DESIRED   CURRENT   READY   AGE     CONTAINERS    IMAGES                            SELECTOR
#    replicaset.apps/dockercoins-hasher-dockercoins-754494669f   1         1         1       3m24s   dockercoins   training/dockercoins-hasher:1.0   app.kubernetes.io/instance=dockercoins,app.kubernetes.io/name=dockercoins,component=hasher,pod-template-hash=754494669f
#    replicaset.apps/dockercoins-webui-dockercoins-86b9bcd747    1         1         1       3m24s   dockercoins   training/dockercoins-webui:1.0    app.kubernetes.io/instance=dockercoins,app.kubernetes.io/name=dockercoins,component=webui,pod-template-hash=86b9bcd747
#    replicaset.apps/dockercoins-worker-dockercoins-5c58cfd45d   0         0         0       3m24s   dockercoins   training/dockercoins-worker:1.0   app.kubernetes.io/instance=dockercoins,app.kubernetes.io/name=dockercoins,component=worker,pod-template-hash=5c58cfd45d
#    replicaset.apps/dockercoins-worker-dockercoins-7c4c96bdd9   2         2         2       34s     dockercoins   training/dockercoins-worker:1.1   app.kubernetes.io/instance=dockercoins,app.kubernetes.io/name=dockercoins,component=worker,pod-template-hash=7c4c96bdd9
#
#    NAME                                         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE     CONTAINERS    IMAGES                         SELECTOR
#    daemonset.apps/dockercoins-rng-dockercoins   3         3         3       3            3           <none>          3m24s   dockercoins   training/dockercoins-rng:1.0   app.kubernetes.io/instance=dockercoins,app.kubernetes.io/name=dockercoins,component=rng
#
#    NAME                                             READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS    IMAGES                            SELECTOR
#    deployment.apps/dockercoins-hasher-dockercoins   1/1     1            1           3m24s   dockercoins   training/dockercoins-hasher:1.0   app.kubernetes.io/instance=dockercoins,app.kubernetes.io/name=dockercoins,component=hasher
#    deployment.apps/dockercoins-webui-dockercoins    1/1     1            1           3m24s   dockercoins   training/dockercoins-webui:1.0    app.kubernetes.io/instance=dockercoins,app.kubernetes.io/name=dockercoins,component=webui
#    deployment.apps/dockercoins-worker-dockercoins   2/2     2            2           3m24s   dockercoins   training/dockercoins-worker:1.1   app.kubernetes.io/instance=dockercoins,app.kubernetes.io/name=dockercoins,component=worker
#
#    NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE     SELECTOR
#    service/hasher           ClusterIP   10.101.5.212     <none>        80/TCP               3m24s   app.kubernetes.io/instance=dockercoins,app.kubernetes.io/name=dockercoins,component=hasher
#    service/redis            ClusterIP   10.108.254.79    <none>        6379/TCP,26379/TCP   3m24s   app.kubernetes.io/component=node,app.kubernetes.io/instance=dockercoins,app.kubernetes.io/name=redis
#    service/redis-headless   ClusterIP   None             <none>        6379/TCP,26379/TCP   3m24s   app.kubernetes.io/instance=dockercoins,app.kubernetes.io/name=redis
#    service/rng              ClusterIP   10.101.219.207   <none>        80/TCP               3m24s   app.kubernetes.io/instance=dockercoins,app.kubernetes.io/name=dockercoins,component=rng
#    service/webui            ClusterIP   10.105.80.35     <none>        80/TCP               3m24s   app.kubernetes.io/instance=dockercoins,app.kubernetes.io/name=dockercoins,component=webui
#
#    NAME                                                      CLASS   HOSTS                              ADDRESS        PORTS   AGE
#    ingress.networking.k8s.io/dockercoins-webui-dockercoins   nginx   dockercoins.51.44.6.238.sslip.io   192.168.49.2   80      3m24s
```

```shell
helm delete dockercoins
#    release "dockercoins" uninstalled
```

```shell
kubectl ns default
#    Context "minikube" modified.
#    Active namespace is "default"
kubectl delete namespace helm-dockercoins
#    namespace "helm-dockercoins" deleted
```
