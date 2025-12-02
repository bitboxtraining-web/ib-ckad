
# Lab 4: Services

## Lab 4.1: Services and Service Discovery

### Preparation

```shell
kubectl create namespace shopping
#    namespace/shopping created

kubectl ns shopping
#    Context "minikube" modified.
#    Active namespace is "shopping".
```

```shell
kubectl apply -f ~/workspaces/Lab4/rs--whoami--4.1.yml
#    replicaset.apps/whoami created
```

### First Service creation

```shell
kubectl apply -f ~/workspaces/corrections/Lab4/svc--whoami.yml
#    service/whoami created
```

```shell
kubectl get svc whoami
#    NAME     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
#    whoami   ClusterIP   10.107.142.50   <none>        8080/TCP   3s
```

```shell
kubectl describe svc whoami
#    Name:              whoami
#    Namespace:         shopping
#    Labels:            stack=whoami
#    Annotations:       <none>
#    Selector:          app=whoami,level=expert
#    Type:              ClusterIP
#    IP Family Policy:  SingleStack
#    IP Families:       IPv4
#    IP:                10.107.142.50
#    IPs:               10.107.142.50
#    Port:              main  8080/TCP
#    TargetPort:        main-port/TCP
#    Endpoints:         10.244.1.22:80,10.244.2.16:80
#    Session Affinity:  None
#    Events:            <none>
```

```shell
export WHOAMI_IP=$(kubectl get svc whoami -o go-template --template '{{ .spec.clusterIP }}')
echo ${WHOAMI_IP}
#    10.107.142.50
```

### Create a gateway Pod

```shell
kubectl apply -f ~/workspaces/Lab4/pod--gateway.yml
#    pod/gateway created
```

```shell
kubectl exec gateway -- ping -c 1 -W 1 ${WHOAMI_IP}
#    PING 10.107.142.50 (10.107.142.50) 56(84) bytes of data.
#
#    --- 10.107.142.50 ping statistics ---
#    1 packets transmitted, 0 received, 100% packet loss, time 0ms
#
#    command terminated with exit code 1

kubectl exec gateway -- curl -s ${WHOAMI_IP}:8080/api
#    {"hostname":"whoami-g6pgx","ip":["127.0.0.1","10.244.2.16"],"headers":{"Accept":["*/*"],"User-Agent":["curl/7.88.1"]},"url":"/api","host":"10.107.142.50:8080","method":"GET"}
```

### Does it still work when you kill the Pods?

```shell
kubectl exec gateway -- curl -s ${WHOAMI_IP}:8080/api | jq -r .hostname
#    whoami-g6pgx
kubectl exec gateway -- curl -s ${WHOAMI_IP}:8080/api | jq -r .hostname
#    whoami-5xzws
kubectl exec gateway -- curl -s ${WHOAMI_IP}:8080/api | jq -r .hostname
#    whoami-5xzws
kubectl exec gateway -- curl -s ${WHOAMI_IP}:8080/api | jq -r .hostname
#    whoami-g6pgx
kubectl exec gateway -- curl -s ${WHOAMI_IP}:8080/api | jq -r .hostname
#    whoami-5xzws
```

```shell
kubectl delete pods -l app=whoami
#    pod "whoami-5xzws" deleted
#    pod "whoami-g6pgx" deleted
```

```shell
kubectl exec gateway -- curl -s ${WHOAMI_IP}:8080/api | jq -r .hostname
#    whoami-9f5x5
kubectl exec gateway -- curl -s ${WHOAMI_IP}:8080/api | jq -r .hostname
#    whoami-z89rh
kubectl exec gateway -- curl -s ${WHOAMI_IP}:8080/api | jq -r .hostname
#    whoami-z89rh
kubectl exec gateway -- curl -s ${WHOAMI_IP}:8080/api | jq -r .hostname
#    whoami-9f5x5
```

### What if we increase the number of replicas?

```shell
kubectl scale --replicas=5 rs whoami
#    replicaset.apps/whoami scaled
```

