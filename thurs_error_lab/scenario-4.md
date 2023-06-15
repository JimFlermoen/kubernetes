# Scenario-4.md

-   Example approach


>  Check the contents of the scenario-4.txt file to understand the problem statement as part of Troubleshooting Task 1.
```shell
cat scenario-4.txt
```

>  Check the directory contents and ensure that the deployment.yaml file is present.
```shell
ls -l
```

>  Try creating resources from deployment.yaml file to see if it generates.
```shell
kubectl apply -f deployment.yaml
```

-   At this stage, we get an error stating "prod-deployment" namespace not found.

>  Create a new namespace called prod-deployment to fix the above error.
```shell
kubectl create namespace prod-deployment
```

>  Try creating resources from deployment.yaml file to see if it gets created.
```shell
kubectl apply -f deployment.yaml
```
-   This time we got an error related to the selector not matching the labels.

>  Open the deployment.yaml file in a text editor.
```shell
nano deployment.yaml
```

>  Check the contents of labels and selectors. Verify whether they match or not.
```shell
Labels -->    env : production
Selector --> app: production
metadata.labels --> app: production
```

-   Ensure that all labels and selectors have the same set of values.

-   Once completed, save and close the file.


>  Verify if you are able to create a deployment from the manifest file.
```shell
kubectl apply -f deployment.yaml
```

>  Ensure that the deployment is successfully running.
```shell
kubectl get deployments -n prod-deployment
```



