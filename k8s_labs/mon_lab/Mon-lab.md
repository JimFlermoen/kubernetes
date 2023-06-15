 # Deploy Applications in Kubernetes using Deployments


1. Step one
> Create custom namespace for application environment called stage-app
```shell
kubectl create namespace stage-app
# Verify
kubectl get ns
```


2. Step Two
> Setting a new default namespace
```shell
kubectl config set-content --current --namespace=stage-app
# Validate it
kubectl config view --minify | grep namespace:
# also check
kubectl config get-contexts
```


3. Step Three
> Create a Deployment with 2 replicas, the namespace and verify pods created
 I used yaml file but can also use:
```shell
kubectl create deploy nginx-deployment --image=nginx --replicas=2
# Verify
kubectl get deployment
# And
kubectl get pods
```


4. Step four
> Scale the Deployment and verify if new Pods are created based on the specifications.
```shell
# scale deployment
kubectl scale deployment/app-deployment --replicas=5
# Auto scale deployment
kubectl autoscale deployment/nginx-deployment --min=2 --max=15 --cpu-percent=80
# Check 
kubectl get deployment
# verify number of pods
kubectl get pods
```


5. Step five
> Test Rolling Update and Roll Back functionality of Deployments
```shell
#1. Set a new image for the deployment. Change it from nginx image to httpd.
kubectl set image deployment/nginx-deployment nginx=httpd
#2. Verify new Pods
kubectl get pods
#3. Set a new image for the deployment. This time add an image name that does not exist so that we can test the failure scenario.
kubectl set image deployment/nginx-deployment nginx=a423sdg
#4. Display the list of Pods. You should see that there are 3 Pods created that have failed (in the ImagePullBackOff state).
kubectl get pods
#5. Rollback the changes to the previous working version.
kubectl rollout undo deployment/nginx-deployment
#6. Display the list of Pods. You should see that all 5 Pods are now actively running.
kubectl get pods
#7. Manually delete a specific Pod from the deployment to verify if deployment detects and auto-creates a new Pod. Make sure to replace the pod-name in the below command.
kubectl delete pod nginx-deployment-dc9dcc5d6-l2qgj
#8. Display the list of Pods. You should see that a new pod (last in the list) has been recently created.
kubectl get pods
