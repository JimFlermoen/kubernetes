# Manage Kubernetes Traffic using Nginx Ingress Controller


# 1. Create Two Instances (Kubernetes Pods) For Loan Applications

-   Create loan applications with two Kubernetes pod replicas. The application docker image is located in the following docker hub : timpamungkas/devsecops-loan:1.0.0
-   Application base path is at /loan

-Create and expose a Kubernetes load balancer service for the pod. It will need the following requirements for Kubernetes:
  - application port : 8111
  - healthiness endpoint : /actuator/health
  - readiness endpoint : /actuator/readiness

>  Create a yml file for Kubernetes declarative configuration using the following command
 ```shell
vim devsecops-loan.yml
 ```

>  Create k8s objects
 ```shell
kubectl apply -f devsecops-loan.yml
 ```

>  Verify pods are ready. Wait 3 min.
 ```shell
kubectl get pod -n devsecops
 ```

>  Hit the endpoint /loan/api/hello several times. Use this terminal command (several times):
 ```shell
curl --location 'http://localhost:8111/loan/api/hello'
 ```
----------------------------------------------------------------------------------------------------------------


# 2. Install Nginx Ingress Controller

- Install the latest Nginx ingress controller on the Kubernetes cluster.

> Configure local hosts name and Nginx so the url loan.mycompany.local will redirect traffic to the running loan pod instances.

```shell
#The current cluster already has a traefik service that occupy the port 80. Since Nginx ingress controller also requires port 80, you must delete the traefik pod.
kubectl delete svc -n kube-system traefik

# Replace the <version> with the latest controller release. 
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.0/deploy/static/provider/cloud/deploy.yaml 

# Wait for around 3 minutes and make sure the Nginx ingress controller pod is running. Use this terminal command and make sure the controller pod is READY
kubectl get pods --namespace=ingress-nginx

# Since you will expose the loan application through Nginx ingress controller, it is better to use Kubernetes service with type ClusterIP instead of LoadBalancer that was created on Task 1, as ClusterIP will make the loan application available only at the other pod on the Kubernetes cluster and will not expose it on public IP address. Create cluster Ip Yaml file

vim service-loan-clusterip.yml

# Apply the ClusterIP
kubectl apply -f service-loan-clusterip.yml

# Verify you have two services to expose loan application.
kubectl get services -n devsecops

# You won't need the devsecops-loan-loadbalancer service, so delete it
kubectl delete service -n devsecops devsecops-loan-loadbalancer

# Verify that you now have only one service to expose the loan application, which is ClusterIP.
kubectl get services -n devsecops
```
----------------------------------------------------------------------------------------------------------------


# 3. Create an Ingress Rule for the Loan Application
> Create an ingress rule so that the  traffic to http://<nginx-address>/loan/... (has prefix /loan) will access one of the existing loan application pods.

- The Nginx ingress controller is only the tool. You need to define your ingress rule so the tool (Nginx) knows how to route traffic. You must route all incoming traffic with the pattern http://<nginx-address>/loan/...  (has prefix /loan) to the Kubernetes service with the name service-loan-clusterip (defined on the Task 2).  

```shell
# Create the ingress rule by creating the file ingress-nginx.yml 
vim ingress-nginx.yml

# Apply the ingress rule using this command
kubectl apply -f ingress-nginx.yml

# You can now access the loan application through Nginx. Try to hit this endpoint several times
curl http://localhost/loan/api/hello
```
----------------------------------------------------------------------------------------------------------------


# 4. Reduce the Loan Replica to One and Create a Marketing Pod with One Replica

> Reduce the loan application replica to one. Create marketing applications with one Kubernetes pod replica. The marketing application docker image is located in the following docker hub: 
 timpamungkas/devsecops-marketing:1.0.0

- The marketing application runs on port: 8222
- Application base path is at: /marketing
- Make sure both loan and marketing have ClusterIP service

```shell
# Edit devsecops-loan.yml by changing replicas to 1
vim devsecops-loan.yml

# Update the yml file
kubectl apply -f devsecops-loan.yml

# Create the devsecops-marketing.yml file for Kubernetes declarative configuration
vim devsecops-marketing.yml

#Create Kubernetes objects by applying the yml file
kubectl apply -f devsecops-marketing.yml

# To make sure all pods are re-created, restart the deployment using the following command
kubectl rollout restart deployment -n devsecops

# Wait for around 3 minutes until the pod is ready. Verify that there is one loan pod and one marketing pod. They must be READY before continuing. Use this terminal command to check:
kubectl get pod -n devsecops

# Make sure there are two ClusterIP services, one for loan and one for marketing. Use the following command
kubectl get services -n devsecops
```
----------------------------------------------------------------------------------------------------------------


