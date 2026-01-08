# Kubernetes


Kubernetes is OS of the cloud.

Kubernetes is a bunch of virtual machines, who are able to communicate propertly with each other and to divide their workload.

So Kubernetes is an intelligent way of running container workloads at scale.

## Pod

A pod is the smallest eleemnt on a kubernetes cluster.

A pod is not a container.

A pod is a collection of containers + other resources.


- single container
- multi container
- initcontainer

Pod will contain info about:
  - networking
  - storage
  - ...
  
  
multiples containers are attached to the pod, but the storage is attached to the pod. 
So the containers can all use the same storage

### YAML
  
get a little yaml definition
```
$ kubectl run nginx-yaml --image=nginx --dry-run=client -o yaml

apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx-yaml
  name: nginx-yaml
spec:
  containers:
  - image: nginx
    name: nginx-yaml
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Save in a file:
```
kubectl run nginx-yaml --image=nginx --dry-run=client -o yaml > nginx.yaml
```

Create a pod with a yaml file
```
$ kubectl apply -f nginx.yaml
pod/nginx-yaml created
$ k get pods
NAME           READY   STATUS    RESTARTS      AGE
httpd-sergio   1/1     Running   1 (23h ago)   24h
nginx-sergio   1/1     Running   1 (23h ago)   24h
nginx-yaml     1/1     Running   0             8s
$ k describe pod nginx-yaml
....
 
```

Apply changes

```
$ kubectl apply -f nginx.yaml
pod/nginx-yaml configured
```

### Commands

Get cluster contexts:

```
kubectl config get-contexts
```

```
kubctl config use-context ....
```

Create a Pod
```
kubectl run nginx-sergio --image=nginx
```

Get the yaml

```
k get pod httpd-sergio -o yaml
```

:set paste -> for preserving original format in Vim


To see more info

```
k get pods -o wide
NAME           READY   STATUS    RESTARTS      AGE   IP           NODE                   NOMINATED NODE   READINESS GATES
httpd-sergio   1/1     Running   3 (15m ago)   8d    10.42.0.47   lima-rancher-desktop   <none>           <none>
nginx-sergio   1/1     Running   3 (15m ago)   8d    10.42.0.51   lima-rancher-desktop   <none>           <none>
nginx-yaml     1/1     Running   2 (15m ago)   7d    10.42.0.48   lima-rancher-desktop   <none>           <none>
```

For delete pod

```
k get pods

k delete pod nginx-yaml
```

Execute inside pod

```
k exec -it nginx-sergio -- /bin/bash
```

## Deployments

It is a way of defining the desired state to Kubernetes.

```
k create deployment -h

```

```
~ kubectl create deploy test --image=httpd --replicas=3
deployment.apps/test created
~ kgp
NAME                    READY   STATUS    RESTARTS      AGE
httpd                   1/1     Running   2 (72m ago)   5d
nginx-sergio            1/1     Running   5 (72m ago)   13d
nginx-yaml              1/1     Running   4 (72m ago)   12d
test-6546ccdcf9-56d4r   1/1     Running   0             7s
test-6546ccdcf9-rrszk   1/1     Running   0             7s
test-6546ccdcf9-xn4h9   1/1     Running   0             7s

~ k get deployments
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
test   3/3     3            3           105s

~ k edit deployments.apps test
```

Easy to check the properties:

```
 $ k describe deployments.apps test
