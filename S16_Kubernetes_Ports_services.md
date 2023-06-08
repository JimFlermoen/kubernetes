## Creating ClusterIP Service

# Start simple http server using sample code
kubectl create deploy httpenv --image=bretfisher/httpenv

# Scale up 5 replicas
kubectl scale deploy httpenv --replicas=5

# Create a ClusterIP Service (default)
kubectl expose deploy httpenv --port 8888

# Inspecting ClusterIP service
(Internal Ip)

# Lookup what IP was allocated 
kubetl get service

# Run a single pod named "tmp-shell" that is removed(--rm) after exiting with a shell(-it) in the pod with an 
# image that has curl in it. " -- " changes the cmd or what runs in the container, bash
kubectl run tmp-shell --rm -it --image bretfisher/netshoot -- bash

# Shows all environment variables on machine in the pod
curl <deploy name>:<port>

# If on a linux host: (kubectl get service) to get IP Address
curl <IP of Service>:<port>

##################################################################################################################### 

## Create NodePort Service 

# Expose a NodePort so we can access it via the host IP
kubectl expose deploy httpenv --port --name httpenv-np --type NodePort

# To get NodePort port:
kubectl get service

# NodePort rang id default between: 30000-32767
curl localhost:<NodePort>

############################################################################################################

## Create/add Load Balancer Service

# Adds Load Balancer to deployment
kubectl expose deploy httpenv --port 8888 --name httpenv-lb --type LoadBalancer

# Run to get Ports
kubectl get service

# Deletes everyhing
kubectl delete service/httpenv service/httpenv-np service/httpenv-lb deploy/httpenv