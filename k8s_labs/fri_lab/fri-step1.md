Step 1

# Create Users in Kubernetes

> In this task, you will create a user for the development team called John. The goal of this task is to evaluate how a user is created in Kubernetes with a specific group and how to log in as the new user that you have created.
- Remember that Kubernetes doesn't provide an API to create users, so you must rely on an external authentication method, such as self-signed certificates.
- Note: You only need to create a user for this task and you should test its login. It's completely acceptable to receive errors such as: User "xyz" cannot list resource "xyz"  - This error indicates that the user is a valid user, but it still doesn't have any permissions attached.
- On the other hand, receiving errors similar to error: You must be logged in to the server (Unauthorized)  is unacceptable and implies that the new user is not able to login to the cluster.


# Example approach
- Note: This task will use self-signed certificates as a way of authenticating users to Kubernetes.

>  Create a directory for the team and the user. This is not a requirement, but it simplifies and logically organizes teams and users.
```shell
mkdir -p ~/teams/development/john
```


>  Create a private key for the user John using OpenSSL.
-   Changes directory
```shell

cd ~/teams/development/john
 ```
-   Creates the private key
```shell
openssl genrsa -out john.key 2048
# Output Below
Generating RSA private key, 2048 bit long modulus (2 primes)
---
ls -l
john.key # You should see the key here
```


>  Create a certificate signing request. You do this to let Kubernetes know that you'd like to trust this new john.key  certificate, so that it can log in and access the cluster as a valid user.
```shell

cd ~/teams/development/john
 # Create file that holds the certificate signing request
## Note how the group is passed in the form of "/O=GROUP_NAME"
openssl req -new -key john.key -out john.csr -subj "/CN=john/O=development"
 # Check that the csr file was created
ls
john.csr  john.key # See john.csr
```


>  Create a CertificateSigningRequest object in Kubernetes to request the approval of the new certificate for John.
```shell

cat <<EOF | kubectl create -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john-development # Any name can be used here. 
spec:
  signerName: kubernetes.io/kube-apiserver-client
  groups:
  - system:authenticated
  ### Be sure to pass the correct file name.
  request: $(cat john.csr | base64 | tr -d '\n')
  usages:
  - digital signature
  - key encipherment
  - client auth
EOF
# Check the CertificateSigningRequest objects. You will see more fields
# in the output, but below are the important ones
$ kubectl get CertificateSigningRequest
# Output Below
NAME               REQUESTOR        CONDITION
john-development  system:admin     Pending # This should be "Pending"
```


>  Now that you as a Kubernetes admin have sent a request to approve the "creation of the user John" - or more specifically, the approval of his certificate - it's time to approve the request. 
```shell

# Approve the request
kubectl certificate approve john-development
 # Ensure that the certificate was approved and issued
kubectl get CertificateSigningRequest
NAME               REQUESTOR        CONDITION
john-development  system:admin     Approved,Issued # This should be "Approved, Issued"
 # Retrieve the generated certificate for John
cd ~/teams/development/john
 kubectl get csr john-development -o jsonpath='{.status.certificate}'  | base64 -d > john.crt
 # The content of the certificate should look like below
cat john.crt 
# Output Below
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
```


>  Congrats! You have successfully granted John access to the Kubernetes cluster. It's time to configure the access for John using the certificates that you approved above.
```shell

cd ~/teams/development/john
 # Configure the certificates for the user john
kubectl config set-credentials john \
  --client-certificate=$PWD/john.crt \
  --client-key=$PWD/john.key
 # Add John as a new "user" - to be more specific, it'll be added as a kubernetes context to your kubeconfig file.
kubectl config set-context john \
  --cluster=default \
  --user=john
 # List the available "users" (contexts) in your system and you should now see john's one poping up.
kubectl  config get-contexts
# Output Below
CURRENT   NAME      CLUSTER   AUTHINFO   NAMESPACE
*         default   default   default    
          john      default   john    
```


>  Switch to the "john" context and try to access any resource in Kubernetes.
```shell

kubectl config use-context john 
# Output Below
Switched to context "john".
 # Check your current context
kubectl  config get-contexts
# Output Below
CURRENT   NAME      CLUSTER   AUTHINFO   NAMESPACE
          default   default   default    
 *        john      default   john    
 
kubectl  get pods 
# Output Below
Error from server (Forbidden): pods is forbidden: User "john" cannot list resource "pods" in API group "" in the namespace "default"
It's important to notice that the error clearly states: User "john" cannot list resource "XYZ", which means that John is a valid user within Kubernetes but needs some additional configuration in terms of permissions.  But, the main goal of this task was to create a valid user, and it's now done!


# If you receive an error with a different message, please check that you followed all the steps as defined above.
# Note: Since John doesn't have any permissions, yet, switch back to the admin context.

kubectl config use-context default
# Output Below
kubectl  config get-contexts
CURRENT   NAME      CLUSTER   AUTHINFO   NAMESPACE
*         default   default   default    
          john      default   john
```            