Step 4

# Limit the Amount of Pods Running in the Development Namespace
- In this task, you will ensure that the development team can't create more than 5 pods at a time in the development namespace. This is to respect resources in the cluster and prevent one team from using all of the available resources.
- To test that it works, apply the rule as the admin user and the login as "John" to create more than 5 pods. If the rule works correctly, John won't be able to create 6 pods at a time.


# Example approach


>  Ensure that you run these commands as the admin user
```shell
kubectl config use-context default
```
Switched to context "default".


>  Modify the k8s.yaml file and add a resource quota to limit the number of pods in the development namespace.
```shell
vim ~/teams/development/k8s.yaml
```
--- # Add a new component at the end of the file
apiVersion: v1
kind: ResourceQuota
metadata:
  name: development-pods-limits
  namespace: development
spec:
  hard:
    pods: "5"
# Save the file and exit
 
- Apply the config
```shell
kubectl apply -f ~/teams/development/k8s.yaml
```
resourcequota/development-pods-limits created


>  Become John and test the new constraint by creating 6 pods with the below command:
```shell
for i in {1..6}; do kubectl -n development run max-pods-$i --image=nginx:alpine ; done
``` 
pod/max-pods-1 created
pod/max-pods-2 created
pod/max-pods-3 created
Error from server (Forbidden): pods "max-pods-4" is forbidden: exceeded quota
Error from server (Forbidden): pods "max-pods-5" is forbidden: exceeded quota
 
>  Check how many pods were actually created
```shell
kubectl  get pods -n development
```
NAME                               READY   STATUS    RESTARTS      AGE
deployment-test-6647479965-7jhd5   1/1     Running   1 (38m ago)   42h
deployment-test-6647479965-jgd64   1/1     Running   1 (38m ago)   42h
max-pods-1                         1/1     Running   0             98s
max-pods-2                         1/1     Running   0             98s
max-pods-3                         1/1     Running   0             98s
- You can see that only 3 pods were created because there were two pods from previous tasks. The total is 5, and any attempt of creating a new pod in the "development" namespace will end up throwing an error since the limit of 5 pods is already met.


# Congratulations! You have just configured multitenancy in a Kubernetes cluster, allowing different teams to securely and separately live in different namespaces with rules that will enforce limits per team.