# 5  Create an Ingress Rule for Marketing

> Which will be accessible on path http://<nginx-address>/marketing/... (has prefix /marketing)
  - Make sure both loan and marketing applications are accessible via Nginx ingress controller
  
  > Objects can have many rules.
```shell
# To add a rule for marketing application, edit the existing ingress-nginx.yml file
vim ingress-nginx.yml

# Update the ingress rule by re-applying it
kubectl apply -f ingress-nginx.yml

# You can now access the marketing application through Nginx. Try to hit the marketing endpoint using the command below
curl http://localhost/marketing/api/hello

# Make sure you can still access the loan application through Nginx. Try to hit the loan endpoint using the following command
curl http://localhost/loan/api/hello
```
----------------------------------------------------------------------------------------------------------------


# 6 Add Request & Response Headers for All Traffics

#  Custom HTTP Headers
-   All API requests received by applications need to receive the following HTTP request headers:
      -nginx-request-time: when the request was received (on ISO-8601 format)
      -nginx-source: mycompany-nginx
      -nginx-more-request-header: just-sample-dummy-header

-   To test the request headers, both loan and marketing application have: /api/echo endpoint.
-   For all API responses, you need to add the following:
      -nginx-response-time: when the response was sent (on ISO-8601 format)
      -nginx-version: current nginx version

#  Nginx Timeout
-   If the backend (loan/marketing) does not respond in 5 seconds, close the connection and give the appropriate HTTP    status.

>   Create a yml file for Kubernetes declarative configuration using the following command
 ```shell
vim ingress-nginx-configmap-headers.yml
  # Create the ConfigMap configuration using the following command
kubectl apply -f ingress-nginx-configmap-headers.yml
  ```

>   You can configure Nginx ingress controller globally, which means the configuration will be effective for all traffic. To do this, you need to create a Kubernetes ConfigMap for the ingress controller. Create a new file ingress-nginx-configmap-config.yml using the following command
 ```shell
vim ingress-nginx-configmap-config.yml
# Apply the configuration using the command below
kubectl apply -f ingress-nginx-configmap-config.yml
 ```

>  Restart the Nginx ingress controller
 ```shell
# Get the Nginx deployment name using the command
kubectl get deployment -n ingress-nginx
# Restart the Nginx using command
kubectl rollout restart deployment -n ingress-nginx ingress-nginx-controller 
 ```
--------------------------------------------------------------------------------------------------------------------------


# 7 Test your Global Nginx Configuration

-  Test the global Nginx configurations:
    - Use /loan/api/echo and /marketing/api/echo to check whether or not the HTTP request headers were added and passed to backend server
    - Use /loan/api/hello and /marketing/api/hello to check whether or not the request headers were added and passed to client
    - Use /loan/api/delay/<delay-in-seconds> and /marketing/api/delay/<delay-in-seconds> to check whether or not the long requests (more than 5 seconds) will be timed out, but short requests (less than 5 seconds) will pass through

>  Use the following curl command to check whether the HTTP request headers were added by Nginx and received by the loan application
 ```shell
curl http://localhost/loan/api/echo
 ```

>  Use the following curl command to check whether the HTTP request headers were added by Nginx and received by the marketing application
 ```shell
curl http://localhost/marketing/api/echo
 ```

>  Use the following curl command to check whether the HTTP response headers were added by Nginx and received by the client who called the loan API
 ```shell
curl -I http://localhost/loan/api/hello
 ```

>  Use the following curl command to check whether the HTTP response headers were added by Nginx and received by the client who called the marketing API
 ```shell
curl -I http://localhost/marketing/api/hello
 ```

>  Use the following curl command to check whether a slow HTTP call will be timed out in the loan API. Use a 7 seconds delay
 ```shell
curl http://localhost/loan/api/delay/7
 ```

>  Use the following curl command to check whether a slow HTTP call will be timed out in the marketing API. Use a 6 seconds delay
 ```shell
curl http://localhost/marketing/api/delay/6
 ```

