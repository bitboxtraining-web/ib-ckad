
# Lab 2: Pods, labels, annotations, Namespaces

## Lab 2.1: Pods

### Getting started with 'kubectl apply'

```shell
kubectl apply --help
# Apply a configuration to a resource by filename or stdin. The resource name must be specified. This resource will be
# created if it doesn't exist yet. To use 'apply', always create the resource initially with either 'apply' or 'create
# --save-config'.
#
#  JSON and YAML formats are accepted.
#
#  Alpha Disclaimer: the --prune functionality is not yet complete. Do not use unless you are aware of what the current
# state is. See https://issues.k8s.io/34274.
#
# Examples:
#   # Apply the configuration in pod.json to a pod.
#   kubectl apply -f ./pod.json
#
#    [...]
```

### Use a YAML descriptor

```shell
kubectl run yaml-pod --image=traefik/whoami:v1.10 --port=80 --dry-run=client -o yaml > yaml-pod.yml
```

```yaml
# yaml-pod.yml
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
      image: traefik/whoami:v1.10
      ports:
        - containerPort: 80
      resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Should be modified to look like `corrections/Lab2/pod--source-yaml.yml`

```shell
kubectl apply -f ~/workspaces/corrections/Lab2/pod--source-yaml.yml
#    pod/yaml-pod created
```

Get the Pod's IP address:
```shell
YAML_POD_IP=$(kubectl get -o go-template --template='{{.status.podIP}}' pod yaml-pod)
echo ${YAML_POD_IP}
#    10.244.1.5
```

```shell
minikube ssh -- curl -s ${YAML_POD_IP}:80
#    Hostname: yaml-pod
#    IP: 127.0.0.1
#    IP: 10.244.1.5
#    RemoteAddr: 192.168.49.2:52612
#    GET / HTTP/1.1
#    Host: 10.244.1.5
#    User-Agent: curl/7.81.0
#    Accept: */*
```

### Adding a container in the Pod

```shell
kubectl delete pod yaml-pod
#     pod "yaml-pod" deleted
```

Update the manifest to make it look like `pod--source-yaml-multi-container.yml`

```shell
kubectl apply -f ~/workspaces/corrections/Lab2/pod--source-yaml-multi-container.yml
#     pod/yaml-pod created
```

Get the Pod's IP address:
```shell
YAML_POD_IP=$(kubectl get -o go-template --template='{{.status.podIP}}' pod yaml-pod)
echo ${YAML_POD_IP}
#    10.244.1.6
```

```shell
minikube ssh -- curl -s ${YAML_POD_IP}:80
#    Hostname: yaml-pod
#    IP: 127.0.0.1
#    IP: 10.244.1.6
#    RemoteAddr: 192.168.49.2:50614
#    GET / HTTP/1.1
#    Host: 10.244.1.6
#    User-Agent: curl/7.81.0
#    Accept: */*
```

### Test whoami from the sidecar container

```shell
kubectl exec --help
#    Execute a command in a container.
#
#    Examples:
#      # Get output from running 'date' command from pod mypod, using the first container by default
#      kubectl exec mypod -- date
#
#      # Get output from running 'date' command in ruby-container from pod mypod
#      kubectl exec mypod -c ruby-container -- date
#    [...]
```

```shell
kubectl exec -ti yaml-pod --container shell-in-pod -- bash
> curl localhost:80
#    Hostname: yaml-pod
#    IP: 127.0.0.1
#    IP: 10.244.1.6
#    RemoteAddr: 127.0.0.1:45946
#    GET / HTTP/1.1
#    Host: localhost
#    User-Agent: curl/7.88.1
#    Accept: */*
> ps auxwww
#    USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
#    root         1  0.0  0.0   4364   356 ?        Ss   17:46   0:00 sleep infinity
#    root         6  0.0  0.0  11828  1896 pts/0    Ss   17:48   0:00 bash
#    root        22  0.0  0.0  51756  1704 pts/0    R+   17:48   0:00 ps auxwww
> exit
```

### Edit a Pod

```shell
kubectl describe pod yaml-pod
#    Name:             yaml-pod
#    Namespace:        default
#    Priority:         0
#    Service Account:  default
#    Node:             minikube-m02/192.168.49.3
#    Start Time:       Sun, 18 Feb 2024 10:22:53 +0000
#    Labels:           run=yaml-pod
#    Annotations:      <none>
#    Status:           Running
#    IP:               10.244.1.6
#    [...]
```

```shell
kubectl get pod yaml-pod -o yaml
#    apiVersion: v1
#    kind: Pod
#    metadata:
#      annotations:
#        kubectl.kubernetes.io/last-applied-configuration: | [...]
#      creationTimestamp: "2024-02-18T10:22:53Z"
#      labels:
#        run: yaml-pod
#    [...]
```

```shell
kubectl apply -f ~/workspaces/corrections/Lab2/pod--source-yaml-multi-container-modified.yml
#    pod/yaml-pod configured
```

### Display the definition of a Pod

```shell
kubectl get po yaml-pod -o yaml
#    apiVersion: v1
#    kind: Pod
#    metadata:
#      annotations:
#        kubectl.kubernetes.io/last-applied-configuration: | [...]
#      creationTimestamp: "2024-02-18T10:22:53Z"
#      labels:
#        from-descriptor: yaml
#        run: yaml-pod
```

## Lab 2.2: Labels

### Getting started

```shell
kubectl label --help
#    Update the labels on a resource.
#
#      *  A label key and value must begin with a letter or number, and may contain letters, numbers, hyphens, dots, and
#    underscores, up to  63 characters each.
#      *  Optionally, the key can begin with a DNS subdomain prefix and a single '/', like example.com/my-app
#      *  If --overwrite is true, then existing labels can be overwritten, otherwise attempting to overwrite a label will
#    result in an error.
#      *  If --resource-version is specified, then updates will use this resource version, otherwise the existing
#    resource-version will be used.
#
#    Examples:
#      # Update pod 'foo' with the label 'unhealthy' and the value 'true'.
#      kubectl label pods foo unhealthy=true
#
#      # Update pod 'foo' with the label 'status' and the value 'unhealthy', overwriting any existing value.
#      kubectl label --overwrite pods foo status=unhealthy
#    [...]
```

```shell
kubectl label pod yaml-pod --list
#    from-descriptor=yaml
#    run=yaml-pod
```

```shell
kubectl label pod yaml-pod release=stable stack=market
#    pod/yaml-pod labeled
```

```shell
kubectl label pod yaml-pod --overwrite release=unstable
#    pod/yaml-pod labeled
```

```shell
kubectl label pod yaml-pod --list
#    from-descriptor=yaml
#    release=unstable
#    run=yaml-pod
#    stack=market
```

```shell
kubectl describe pod yaml-pod
#    Name:             yaml-pod
#    Namespace:        default
#    Priority:         0
#    Service Account:  default
#    Node:             minikube-m02/192.168.49.3
#    Start Time:       Sun, 18 Feb 2024 10:22:53 +0000
#    Labels:           from-descriptor=yaml
#                      release=unstable
#                      run=yaml-pod
#                      stack=market
#    [...]
```

### Show Pod Labels

```shell
kubectl get pod --show-labels
#    NAME             READY   STATUS    RESTARTS   AGE     LABELS
#    training-shell   1/1     Running   0          43m     run=training-shell
#    whoami           1/1     Running   0          45m     run=whoami
#    yaml-pod         2/2     Running   0          5m38s   from-descriptor=yaml,release=unstable,run=yaml-pod,stack=market
```

```shell
kubectl get pod --label-columns run,release,stack
#    NAME             READY   STATUS    RESTARTS   AGE    RUN              RELEASE    STACK
#    training-shell   1/1     Running   0          43m    training-shell
#    whoami           1/1     Running   0          46m    whoami
#    yaml-pod         2/2     Running   0          6m3s   yaml-pod         unstable   market
```

```shell
kubectl get pod --selector stack
#    NAME       READY   STATUS    RESTARTS   AGE
#    yaml-pod   2/2     Running   0          6m19s
```

```shell
kubectl get pod --selector '!release'
#    NAME             READY   STATUS    RESTARTS   AGE
#    training-shell   1/1     Running   0          43m
#    whoami           1/1     Running   0          46m
```

```shell
kubectl get pod --selector 'run in (whoami,yaml-pod)'
#    NAME       READY   STATUS    RESTARTS   AGE
#    whoami     1/1     Running   0          46m
#    yaml-pod   2/2     Running   0          6m51s
```

```shell
kubectl get pod --selector 'run in (whoami,yaml-pod),!stack'
#    NAME     READY   STATUS    RESTARTS   AGE
#    whoami   1/1     Running   0          47m
```

### Using a nodeSelector

```shell
kubectl label nodes --all --list
#    Listing labels for Node./minikube:
#     beta.kubernetes.io/arch=amd64
#     kubernetes.io/hostname=minikube
#     kubernetes.io/arch=amd64
#     minikube.k8s.io/name=minikube
#     node.kubernetes.io/exclude-from-external-load-balancers=
#     minikube.k8s.io/commit=86fc9d54fca63f295d8737c8eacdbb7987e89c67
#     beta.kubernetes.io/os=linux
#     kubernetes.io/os=linux
#     node-role.kubernetes.io/control-plane=
#     minikube.k8s.io/updated_at=2024_05_08T13_50_57_0700
#     minikube.k8s.io/version=v1.33.0
#     minikube.k8s.io/primary=true
#    Listing labels for Node./minikube-m02:
#     minikube.k8s.io/commit=86fc9d54fca63f295d8737c8eacdbb7987e89c67
#     minikube.k8s.io/version=v1.33.0
#     kubernetes.io/os=linux
#     minikube.k8s.io/name=minikube
#     beta.kubernetes.io/arch=amd64
#     beta.kubernetes.io/os=linux
#     kubernetes.io/hostname=minikube-m02
#     minikube.k8s.io/primary=false
#     kubernetes.io/arch=amd64
#     minikube.k8s.io/updated_at=2024_05_08T13_51_30_0700
#    Listing labels for Node./minikube-m03:
#     kubernetes.io/arch=amd64
#     minikube.k8s.io/commit=86fc9d54fca63f295d8737c8eacdbb7987e89c67
#     minikube.k8s.io/primary=false
#     beta.kubernetes.io/arch=amd64
#     beta.kubernetes.io/os=linux
#     kubernetes.io/hostname=minikube-m03
#     kubernetes.io/os=linux
#     minikube.k8s.io/version=v1.33.0
#     minikube.k8s.io/name=minikube
#     minikube.k8s.io/updated_at=2024_05_08T13_51_53_0700
```

```shell
kubectl apply -f ~/workspaces/corrections/Lab2/pod--light-sleeper.yml
#    pod/light-sleeper created
```

```shell
kubectl get pod light-sleeper
#    NAME            READY   STATUS    RESTARTS   AGE
#    light-sleeper   0/1     Pending   0          6s
```

The Pod's state is "pending" because no Node matches these criteria.

```shell
kubectl get nodes
#    NAME           STATUS   ROLES           AGE   VERSION
#    minikube       Ready    control-plane   58m   v1.30.0
#    minikube-m02   Ready    <none>          58m   v1.30.0
#    minikube-m03   Ready    <none>          57m   v1.30.0
```

```shell
kubectl label node minikube-m02 cluster-distribution=minikube
#    node/minikube-m02 labeled
```

```shell
kubectl get pod light-sleeper
#    NAME            READY   STATUS    RESTARTS   AGE
#    light-sleeper   1/1     Running   0          81s
```

## Lab 2.3: Annotations and Namespaces

### Annotations

```shell
kubectl annotate --help
#    Update the annotations on one or more resources
#
#     All Kubernetes objects support the ability to store additional data with the object as annotations. Annotations are
#    key/value pairs that can be larger than labels and include arbitrary string values such as structured JSON. Tools and
#    system extensions may use annotations to store their own data.
#
#     Attempting to set an annotation that already exists will fail unless --overwrite is set. If --resource-version is
#    specified and does not match the current resource version on the server the command will fail.
#
#    Use "kubectl api-resources" for a complete list of supported resources.
#
#    Examples:
#      # Update pod 'foo' with the annotation 'description' and the value 'my frontend'.
#      # If the same annotation is set multiple times, only the last value will be applied
#      kubectl annotate pods foo description='my frontend'
#
#      # Update a pod identified by type and name in "pod.json"
#      kubectl annotate -f pod.json description='my frontend'
#    [...]
```

```shell
kubectl annotate pod yaml-pod --list
#    kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"from-descriptor":"yaml","run":"yaml-pod"},"name":"yaml-pod","namespace":"default"},"spec":{"containers":[{"image":"traefik/whoami:v1.8","name":"yaml-pod","ports":[{"containerPort":80}]},{"args":["infinity"],"command":["sleep"],"image":"bitboxtraining/k8s-training-tools:v5","imagePullPolicy":"Always","name":"shell-in-pod"}]}}
```

```shell
kubectl annotate pod yaml-pod super.mycompany.com/ci-build-number=722
#    pod/yaml-pod annotated
```

```shell
kubectl annotate pod yaml-pod --list
#    super.mycompany.com/ci-build-number=722
#    kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"from-descriptor":"yaml","run":"yaml-pod"},"name":"yaml-pod","namespace":"default"},"spec":{"containers":[{"image":"traefik/whoami:v1.8","name":"yaml-pod","ports":[{"containerPort":80}]},{"args":["infinity"],"command":["sleep"],"image":"bitboxtraining/k8s-training-tools:v5","imagePullPolicy":"Always","name":"shell-in-pod"}]}}
```

### Namespaces

```shell
kubectl get namespaces
#    NAME                   STATUS   AGE
#    default                Active   60m
#    kube-node-lease        Active   60m
#    kube-public            Active   60m
#    kube-system            Active   60m
#    kubernetes-dashboard   Active   49m
```

```shell
kubectl get pods --namespace kube-system
#    NAME                               READY   STATUS    RESTARTS      AGE
#    coredns-76f75df574-xfddl           1/1     Running   2 (60m ago)   60m
#    etcd-minikube                      1/1     Running   0             60m
#    kindnet-96xfc                      1/1     Running   0             59m
#    kindnet-9tdgv                      1/1     Running   0             60m
#    kindnet-d6kvc                      1/1     Running   0             60m
#    kube-apiserver-minikube            1/1     Running   0             60m
#    kube-controller-manager-minikube   1/1     Running   0             60m
#    kube-proxy-59cj6                   1/1     Running   0             60m
#    kube-proxy-8h9dn                   1/1     Running   0             59m
#    kube-proxy-llbqc                   1/1     Running   0             60m
#    kube-scheduler-minikube            1/1     Running   0             60m
#    metrics-server-7c66d45ddc-pvbjd    1/1     Running   0             57m
#    storage-provisioner                1/1     Running   1 (60m ago)   60m
```

```shell
kubectl get pods --all-namespaces
#    NAMESPACE              NAME                                         READY   STATUS    RESTARTS      AGE
#    default                light-sleeper                                1/1     Running   0             3m25s
#    default                training-shell                               1/1     Running   0             51m
#    default                whoami                                       1/1     Running   0             53m
#    default                yaml-pod                                     2/2     Running   0             13m
#    kube-system            coredns-76f75df574-xfddl                     1/1     Running   2 (61m ago)   61m
#    kube-system            etcd-minikube                                1/1     Running   0             61m
#    kube-system            kindnet-96xfc                                1/1     Running   0             60m
#    kube-system            kindnet-9tdgv                                1/1     Running   0             61m
#    kube-system            kindnet-d6kvc                                1/1     Running   0             61m
#    kube-system            kube-apiserver-minikube                      1/1     Running   0             61m
#    kube-system            kube-controller-manager-minikube             1/1     Running   0             61m
#    kube-system            kube-proxy-59cj6                             1/1     Running   0             61m
#    kube-system            kube-proxy-8h9dn                             1/1     Running   0             60m
#    kube-system            kube-proxy-llbqc                             1/1     Running   0             61m
#    kube-system            kube-scheduler-minikube                      1/1     Running   0             61m
#    kube-system            metrics-server-7c66d45ddc-pvbjd              1/1     Running   0             57m
#    kube-system            storage-provisioner                          1/1     Running   1 (60m ago)   61m
#    kubernetes-dashboard   dashboard-metrics-scraper-7fd5cb4ddc-h99ql   1/1     Running   0             50m
#    kubernetes-dashboard   kubernetes-dashboard-8694d4445c-k9xv7        1/1     Running   0             50m
```

```shell
kubectl get pods --selector run=yaml-pod --all-namespaces
#    NAMESPACE    NAME       READY   STATUS              RESTARTS   AGE
#    default      yaml-pod   2/2     Running             0          14m
#    my-sandbox   yaml-pod   0/2     ContainerCreating   0          4s
```

### Creating a new Namespace

```shell
kubectl apply -f ~/workspaces/corrections/Lab2/ns--my-sandbox.yml
#    namespace/my-sandbox created
```

### Using the new Namespace

```shell
kubectl apply -f ~/workspaces/corrections/Lab2/pod--source-yaml-multi-container-in-namespace.yml
#    pod/yaml-pod created
```

```shell
kubectl get pods -n my-sandbox
#    NAME       READY   STATUS    RESTARTS   AGE
#    yaml-pod   2/2     Running   0          18s
```

```shell
kubectl delete namespace my-sandbox
#    namespace "my-sandbox" deleted
```

## Lab 2.4: logs

### Getting Started

```shell
kubectl logs --help
#    Print the logs for a container in a pod or specified resource. If the pod has only one container, the container name is
#    optional.
#
#    Examples:
#      # Return snapshot logs from pod nginx with only one container
#      kubectl logs nginx
#
#      # Return snapshot logs from pod nginx with multi containers
#      kubectl logs nginx --all-containers=true
#
#      # Return snapshot logs from all containers in pods defined by label app=nginx
#      kubectl logs -lapp=nginx --all-containers=true
```

```shell
kubectl apply -f ~/workspaces/Lab2/pod--whoami-and-clock--2.4.yml
#    pod/whoami-and-clock created
```

```shell
kubectl logs whoami-and-clock -c clock
#    Sun Feb 18 10:38:40 UTC 2024
#    Sun Feb 18 10:38:40 UTC 2024
#    Sun Feb 18 10:38:41 UTC 2024
#    Sun Feb 18 10:38:41 UTC 2024
#    Sun Feb 18 10:38:42 UTC 2024
#    Sun Feb 18 10:38:42 UTC 2024
#    Sun Feb 18 10:38:43 UTC 2024
```

```shell
kubectl logs whoami-and-clock -c clock --follow --tail 10
#    Sun Feb 18 10:39:11 UTC 2024
#    Sun Feb 18 10:39:11 UTC 2024
#    Sun Feb 18 10:39:12 UTC 2024
#    Sun Feb 18 10:39:12 UTC 2024
#    Sun Feb 18 10:39:13 UTC 2024
#    Sun Feb 18 10:39:13 UTC 2024
#    Sun Feb 18 10:39:14 UTC 2024
#    Sun Feb 18 10:39:14 UTC 2024
#    Sun Feb 18 10:39:15 UTC 2024
#    Sun Feb 18 10:39:15 UTC 2024
#    Sun Feb 18 10:39:16 UTC 2024
#    Sun Feb 18 10:39:16 UTC 2024
#    Sun Feb 18 10:39:17 UTC 2024
#    Sun Feb 18 10:39:17 UTC 2024
#    [...]
```

## Lab 2.5: Init Containers

### Experiments

```shell
kubectl explain pod.spec.initContainers
#    KIND:       Pod
#    VERSION:    v1
#
#    FIELD: initContainers <[]Container>
#
#
#    DESCRIPTION:
#        List of initialization containers belonging to the pod. Init containers are
#        executed in order prior to containers being started. If any init container
#        fails, the pod is considered to have failed and is handled according to its
#        restartPolicy. The name for an init container or normal container must be
#        unique among all containers. Init containers may not have Lifecycle actions,
#        Readiness probes, Liveness probes, or Startup probes. The
#        resourceRequirements of an init container are taken into account during
#        scheduling by finding the highest request/limit for each resource type, and
#        then using the max of of that value or the sum of the normal containers.
#        Limits are applied to init containers in a similar fashion. Init containers
#        cannot currently be added or removed. Cannot be updated. More info:
#        https://kubernetes.io/docs/concepts/workloads/pods/init-containers/
#        A single application container that you want to run within a pod.
#    [...]
```

```shell
kubectl apply -f ~/workspaces/corrections/Lab2/pod--whoami-and-clock-with-init-containers.yml
#    pod/whoami-and-clock-with-init created
```

```shell
kubectl get pod whoami-and-clock-with-init
#    NAME                         READY   STATUS     RESTARTS   AGE
#    whoami-and-clock-with-init   0/1     Init:0/1   0          2s
```

```shell
kubectl logs whoami-and-clock-with-init -c timer
#    Sun Feb 18 10:40:00 UTC 2024
#    Sun Feb 18 10:40:01 UTC 2024
#    Sun Feb 18 10:40:02 UTC 2024
#    Sun Feb 18 10:40:03 UTC 2024
#    Sun Feb 18 10:40:04 UTC 2024
#    Sun Feb 18 10:40:05 UTC 2024
#    Sun Feb 18 10:40:06 UTC 2024
#    Sun Feb 18 10:40:07 UTC 2024
#    Sun Feb 18 10:40:08 UTC 2024
#    Sun Feb 18 10:40:09 UTC 2024
#    Sun Feb 18 10:40:10 UTC 2024
#    Sun Feb 18 10:40:11 UTC 2024
#    Sun Feb 18 10:40:12 UTC 2024
#    Sun Feb 18 10:40:13 UTC 2024
#    Sun Feb 18 10:40:14 UTC 2024
```

## Lab 2.6: Life cycle

```shell
minikube ssh --node minikube-m02 -- crictl ps --label io.kubernetes.pod.name=whoami-and-clock --label io.kubernetes.container.name=clock
#    CONTAINER           IMAGE                                                                            CREATED             STATE               NAME                ATTEMPT             POD ID              POD
#    420f05b6b0b99       debian@sha256:c6d9e246479d56687c1a579a7a0336956a5ce6f2bc26bd7925b0c7405e81dbff   2 minutes ago       Running             clock               0                   4a1db3be9c141       whoami-and-clock
```

```shell
kubectl get pod whoami-and-clock
#    NAME               READY   STATUS    RESTARTS   AGE
#    whoami-and-clock   2/2     Running   0          3m5s
```

```shell
CONTAINER_ID=$(minikube ssh --node minikube-m02 -- crictl ps --quiet --label io.kubernetes.pod.name=whoami-and-clock --label io.kubernetes.container.name=clock | tr -d '\r')
echo ${CONTAINER_ID}
#    420f05b6b0b9915060a1f186cb29e83397c596f2fdc704e04b8621ea41a88a62
minikube ssh --node minikube-m02 -- crictl stop ${CONTAINER_ID}
#    420f05b6b0b9915060a1f186cb29e83397c596f2fdc704e04b8621ea41a88a62
```

```shell
kubectl get pod whoami-and-clock
#    NAME               READY   STATUS    RESTARTS     AGE
#    whoami-and-clock   2/2     Running   1 (9s ago)   5m58s
```

### Restart Policy

```shell
kubectl apply -f ~/workspaces/Lab2/pod--restart-policy-rp-check--2.6.yml
#    pod/rp-check created
```

```shell
kubectl get pod rp-check
#    NAME       READY   STATUS      RESTARTS      AGE
#    rp-check   0/1     Completed   2 (39s ago)   71s
```

Notice the restart counter increment.

```shell
kubectl delete pod rp-check
#    pod "rp-check" deleted
```

```shell
kubectl apply -f ~/workspaces/corrections/Lab2/pod--restart-policy-rp-check--2.6.yml
#    pod/rp-check created
```

```shell
kubectl get pod rp-check
#    NAME       READY   STATUS      RESTARTS   AGE
#    rp-check   0/1     Completed   0          43s
```

Notice the number of restarts at 0 but the status as Completed.


```shell
kubectl delete pod rp-check
#    pod "rp-check" deleted
```

```shell
kubectl apply -f ~/workspaces/corrections/Lab2/pod--restart-policy-rp-check-and-false--2.6.yml
#    pod/rp-check created
```

```shell
kubectl get pod rp-check
#    NAME       READY   STATUS    RESTARTS      AGE
#    rp-check   1/1     Running   2 (18s ago)   50s
```

Note the increase in the restart counter again.

## Lab 2.7: Liveness probes

### Liveness

```shell
kubectl apply -f ~/workspaces/corrections/Lab2/pod--liveness-http-probe.yml
#    pod/liveness-http created
```
Image documentation: https://github.com/kubernetes/kubernetes/tree/master/test/images/agnhost#liveness

```shell
kubectl get pod liveness-http
#    NAME            READY   STATUS    RESTARTS   AGE
#    liveness-http   1/1     Running   0          18s

