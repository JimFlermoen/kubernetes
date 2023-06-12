# Your First Pods

> Shell commands and reference links, per lecture

## Our First Pod With Kubectl run

- [Kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Kubectl Commands for Docker Users](https://kubernetes.io/docs/reference/kubectl/docker-cli-to-kubectl/)

```shell
kubectl version
kubectl version --short
```
> Run a Pod named my_nginx with NGINX webserver
```shell
kubectl run my-nginx --image nginx  
```
> List the Pods
```shell
kubectl get pods
```
> See all objects
```shell
kubectl get all
```


## Your First Deployment with kubectl create

> Creates a deployment of the NGINX web server
```shell
kubectl create deployment my-nginx --image nginx
```
> List the Pods
```shell
kubectl get pods
```
> See all objects
```shell
kubectl get all
````
> Deletes the pod
```shell
kubectl delete pod my-nginx
```
> Deletes the Deployment> Can use "deploy" =="deployment", and "deployments"
```shell
kubectl delete deployment my-nginx
```

## Scaling ReplicaSets

> Start a new deployment for one replica/Pod
```shell
kubectl create deploy my-apache --image httpd

kubectl get all
```
> Scale to 2 replicas
```shell
kubectl scale deploy/my-apache --replicas 2

kubectl get all
```

##  Inspect Deployments

> Get container logs
```shell
kubectl logs deployment/my-apache
```
> Follow and list the last line of the logs for this deployment
```shell
 kubectl logs deployments/my-apache --follow --tail 1
```
> Get a bunch of details about an object, including events (Get name of pod first)
```shell
kubectl describe pod my-apache-55878d86f4-8rmv6
```
> Watch command (without needing watch)
```shell
kubectl get pods -w
```