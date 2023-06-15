-----Deploy Canary Releases with Istio Lab-----


# 1. Deploy the stable version of the application using Kubernetes

-   In this task, you will deploy version 1 of the application that we provided. You will have to deploy to Kubernetes the file named app-stable-deployment.yaml (find it in the attached asset folder). You will also have to deploy a ClusterIP type service so that other services within Kubernetes can talk to the Deployment that you previously created on port 80.
-   Once done, ensure that you can see two replicas for the deployment that you created. You also need to ensure that the ClusterIP service that you created responds to requests on port 80.
-   Note: Since this deployment will be considered a stable deployment, it's good practice to name this ClusterIP service as xyz-stable or in this case: api-stable. Feel free to name it as you wish.

> Create a main directory named "canary-deployments" with a k8s subdirectory to better organize the project.   
  ```shell
mkdir ~/canary-deployments/k8s -p
  ```

> Create a new file with the content of the app-stable-deployment.yaml asset, and Create the ClusterIP that will allow other services to communicate with these pods.
  ```shell
vim ~/canary-deployments/k8s/app-stable-deployment.yaml
  ```

> Apply the file to Kubernetes using kubectl
  ```shell
kubectl apply -f ~/canary-deployments/k8s/app-stable-deployment.yaml
  ```
> Ensure that the pods and service were created 
  ```shell
kubectl get pods
kubectl get svc
# or run
kubectl get all
  ```

>  Make sure the service works as expected, you can use kubectl port-forward to test this.
- This will port-forward port 80 from the service to port 8080 on your workspace.
```shell
kubectl port-forward service/api 8080:80 --address='0.0.0.0'
  ```
----------------------------------------------------------------------------------------------------------------


# 2. Deploy a canary release with Kubernetes

-   In this task, you will simulate a new release with new changes to our existing application. Please start by creating a new Kubernetes deployment (yaml file) named "app-canary".
-   The new deployment should create pods with a label named lifecycle (just like the stable deployment). Thislifecycle label should be set to canary.
-   You will also need to create a new ClusterIP service, preferably named api-canary, and you have to ensure that this service picks up the new canary pods.
Note: Do not remove the old app-stable deployment.

>  Create the new canary deployment. To do this, create a new file called app-canary-deployment.yaml  at ~/canary-deployments/k8s . The only change that is required - as compared to api-stable - is that the lifecycle label should be set to canary instead of stable and create a ClusterIP service named api-canary and ensure that the selector picks up the new pods.
  ```shell
vim ~/canary-deployments/k8s/app-canary-deployment.yaml
  ```

>  Apply the config
  ```shell
kubectl  apply -f ~/canary-deployments/k8s/app-canary-deployment.yaml
  ```

>  Ensure that both of the deployments are running. The stable deployment should have two replicas and the canary deployment should have three.
  ```shell
# Check Stable
kubectl get pods -l app=api,lifecycle=stable
# Check Canary
kubectl get pods -l app=api,lifecycle=canary
  ```

>  Ensure that you can access the new canary release using the api-canary service, This will port-forward port 80 from the service to port 8080 on your workspace.
  ```shell
kubectl port-forward service/api-canary 8080:80 --address='0.0.0.0'
  ```
----------------------------------------------------------------------------------------------------------------


# 3. Install Istio on Kubernetes

-   In this task, you will install Istio to enable service mesh features in Kubernetes. To begin, please install Istio 1.17.0  using the istioctl utility.
-   You are also provided with a custom config file that was handed over by the security team since they want to customize the installation for security purposes (find it as an asset named istio-security-config.yaml ). Please ensure that you pass this custom config file during the Istio installation.

>  Create a directory to create a logical distinction between Istio and Kubernetes.
 ```shell
mkdir -p ~/canary-deployments/istio
# Ensure that you change to the new directory
cd ~/canary-deployments/istio
 ```

>  Download Istio 1.17.0 as described in the task.
 ```shell
curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.17.0 TARGET_ARCH=x86_64 sh -
# Ensure the download was successful
ls
 ```