Name:                   test
Namespace:              default
CreationTimestamp:      Mon, 03 Nov 2025 20:58:34 +0100
Labels:                 app=test
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=test
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=test
  Containers:
   httpd:
    Image:         httpd
    Port:          <none>
    Host Port:     <none>
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Progressing    True    NewReplicaSetAvailable
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   test-6546ccdcf9 (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  20h   deployment-controller  Scaled up replica set test-6546ccdcf9 from 0 to 3

```


### How we can get YAML files:
1. copying from Kubernetes docs
2. run a comand for creating a pod and generate YAML file without running it on the cluster

```
 $ kgp
NAME           READY   STATUS    RESTARTS        AGE
httpd          1/1     Running   3 (5m34s ago)   5d20h
nginx-sergio   1/1     Running   6 (5m34s ago)   13d
nginx-yaml     1/1     Running   5 (5m34s ago)   12d
 $ k run sergio --image=nginx --dry-run=client -o yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: sergio
  name: sergio
spec:
  containers:
  - image: nginx
    name: sergio
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
 $ kgp
NAME           READY   STATUS    RESTARTS      AGE
httpd          1/1     Running   3 (11m ago)   5d20h
nginx-sergio   1/1     Running   6 (11m ago)   13d
nginx-yaml     1/1     Running   5 (11m ago)   12d
```

### How to create a deployment
Same for deployments:

```
k create deploy test --image=httpd --replicas=10 --dry-run=client -o yaml > deploy.yaml
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: test
  name: test
spec:
  replicas: 10
  selector:
    matchLabels:
      app: test  -> which pod is managing
  strategy: {}
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
      - image: httpd
        name: httpd
        resources: {}
status: {}

```

deployment is to define the desired state.
```
$ k apply -f deploy.yaml
deployment.apps/test created
$ kgp
NAME                    READY   STATUS    RESTARTS      AGE
httpd                   1/1     Running   3 (21m ago)   5d21h
nginx-sergio            1/1     Running   6 (21m ago)   13d
nginx-yaml              1/1     Running   5 (21m ago)   12d
test-6546ccdcf9-6szwk   1/1     Running   0             5s
test-6546ccdcf9-glwq2   1/1     Running   0             5s
test-6546ccdcf9-nsxbr   1/1     Running   0             5s
test-6546ccdcf9-qlqlv   1/1     Running   0             5s
test-6546ccdcf9-smmch   1/1     Running   0             5s
test-6546ccdcf9-svnhx   1/1     Running   0             5s
test-6546ccdcf9-t856j   1/1     Running   0             5s
test-6546ccdcf9-z4q85   1/1     Running   0             5s
test-6546ccdcf9-znxv5   1/1     Running   0             5s
test-6546ccdcf9-zws89   1/1     Running   0             5s
```
The deployment is not actually creating the pods.
The deployment controls the replica set, and a replica set is basically a set of replicas of a certain pod. 

```
$ k describe deployments.apps test
Name:                   test
Namespace:              default
CreationTimestamp:      Tue, 04 Nov 2025 17:41:01 +0100
Labels:                 app=test
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=test
Replicas:               10 desired | 10 updated | 10 total | 10 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=test
  Containers:
   httpd:
    Image:         httpd
    Port:          <none>
    Host Port:     <none>
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   test-6546ccdcf9 (10/10 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  2m4s  deployment-controller  Scaled up replica set test-6546ccdcf9 from 0 to 10
 ~/development/workspaces/lab/kubernetes/deployments $  main ± $ k get replicasets.apps
NAME              DESIRED   CURRENT   READY   AGE
test-6546ccdcf9   10        10        10      2m25s
```


```
$ k describe replicasets.apps test-6546ccdcf9
Name:           test-6546ccdcf9
Namespace:      default
Selector:       app=test,pod-template-hash=6546ccdcf9
Labels:         app=test
                pod-template-hash=6546ccdcf9
Annotations:    deployment.kubernetes.io/desired-replicas: 10
                deployment.kubernetes.io/max-replicas: 13
                deployment.kubernetes.io/revision: 1
Controlled By:  Deployment/test
Replicas:       10 current / 10 desired
Pods Status:    10 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=test
           pod-template-hash=6546ccdcf9
  Containers:
   httpd: 
    Image:         httpd
    Port:          <none>
    Host Port:     <none>
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Events:
  Type    Reason            Age    From                   Message
  ----    ------            ----   ----                   -------
  Normal  SuccessfulCreate  6m19s  replicaset-controller  Created pod: test-6546ccdcf9-nsxbr
  Normal  SuccessfulCreate  6m19s  replicaset-controller  Created pod: test-6546ccdcf9-qlqlv
  Normal  SuccessfulCreate  6m19s  replicaset-controller  Created pod: test-6546ccdcf9-znxv5
  Normal  SuccessfulCreate  6m19s  replicaset-controller  Created pod: test-6546ccdcf9-svnhx
  Normal  SuccessfulCreate  6m19s  replicaset-controller  Created pod: test-6546ccdcf9-z4q85
  Normal  SuccessfulCreate  6m19s  replicaset-controller  Created pod: test-6546ccdcf9-smmch
  Normal  SuccessfulCreate  6m19s  replicaset-controller  Created pod: test-6546ccdcf9-zws89
  Normal  SuccessfulCreate  6m19s  replicaset-controller  Created pod: test-6546ccdcf9-6szwk
  Normal  SuccessfulCreate  6m19s  replicaset-controller  Created pod: test-6546ccdcf9-glwq2
  Normal  SuccessfulCreate  6m19s  replicaset-controller  (combined from similar events): Created pod: test-6546ccdcf9-t856j
  ```
Replicas should never really be managed by people by by the user.
Kubernetes manages replica sets for you.
This is something that is managed by the deployments.

### How to update to a new version without losing service
10 replicas running
Some of the will be updated, but the rest will still running.
Once the updated ones are ready, we will do the same to the rest.

1. change yaml file

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: test
  name: test
spec:
  replicas: 10
  selector:
    matchLabels:
      app: test
  strategy: {}
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
        - image: httpd:alpine3.18
          name: httpd
          resources: {}
status: {}
```

2. Apply the change

```
kubectl apply -f deploy.yaml 
```

to see the trick use 
```
watch -n 1 "kubectl get pods"
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: test
  name: test
spec:
  replicas: 10
  selector:
    matchLabels:
      app: test
  strategy:
    type: RollingUpdate -> it is by default
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1  // max nmber of pods that can be created over the desired number of pods. We have 10 replicas, only 11.
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
        - image: httpd:alpine3.18
          name: httpd
          resources: {}
status: {}
```

Also with %

```
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
```

Apply again

```
kubectl apply -f deploy.yaml
```

With another strategy

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: test
  name: test
spec:
  replicas: 10
  selector:
    matchLabels:
      app: test
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
        - image: httpd:alpine3.22
          name: httpd
          resources: {}
status: {}
```

Kill all pods at same time and recreate them.
```
  strategy:
    type: Recreate
```

#### Something can be wrong

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: test
  name: test
spec:
  replicas: 10
  selector:
    matchLabels:
      app: test
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
        - image: httpd:alpine3.25
          name: httpd
          resources: {}
          command: ["/bin/bash", "-c"] # override the default command
          args: ["sleep 5; exit 1"] # sleep for 30 seconds then exit with an error
status: {}
```

### Namespaces

A mechanism for isoliting groups of resources.

For production cluster, not use default namespace.

Every app should have its own namespace.

```
$ k create namespace mealie
namespace/mealie created
$ k get namespaces
NAME              STATUS   AGE
default           Active   27d
kube-node-lease   Active   27d
kube-public       Active   27d
kube-system       Active   27d
mealie            Active   8s

$ k create namespace mealie -o yaml --dry-run=client
apiVersion: v1
kind: Namespace
metadata:
  name: mealie
spec: {}
status: {}
```

```
$ k create namespace mealie --dry-run=client -o yaml > namespace.yaml

apiVersion: v1
kind: Namespace
metadata:
  name: mealie
spec: {}
status: {}
```

- Delete a namespace
it will delete anything associated to the namespace.
```
$ k delete namespaces mealie
namespace "mealie" deleted
```

- Create a namespace with the yaml file
```
$ k apply -f namespace.yaml
namespace/mealie created
$ k get ns
NAME              STATUS   AGE
default           Active   27d
kube-node-lease   Active   27d
kube-public       Active   27d
kube-system       Active   27d
mealie            Active   5s
```

- Creating a pod within a namespace

```
$ k run mischa --image=nginx
pod/mischa created
$ kgp
NAME     READY   STATUS         RESTARTS   AGE
mischa   1/1     Running   0          3s
$ kubectl get pods -n mealie
No resources found in mealie namespace.
$ kubectl get pods -n default
NAME     READY   STATUS         RESTARTS   AGE
mischa   1/1     Running   0          45s
$ k run mischa-mealie --image=nginx --namespace mealie
pod/mischa-mealie created
$ kgp
NAME     READY   STATUS         RESTARTS   AGE
mischa   0/1     ErrImagePull   0          108s
$ k get pods -n mealie
NAME            READY   STATUS    RESTARTS   AGE
mischa-mealie   1/1     Running   0          38s
```

- set namespace as default

```
$ k config current-context
rancher-desktop
$ k config set-context --current --namespace=mealie
Context "rancher-desktop" modified.
$ kgp
NAME            READY   STATUS    RESTARTS   AGE
mischa-mealie   1/1     Running   0          5m25s
$ kgp -n default
NAME     READY   STATUS    RESTARTS   AGE
mischa   1/1     Running   0          2m10s
$ kgp
NAME            READY   STATUS    RESTARTS   AGE
mischa-mealie   1/1     Running   0          5m45s
```

### First app

create yaml file
```
k create deployment mealie --image=nginx --dry-run=client -o yaml > deployment.yaml
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: mealie
  name: mealie
  namespace: mealie
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mealie
  template:
    metadata:
      labels:
        app: mealie
    spec:
      containers:
      - image: nginx
        name: nginx
```

ghcr.io/mealie-recipes/mealie:<version>


```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: mealie
  name: mealie
  namespace: mealie
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mealie
  template:
    metadata:
      labels:
        app: mealie
    spec:
      containers:
      - image: ghcr.io/mealie-recipes/mealie:v1.2.0
        name: mealie
```

Apply
  
```
k apply -f deployment.yaml
deployment.apps/mealie created
```
  
```
$ k describe pod mealie
```

Port forwarding
```
$ k port-forward pods/mealie-5479dbb894-mk7jw 9000
Forwarding from 127.0.0.1:9000 -> 9000
Forwarding from [::1]:9000 -> 9000
```
  
List deployments

```
$ k get deployments.apps
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
mealie   1/1     1            1           27m
```

Update the version
  
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: mealie
  name: mealie
  namespace: mealie
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mealie
  template:
    metadata:
      labels:
        app: mealie
    spec:
      containers:
      - image: ghcr.io/mealie-recipes/mealie:v3.5.0
        name: mealie
        ports:
          - containerPort: 9000

And apply again

k apply -f deployment.yaml
```
  
For deleting all pod instances created with a deployment:
  
```
kubectl delete deployment name
```

## Networking
### Intro
  
List all pods in all namespaces

kgp -A
k get pods --all-namespaces -o wide

Each pod gets its own IP address.
By default, pods can connect to all pods on all nodes.
Containers in pods can comminicate with each other through localhost.

  
CNI plugin provides network connectivity to containers.
  
  
Implemented by CNI plugins:
  - Cilium
  - Calico
  - Flannel


  
rdctl -h -> Rancher desktop ctl
  
rdctl shell bash -> Rancher desktop vm.

```
$ cd /etc/cni/
/etc/cni $ ls
net.d
/etc/cni $ tree
.
└── net.d
    └── 10-flannel.conflist

1 directories, 1 files
/etc/cni $ cat net.d/10-flannel.conflist
{
  "name":"cbr0",
  "cniVersion":"0.3.1",
  "plugins":[
    {
      "type":"flannel",
      "delegate":{
        "hairpinMode":true,
        "forceAddress":true,
        "isDefaultGateway":true
      }
    },
    {
      "type":"portmap",
      "capabilities":{
        "portMappings":true
      }
    }
  ]
}
```
  
### Services

A service offers a consistent address to access a set of pods.
  
Pods are ephemeral.
  
A service is a group of pods.
  
Pods are constantly changing and being moved across nodes.
  

How will the system keep track of the constantly changing IP addresses? -> service
  
  
-> expose generates the service.

```
$ k expose deployment frontend --port 8080
service/frontend exposed
$ k get service
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
frontend     ClusterIP   10.43.132.58   <none>        8080/TCP   9s
kubernetes   ClusterIP   10.43.0.1      <none>        443/TCP    41d
$ k get service -o wide
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE   SELECTOR
frontend     ClusterIP   10.43.132.58   <none>        8080/TCP   27s   app=frontend
kubernetes   ClusterIP   10.43.0.1      <none>        443/TCP    41d   <none>
```

The cluster ip is 10.43.132.58 and will remain the same.
  
type of services:
  - cluster IP -> default one
  - node port: Exposes a port on each node allowing direct access to the service throught any node's IP address.
  
```
$ k get nodes -o wide
NAME                   STATUS   ROLES                  AGE   VERSION        INTERNAL-IP    EXTERNAL-IP    OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
lima-rancher-desktop   Ready    control-plane,master   47d   v1.33.5+k3s1   192.168.5.15   192.168.64.2   Alpine Linux v3.21   6.6.96-0-virt    docker://27.3.1
```

192.168.64.2  is the external IP.
  
If I have a node port service, I can exposte a port on that node directly. Then I could reach that port from by entering that IP address and the port.
  
Not very common.
  
  - Load balancer: For cloud providers.
  
```
k get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
frontend     ClusterIP   10.43.132.58   <none>        8080/TCP   6d
kubernetes   ClusterIP   10.43.0.1      <none>        443/TCP    47d
  
  
k edit svc frontend
...
  type: ClusterIP
...

  
...
  type: LoadBalancer
...
  
  
$ k get svc
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)          AGE
frontend     LoadBalancer   10.43.132.58   192.168.64.2   8080:30922/TCP   6d
kubernetes   ClusterIP      10.43.0.1      <none>         443/TCP          47d
```  
This is a way how to reach your your services, your applications from outside of your Kubernetes cluster.
  

#### Mealie
doing a port forward of the pod itself.

change to mealie context
```
$ k config set-context --current --namespace=mealie
  
$ kgp
NAME                      READY   STATUS    RESTARTS       AGE
mealie-5d545757cf-275qj   1/1     Running   7 (138m ago)   12d
```

Expose mealie deployment
  
```
k expose deployment mealie --port 9000
service/mealie exposed
$ k get svc
NAME     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
mealie   ClusterIP   10.43.153.184   <none>        9000/TCP   85s
```

Port forward to a service
```
k port-forward services/mealie 9000
Forwarding from 127.0.0.1:9000 -> 9000
Forwarding from [::1]:9000 -> 9000
```

The deployment is exposed through a service instead of doing the pod directly, because if we expose the pod and we kill that pod, the port for remote works.
But if we close terminal, we will not able to access to it.
  
We need a load balancer.

get the yaml of the service
```
k get svc mealie -o yaml > service.yaml
```

```
frontend     LoadBalancer   10.43.132.58   192.168.64.2   8080:30922/TCP   6d
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2025-12-02T19:19:47Z"
  labels:
    app: mealie
  name: mealie
  namespace: mealie
  resourceVersion: "55305"
  uid: dd234a85-5b6f-4b40-8655-e83e1cd04468
spec:
  clusterIP: 10.43.153.184
  clusterIPs:
  - 10.43.153.184
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - port: 9000
    protocol: TCP
    targetPort: 9000
  selector:
    app: mealie
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
~
```

Simplified to
```
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2025-12-02T19:19:47Z"
  labels:
    app: mealie
  name: mealie
  namespace: mealie
spec:
  ports:
  - port: 9000
    protocol: TCP
    targetPort: 9000
  selector:
    app: mealie
  type: ClusterIP
```
type changd to LoadBalancer
  
Kill the existing service
```
$ k get svc
NAME     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
mealie   ClusterIP   10.43.153.184   <none>        9000/TCP   12m
$ k delete svc mealie
service "mealie" deleted from mealie namespace
```

Apply new one:
  
```
$ k apply -f service.yaml
service/mealie created
$ k get svc
NAME     TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)          AGE
mealie   LoadBalancer   10.43.174.220   192.168.64.2   9000:32530/TCP   5s
```

### Ingress
  
It is a resource on the cluster, and it exposes http and https routes from outside the cluster to services within the cluster.

So the cluster is listening to a certain domain, and it has a root configured to root that domain to a service in the cluster.

The cluster is listeing to a certain domain, and it has a route configured to route that domain to a service in the cluster.
  
For an app running we would need a FQDN (fully qualified domain name).

Provides:
  - SSL and TLS termination.
  - external urls
  - path based routing: you will the route.
  
Ingress controller types:
  - nginx
  - traefik
  - cilium
  - cloud: AGIC
  
  
Possible:
  domain in cloudfare pointing to IP
  that IP address is pointing to a kubernetes load balancer created by traffic or nginx
  the traffic is the ingres controller that running on my cluster
  the cluster is going to look at the ingress resources configured, and it is going to look at the ingress
  The ingress has a routing rule to ta service.

  
## Storage
### Ephemeral Storage
So in order for a container to save data somewhere, it needs to have a volume.
It needs to have, in other terms a disk mounted to it.
So a volume is just a piece of the file system where the container is hosted.

  
```
k describe pod mealie-5d545757cf-275qj | less

