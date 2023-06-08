## Week 18 Kubernetes project 

> Spin up two deployments. One deployment contains 2 pods running the nginx image. Include a ConfigMap that points to a custom index.html page that contains the line “This is Deployment One”. The other deployment contains 2 pods running the nginx image. Include a ConfigMap that points to a custom index.html page that contains the line “This is Deployment Two”.

# First Deployment
```shell
kubectl create deployment nginx-1 --image nginx