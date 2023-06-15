#  scenario-2.txt 

1. There is a POD created named troubleshoot-scenario-2-pod
2. This POD fails to start due to some reason.
3. This POD is created with custom config related to hostNetwork, DNS Policy as well as terminationGracePeriodSeconds.
4. Fix the Issue and Ensure POD with same set of specification runs. Rest configurations you can edit.
                                                                         

#  Troubleshoot-Scenario-2 = scenario-2-solution.yaml

>   Check the contents of the scenario-2.txt file to understand the problem statement as part of Troubleshooting Task 2.
```shell
cat scenario-2.txt
```

>  Find the Pod in the cluster and the namespace in which it is running.
```shell
kubectl get pods --all-namespaces
```

- From the above step, we can identify that a Pod named "troubleshoot-scenario-2-pod" is running in the default namespace.

>  Verify if the Pod is running in the default namespace.
```shell
kubectl get pods
```

-   From this step, we can also identify that the Pod is in Pending Status.


>  Check the events associated with Pod to understand the cause of failure. After running kubectl describe, scroll down to the Events section to get the Pod events.
```shell
kubectl describe pod troubleshoot-scenario-2-pod
```

-   From the events, we can analyze that the error is caused due to "Insufficient cpu" and no nodes are matching the CPU requests and limits that are set at the Pod level.

>  Get the Pod manifest and store it in a file so that we can edit the resource to resolve it.
```shell
kubectl get pod troubleshoot-scenario-2-pod -o yaml > scenario-2-solution.yaml
```

>  Open the file where the Pod manifest is stored.
```shell
nano scenario-2-solution.yaml
```

-   Identify the requests and limits-related details in the Pod manifest. We see that memory of 2Gi and 4Gi are set as part of requests and limits and also certain CPU requirements.

-   Since the scenario clearly states that we are free to modify the manifest as required to resolve the issue, let us remove the specific resources section from the Pod manifest.
-   Save the file in nano using CTRL+O and exit using CTRL+X

>  Delete the Pod so that we can recreate the Pod using the latest manifest.
```shell
kubectl delete pod troubleshoot-scenario-2-pod
```

>  Create a new Pod from the manifest that we have created.
```shell
kubectl apply -f scenario-2-solution.yaml
```

>  Verify if the Pod is running.
```shell
kubectl get pods
```
