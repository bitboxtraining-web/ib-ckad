
# TP 2 : Pods, labels, annotations, Namespaces

## TP 2.1 : Pods

### Prise en main de 'kubectl apply'

```shell
kubectl apply --help
```

### Utilisation d'un descripteur au format yaml

```shell
kubectl run yaml-pod --image=containous/whoami:latest --port=80 --dry-run=client -o yaml > pod--run-yaml.yml
```

```yaml
# pod--run-yaml.yml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: yaml-pod
  name: yaml-pod
spec:
  containers:
    - name: yaml-pod
      image: containous/whoami:latest
      ports:
        - containerPort: 80
      resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

À ajuster pour donner `corrections/Lab2/pod--source-yaml.yml`

```shell
kubectl apply -f corrections/Lab2/pod--source-yaml.yml
#    pod/yaml-pod created
```

Récupération de l'IP du pod : 
```shell
kubectl describe pod yaml-pod | grep IP: | head -1
# IP:           172.17.0.7
```

```shell
curl 172.17.0.7
```

### Ajout d'un conteneur dans le Pod

```shell
kubectl delete pod yaml-pod
# pod "yaml-pod" deleted
```

Modifier le descripteur de déploiement pour obtenir `pod--source-yaml-multi-container.yml`

```shell
kubectl apply -f corrections/Lab2/pod--source-yaml-multi-container.yml
# pod/yaml-pod created
```

Récupération de l'IP du pod :
```shell
kubectl describe pod yaml-pod | grep IP: | head -1
# IP:           172.17.0.7
```

```shell
curl 172.17.0.7
```

### Tester le whoami en passant par le conteneur sidecar

```shell
kubectl exec --help
```

```shell
kubectl exec -ti yaml-pod --container shell-in-pod -- bash
> curl localhost:80

> ps auxwww

> exit
```

### Modifier un Pod

```shell
kubectl describe pod yaml-pod
```

```shell
kubectl get pod yaml-pod -o yaml
```

```shell
kubectl apply -f corrections/Lab2/pod--source-yaml-multi-container-modified.yml
#    pod/yaml-pod configured
```

### Visualiser la definition d'un pod

```shell
kubectl get po yaml-pod -o yaml
```

### Optionnel : utilisation d'un descripteur au format json

```shell
kubectl create -f corrections/Lab2/pod--source-json.json
#    pod/json-pod created
```
## TP 2.2 : Labels

### Prise en main

```shell
kubectl label --help
```

```shell
kubectl get pod --show-labels
```

```shell
kubectl label pod yaml-pod release=stable stack=market
#    pod/yaml-pod labeled
kubectl label pod json-pod release=stable stack=market
#    pod/json-pod labeled
```

```shell
kubectl label pod yaml-pod --overwrite release=unstable
#    pod/yaml-pod labeled
```

```shell
kubectl describe pod yaml-pod
```

### Afficher les labels des pods

```shell
kubectl get pod --all-namespaces --show-labels
```

```shell
kubectl get pod --label-columns run,release,stack
```

```shell
kubectl get pod --selector stack
```

```shell
kubectl get pod --selector '!release'
```

```shell
kubectl get pod --selector 'run in (whoami,yaml-pod)'
```

```shell
kubectl get pod --selector 'run in (whoami,yaml-pod),!stack'
```

### Utilisation du nodeSelector

```shell
kubectl get nodes --show-labels
```

```shell
kubectl apply -f corrections/Lab2/pod--light-sleeper.yml
#    pod/light-sleeper created
```

```shell
kubectl get pod light-sleeper
#    NAME            READY   STATUS    RESTARTS   AGE
#    light-sleeper   0/1     Pending   0          6s
```

Le Pod est "pending" car aucun noeud ne correspond à ses critères.

```shell
kubectl get nodes
#    NAME              STATUS   ROLES                  AGE   VERSION
#    ip-172-31-28-75   Ready    control-plane,master   84m   v1.18.3
```

```shell
kubectl label node ip-172-31-28-75 container-runtime=docker
#    node/ip-172-31-28-75 labeled
```

```shell
kubectl get pod light-sleeper
#    NAME            READY   STATUS    RESTARTS   AGE
#    light-sleeper   1/1     Running   0          2m34s
```
## TP 2.3 : Annotations et Namespaces

### Annotations

```shell
kubectl annotate --help
```

```shell
kubectl get -o json pod yaml-pod | jq .metadata.annotations
```

```shell
kubectl annotate pod yaml-pod super.mycompany.com/ci-build-number=722
#    pod/yaml-pod annotated
```

```shell
kubectl get -o json pod yaml-pod | jq .metadata.annotations
```

### Namespaces

```shell
kubectl get namespaces
```

```shell
kubectl get pods --namespace kube-system
```

```shell
kubectl get pods --all-namespaces
```

### Création d'un nouveau Namespace

```shell
kubectl create -f corrections/Lab2/ns--my-sandbox.yml
#    namespace/my-sandbox created
```

```shell
kubectl apply -n my-sandbox -f corrections/Lab2/pod--source-yaml-multi-container.yml
#    pod/yaml-pod created
```

```shell
kubectl get pods --selector run=yaml-pod --all-namespaces
#    NAMESPACE    NAME       READY   STATUS    RESTARTS   AGE
#    default      yaml-pod   2/2     Running   0          40m
#    my-sandbox   yaml-pod   2/2     Running   0          46s
```

```shell
kubectl apply -f corrections/Lab2/pod--source-json-in-ns-my-sandbox.json
#    pod/json-pod created
```

```shell
kubectl get pods -n my-sandbox
#    NAME       READY   STATUS    RESTARTS   AGE
#    json-pod   2/2     Running   0          45s
#    yaml-pod   2/2     Running   0          2m15s
```

```shell
kubectl delete namespace my-sandbox
#    namespace "my-sandbox" deleted
```
## TP 2.4 : logs et cycle de vie

### Prise en main

```shell
kubectl logs --help
```

```shell
kubectl logs yaml-pod -c yaml-pod
#    Starting up on port 80
```

```shell
kubectl apply -f workspaces/Lab2/pod--whoami-and-clock--2.4.yml
#    pod/whoami-and-clock created
```

```shell
kubectl logs whoami-and-clock -c clock
```

```shell
kubectl logs whoami-and-clock -c clock --follow --tail 10
```

## TP 2.5 : Init Containers

### Experimentations

```shell
kubectl explain pod.spec.initContainers
```

```shell
kubectl apply -f corrections/Lab2/pod--whoami-and-clock-with-init-containers.yml
#    pod/whoami-and-clock-with-init created
```

```shell
kubectl get pod whoami-and-clock-with-init
#    NAME                         READY   STATUS     RESTARTS   AGE
#    whoami-and-clock-with-init   0/1     Init:0/1   0          2s
```

```shell
kubectl logs whoami-and-clock-with-init -c timer