```shell
kubectl exec gateway -- curl -s ${WHOAMI_IP}:8080/api | jq -r .hostname
#    whoami-kd5sm
kubectl exec gateway -- curl -s ${WHOAMI_IP}:8080/api | jq -r .hostname
#    whoami-z89rh
kubectl exec gateway -- curl -s ${WHOAMI_IP}:8080/api | jq -r .hostname
#    whoami-pjtg4
kubectl exec gateway -- curl -s ${WHOAMI_IP}:8080/api | jq -r .hostname
#    whoami-hh9t2
kubectl exec gateway -- curl -s ${WHOAMI_IP}:8080/api | jq -r .hostname
#    whoami-9f5x5
```

### Using DNS

```shell
kubectl exec gateway -- curl -s whoami:8080/api
#    {"hostname":"whoami-kd5sm","ip":["127.0.0.1","10.244.1.25"],"headers":{"Accept":["*/*"],"User-Agent":["curl/7.88.1"]},"url":"/api","host":"whoami:8080","method":"GET"}

kubectl exec gateway -- curl -s whoami.shopping.svc.cluster.local:8080/api
#    {"hostname":"whoami-z89rh","ip":["127.0.0.1","10.244.1.24"],"headers":{"Accept":["*/*"],"User-Agent":["curl/7.88.1"]},"url":"/api","host":"whoami.shopping.svc.cluster.local:8080","method":"GET"}
```

```shell
kubectl exec gateway -- nslookup whoami
kubectl exec gateway -- nslookup whoami.shopping
kubectl exec gateway -- nslookup whoami.shopping.svc
kubectl exec gateway -- nslookup whoami.shopping.svc.cluster.local
#    Server:         10.96.0.10
#    Address:        10.96.0.10#53
#
#    Name:   whoami.shopping.svc.cluster.local
#    Address: 10.107.142.50
```

### Service with multiple ports (Optional)

```shell
kubectl apply -f ~/workspaces/Lab4/rs--multi-ports.yml
#    replicaset.apps/multi-ports created
```

```shell
kubectl apply -f ~/workspaces/corrections/Lab4/svc--multi-ports.yml
#    service/multi-ports created
```

```shell
export MULTI_PORTS_IP=$(kubectl get svc multi-ports -o go-template --template '{{ .spec.clusterIP }}')
echo ${MULTI_PORTS_IP}
#    10.99.75.121

kubectl exec gateway -- curl -s ${MULTI_PORTS_IP}:80
#    web Sun Feb 18 11:14:54 UTC 2024

kubectl exec gateway -- curl -s ${MULTI_PORTS_IP}:8000
#    admin 2024-02-18T11:14:54+00:00
```

## Lab 4.2: NodePort and LoadBalancer

### NodePort

```shell
kubectl apply -f ~/workspaces/corrections/Lab4/svc--whoami-nodeport.yml
#    service/whoami configured
```

```shell
kubectl get svc whoami
#    NAME     TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
#    whoami   NodePort   10.107.142.50   <none>        8080:30123/TCP   8m22s
```

```shell
WHOAMI_NODE_PORT=$(kubectl get svc -n shopping whoami -ojsonpath="{.spec.ports[0].nodePort}")
echo ${WHOAMI_NODE_PORT}
#    30123
curl $(minikube ip):${WHOAMI_NODE_PORT}/api
#    {"hostname":"whoami-pjtg4","ip":["127.0.0.1","10.244.0.8"],"headers":{"Accept":["*/*"],"User-Agent":["curl/7.68.0"]},"url":"/api","host":"192.168.49.2:30123","method":"GET"}
```

```shell
docker container run --name expose-port-${WHOAMI_NODE_PORT} --detach --network minikube --publish ${WHOAMI_NODE_PORT}:${WHOAMI_NODE_PORT} alpine/socat tcp-listen:${WHOAMI_NODE_PORT},fork,reuseaddr tcp-connect:minikube:${WHOAMI_NODE_PORT}
#    23633170ab51bbff5641aba13080ef23ea746de03bc9a33e88697f773e64a115
curl localhost:${WHOAMI_NODE_PORT}/api
#    {"hostname":"whoami-hh9t2","ip":["127.0.0.1","10.244.2.18"],"headers":{"Accept":["*/*"],"User-Agent":["curl/7.68.0"]},"url":"/api","host":"localhost:30123","method":"GET"}
```