kubectl get pod liveness-http
#    NAME            READY   STATUS    RESTARTS     AGE
#    liveness-http   1/1     Running   1 (4s ago)   34s

kubectl get pod liveness-http
#    NAME            READY   STATUS    RESTARTS     AGE
#    liveness-http   1/1     Running   2 (5s ago)   65s
```

Notice the restart counter increment.

```shell
kubectl describe pod liveness-http
#    [...]
#    Events:
#      Type     Reason     Age                From               Message
#      ----     ------     ----               ----               -------
#      Normal   Scheduled  78s                default-scheduler  Successfully assigned default/liveness-http to minikube-m03
#      Normal   Pulling    78s                kubelet            Pulling image "registry.k8s.io/e2e-test-images/agnhost:2.21"
#      Normal   Pulled     74s                kubelet            Successfully pulled image "registry.k8s.io/e2e-test-images/agnhost:2.21" in 4.796s (4.796s including waiting)
#      Normal   Created    19s (x3 over 73s)  kubelet            Created container i-am-alive
#      Warning  Unhealthy  19s (x4 over 59s)  kubelet            Liveness probe failed: HTTP probe failed with statuscode: 500
#      Normal   Killing    19s (x2 over 49s)  kubelet            Container i-am-alive failed liveness probe, will be restarted
#      Normal   Pulled     19s (x2 over 49s)  kubelet            Container image "registry.k8s.io/e2e-test-images/agnhost:2.21" already present on machine
#      Normal   Started    18s (x3 over 73s)  kubelet            Started container i-am-alive
```
