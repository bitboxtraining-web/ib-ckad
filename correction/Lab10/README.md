
# Lab 10: Kustomize

```shell
ls -lR ~/workspaces/Lab10/
#    ~/workspaces/Lab10/:
#    total 8,0K
#    drwxrwxr-x 2 user user 4,0K mai   12 19:06 base/
#    drwxrwxr-x 4 user user 4,0K mai   12 18:59 overlays/
#
#    ~/workspaces/Lab10/base:
#    total 24K
#    -rw-rw-r-- 1 user user  899 mai   12 19:04 hasher.yml
#    -rw-rw-r-- 1 user user  443 mai   12 19:20 kustomization.yaml
#    -rw-rw-r-- 1 user user  652 mai   12 18:33 redis.yml
#    -rw-rw-r-- 1 user user  857 mai   12 18:16 rng.yml
#    -rw-rw-r-- 1 user user 1,3K mai   12 19:08 webui.yml
#    -rw-rw-r-- 1 user user  377 mai   12 18:16 worker.yml
#
#    ~/workspaces/Lab10/overlays:
#    total 8,0K
#    drwxrwxr-x 2 user user 4,0K mai   12 18:16 perfs/
#    drwxrwxr-x 2 user user 4,0K mai   12 18:59 upgrade/
#
#    ~/workspaces/Lab10/overlays/perfs:
#    total 4,0K
#    -rw-rw-r-- 1 user user 136 mai   12 18:59 kustomization.yaml
#
#    ~/workspaces/Lab10/overlays/upgrade:
#    total 4,0K
#    -rw-rw-r-- 1 user user 158 mai   12 19:01 kustomization.yaml
```

```shell
cat ~/workspaces/Lab10/base/kustomization.yaml
#    ---
#    apiVersion: kustomize.config.k8s.io/v1beta1
#    kind: Kustomization
#
#    namespace: kustomize-dockercoins
#
#    commonLabels:
#      app.kubernetes.io/name: dockercoins
#      app.kubernetes.io/managed-by: kustomize
#    [...]
```

```shell
kubectl kustomize ~/workspaces/Lab10/base
#    apiVersion: v1
#    kind: Service
#    metadata:
#      labels:
#        app.kubernetes.io/managed-by: kustomize
#        app.kubernetes.io/name: dockercoins
#        component: hasher
#      name: hasher
#      namespace: kustomize-dockercoins
#    spec:
#      ports:
#      - name: http
#        port: 80
#        protocol: TCP
#        targetPort: http
#      selector:
#        app.kubernetes.io/managed-by: kustomize
#        app.kubernetes.io/name: dockercoins
#        component: hasher
#      type: ClusterIP
#    ---
#    apiVersion: v1
#    kind: Service
#    metadata:
#      labels:
#        app.kubernetes.io/managed-by: kustomize
#        app.kubernetes.io/name: dockercoins
#        component: redis
#      name: redis
#      namespace: kustomize-dockercoins
#    [...]
```

```shell
export MINIKUBE_IP=${PUBLIC_IP}
# Ou
# export MINIKUBE_IP=$(minikube ip)
echo ${MINIKUBE_IP}
#    51.44.6.238
sed --in-place "s/FIXME/${MINIKUBE_IP}/" ~/workspaces/Lab10/base/kustomization.yaml
```

```shell
kubectl create namespace kustomize-dockercoins
#    namespace/kustomize-dockercoins created
kubectl config set-context --current --namespace=kustomize-dockercoins
#    Context "minikube" modified.
```

```shell
kubectl apply --kustomize ~/workspaces/Lab10/base
#    service/hasher created
#    service/redis created
#    service/rng created
#    service/webui created
#    deployment.apps/hasher created
#    deployment.apps/redis created
#    deployment.apps/webui created
#    deployment.apps/worker created
#    daemonset.apps/rng created
#    ingress.networking.k8s.io/webui created
```

```shell
kubectl apply --kustomize ~/workspaces/Lab10/overlays/perfs
#    service/hasher unchanged
#    service/redis unchanged
#    service/rng unchanged
#    service/webui unchanged
#    deployment.apps/hasher unchanged
#    deployment.apps/redis unchanged
#    deployment.apps/webui unchanged
#    deployment.apps/worker configured
#    daemonset.apps/rng unchanged
#    ingress.networking.k8s.io/webui unchanged
```

```shell
kubectl get pod,svc,ds,deploy,rs
#    NAME                         READY   STATUS    RESTARTS   AGE
#    pod/hasher-d886b99dc-nvbsl   1/1     Running   0          46s
#    pod/redis-7f46f46c-mf8js     1/1     Running   0          46s
#    pod/rng-2s94t                1/1     Running   0          46s
#    pod/rng-crkwx                1/1     Running   0          46s
#    pod/rng-sl5pn                1/1     Running   0          46s
#    pod/webui-6d48d8f996-flqfg   1/1     Running   0          46s
#    pod/worker-c6cf48d4f-846bx   1/1     Running   0          17s
#    pod/worker-c6cf48d4f-wptf9   1/1     Running   0          46s
#
#    NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
#    service/hasher   ClusterIP   10.99.179.228    <none>        80/TCP     46s
#    service/redis    ClusterIP   10.100.210.62    <none>        6379/TCP   46s
#    service/rng      ClusterIP   10.105.153.247   <none>        80/TCP     46s
#    service/webui    ClusterIP   10.108.128.245   <none>        80/TCP     46s
#
#    NAME                 DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
#    daemonset.apps/rng   3         3         3       3            3           <none>          46s
#
#    NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
#    deployment.apps/hasher   1/1     1            1           46s
#    deployment.apps/redis    1/1     1            1           46s
#    deployment.apps/webui    1/1     1            1           46s
#    deployment.apps/worker   2/2     2            2           46s
#
#    NAME                               DESIRED   CURRENT   READY   AGE
#    replicaset.apps/hasher-d886b99dc   1         1         1       46s
#    replicaset.apps/redis-7f46f46c     1         1         1       46s
#    replicaset.apps/webui-6d48d8f996   1         1         1       46s
#    replicaset.apps/worker-c6cf48d4f   2         2         2       46s
```

```shell
kubectl apply --kustomize ~/workspaces/Lab10/overlays/upgrade
#    service/hasher unchanged
#    service/redis unchanged
#    service/rng unchanged
#    service/webui unchanged
#    deployment.apps/hasher unchanged
#    deployment.apps/redis unchanged
#    deployment.apps/webui unchanged
#    deployment.apps/worker configured
#    daemonset.apps/rng unchanged
#    ingress.networking.k8s.io/webui unchanged
```

```shell
kubectl delete --kustomize ~/workspaces/Lab10/overlays/upgrade
#    service "hasher" deleted
#    service "redis" deleted
#    service "rng" deleted
#    service "webui" deleted
#    deployment.apps "hasher" deleted
#    deployment.apps "redis" deleted
#    deployment.apps "webui" deleted
#    deployment.apps "worker" deleted
#    daemonset.apps "rng" deleted
#    ingress.networking.k8s.io "webui" deleted
```

```shell
kubectl ns default
#    Context "minikube" modified.
#    Active namespace is "default"
kubectl delete namespace kustomize-dockercoins
#    namespace "kustomize-dockercoins" deleted
```