```shell
kubectl describe svc whoami
#    Name:                     whoami
#    Namespace:                shopping
#    Labels:                   stack=whoami
#    Annotations:              <none>
#    Selector:                 app=whoami,level=expert
#    Type:                     NodePort
#    IP Family Policy:         SingleStack
#    IP Families:              IPv4
#    IP:                       10.107.142.50
#    IPs:                      10.107.142.50
#    Port:                     <unset>  8080/TCP
#    TargetPort:               80/TCP
#    NodePort:                 <unset>  30123/TCP
#    Endpoints:                10.244.0.8:80,10.244.1.24:80,10.244.1.25:80 + 2 more...
#    Session Affinity:         None
#    External Traffic Policy:  Cluster
#    Events:                   <none>
```

### LoadBalancer

```shell
minikube ip
#    192.168.49.2
minikube addons configure metallb
#    -- Enter Load Balancer Start IP: 192.168.49.10
#    -- Enter Load Balancer End IP: 192.168.49.255
#        ▪ Using image quay.io/metallb/controller:v0.9.6
#        ▪ Using image quay.io/metallb/speaker:v0.9.6
#    ✅  metallb was successfully configured
```

```shell
minikube addons enable metallb
#    ❗  metallb is a 3rd party addon and is not maintained or verified by minikube maintainers, enable at your own risk.
#    ❗  metallb does not currently have an associated maintainer.
#        ▪ Using image quay.io/metallb/speaker:v0.9.6
#        ▪ Using image quay.io/metallb/controller:v0.9.6
#    🌟  The 'metallb' addon is enabled
```

```shell
kubectl delete svc whoami
#    service "whoami" deleted
```

```shell
kubectl apply -f ~/workspaces/corrections/Lab4/svc--whoami-loadbalancer.yml
#    service/whoami created
```

```shell
kubectl get svc whoami
#    NAME     TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)          AGE
#    whoami   LoadBalancer   10.110.218.67   192.168.49.10   8080:30620/TCP   5s
```

```shell
export WHOAMI_EXTERNAL_IP=$(kubectl get svc whoami -o json | jq -r .status.loadBalancer.ingress[].ip)
echo ${WHOAMI_EXTERNAL_IP}
#    192.168.49.10
curl ${WHOAMI_EXTERNAL_IP}:8080/api
#    {"hostname":"whoami-pjtg4","ip":["127.0.0.1","10.244.0.8"],"headers":{"Accept":["*/*"],"User-Agent":["curl/7.68.0"]},"url":"/api","host":"192.168.49.10:8080","method":"GET"}
```

```shell
kubectl describe svc whoami
#    Name:                     whoami
#    Namespace:                shopping
#    Labels:                   stack=whoami
#    Annotations:              <none>
#    Selector:                 app=whoami,level=expert
#    Type:                     LoadBalancer
#    IP Family Policy:         SingleStack
#    IP Families:              IPv4
#    IP:                       10.110.218.67
#    IPs:                      10.110.218.67
#    LoadBalancer Ingress:     192.168.49.10
#    Port:                     <unset>  8080/TCP
#    TargetPort:               80/TCP
#    NodePort:                 <unset>  30620/TCP
#    Endpoints:                10.244.0.8:80,10.244.1.24:80,10.244.1.25:80 + 2 more...
#    Session Affinity:         None
#    External Traffic Policy:  Cluster
#    Events:
#      Type    Reason        Age   From                Message
#      ----    ------        ----  ----                -------
#      Normal  IPAllocated   3m3s  metallb-controller  Assigned IP "192.168.49.10"
#      Normal  nodeAssigned  3m3s  metallb-speaker     announcing from node "minikube-m02"
```

## Lab 4.3: Ingress