Volumes:
  kube-api-access-h65zs:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
  
```
  
  
```

apiVersion: v1
kind: Pod
metadata:
  labels:
  name: nginx-storage
spec:
  containers:
    - image: nginx
      name: nginx
      volumeMounts:
        - mountPath: /scratch
          name: scratch-volume
  volumes:
    - name: scratch-volume
      emptyDir:       ---> it will be deleted when pod is deleted
        sizeLimit: 500Mi

  
k apply -f nginx-pod.yaml
pod/nginx-storage created  
  
  
k describe pod nginx-pod
...
Volumes:
  scratch-volume:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  500Mi
  kube-api-access-m895p:
  

  
k exec -it nginx-storage -- bash

root@nginx-storage:/# ls
bin  boot  dev	docker-entrypoint.d  docker-entrypoint.sh  etc	home  lib  media  mnt  opt  proc  root	run  sbin  scratch  srv  sys  tmp  usr	va
  
```
  
```
apiVersion: v1
kind: Pod
metadata:
  labels:
  name: nginx-storage
spec:
  containers:
    - image: nginx
      name: nginx
      volumeMounts:
        - mountPath: /scratch
          name: scratch-volume
    - image: busybox
      name: busybox
      command: ["/bin/sh", "-c"]
      args: ["sleep 1000"]
      volumeMounts:
        - mountPath: /scratch
          name: scratch-volume
  volumes:
    - name: scratch-volume
      emptyDir:
        sizeLimit: 500Mi
  
