# Overview

Linkerd is `kubernetes first`, which means that a basic understanding of Kubernetes is required to work through these exercises.

This module is designed to familiarize users who are new to Kubernetes with the basic kubernetes commands and assumes that the reader already has access to a Kubernetes cluster.

## Check the `kubectl` and `kubernetes` versions
`kubectl version --short`

### Example Output
```
Client Version: v1.15.0
Server Version: v1.15.3
``` 

## View all the namespaces in the cluster
`kubectl get ns`

### Example Output
```
NAME              STATUS   AGE
booksapp          Active   10d
default           Active   18d
emojivoto         Active   16d
kube-node-lease   Active   18d
kube-public       Active   18d
kube-system       Active   18d
linkerd           Active   10d
```

## Change kubectl namespace
`kubectl config set-context --current --namespace=<your-desired-namespace>`

## Get kubernetes resources

### Get all the pods in a namespace
`kubectl get po`

#### Example Output
```
NAME                        READY   STATUS    RESTARTS   AGE
NAME                                      READY   STATUS    RESTARTS   AGE
linkerd-controller-6c8cbbd9d4-fp76c       2/2     Running   0          4d2h
linkerd-destination-7f8bb7c84f-qznx5      2/2     Running   0          4d2h
linkerd-grafana-5f664f94c5-z7ntz          2/2     Running   0          4d2h
linkerd-identity-56bf4597f4-42zjs         2/2     Running   0          4d2h
linkerd-prometheus-f8697996c-r78c9        2/2     Running   0          4d2h
linkerd-proxy-injector-7dbd97d46f-qnxm5   2/2     Running   0          4d2h
linkerd-sp-validator-8555c5f8d6-rmbtl     2/2     Running   0          4d2h
linkerd-tap-6f57b78499-gqgvq              2/2     Running   0          4d2h
linkerd-web-5dd956f7c9-rpks2              2/2     Running   0          4d2h
```

*NB* You can use the `-n` flag to get resources from any namespace, i.e. `kubectl get po -n linkerd`

### Get all the Deployment resources in a namespace
`kubectl get deploy`

#### Example Output
```
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
linkerd-controller       1/1     1            1           10d
linkerd-destination      1/1     1            1           10d
linkerd-grafana          1/1     1            1           10d
linkerd-identity         1/1     1            1           10d
linkerd-prometheus       1/1     1            1           10d
linkerd-proxy-injector   1/1     1            1           10d
linkerd-sp-validator     1/1     1            1           10d
linkerd-tap              1/1     1            1           10d
linkerd-web              1/1     1            1           10d
```

## Get all the Sevice resources in a namespace
`kubectl get svc`

#### Example Output
```
NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
linkerd-controller-api   ClusterIP   10.106.136.219   <none>        8085/TCP            10d
linkerd-dst              ClusterIP   10.107.146.69    <none>        8086/TCP            10d
linkerd-grafana          ClusterIP   10.107.173.198   <none>        3000/TCP            10d
linkerd-identity         ClusterIP   10.97.226.221    <none>        8080/TCP            10d
linkerd-prometheus       ClusterIP   10.107.221.103   <none>        9090/TCP            10d
linkerd-proxy-injector   ClusterIP   10.109.1.253     <none>        443/TCP             10d
linkerd-sp-validator     ClusterIP   10.111.62.85     <none>        443/TCP             10d
linkerd-tap              ClusterIP   10.97.73.51      <none>        8088/TCP,443/TCP    10d
linkerd-web              ClusterIP   10.111.201.72    <none>        8084/TCP,9994/TCP   10d
```