### Ingress Controller

```shell
minikube addons enable ingress
#    💡  ingress is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
#    You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
#        ▪ Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.4.0
#        ▪ Using image registry.k8s.io/ingress-nginx/controller:v1.10.0
#        ▪ Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.4.0
#    🔎  Verifying ingress addon...
#    🌟  The 'ingress' addon is enabled

kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
#    pod/ingress-nginx-controller-76c7c4c767-zff9l condition met

kubectl get pod,svc,deploy,rs,job -n ingress-nginx
#    NAME                                            READY   STATUS      RESTARTS   AGE
#    pod/ingress-nginx-admission-create-6rj7j        0/1     Completed   0          4m
#    pod/ingress-nginx-admission-patch-s6rwl         0/1     Completed   1          4m
#    pod/ingress-nginx-controller-76c7c4c767-zff9l   1/1     Running     0          4m
#
#    NAME                                         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
#    service/ingress-nginx-controller             NodePort    10.109.169.107   <none>        80:32537/TCP,443:32029/TCP   4m1s
#    service/ingress-nginx-controller-admission   ClusterIP   10.107.85.194    <none>        443/TCP                      4m
#
#    NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
#    deployment.apps/ingress-nginx-controller   1/1     1            1           4m
#
#    NAME                                                  DESIRED   CURRENT   READY   AGE
#    replicaset.apps/ingress-nginx-controller-76c7c4c767   1         1         1       4m
#
#    NAME                                       COMPLETIONS   DURATION   AGE
#    job.batch/ingress-nginx-admission-create   1/1           7s         4m
#    job.batch/ingress-nginx-admission-patch    1/1           8s         4m

docker container run --name expose-ingress-controller --detach --network minikube --publish 80:80 alpine/socat tcp-listen:80,fork,reuseaddr tcp-connect:minikube:80
#    8b994ea4c199092c1b5e83cdb97478d651618b587230c7e4a506de5e193aca8c
```

### Ingress whoami

```shell
export MINIKUBE_IP=${PUBLIC_IP}
# Ou
# export MINIKUBE_IP=$(minikube ip)
echo ${MINIKUBE_IP}
```

```shell
sed "s/FIXME/${MINIKUBE_IP}/" ~/workspaces/corrections/Lab4/ingress--whoami.yml | kubectl apply -f -
#    ingress.networking.k8s.io/whoami created
```

```shell
kubectl get ingress
#    NAME     CLASS   HOSTS                         ADDRESS   PORTS   AGE
#    whoami   nginx   whoami.51.44.6.238.sslip.io             80      3s
```

```shell
curl whoami.${MINIKUBE_IP}.sslip.io
#    Hostname: whoami-hh9t2
#    IP: 127.0.0.1
#    IP: 10.244.2.18
#    RemoteAddr: 10.244.0.11:57592
#    GET / HTTP/1.1
#    Host: whoami.51.44.6.238.sslip.io
#    User-Agent: curl/7.68.0
#    Accept: */*
#    X-Forwarded-For: 192.168.49.6
#    X-Forwarded-Host: whoami.51.44.6.238.sslip.io
#    X-Forwarded-Port: 80
#    X-Forwarded-Proto: http
#    X-Forwarded-Scheme: http
#    X-Real-Ip: 192.168.49.6
#    X-Request-Id: 4bbe5ba0a2a15ada222fd64f517087b8
#    X-Scheme: http
```

