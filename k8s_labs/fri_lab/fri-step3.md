Step 3

# In this task, you will ensure that John or any other member can't consume all the cluster resources. This means, that you will have to constrain the development team to 32Mi of RAM and 20m of CPU per pod. If a pod wants to use more than that, its creation should fail.
- To ensure that everything works as expected, create a dummy deployment using the nginx:alpine image, and ensure that the conditions mentioned above are enforced.


# Example approach

>  Ensure that you become the admin user to be able to create the restrictions in the namespace.
```shell
kubectl config use-context default
```
Switched to context "default".

>  Create the constraints in the development namespace with 20m for CPU and 32Mi for RAM.
```shell
vim ~/teams/development/k8s.yaml
```
...
kind: RoleBinding
metadata:
  namespace: development
  name: deployment-team
...
---  # Add a new object to limit the resources
apiVersion: v1
kind: LimitRange
metadata:
  name: development-limits # Arbitrary name
  namespace: development   # Namespace to constrain
spec:
  limits:
    - type: "Container"
      max:
        cpu: 20m
        memory: 32Mi
 # Save and exit
 
- Apply using kubectl
```shell
kubectl apply -f ~/teams/development/k8s.yaml
```
limitrange/development-limits created


>  Create a deployment to test that the development team is constrained in terms of pod resources. This deployment will ask for more CPU and RAM than the values allowed in the constraint, and it should fail.
```shell
vim ~/teams/development/pods-demo.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-test
  namespace: development # development namespace
spec:
  replicas: 2
  selector:
    matchLabels:
      app: deployment-test
  template:
    metadata:
      labels:
        app: deployment-test
    spec:
      containers:
      - name: nginx
        image: nginx:alpine # Use a tiny image
        resources:
          requests:
            cpu: 100m       # Ask for lots of cpu
            memory: 200Mi   # Ask for lots of RAM
 # Save the file and exit
 
-  Switch to User John
```shell
kubectl config use-context john
```
- Switched to context "john".

- Apply the file with kubectl as John
```shell
kubectl apply -f ~/teams/development/pods-demo.yaml
```
deployment.apps/deployment-test created
 
- Ensure that the pods are not created, because the violate the contraints
```shell
kubectl get -f ~/teams/development/pods-demo.yaml -o yaml
```
status:
  conditions:
  ...
    message: 'Pod "deployment-test-56c87dc4d5-nx58b" is invalid: [spec.containers[0].resources.requests:
      Invalid value: "100m": must be less than or equal to cpu limit, spec.containers[0].resources.requests:
      Invalid value: "200Mi": must be less than or equal to memory limit]'



>  Provide valid values to ensure that members of the development team can create pods within the enforced limits.
```shell
vim~/teams/development/pods-demo.yaml
```
apiVersion: apps/v1
kind: Deployment
...
    spec:
      containers:
      - name: nginx
        image: nginx:alpine # Use a tiny image
        resources:
          requests:
            cpu: 10m       # Ask for cpu within the limits
            memory: 10Mi   # Ask for ram within the limits
 # Save the file and exit
 
- Apply with kubectl
```shell
kubectl apply -f ~/teams/development/pods-demo.yaml
 # Check the pods
kubectl  get pods -n development
 # Works@
NAME                               READY   STATUS    RESTARTS   AGE
deployment-test-6647479965-7jhd5   1/1     Running   0          19s
deployment-test-6647479965-jgd64   1/1     Running   0          10s
```