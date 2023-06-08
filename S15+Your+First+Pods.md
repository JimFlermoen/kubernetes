# Your First Pods

> Shell commands and reference links, per lecture

## Our First Pod With Kubectl run

- [Kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Kubectl Commands for Docker Users](https://kubernetes.io/docs/reference/kubectl/docker-cli-to-kubectl/)

```shell
kubectl version
kubectl version --short

# Run a Pod named my_nginx with NGINX webserver
kubectl run my-nginx --image nginx  

# List the Pods
kubectl get pods

# See all objects
kubectl get all
```


## Your First Deployment with kubectl create

```shell
# Creates a deployment of the NGINX web server
kubectl create deployment my-nginx --image nginx

# List the Pods
kubectl get pods

# See all objects
kubectl get all

#Deletes the pod
kubectl delete pod my-nginx

# Deletes the Deployment> Can use "deploy" =="deployment", and "deployments"
kubectl delete deployment my-nginx
```

## Scaling ReplicaSets

```shell
# Start a new deployment for one replica/Pod
kubectl create deploy my-apache --image httpd
kubectl get all

# Scale to 2 replicas
kubectl scale deploy/my-apache --replicas 2
kubectl get all
```

##  Inspect Deployments

```shell
# Get container logs
kubectl logs deployment/my-apache

# Follow and list the last line of the logs for this deployment
 kubectl logs deployments/my-apache --follow --tail 1

# Get a bunch of details about an object, including events (Get name of pod first)
kubectl describe pod my-apache-55878d86f4-8rmv6

# Watch command (without needing watch)
kubectl get pods -w