```shell
kubectl logs -n ingress-nginx --selector app.kubernetes.io/name=ingress-nginx
#    W0218 12:10:03.371026       1 client_config.go:618] Neither --kubeconfig nor --master was specified.  Using the inClusterConfig.  This might not work.
#    {"err":"secrets \"ingress-nginx-admission\" not found","level":"info","msg":"no secret found","source":"k8s/k8s.go:229","time":"2024-02-18T12:10:03Z"}
#    {"level":"info","msg":"creating new secret","source":"cmd/create.go:28","time":"2024-02-18T12:10:03Z"}
#    W0218 12:10:04.282119       1 client_config.go:618] Neither --kubeconfig nor --master was specified.  Using the inClusterConfig.  This might not work.
#    {"level":"info","msg":"patching webhook configurations 'ingress-nginx-admission' mutating=false, validating=true, failurePolicy=Fail","source":"k8s/k8s.go:118","time":"2024-02-18T12:10:04Z"}
#    {"level":"info","msg":"Patched hook(s)","source":"k8s/k8s.go:138","time":"2024-02-18T12:10:04Z"}
#    -------------------------------------------------------------------------------
#    NGINX Ingress controller
#      Release:       v1.10.0
#      Build:         71f78d49f0a496c31d4c19f095469f3f23900f8a
#      Repository:    https://github.com/kubernetes/ingress-nginx
#      nginx version: nginx/1.25.3
#
#    -------------------------------------------------------------------------------
#
#    W0218 12:10:16.511796       6 client_config.go:618] Neither --kubeconfig nor --master was specified.  Using the inClusterConfig.  This might not work.
#    I0218 12:10:16.511952       6 main.go:205] "Creating API client" host="https://10.96.0.1:443"
#    I0218 12:10:16.521887       6 main.go:249] "Running in Kubernetes cluster" major="1" minor="29" git="v1.30.0" state="clean" commit="3f7a50f38688eb332e2a1b013678c6435d539ae6" platform="linux/amd64"
#    I0218 12:10:16.716660       6 main.go:101] "SSL fake certificate created" file="/etc/ingress-controller/ssl/default-fake-certificate.pem"
#    I0218 12:10:16.737013       6 ssl.go:536] "loading tls certificate" path="/usr/local/certificates/cert" key="/usr/local/certificates/key"
#    I0218 12:10:16.748979       6 nginx.go:260] "Starting NGINX Ingress controller"
#    I0218 12:10:16.767659       6 event.go:298] Event(v1.ObjectReference{Kind:"ConfigMap", Namespace:"ingress-nginx", Name:"ingress-nginx-controller", UID:"6a7236d0-0906-4fa2-8be8-e4ca28d3c6ab", APIVersion:"v1", ResourceVersion:"11250", FieldPath:""}): type: 'Normal' reason: 'CREATE' ConfigMap ingress-nginx/ingress-nginx-controller
#    I0218 12:10:16.773081       6 event.go:298] Event(v1.ObjectReference{Kind:"ConfigMap", Namespace:"ingress-nginx", Name:"tcp-services", UID:"0717d2f6-15a2-42c6-a712-99d8010a388c", APIVersion:"v1", ResourceVersion:"11251", FieldPath:""}): type: 'Normal' reason: 'CREATE' ConfigMap ingress-nginx/tcp-services
#    I0218 12:10:16.773341       6 event.go:298] Event(v1.ObjectReference{Kind:"ConfigMap", Namespace:"ingress-nginx", Name:"udp-services", UID:"53e170b7-82e0-4919-8d6a-bf3025fce0e0", APIVersion:"v1", ResourceVersion:"11252", FieldPath:""}): type: 'Normal' reason: 'CREATE' ConfigMap ingress-nginx/udp-services
#    I0218 12:10:17.951388       6 nginx.go:303] "Starting NGINX process"
#    I0218 12:10:17.951643       6 leaderelection.go:245] attempting to acquire leader lease ingress-nginx/ingress-nginx-leader...
#    I0218 12:10:17.951946       6 nginx.go:323] "Starting validation webhook" address=":8443" certPath="/usr/local/certificates/cert" keyPath="/usr/local/certificates/key"
#    I0218 12:10:17.954462       6 controller.go:190] "Configuration changes detected, backend reload required"
#    I0218 12:10:17.964695       6 leaderelection.go:255] successfully acquired lease ingress-nginx/ingress-nginx-leader
#    I0218 12:10:17.965092       6 status.go:84] "New leader elected" identity="ingress-nginx-controller-76c7c4c767-zff9l"
#    I0218 12:10:17.977796       6 status.go:219] "POD is not ready" pod="ingress-nginx/ingress-nginx-controller-76c7c4c767-zff9l" node="minikube"
#    I0218 12:10:18.050988       6 controller.go:210] "Backend successfully reloaded"
#    I0218 12:10:18.051232       6 controller.go:221] "Initial sync, sleeping for 1 second"
#    I0218 12:10:18.051251       6 event.go:298] Event(v1.ObjectReference{Kind:"Pod", Namespace:"ingress-nginx", Name:"ingress-nginx-controller-76c7c4c767-zff9l", UID:"2dbc00af-a66f-4e98-999a-5b6560d75ab2", APIVersion:"v1", ResourceVersion:"11290", FieldPath:""}): type: 'Normal' reason: 'RELOAD' NGINX reload triggered due to a change in configuration
#    I0218 12:15:12.440105       6 admission.go:149] processed ingress via admission controller {testedIngressLength:1 testedIngressTime:0.054s renderingIngressLength:1 renderingIngressTime:0s admissionTime:18.0kBs testedConfigurationSize:0.054}
#    I0218 12:15:12.440154       6 main.go:107] "successfully validated configuration, accepting" ingress="shopping/whoami"
#    I0218 12:15:12.450729       6 store.go:440] "Found valid IngressClass" ingress="shopping/whoami" ingressclass="nginx"
#    I0218 12:15:12.451179       6 event.go:298] Event(v1.ObjectReference{Kind:"Ingress", Namespace:"shopping", Name:"whoami", UID:"4413c6ab-d927-4c2a-9096-473cdb6d887b", APIVersion:"networking.k8s.io/v1", ResourceVersion:"11707", FieldPath:""}): type: 'Normal' reason: 'Sync' Scheduled for sync
#    I0218 12:15:12.451917       6 controller.go:190] "Configuration changes detected, backend reload required"
#    I0218 12:15:12.568152       6 controller.go:210] "Backend successfully reloaded"
#    I0218 12:15:12.568487       6 event.go:298] Event(v1.ObjectReference{Kind:"Pod", Namespace:"ingress-nginx", Name:"ingress-nginx-controller-76c7c4c767-zff9l", UID:"2dbc00af-a66f-4e98-999a-5b6560d75ab2", APIVersion:"v1", ResourceVersion:"11290", FieldPath:""}): type: 'Normal' reason: 'RELOAD' NGINX reload triggered due to a change in configuration
#    I0218 12:15:17.976329       6 status.go:304] "updating Ingress status" namespace="shopping" ingress="whoami" currentValue=null newValue=[{"ip":"192.168.49.2"}]
#    I0218 12:15:17.984209       6 event.go:298] Event(v1.ObjectReference{Kind:"Ingress", Namespace:"shopping", Name:"whoami", UID:"4413c6ab-d927-4c2a-9096-473cdb6d887b", APIVersion:"networking.k8s.io/v1", ResourceVersion:"11713", FieldPath:""}): type: 'Normal' reason: 'Sync' Scheduled for sync
#    192.168.49.6 - - [18/Feb/2024:12:15:30 +0000] "GET / HTTP/1.1" 200 410 "-" "curl/7.68.0" 89 0.001 [shopping-whoami-8080] [] 10.244.2.18:80 410 0.001 200 4bbe5ba0a2a15ada222fd64f517087b8
#    192.168.49.6 - - [18/Feb/2024:12:16:25 +0000] "GET / HTTP/1.1" 200 618 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:122.0) Gecko/20100101 Firefox/122.0" 321 0.001 [shopping-whoami-8080] [] 10.244.2.18:80 618 0.001 200 07f872d1f13c69b16009be22ebc88aa5
#    192.168.49.6 - - [18/Feb/2024:12:16:27 +0000] "GET / HTTP/1.1" 200 687 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:122.0) Gecko/20100101 Firefox/122.0" 390 0.046 [shopping-whoami-8080] [] 10.244.1.25:80 687 0.047 200 621e458d5255012c3150eb58e824f0fe
#    192.168.49.6 - - [18/Feb/2024:12:16:53 +0000] "GET / HTTP/1.1" 200 409 "-" "curl/7.68.0" 89 0.001 [shopping-whoami-8080] [] 10.244.0.8:80 409 0.000 200 348b438d53be2a5a30edd40679a2fe80
#    192.168.49.6 - - [18/Feb/2024:12:16:54 +0000] "GET / HTTP/1.1" 200 410 "-" "curl/7.68.0" 89 0.004 [shopping-whoami-8080] [] 10.244.1.24:80 410 0.003 200 b98e9f1496d79812d8f2401309fa4186
#    192.168.49.6 - - [18/Feb/2024:12:16:56 +0000] "GET / HTTP/1.1" 200 687 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:122.0) Gecko/20100101 Firefox/122.0" 390 0.001 [shopping-whoami-8080] [] 10.244.2.18:80 687 0.001 200 6f8c6b1c0efe2f201654985241cb8972
#    192.168.49.6 - - [18/Feb/2024:12:16:57 +0000] "GET / HTTP/1.1" 200 687 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:122.0) Gecko/20100101 Firefox/122.0" 390 0.001 [shopping-whoami-8080] [] 10.244.1.25:80 687 0.000 200 b04cd8279fc0278169fee32f097eebe2
```

