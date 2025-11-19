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