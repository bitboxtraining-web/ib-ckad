
# Lab 1: First steps

## Lab 1.1: minikube usage

### minikube startup

```shell
minikube start --kubernetes-version 1.30.0 --nodes 3
#    😄  minikube v1.33.0 on Ubuntu 20.04
#        ▪ MINIKUBE_WANTREPORTERRORPROMPT=false
#        ▪ MINIKUBE_KUBERNETES_VERSION=1.29.0
#        ▪ MINIKUBE_DRIVER=docker
#        ▪ MINIKUBE_IN_STYLE=true
#        ▪ MINIKUBE_WANTUPDATENOTIFICATION=false
#    ✨  Using the docker driver based on user configuration
#    📌  Using Docker driver with root privileges
#    👍  Starting "minikube" primary control-plane node in "minikube" cluster
#    🚜  Pulling base image v0.0.43 ...
#    💾  Downloading Kubernetes v1.30.0 preload ...
#        > preloaded-images-k8s-v18-v1...:  342.90 MiB / 342.90 MiB  100.00% 40.01 M
#        > gcr.io/k8s-minikube/kicbase...:  480.29 MiB / 480.29 MiB  100.00% 31.27 M
#    🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
#    🐳  Preparing Kubernetes v1.30.0 on Docker 26.0.1 ...
#        ▪ Generating certificates and keys ...
#        ▪ Booting up control plane ...
#        ▪ Configuring RBAC rules ...
#    🔗  Configuring CNI (Container Networking Interface) ...
#    🔎  Verifying Kubernetes components...
#        ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
#    🌟  Enabled addons: storage-provisioner, default-storageclass
#
#    👍  Starting "minikube-m02" worker node in "minikube" cluster
#    🚜  Pulling base image v0.0.43 ...
#    🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
#    🌐  Found network options:
#        ▪ NO_PROXY=192.168.49.2
#    🐳  Preparing Kubernetes v1.30.0 on Docker 26.0.1 ...
#        ▪ env NO_PROXY=192.168.49.2
#    🔎  Verifying Kubernetes components...
#
#    👍  Starting "minikube-m03" worker node in "minikube" cluster
#    🚜  Pulling base image v0.0.43 ...
#    🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
#    🌐  Found network options:
#        ▪ NO_PROXY=192.168.49.2,192.168.49.3
#    🐳  Preparing Kubernetes v1.30.0 on Docker 26.0.1 ...
#        ▪ env NO_PROXY=192.168.49.2
#        ▪ env NO_PROXY=192.168.49.2,192.168.49.3
#    🔎  Verifying Kubernetes components...
#    🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

```shell
minikube status
#    minikube
#    type: Control Plane
#    host: Running
#    kubelet: Running
#    apiserver: Running
#    kubeconfig: Configured
#
#    minikube-m02
#    type: Worker
#    host: Running
#    kubelet: Running
#
#    minikube-m03
#    type: Worker
#    host: Running
#    kubelet: Running
```

### Addons management

```shell
minikube addons list
#    |-----------------------------|----------|--------------|--------------------------------|
#    |         ADDON NAME          | PROFILE  |    STATUS    |           MAINTAINER           |
#    |-----------------------------|----------|--------------|--------------------------------|
#    | ambassador                  | minikube | disabled     | 3rd party (Ambassador)         |
#    | auto-pause                  | minikube | disabled     | minikube                       |
#    | cloud-spanner               | minikube | disabled     | Google                         |
#    | csi-hostpath-driver         | minikube | disabled     | Kubernetes                     |
#    | dashboard                   | minikube | disabled     | Kubernetes                     |
#    | default-storageclass        | minikube | disabled     | Kubernetes                     |
#    | efk                         | minikube | disabled     | 3rd party (Elastic)            |
#    | freshpod                    | minikube | disabled     | Google                         |
#    | gcp-auth                    | minikube | disabled     | Google                         |
#    | gvisor                      | minikube | disabled     | minikube                       |
#    | headlamp                    | minikube | disabled     | 3rd party (kinvolk.io)         |
#    | helm-tiller                 | minikube | disabled     | 3rd party (Helm)               |
#    | inaccel                     | minikube | disabled     | 3rd party (InAccel             |
#    |                             |          |              | [info@inaccel.com])            |
#    | ingress                     | minikube | disabled     | Kubernetes                     |
#    | ingress-dns                 | minikube | disabled     | minikube                       |
#    | inspektor-gadget            | minikube | disabled     | 3rd party                      |
#    |                             |          |              | (inspektor-gadget.io)          |
#    | istio                       | minikube | disabled     | 3rd party (Istio)              |
#    | istio-provisioner           | minikube | disabled     | 3rd party (Istio)              |
#    | kong                        | minikube | disabled     | 3rd party (Kong HQ)            |
#    | kubeflow                    | minikube | disabled     | 3rd party                      |
#    | kubevirt                    | minikube | disabled     | 3rd party (KubeVirt)           |
#    | logviewer                   | minikube | disabled     | 3rd party (unknown)            |
#    | metallb                     | minikube | disabled     | 3rd party (MetalLB)            |
#    | metrics-server              | minikube | disabled     | Kubernetes                     |
#    | nvidia-device-plugin        | minikube | disabled     | 3rd party (NVIDIA)             |
#    | nvidia-driver-installer     | minikube | disabled     | 3rd party (Nvidia)             |
#    | nvidia-gpu-device-plugin    | minikube | disabled     | 3rd party (Nvidia)             |
#    | olm                         | minikube | disabled     | 3rd party (Operator Framework) |
#    | pod-security-policy         | minikube | disabled     | 3rd party (unknown)            |
#    | portainer                   | minikube | disabled     | 3rd party (Portainer.io)       |
#    | registry                    | minikube | disabled     | minikube                       |
#    | registry-aliases            | minikube | disabled     | 3rd party (unknown)            |
#    | registry-creds              | minikube | disabled     | 3rd party (UPMC Enterprises)   |
#    | storage-provisioner         | minikube | disabled     | minikube                       |
#    | storage-provisioner-gluster | minikube | disabled     | 3rd party (Gluster)            |
#    | storage-provisioner-rancher | minikube | disabled     | 3rd party (Rancher)            |
#    | volumesnapshots             | minikube | disabled     | Kubernetes                     |
#    | yakd                        | minikube | disabled     | 3rd party (marcnuri.com)       |
#    |-----------------------------|----------|--------------|--------------------------------|
```

```shell
minikube addons enable metrics-server
#    💡  metrics-server is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
#    You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
#        ▪ Using image registry.k8s.io/metrics-server/metrics-server:v0.7.1
#    🌟  The 'metrics-server' addon is enabled
```

## Lab 1.2: kubectl

### Basic usage

```shell
kubectl
#    kubectl controls the Kubernetes cluster manager.
#
#     Find more information at: https://kubernetes.io/docs/reference/kubectl/
#
#    Basic Commands (Beginner):
#      create          Create a resource from a file or from stdin
#      expose          Take a replication controller, service, deployment or pod and
#    expose it as a new Kubernetes service
#      run             Run a particular image on the cluster
#      set             Set specific features on objects
#
#    Basic Commands (Intermediate):
#      explain         Get documentation for a resource
#      get             Display one or many resources
#      edit            Edit a resource on the server
#      delete          Delete resources by file names, stdin, resources and names, or
#    by resources and label selector
#
#    Deploy Commands:
#      rollout         Manage the rollout of a resource
#      scale           Set a new size for a deployment, replica set, or replication
#    controller
#      autoscale       Auto-scale a deployment, replica set, stateful set, or
#    replication controller
#
#    Cluster Management Commands:
#      certificate     Modify certificate resources
#      cluster-info    Display cluster information
#      top             Display resource (CPU/memory) usage
#      cordon          Mark node as unschedulable
#      uncordon        Mark node as schedulable
#      drain           Drain node in preparation for maintenance
#      taint           Update the taints on one or more nodes
#
#    Troubleshooting and Debugging Commands:
#      describe        Show details of a specific resource or group of resources
#      logs            Print the logs for a container in a pod
#      attach          Attach to a running container
#      exec            Execute a command in a container
#      port-forward    Forward one or more local ports to a pod
#      proxy           Run a proxy to the Kubernetes API server
#      cp              Copy files and directories to and from containers
#      auth            Inspect authorization
#      debug           Create debugging sessions for troubleshooting workloads and
#    nodes
#      events          List events
#
#    Advanced Commands:
#      diff            Diff the live version against a would-be applied version
#      apply           Apply a configuration to a resource by file name or stdin
#      patch           Update fields of a resource
#      replace         Replace a resource by file name or stdin
#      wait            Experimental: Wait for a specific condition on one or many
#    resources
#      kustomize       Build a kustomization target from a directory or URL
#
#    Settings Commands:
#      label           Update the labels on a resource
#      annotate        Update the annotations on a resource
#      completion      Output shell completion code for the specified shell (bash,
#    zsh, fish, or powershell)
#
#    Subcommands provided by plugins:
#      ctx           The command ctx is a plugin installed by the user
#      ns            The command ns is a plugin installed by the user
#      krew          The command krew is a plugin installed by the user
#
#    Other Commands:
#      api-resources   Print the supported API resources on the server
#      api-versions    Print the supported API versions on the server, in the form of
#    "group/version"
#      config          Modify kubeconfig files
#      plugin          Provides utilities for interacting with plugins
#      version         Print the client and server version information
#
#    Usage:
#      kubectl [flags] [options]
#
#    Use "kubectl <command> --help" for more information about a given command.
#    Use "kubectl options" for a list of global command-line options (applies to all
#    commands).
```

```shell
kubectl <TAB><TAB>
#    annotate       (Update the annotations on a resource)
#    api-resources  (Print the supported API resources on the server)
#    api-versions   (Print the supported API versions on the server, in the form of "gro…)
#    apply          (Apply a configuration to a resource by file name or stdin)
#    attach         (Attach to a running container)
#    auth           (Inspect authorization)
#    autoscale      (Auto-scale a deployment, replica set, stateful set, or replication …)
#    certificate    (Modify certificate resources)
#    cluster-info   (Display cluster information)
#    completion     (Output shell completion code for the specified shell (bash, zsh, fi…)
#    config         (Modify kubeconfig files)
#    cordon         (Mark node as unschedulable)
#    cp             (Copy files and directories to and from containers)
#    create         (Create a resource from a file or from stdin)
#    ctx            (The command ctx is a plugin installed by the user)
#    debug          (Create debugging sessions for troubleshooting workloads and nodes)
#    delete         (Delete resources by file names, stdin, resources and names, or by r…)
#    describe       (Show details of a specific resource or group of resources)
#    diff           (Diff the live version against a would-be applied version)
#    drain          (Drain node in preparation for maintenance)
#    edit           (Edit a resource on the server)
#    events         (List events)
#    exec           (Execute a command in a container)
#    explain        (Get documentation for a resource)
#    expose         (Take a replication controller, service, deployment or pod and expos…)
#    get            (Display one or many resources)
#    help           (Help about any command)
#    krew           (The command krew is a plugin installed by the user)
#    kustomize      (Build a kustomization target from a directory or URL)
#    label          (Update the labels on a resource)
#    logs           (Print the logs for a container in a pod)
#    ns             (The command ns is a plugin installed by the user)
#    options        (Print the list of flags inherited by all commands)
#    patch          (Update fields of a resource)
#    plugin         (Provides utilities for interacting with plugins)
#    port-forward   (Forward one or more local ports to a pod)
#    proxy          (Run a proxy to the Kubernetes API server)
#    replace        (Replace a resource by file name or stdin)
#    rollout        (Manage the rollout of a resource)
#    run            (Run a particular image on the cluster)
#    scale          (Set a new size for a deployment, replica set, or replication contro…)
#    set            (Set specific features on objects)
#    taint          (Update the taints on one or more nodes)
#    top            (Display resource (CPU/memory) usage)
#    uncordon       (Mark node as schedulable)
#    version        (Print the client and server version information)
#    wait           (Experimental: Wait for a specific condition on one or many resources)
```

```shell
kubectl version --output yaml
#    clientVersion:
#      buildDate: "2024-04-17T17:36:05Z"
#      compiler: gc
#      gitCommit: 7c48c2bd72b9bf5c44d21d7338cc7bea77d0ad2a
#      gitTreeState: clean
#      gitVersion: v1.30.0
#      goVersion: go1.22.2
#      major: "1"
#      minor: "30"
#      platform: linux/amd64
#    kustomizeVersion: v5.0.4-0.20230601165947-6ce0bf390ce3
#    serverVersion:
#      buildDate: "2024-04-17T17:27:03Z"
#      compiler: gc
#      gitCommit: 7c48c2bd72b9bf5c44d21d7338cc7bea77d0ad2a
#      gitTreeState: clean
#      gitVersion: v1.30.0
#      goVersion: go1.22.2
#      major: "1"
#      minor: "30"
#      platform: linux/amd64
```

```shell
kubectl options
#    The following options can be passed to any command:
#
#        --as='':
#            Username to impersonate for the operation. User could be a regular
#            user or a service account in a namespace.
#
#        --as-group=[]:
#            Group to impersonate for the operation, this flag can be repeated to
#            specify multiple groups.
#
#        --as-uid='':
#            UID to impersonate for the operation.
#
#        --cache-dir='/home/ubuntu/.kube/cache':
#            Default cache directory
#
#        --certificate-authority='':
#            Path to a cert file for the certificate authority
#
#        --client-certificate='':
#            Path to a client certificate file for TLS
#
#        --client-key='':
#            Path to a client key file for TLS
#
#        --cluster='':
#            The name of the kubeconfig cluster to use
#
#        --context='':
#            The name of the kubeconfig context to use
#
#        --disable-compression=false:
#            If true, opt-out of response compression for all requests to the
#            server
#
#        --insecure-skip-tls-verify=false:
#            If true, the server's certificate will not be checked for validity.
#            This will make your HTTPS connections insecure
#
#        --kubeconfig='':
#            Path to the kubeconfig file to use for CLI requests.
#
#        --log-flush-frequency=5s:
#            Maximum number of seconds between log flushes
#
#        --match-server-version=false:
#            Require server version to match client version
#
#        -n, --namespace='':
#            If present, the namespace scope for this CLI request
#
#        --password='':
#            Password for basic authentication to the API server
#
#        --profile='none':
#            Name of profile to capture. One of
#            (none|cpu|heap|goroutine|threadcreate|block|mutex)
#
#        --profile-output='profile.pprof':
#            Name of the file to write the profile to
#
#        --request-timeout='0':
#            The length of time to wait before giving up on a single server
#            request. Non-zero values should contain a corresponding time unit
#            (e.g. 1s, 2m, 3h). A value of zero means don't timeout requests.
#
#        -s, --server='':
#            The address and port of the Kubernetes API server
#
#        --tls-server-name='':
#            Server name to use for server certificate validation. If it is not
#            provided, the hostname used to contact the server is used
#
#        --token='':
#            Bearer token for authentication to the API server
#
#        --user='':
#            The name of the kubeconfig user to use
#
#        --username='':
#            Username for basic authentication to the API server
#
#        -v, --v=0:
#            number for the log level verbosity
#
#        --vmodule=:
#            comma-separated list of pattern=N settings for file-filtered logging
#            (only works for the default text log format)
#
#        --warnings-as-errors=false:
#            Treat warnings received from the server as errors and exit with a
#            non-zero exit code
```

### Cluster info

```shell
kubectl config current-context
#    minikube
```

```shell
kubectl get nodes
#    NAME           STATUS   ROLES           AGE     VERSION
#    minikube       Ready    control-plane   5m10s   v1.30.0
#    minikube-m02   Ready    <none>          4m34s   v1.30.0
#    minikube-m03   Ready    <none>          4m8s    v1.30.0
```

### Checking available API groups and resources

```shell
kubectl api-resources
#    NAME                                SHORTNAMES   APIVERSION                        NAMESPACED   KIND
#    bindings                                         v1                                true         Binding
#    componentstatuses                   cs           v1                                false        ComponentStatus
#    configmaps                          cm           v1                                true         ConfigMap
#    endpoints                           ep           v1                                true         Endpoints
#    events                              ev           v1                                true         Event
#    limitranges                         limits       v1                                true         LimitRange
#    namespaces                          ns           v1                                false        Namespace
#    nodes                               no           v1                                false        Node
#    persistentvolumeclaims              pvc          v1                                true         PersistentVolumeClaim
#    persistentvolumes                   pv           v1                                false        PersistentVolume
#    pods                                po           v1                                true         Pod
#    podtemplates                                     v1                                true         PodTemplate
#    replicationcontrollers              rc           v1                                true         ReplicationController
#    resourcequotas                      quota        v1                                true         ResourceQuota
#    secrets                                          v1                                true         Secret
#    serviceaccounts                     sa           v1                                true         ServiceAccount
#    services                            svc          v1                                true         Service
#    mutatingwebhookconfigurations                    admissionregistration.k8s.io/v1   false        MutatingWebhookConfiguration
#    validatingadmissionpolicies                      admissionregistration.k8s.io/v1   false        ValidatingAdmissionPolicy
#    validatingadmissionpolicybindings                admissionregistration.k8s.io/v1   false        ValidatingAdmissionPolicyBinding
#    validatingwebhookconfigurations                  admissionregistration.k8s.io/v1   false        ValidatingWebhookConfiguration
#    customresourcedefinitions           crd,crds     apiextensions.k8s.io/v1           false        CustomResourceDefinition
#    apiservices                                      apiregistration.k8s.io/v1         false        APIService
#    controllerrevisions                              apps/v1                           true         ControllerRevision
#    daemonsets                          ds           apps/v1                           true         DaemonSet
#    deployments                         deploy       apps/v1                           true         Deployment
#    replicasets                         rs           apps/v1                           true         ReplicaSet
#    statefulsets                        sts          apps/v1                           true         StatefulSet
#    selfsubjectreviews                               authentication.k8s.io/v1          false        SelfSubjectReview
#    tokenreviews                                     authentication.k8s.io/v1          false        TokenReview
#    localsubjectaccessreviews                        authorization.k8s.io/v1           true         LocalSubjectAccessReview
#    selfsubjectaccessreviews                         authorization.k8s.io/v1           false        SelfSubjectAccessReview
#    selfsubjectrulesreviews                          authorization.k8s.io/v1           false        SelfSubjectRulesReview
#    subjectaccessreviews                             authorization.k8s.io/v1           false        SubjectAccessReview
#    horizontalpodautoscalers            hpa          autoscaling/v2                    true         HorizontalPodAutoscaler
#    cronjobs                            cj           batch/v1                          true         CronJob
#    jobs                                             batch/v1                          true         Job
#    certificatesigningrequests          csr          certificates.k8s.io/v1            false        CertificateSigningRequest
#    leases                                           coordination.k8s.io/v1            true         Lease
#    endpointslices                                   discovery.k8s.io/v1               true         EndpointSlice
#    events                              ev           events.k8s.io/v1                  true         Event
#    flowschemas                                      flowcontrol.apiserver.k8s.io/v1   false        FlowSchema
#    prioritylevelconfigurations                      flowcontrol.apiserver.k8s.io/v1   false        PriorityLevelConfiguration
#    nodes                                            metrics.k8s.io/v1beta1            false        NodeMetrics
#    pods                                             metrics.k8s.io/v1beta1            true         PodMetrics
#    ingressclasses                                   networking.k8s.io/v1              false        IngressClass
#    ingresses                           ing          networking.k8s.io/v1              true         Ingress
#    networkpolicies                     netpol       networking.k8s.io/v1              true         NetworkPolicy
#    runtimeclasses                                   node.k8s.io/v1                    false        RuntimeClass
#    poddisruptionbudgets                pdb          policy/v1                         true         PodDisruptionBudget
#    clusterrolebindings                              rbac.authorization.k8s.io/v1      false        ClusterRoleBinding
#    clusterroles                                     rbac.authorization.k8s.io/v1      false        ClusterRole
#    rolebindings                                     rbac.authorization.k8s.io/v1      true         RoleBinding
#    roles                                            rbac.authorization.k8s.io/v1      true         Role
#    priorityclasses                     pc           scheduling.k8s.io/v1              false        PriorityClass
#    csidrivers                                       storage.k8s.io/v1                 false        CSIDriver
#    csinodes                                         storage.k8s.io/v1                 false        CSINode
#    csistoragecapacities                             storage.k8s.io/v1                 true         CSIStorageCapacity
#    storageclasses                      sc           storage.k8s.io/v1                 false        StorageClass
#    volumeattachments                                storage.k8s.io/v1                 false        VolumeAttachment
```

## Lab 1.3: kubectl run

### Basic usage

```shell
kubectl run --help
#    Create and run a particular image in a pod.
#
#    Examples:
#      # Start a nginx pod.
#      kubectl run nginx --image=nginx
#
#      # Start a hazelcast pod and let the container expose port 5701.
#      kubectl run hazelcast --image=hazelcast/hazelcast --port=5701
#
#      # Start a hazelcast pod and set environment variables "DNS_DOMAIN=cluster" and "POD_NAMESPACE=default" in the
#    container.
#      kubectl run hazelcast --image=hazelcast/hazelcast --env="DNS_DOMAIN=cluster" --env="POD_NAMESPACE=default"
#    [...]
```

### Start a Pod

```shell
kubectl run whoami --image=traefik/whoami:v1.10 --port=80
#    pod/whoami created
```

```shell
kubectl get pods
#    NAME            READY   STATUS             RESTARTS   AGE
#    whoami          1/1     Running            0          7s
```

```shell
kubectl describe pod whoami
#    Name:             whoami
#    Namespace:        default
#    Priority:         0
#    Service Account:  default
#    Node:             minikube-m03/192.168.49.4
#    Start Time:       Sun, 18 Feb 2024 09:42:55 +0000
#    Labels:           run=whoami
#    Annotations:      <none>
#    Status:           Running
#    IP:               10.244.2.2
#    IPs:
#      IP:  10.244.2.2
#    Containers:
#      whoami:
#        Container ID:   docker://626db96998c6cf93ac074823630c1999d050129ecbba8bf6aa688cc3e6d56a51
#        Image:          traefik/whoami:v1.10
#        Image ID:       docker-pullable://traefik/whoami@sha256:24829edb0dbaea072dabd7d902769168542403a8c78a6f743676af431166d7f0
#        Port:           80/TCP
#        Host Port:      0/TCP
#        State:          Running
#          Started:      Sun, 18 Feb 2024 09:42:58 +0000
#        Ready:          True
#        Restart Count:  0
#        Environment:    <none>
#        Mounts:
#          /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-dkd4z (ro)
#    Conditions:
#      Type                        Status
#      PodReadyToStartContainers   True 
#      Initialized                 True 
#      Ready                       True 
#      ContainersReady             True 
#      PodScheduled                True 
#    Volumes:
#      kube-api-access-dkd4z:
#        Type:                    Projected (a volume that contains injected data from multiple sources)
#        TokenExpirationSeconds:  3607
#        ConfigMapName:           kube-root-ca.crt
#        ConfigMapOptional:       <nil>
#        DownwardAPI:             true
#    QoS Class:                   BestEffort
#    Node-Selectors:              <none>
#    Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
#                                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
#    Events:
#      Type    Reason     Age   From               Message
#      ----    ------     ----  ----               -------
#      Normal  Scheduled  13s   default-scheduler  Successfully assigned default/whoami to minikube-m03
#      Normal  Pulling    13s   kubelet            Pulling image "traefik/whoami:v1.10"
#      Normal  Pulled     10s   kubelet            Successfully pulled image "traefik/whoami:v1.10" in 2.437s (2.437s including waiting)
#      Normal  Created    10s   kubelet            Created container whoami
#      Normal  Started    10s   kubelet            Started container whoami