## Lab 4.4: Readiness Probe

### Simulate errors on the Pods of the ReplicaSet whoami

```shell
kubectl exec gateway -- curl -s --head whoami:8080/health
#    HTTP/1.1 200 OK
#    Date: Sun, 18 Feb 2024 12:18:56 GMT
```

```shell
kubectl get endpoints whoami --output jsonpath='{range .subsets[0].addresses[*]}{.ip}{"\n"}{end}' | while read ip; do echo "Send 500 to ${ip}/health"; kubectl exec gateway -- curl -si --data 500 ${ip}/health ; done
#    Send 500 to 10.244.0.8/health
#    HTTP/1.1 200 OK
#    Date: Sun, 18 Feb 2024 12:19:09 GMT
#    Content-Length: 0
#
#    Send 500 to 10.244.1.24/health
#    HTTP/1.1 200 OK
#    Date: Sun, 18 Feb 2024 12:19:09 GMT
#    Content-Length: 0
#
#    Send 500 to 10.244.1.25/health
#    HTTP/1.1 200 OK
#    Date: Sun, 18 Feb 2024 12:19:10 GMT
#    Content-Length: 0
#
#    Send 500 to 10.244.2.17/health
#    HTTP/1.1 200 OK
#    Date: Sun, 18 Feb 2024 12:19:10 GMT
#    Content-Length: 0
#
#    Send 500 to 10.244.2.18/health
#    HTTP/1.1 200 OK
#    Date: Sun, 18 Feb 2024 12:19:10 GMT
#    Content-Length: 0
```