```
  
**To a running pod, we cannot add a container.**
  
First we need to delete it:
k delete pod nginx-storage
pod "nginx-storage" deleted from mealie namespace
  
```
k apply -f nginx-pod.yaml

 kgp
NAME                      READY   STATUS    RESTARTS       AGE
mealie-5d545757cf-275qj   1/1     Running   16 (73m ago)   28d
nginx-storage             2/2     Running   0              13m

```
1 volumen, 2 containers

```
 k exec -it nginx-storage -c nginx -- bash
root@nginx-storage:/# cd /scratch/
root@nginx-storage:/scratch# ls
root@nginx-storage:/scratch# echo hello > hello.txt
root@nginx-storage:/scratch# cat hello.txt
  
 k exec -it nginx-storage -c busybox -- sh
/ # ls /scratch/
hello.txt
/ # cat /scratch/hello.txt
hello
  
watch -n 1 "ls -la" -> every second run ls -la
```
  
### Persistent Storage

persistent volume are like a huge disck that is living in the cluster where is a huge disk. Every app living in cluster is using a part of this.
that is a persistent volumen claim.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mealie-data
  namespace: mealie
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

as ther is not storage class name, then it takes the one by default.

```
$ k apply -f storage.yaml
persistentvolumeclaim/mealie-data created
$ k get persistentvolume
No resources found
  
