# scenario-3

- Example approach


>  Check the contents of the scenario-3.txt file to understand the problem statement as part of Troubleshooting Task 1.
```shell
cat scenario-3.txt
```

>  Find the service using the kubectl get service command.
```shell
kubectl get service
```

>  Describe the service to understand the issue.
```shell
kubectl describe service kplabs-service
```

-   From the above screenshot, we can understand that there is a selector of env=stage and there are no endpoints that are added.


>  Find the deployment named nginx-deployment and check which namespace it is running.
```shell
kubectl get deployments --all-namespaces
```

-   From the above screenshot, we can identify that nginx-deployment is running in stage-ns namespace.


>  Verify whether the deployment has the same set of labels that are part of the selector in kplabs-service. Ensure that label of env=stage is present.
```shell
kubectl describe deployment nginx-deployment -n stage-ns
```

>  Download the service resource manifest to a file. We will modify the namespace of service to ensure it is running in the same namespace as that of the deployment.
```shell
kubectl get service kplabs-service -o yaml > scenario-3-solution.yaml
```

>  Verify if the manifest is available in the file.
```shell
cat scenario-3-solution.yaml
```

>  Open the yaml file where the manifest is stored.
```shell
nano scenario-3-solution.yaml
```

-   Change the namespace from default to stage-ns. Once edited, save the file.


>  Delete the existing service from the default namespace.
```shell
kubectl delete service kplabs-service
```

>  Create a new service resource from the yaml file.
```shell
kubectl apply -f scenario-3-solution.yaml
```

>  Verify if the service is created successfully in the stage-ns namespace.
```shell
kubectl get service -n stage-ns
```





>  Check whether the endpoints are populated for service. There should be a total of 3 IPs in the endpoints.
```shell
kubectl describe service -n stage-ns
```

-   Also, take note of the Service IP. In my case, it is 10.43.207.30.


>  Send a curl request to service-ip to ensure if you are getting the application response.
```shell
curl <service-ip>
```