```shell
kubectl exec gateway -- curl -s --head whoami:8080/health
#    HTTP/1.1 500 Internal Server Error
#    Date: Sun, 18 Feb 2024 12:19:37 GMT
```

### Addition of a Readiness probe on the Pods of the whoami ReplicaSet

```shell
kubectl apply -f ~/workspaces/corrections/Lab4/rs--whoami-with-readiness.yml
#    replicaset.apps/whoami configured
```

The modification of the ReplicaSet has no impact on existing Pods which do not have the probe and so they stay __READY__

```shell
kubectl delete pod -l app=whoami
#    pod "whoami-hh9t2" deleted
#    pod "whoami-z89rh" deleted
```

```shell
kubectl get pods -l app=whoami
#    NAME           READY   STATUS    RESTARTS   AGE
#    whoami-m5zgp   1/1     Running   0          53s
#    whoami-vfgx7   1/1     Running   0          53s
```

```shell
kubectl exec gateway -- curl -s --data 500 whoami:8080/health
```

```shell
kubectl get pods -l app=whoami
#    NAME           READY   STATUS    RESTARTS   AGE
#    whoami-m5zgp   0/1     Running   0          104s
#    whoami-vfgx7   1/1     Running   0          104s
```

```shell
kubectl exec gateway -- curl -s --data 500 whoami:8080/health
```