$ k get persistentvolumeclaims
NAME          STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
mealie-data   Pending                                      local-path     <unset>                 59s
```

The following code is added in pod yaml:
  
```
        volumenMounts:
          - mountPath: /app/data
            name: mealie-datax
      volumes:
        - name: mealie-datax
          persistentVolumeClaim:
            claimName: mealie-data
```

Then apply:
```
 k get pvc
NAME          STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
mealie-data   Pending                                      local-path     <unset>                 11m

 k apply  -f deployment.yaml
deployment.apps/mealie configured
  
 kgp
NAME                      READY   STATUS    RESTARTS       AGE
mealie-79464fdcc9-lx7gd   1/1     Running   0              43s
nginx-storage             2/2     Running   32 (10m ago)   20d

    main ±  k get pvc
NAME          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
mealie-data   Bound    pvc-49145c9f-c5c1-4be5-a4a7-7a9396b91cc6   500Mi      RWO            local-path     <unset>                 13m

   k get persistentvolume
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pvc-49145c9f-c5c1-4be5-a4a7-7a9396b91cc6   500Mi      RWO            Delete           Bound    mealie/mealie-data   local-path     <unset>                          87s

 k get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pvc-49145c9f-c5c1-4be5-a4a7-7a9396b91cc6   500Mi      RWO            Delete           Bound    mealie/mealie-data   local-path     <unset>                          87s
