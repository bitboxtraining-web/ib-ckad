
# Lab 6: ConfigMaps and Secrets

## Lab 6.1: ConfigMaps

### Literal values

```shell
kubectl create configmap special-config --from-literal=special.k=cereales --from-literal=special.ite=kubernetes
#    configmap/special-config created
```

```shell
kubectl describe cm special-config
#    Name:         special-config
#    Namespace:    default
#    Labels:       <none>
#    Annotations:  <none>
#
#    Data
#    ====
#    special.ite:
#    ----
#    kubernetes
#    special.k:
#    ----
#    cereales
#
#    BinaryData
#    ====
#
#    Events:  <none>
```

### Use as environment variable

```shell
kubectl apply -f ~/workspaces/Lab6/pod--cm-one-venv--6.1.yml
#    pod/pod-with-cm-one-venv created
```
```shell
kubectl logs pod-with-cm-one-venv | grep SPECIAL
#    SPECIAL_LEVEL_KEY=cereales
```

### Import all the ConfigMap keys

```shell
kubectl apply -f ~/workspaces/corrections/Lab6/cm--special-config-for-venv.yml
#    configmap/special-config-for-venv created
```

```shell
kubectl apply -f ~/workspaces/Lab6/pod--cm-all-vars--6.1.yml
#    pod/pod-with-cm-all-vars created
```

```shell
kubectl logs pod-with-cm-all-vars | grep SPECIAL
#    SPECIAL_K=cereales
#    SPECIAL_ITE=kubernetes
```

### Import from files

```shell
kubectl create configmap game-config --from-file ~/workspaces/Lab6/configs
#    configmap/game-config created
```

```shell
kubectl describe cm game-config
#    Name:         game-config
#    Namespace:    default
#    Labels:       <none>
#    Annotations:  <none>
#
#    Data
#    ====
#    game.properties:
#    ----
#    enemies=aliens
#    lives=3
#    enemies.cheat=true
#    enemies.cheat.level=noGoodRotten
#    secret.code.passphrase=UUDDLRLRBABAS
#    secret.code.allowed=true
#    secret.code.lives=30
#
#    ui.properties:
#    ----
#    color.good=purple
#    color.bad=yellow
#    allow.textmode=true
#    how.nice.to.look=fairlyNice
#
#
#    BinaryData
#    ====
#
#    Events:  <none>
```

### Use ConfigMap as a volume

```shell
kubectl apply -f ~/workspaces/Lab6/pod--cm-as-vol--6.1.yml
#    pod/cm-as-vol created
```

```shell
kubectl exec cm-as-vol -- ls -l /etc/config
#    total 0
#    lrwxrwxrwx 1 root root 22 Jul  2 14:33 game.properties -> ..data/game.properties
#    lrwxrwxrwx 1 root root 20 Jul  2 14:33 ui.properties -> ..data/ui.properties

kubectl exec cm-as-vol -- cat /etc/config/ui.properties
#    color.good=purple
#    color.bad=yellow
#    allow.textmode=true
#    how.nice.to.look=fairlyNice
```

## Lab 6.2: Secrets

### Create namespace

```shell
kubectl create namespace secret-ops
#    namespace/secret-ops created
```

### Create a secret

```shell
kubectl create secret generic secret-identities \
          --from-literal='spiderman=Peter Parker' \
          --from-literal='superman=Clark Kent' \
          --namespace secret-ops
#    secret/secret-identities created
```

```shell
kubectl describe secrets secret-identities --namespace secret-ops
#    Name:         secret-identities
#    Namespace:    secret-ops
#    Labels:       <none>
#    Annotations:  <none>
#
#    Type:  Opaque
#
#    Data
#    ====
#    spiderman:  12 bytes
#    superman:   10 bytes
```

```shell
kubectl get secret secret-identities --namespace secret-ops -o yaml
#    apiVersion: v1
#    data:
#      spiderman: UGV0ZXIgUGFya2Vy
#      superman: Q2xhcmsgS2VudA==
#    kind: Secret
#    metadata:
#      creationTimestamp: "2023-11-24T10:47:41Z"
#      name: secret-identities
#      namespace: secret-ops
#      resourceVersion: "1738"
#      uid: ef7bfce3-a6d3-4025-ad63-4ce08ba446b4
#    type: Opaque
```

```shell
echo "UGV0ZXIgUGFya2Vy" | base64 -d
#    Peter Parker
```

### Use the Secret / environment variable

```shell
kubectl apply -f ~/workspaces/Lab6/pod--civil-war-expose-secret-identity--6.2.yml
#    pod/civil-war created
```

```shell
kubectl exec --namespace secret-ops civil-war -- sh -c 'echo ${SPIDERMAN_IS}'
#    Peter Parker
```

### Use the Secret / files

```shell
kubectl apply -f ~/workspaces/Lab6/pod--dc-vs-marvel--6.2.yml
#    pod/dc-vs-marvel created
```

```shell
kubectl exec dc-vs-marvel --namespace secret-ops -- ls -l /etc/revelations
#    total 0
#    lrwxrwxrwx 1 root root 16 Jul  2 14:50 spiderman -> ..data/spiderman
#    lrwxrwxrwx 1 root root 15 Jul  2 14:50 superman -> ..data/superman

kubectl exec dc-vs-marvel --namespace secret-ops -- cat /etc/revelations/superman
#    Clark Kent
```