```shell
kubectl get pods -l app=whoami
#    NAME           READY   STATUS    RESTARTS   AGE
#    whoami-m5zgp   0/1     Running   0          2m36s
#    whoami-vfgx7   0/1     Running   0          2m36s
```

```shell
WHOAMI_POD_IP=$(kubectl get pods -l app=whoami -o jsonpath --template='{.items[0].status.podIP}')
kubectl exec gateway -- curl -s --data 200 ${WHOAMI_POD_IP}:80/health
```

```shell
kubectl get pods -l app=whoami
#    NAME           READY   STATUS    RESTARTS   AGE
#    whoami-m5zgp   1/1     Running   0          3m11s
#    whoami-vfgx7   0/1     Running   0          3m11s
```

```shell
kubectl get pods -l app=whoami --output jsonpath='{.items[*].status.podIP}' \
  | kubectl exec --stdin gateway -- sh -c \
  'for pod in $(cat /dev/stdin); do curl -s --data 200 ${pod}:80/health; done'

kubectl get pods -l app=whoami
#    NAME           READY   STATUS    RESTARTS   AGE
#    whoami-m5zgp   1/1     Running   0          3m31s
#    whoami-vfgx7   1/1     Running   0          3m31s
```

## Lab 4.5: Headless services

### Restore previous configuration for whoami ReplicaSet

```shell
kubectl apply -f ~/workspaces/Lab4/rs--whoami--4.1.yml
#    replicaset.apps/whoami configured
```

```shell
kubectl delete pod -l app=whoami
#    pod "whoami-m5zgp" deleted
#    pod "whoami-vfgx7" deleted
```

### whoami headless

```shell
kubectl apply -f ~/workspaces/corrections/Lab4/svc--whoami-headless.yml
#    service/whoami-headless created
```

```shell
kubectl exec gateway -- nslookup whoami-headless
#    Server:         10.96.0.10
#    Address:        10.96.0.10#53
#
#    Name:   whoami-headless.shopping.svc.cluster.local
#    Address: 10.244.2.21
#    Name:   whoami-headless.shopping.svc.cluster.local
#    Address: 10.244.1.30
```

## Lab 4.6: Port forward

```shell
kubectl port-forward --help
#    Forward one or more local ports to a pod. This command requires the node to have 'socat' installed.
#
#     Use resource type/name such as deployment/mydeployment to select a pod. Resource type shoppings to 'pod' if omitted.
#
#     If there are multiple pods matching the criteria, a pod will be selected automatically. The forwarding session ends
#    when the selected pod terminates, and rerun of the command is needed to resume forwarding.
#
#    Examples:
#      # Listen on ports 5000 and 6000 locally, forwarding data to/from ports 5000 and 6000 in the pod
#      kubectl port-forward pod/mypod 5000 6000
#
#      # Listen on ports 5000 and 6000 locally, forwarding data to/from ports 5000 and 6000 in a pod selected by the
#    deployment
#      kubectl port-forward deployment/mydeployment 5000 6000
#    [...]
```

```shell
kubectl run whoami --image=traefik/whoami:v1.10 --port=80
#   pod/whoami created
```

```shell
kubectl port-forward pod/whoami 8888:80 &
#    Forwarding from 127.0.0.1:8888 -> 80
#    Forwarding from [::1]:8888 -> 80
```

```shell
curl http://localhost:8888/api
#    {"hostname":"whoami","ip":["127.0.0.1","10.244.2.22"],"headers":{"Accept":["*/*"],"User-Agent":["curl/7.68.0"]},"url":"/api","host":"localhost:8888","method":"GET"}
```

```shell
kubectl delete pod whoami
#    pod "whoami" deleted
```

Do not forget to stop the `kubectl port-forward` afterwards.

```shell
killall kubectl
#    [1]+  Terminated              kubectl port-forward pod/whoami 8888:80
```

```shell
kubectl ns default
#    Context "minikube" modified.
#    Active namespace is "default"
```