```
## TP 2.6 : Cycle de vie

```shell
docker  ps --filter name=clock_whoami-and-clock
```

```shell
kubectl get pod whoami-and-clock
#    NAME               READY   STATUS    RESTARTS   AGE
#    whoami-and-clock   2/2     Running   0          3m57s
```

```shell
docker stop 74882b79f3f7
#    74882b79f3f7
```

```shell
kubectl get pod whoami-and-clock
#    NAME               READY   STATUS    RESTARTS   AGE
#    whoami-and-clock   2/2     Running   1          5m21s
```

### Restart Policy

```shell
kubectl apply -f workspaces/Lab2/pod--restart-policy-rp-check--2.6.yml
#    pod/rp-check created
```

```shell
kubectl get pod rp-check
#    NAME       READY   STATUS      RESTARTS   AGE
#    rp-check   0/1     Completed   2          71s
```

On constate l'augmentation du compteur de restart.

```shell
kubectl delete pod rp-check
#    pod "rp-check" deleted
```

```shell
kubectl apply -f corrections/Lab2/pod--restart-policy-rp-check--2.6.yml
#    pod/rp-check created
```

```shell
kubectl get pod rp-check
#    NAME       READY   STATUS      RESTARTS   AGE
#    rp-check   0/1     Completed   0          43s
```

On constate le nombre de restart à 0 mais le nombre de ready aussi.

```shell
kubectl delete pod rp-check
#    pod "rp-check" deleted
```

```shell
kubectl apply -f corrections/Lab2/pod--restart-policy-rp-check-and-false--2.6.yml
#    pod/rp-check created
```

```shell
kubectl get pod rp-check
#    NAME       READY   STATUS             RESTARTS   AGE
#    rp-check   0/1     CrashLoopBackOff   2          81s
```

On constate de nouveau les restarts.

## TP 2.7 : Liveness probes

### Liveness

```shell
kubectl apply -f corrections/Lab2/pod--liveness-http-probe.yml
#    pod/liveness-http created
```

```shell
kubectl get pod liveness-http
#    NAME            READY   STATUS    RESTARTS   AGE
#    liveness-http   1/1     Running   0          47s

kubectl get pod liveness-http
#    NAME            READY   STATUS    RESTARTS   AGE
#    liveness-http   1/1     Running   1          59s

kubectl get pod liveness-http
#    NAME            READY   STATUS    RESTARTS   AGE
#    liveness-http   1/1     Running   2          116s
```

On observe que le compteur de "restart" augmente.

Le conteneur démarre.

Après 10 secondes, Kubernetes commence à envoyer des requêtes GET sur /healthz.

Si la réponse est OK (200) → tout va bien.

Si la réponse échoue 5 fois de suite → Kubernetes redémarre le conteneur pour le relancer.

```shell
kubectl describe pod liveness-http
```