WHOAMI_POD_IP=$(kubectl get -o go-template --template='{{.status.podIP}}' pod whoami)
echo ${WHOAMI_POD_IP}
#    10.244.2.2
```

```shell
minikube ssh -- curl ${WHOAMI_POD_IP}:80/api
#    {"hostname":"whoami","ip":["127.0.0.1","10.244.2.2"],"headers":{"Accept":["*/*"],"User-Agent":["curl/7.81.0"]},"url":"/api","host":"10.244.2.2","method":"GET"}
```

### Troubleshooting Pod startup

```shell
kubectl run faulty-whoami --image=traefik/whoami:nil --port=80
#    pod/faulty-whoami created
```

```shell
kubectl get pods
#    NAME            READY   STATUS             RESTARTS   AGE
#    faulty-whoami   0/1     ImagePullBackOff   0          15s
#    whoami          1/1     Running            0          1m
```

The Pod shows an `ImagePullBackOff` error status

```shell
kubectl describe pod faulty-whoami
#    Name:             faulty-whoami
#    Namespace:        default
#    Priority:         0
#    Service Account:  default
#    Node:             minikube-m02/192.168.49.3
#    Start Time:       Sun, 18 Feb 2024 09:44:46 +0000
#    Labels:           run=faulty-whoami
#    Annotations:      <none>
#    Status:           Pending
#    IP:               10.244.1.3
#    IPs:
#      IP:  10.244.1.3
#    Containers:
#      faulty-whoami:
#        Container ID:   
#        Image:          traefik/whoami:nil
#        Image ID:       
#        Port:           80/TCP
#        Host Port:      0/TCP
#        State:          Waiting
#          Reason:       ErrImagePull
#        Ready:          False
#        Restart Count:  0
#        Environment:    <none>
#        Mounts:
#          /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-hmxnp (ro)
#    Conditions:
#      Type                        Status
#      PodReadyToStartContainers   True 
#      Initialized                 True 
#      Ready                       False 
#      ContainersReady             False 
#      PodScheduled                True 
#    Volumes:
#      kube-api-access-hmxnp:
#        Type:                    Projected (a volume that contains injected data from multiple sources)
#        TokenExpirationSeconds:  3607
#        ConfigMapName:           kube-root-ca.crt
#        ConfigMapOptional:       <nil>
#        DownwardAPI:             true
#    QoS Class:                   BestEffort
#    Node-Selectors:              <none>
#    Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
#                                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
#    Events:
#      Type     Reason     Age   From               Message
#      ----     ------     ----  ----               -------
#      Normal   Scheduled  13s   default-scheduler  Successfully assigned default/faulty-whoami to minikube-m02
#      Normal   Pulling    13s   kubelet            Pulling image "traefik/whoami:nil"
#      Warning  Failed     12s   kubelet            Failed to pull image "traefik/whoami:nil": Error response from daemon: manifest for traefik/whoami:nil not found: manifest unknown: manifest unknown
#      Warning  Failed     12s   kubelet            Error: ErrImagePull
#      Normal   BackOff    11s   kubelet            Back-off pulling image "traefik/whoami:nil"
#      Warning  Failed     11s   kubelet            Error: ImagePullBackOff
```

We can see the `ErrImagePull` in the Pod's events.

```shell
kubectl delete pod faulty-whoami
#    pod "faulty-whoami" deleted
```

### Starting a Pod Shell

```shell
kubectl run training-shell --image=bitboxtraining/k8s-training-tools:v5 --command -- sleep infinity
#    pod/training-shell created
```

```shell
kubectl get pod training-shell
#    NAME           READY   STATUS    RESTARTS   AGE
#    training-shell   1/1     Running   0          13s
```

```shell
kubectl get -o name pods
#   pod/training-shell
#   pod/whoami
```

```shell
kubectl exec training-shell -- curl -s ${WHOAMI_POD_IP}:80/api
#    {"hostname":"whoami","ip":["127.0.0.1","10.244.2.2"],"headers":{"Accept":["*/*"],"User-Agent":["curl/7.88.1"]},"url":"/api","host":"10.244.2.2","method":"GET"}
```

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

## k9s

```shell
k9s
```
Use `CTRL+C` to exit.

## Optional: Using the dashboard

```shell
minikube addons enable dashboard
#    💡  dashboard is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
#    You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
#        ▪ Using image docker.io/kubernetesui/dashboard:v2.7.0
#        ▪ Using image docker.io/kubernetesui/metrics-scraper:v1.0.8
#    💡  Some dashboard features require the metrics-server addon. To enable all features please run:
#
#            minikube addons enable metrics-server
#
#
#    🌟  The 'dashboard' addon is enabled
```

```shell
kubectl -n kubernetes-dashboard patch service kubernetes-dashboard --type json --patch \
    '[
      {"op": "replace", "path": "/spec/type", "value": "NodePort"},
      {"op": "add", "path": "/spec/ports/0/nodePort", "value": 30998}
    ]'
#    service/kubernetes-dashboard patched

docker container run --name expose-dashboard --detach --network minikube --publish 9998:9998 alpine/socat tcp-listen:9998,fork,reuseaddr tcp-connect:minikube:30998
#    a4b1edf4bcc199e98aa66117b2cc9db8b4b7dc6800dab81645eeb975480f8d03
```

```shell
echo "Open http://${PUBLIC_DNS}:9998"
#    Open http://Z2iedbcR9R6ZdGHMD-dYmgFvJXRxNz79Cno.labs.strigo.io:9998
```

```shell
docker container rm --force expose-dashboard
#    expose-dashboard
```