>  Move the istioctl binary to /usr/local/bin  so that you can start with the installation
 ```shell
sudo mv ~/canary-deployments/istio/istio-1.17.0/bin/istioctl /usr/local/bin
# Ensure that you can interact with the istioctl tool
istioctl version
 ```

>  Before installing Istio, you need to ensure that the custom config that was provided by the security team exists as a file, so that you can pass it to Istio during the installation. To do that, create a file in the  ~/canary-deployments/istio/ folder with the contents of the istio-security-config.yaml
 ```shell
vim ~/canary-deployments/istio/istio-security-config.yaml
 ```

>  Start the Istio installation and provide the custom config file that was provided by the security team.
 ```shell
istioctl install -f ~/canary-deployments/istio/istio-security-config.yaml -y
 ```

>   To verify that the installation was successful, check the pods and services of the istio-system namespace. All pods should be running and a LoadBalancer type service - named istio-ingressgateway - should exist with its respective external IP.
 ```shell
kubectl  get pods,svc -n istio-system
 ```
 -------------------------------------------------------------------------------------------------------------------------


 # 4. Configure incoming traffic to the Istio mesh

 -  In this task, you will create an Istio Gateway. Keep in mind that there's already one IngressGateway up and running as part of the Istio installation, so you should only create an Object of kind Gateway and it should point to the Istio IngressGateway.
 -  The Gateway should accept traffic on port 8080 (since that's the port provided by the security team) and the protocol that will be managed by the gateway must be set to HTTP. 

>  Locate the IngressGateway deployed in your cluster and find the labels that will be used in the Gateway config.
  ```shell
kubectl  get  pods -n istio-system --show-labels
  ```

>  Create a new file at ~/canary-deployments/istio/gateway.yaml  with the following content:
  ```shell
vim ~/canary-deployments/istio/gateway.yaml
  ```

>  Apply the configuration using kubectl
  ```shell
kubectl  apply -f ~/canary-deployments/istio/gateway.yaml
  ```
--------------------------------------------------------------------------------------------------------------------------


# 5 Configure traffic routing with VirtualServices and DestinationRules

-   In this task, you will configure routing rules for the canary release. The main goal is to let the canary release manage 20% of the real traffic while the stable release will handle the 80% left. By doing this, we reduce the impact on affected users and we can still see how the new release performs with real traffic.
-   Go ahead and create a VirtualService to configure the traffic routing. To facilitate your testing, you can initially send 50% of the traffic to the api-stable deployment and the other 50% to the api-canary service. Once everything works and you have validated that the traffic is being evenly distributed, reconfigure the VirtualService to 80% for the stable release and 20% for the canary release.

>  Create a VirtualService to define the percentage of traffic that will be routed to your pods. It's up to you on where to create the file, it could either be created in the k8s or istio folder.
  ```shell
vim ~/canary-deployments/istio/virtualservice.yaml
  ```

>  Apply the configuration with kubectl
  ```shell 
kubectl apply -f ~/canary-deployments/istio/virtualservice.yaml
  # Ensure everything looks good
kubectl  describe  vs api
  ```

>   Click on your web server button and refresh the browser a couple of times. You should see that, on average, 50% of the requests are being served by api-stable and the other 50 are being served by api-canary .
>  Make it more realistic, and set the canary release to 20% of the traffic, and the stable release to 80%. In real-world scenarios, you would want your canary release to affect the least number of real users, so 20% in this case is a pretty high value, but it will do just fine to test this PoC. In a production environment, you could try setting canary releases somewhere from 1% to 10%, and stable releases somewhere between 90% to 99%. It all depends on how much risk the organization is willing to take.
  ```shell
# Edit the ~/canary-deployments/istio/virtualservice.yaml file
vim ~/canary-deployments/istio/virtualservice.yaml
# Apply with kubectl
kubectl apply -f ~/canary-deployments/istio/virtualservice.yaml
```