```


```
 k describe pod mealie
Name:             mealie-79464fdcc9-lx7gd
Namespace:        mealie
Priority:         0
Service Account:  default
Node:             lima-rancher-desktop/192.168.5.15
Start Time:       Thu, 08 Jan 2026 20:37:11 +0100
Labels:           app=mealie
                  pod-template-hash=79464fdcc9
Annotations:      <none>
Status:           Running
IP:               10.42.0.193
IPs:
  IP:           10.42.0.193
Controlled By:  ReplicaSet/mealie-79464fdcc9
Containers:
  mealie:
    Container ID:   docker://bcb0261899bc33df40eff6bc2780775e7ca74da8e5edf71cabad502de7cc5fd0
    Image:          ghcr.io/mealie-recipes/mealie:v3.5.0
    Image ID:       docker-pullable://ghcr.io/mealie-recipes/mealie@sha256:7f776bbb5457db7f58951c11e3aa881f0167675a78459d7a7f2cd5e42d181fa5
    Port:           9000/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 08 Jan 2026 20:37:11 +0100
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /app/data from mealie-datax (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-2zq52 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  mealie-datax:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  mealie-data
    ReadOnly:   false
  kube-api-access-2zq52:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  111s  default-scheduler  Successfully assigned mealie/mealie-79464fdcc9-lx7gd to lima-rancher-desktop
  Normal  Pulled     112s  kubelet            Container image "ghcr.io/mealie-recipes/mealie:v3.5.0" already present on machine
  Normal  Created    112s  kubelet            Created container: mealie
  Normal  Started    112s  kubelet            Started container mealie
```


  
  
## K9s
  
## Helm
  
## Monitoring
  
