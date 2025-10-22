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

## YAML
  
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

## Commands

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
