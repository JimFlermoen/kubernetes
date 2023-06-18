Step 2 

# Create a Namespace for the Team and Grant RBAC Permissions to Users
- In this task, you will create a namespace that will isolate all the resources of the development team.  You need to ensure that John can access ONLY the development namespace. Additionally, you need to ensure that John is able to perform ALL operations on Deployments, ReplicaSets and Pods.
- John MUST NOT be able to perform any operations on any resources other than the ones mentioned above. For example, John shouldn't be able to create configmaps or secrets - you can test this by trying to create a dummy configmap/secret and it should result in an error.


#Example approach

>  Create the development namespace. Ensure that you run these commands as the admin user.
```shell
kubectl config use-context default
Switched to context "default".
 ```

- Ensure you're using the default context
```shell
kubectl  config get-contexts
CURRENT   NAME      CLUSTER   AUTHINFO   NAMESPACE
*         default   default   default    
          john      default   john
``` 


> Create the namespace

```shell
vim ~/teams/development/k8s.yaml
``` 

apiVersion: v1
kind: Namespace
metadata:
  name: development
-  Save the file and exit
 > Apply with kubectl
```shell
kubectl apply -f ~/teams/development/k8s.yaml
namespace/development created
```

>  Create a role that describes the permissions that the user John will have. Remember these permissions should be scoped to the development namespace.
```shell

vim ~/teams/development/k8s.yaml
```
...
kind: Namespace
...
--- # Add these three hyphens to add a new object definition
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: development        # Scoped namespace
  name: development-team        # An arbitrary name
rules:
- apiGroups: ["", "extensions", "apps"]
 
  # Allow ONLY these resources
  resources: ["deployments", "replicasets", "pods"]
 
  # Allow ALL operations on the above resources
  verbs: ["*"]
 # Save the file and exit



>  Bind the role that you created above with the development group that you configured when you created the certificates for John. It's important to bind the role to the group "development" so that you can add more users to this group in the future.
```shell
vim ~/teams/development/k8s.yaml
```

...
kind: Role
metadata:
  namespace: development        # Scoped namespace
  name: development-team        # An arbitrary name
...
--- # Add a new object
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: development      # Allowed namespace
  name: development-team      # An arbitrary name
subjects:
- kind: Group
  name: development          # development group here
  apiGroup: "rbac.authorization.k8s.io"
roleRef:
  kind: Role
  name: development-team      # role name here
  apiGroup: rbac.authorization.k8s.io
 # Save the file and exit
 

> Apply the config
 ```shell
kubectl apply -f ~/teams/development/k8s.yaml
```
...
role.rbac.authorization.k8s.io/development-team created
rolebinding.rbac.authorization.k8s.io/development-team created



>  Become John and test the permissions. Remember that you granted permissions to Deployments, ReplicaSets and Pods ONLY within the development namespace.
```shell
kubectl config use-context john
Switched to context "john".
 
# Ensure that you became John
kubectl  config get-contexts
CURRENT   NAME      CLUSTER   AUTHINFO   NAMESPACE
          default   default   default    
*         john      default   john     
 
# Test permissions on the development namespace
$ kubectl get deploy,rs,pods -n development
No resources found in development namespace. # Works!
 
# Test permissions for something other than deploy,rs,pods for John
$ kubectl get cm,secrets -n development
 
# John shouldn't be able to access configmaps or secrets
Error from server (Forbidden): configmaps is forbidden ...
Error from server (Forbidden): secrets is forbidden ...
 
# Test the permisions on another namespace
$ kubectl get deploy,rs,pods -n default
 
# John shouldn't be able to access resources that belong to other namespaces
Error from server (Forbidden): deployments.apps is forbidden ...
Error from server (Forbidden): replicasets.apps is forbidden ...
Error from server (Forbidden): pods is forbidden ...
```
