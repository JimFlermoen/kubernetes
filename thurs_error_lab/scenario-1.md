# scenario-1.txt

1. There is a POD named troubleshoot-pod created.
2. This POD is not running due to certain reason.
3. Fix the issue without manually deleting the POD.


# 1. Troubleshoot-Scenario-1

>  Check the contents of the scenario-1.txt file to understand the problem statement as part of Troubleshooting Task 1.
```shell
cat scenario-1.txt
```

>  Find the pod named "troubleshoot-pod" in the Kubernetes cluster.
```shell
kubectl get pods --all-namespaces --selector=run=troubleshoot-pod
```

>  Check the events associated with Pod to understand the cause of failure. After running kubectl describe, scroll down to the Events section to get the Pod events.
```shell
kubectl describe pod troubleshoot-pod -n staging-env
```

-  From the events, we can analyze that Pod is launched based on "ninx" image and Kubernetes is failing to pull the image as the repository does not exist. Here we can identify that there is a typo; instead, the image should have been nginx.

>  Run the kubectl edit to directly edit the Pod resource to fix the typo issue.
```shell
kubectl edit pod troubleshoot-pod -n staging-env
```

>  Under the spec -> containers - image, we can identify the typo "ninx"
-  Fix the typo and update the image name to nginx. Save the file (note: this has opened using Vi editor, hence saving the file requires pressing ESC and wq!)
-   Once the file is saved, you will get a message stating that the pod is edited.

>  Verify if the Pod is running.
```shell
kubectl get pods -n staging-env
```