>  Use the following curl command to check whether a fast HTTP call will still be processed and not timed out in the loan API. Use a 3 seconds delay
 ```shell
curl http://localhost/loan/api/delay/3
 ```

>  Use the following curl command to check whether a fast HTTP call will still be processed and not timed out in the marketing API. Use a 4 seconds delay
 ```shell
curl http://localhost/marketing/api/delay/4
 ```
 -------------------------------------------------------------------------------------------------------------------------


 # 8 Configure the Loan and Marketing Ingresses for Specific Requirements

- Use Nginx annotations to fulfill these requirements:
    - On the loan application:
    - timeout is 3 seconds
    - client can only access the loan API for maximum of 4 requests per minute

- On the marketing application:
    - Endpoint /marketing/api/delay/<duration> must be secured with a username/password combination: marketing-user/    marketing-password
    - All other endpoints on the marketing are freely accessible (no security)

>  Since you need to create different rules for loan and marketing, you must use two separated ingress rules. Delete the existing ingress rules. You can find the ingress rule name by using the following command: 
 ```shell
kubectl get ingress -n devsecops
 # Then delete the ingress rule by using command
kubectl delete ingress -n devsecops devsecops-ingress-nginx
 ```

>  To generate a basic username/password, you need to install apache2-utils using this command:
 ```shell
sudo apt-get update
sudo apt-get install -y apache2-utils
 ```

 >  Generate the username marketing-auth and use password marketing-password. Use htpasswd command like below:
  ```shell
htpasswd -c auth marketing-user
 ```

>  From the generated auth file, create a Kubernetes secret with the name marketing-auth in the namespace devsecops. Use the command below
 ```shell
kubectl create secret generic marketing-auth -n devsecops --from-file=auth
 ```

> Create a new file for the loan's ingress rule using the below command
 ```shell
vim ingress-nginx-loan.yml
 ```

>  Create a new file for marketing's ingress rule (not secured) using the command below
 ```shell
vim ingress-nginx-marketing.yml
 ```

>  Create a new file for the marketing's ingress rule (secured with a username/password) using the following command
 ```shell
vim ingress-nginx-marketing-auth.yml
 ```

>  Apply all three ingress rules
 ```shell
kubectl apply -f ingress-nginx-loan.yml
kubectl apply -f ingress-nginx-marketing.yml
kubectl apply -f ingress-nginx-marketing-auth.yml
```
 -------------------------------------------------------------------------------------------------------------------------


# 9. Test the Loan and Marketing APIs After the Specific Configurations

- Test the configurations in Ingress annotations:

- On the loan application:
    - Any API longer than 3 seconds must timeout. Use /loan/api/delay/<duration-second> to test
    - Clients can only access the loan API maximum of 4 requests per minute. Use any loan API to test
- On the marketing application:
     - Endpoint /marketing/api/delay/<duration> must be accessible only when the client sends the following username/password combination: marketing-user / marketing-password
    - All other endpoints on marketing are freely accessible (no security)
    - APIs longer than 3 seconds but less than 5 seconds must NOT timeout

> Loan API should timeout when it receives a slow API (more than 3 seconds). Test using this command to give a 4 seconds delay
 ```shell
curl http://localhost/loan/api/delay/4
 ```

> Loan API must not timeout when it receives a fast API (less than 3 seconds). Test using this command to give a 2 seconds delay
 ```shell
curl http://localhost/loan/api/delay/2
 ```

> Hit /loan/api/hello at least 5 times within one minute. At the 5th (sometimes 6th) attempt, you will be rejected because the rate limit is only 4 requests per minute. Repeat this command at least for 5 times
 ```shell
curl http://localhost/loan/api/hello
 ```

> After 1 min hit /loan/api/hello
 ```shell
curl http://localhost/loan/api/hello
 ```

> Marketing API should not timeout when it receives an API that is between 3-5 seconds. Use this command to give a 4 seconds delay
 ```shell
curl http://localhost/marketing/api/delay/4
 ```

> Access /marketing/api/delay with the correct username/password combination, using this command
 ```shell
curl -u marketing-user:marketing-password http://localhost/marketing/api/delay/4
 ```

> You should be able to hit marketing API many times within one minute, since there is no rate limit set up. Repeat this command as many as you like within one minute:
 ```shell
curl http://localhost/marketing/api/hello 
 ```
 