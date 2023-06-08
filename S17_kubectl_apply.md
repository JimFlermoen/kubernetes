## Using kubectl apply


> Create/update resources in a file
```shell
kubectl apply -f myfile.yaml
```
> Create/update a whole directory of yaml with -f
```shell
kubectl apply -f myyaml/
```
> create/ update from a URL (This deploys a SINGLE Pod because)
```shell
kubectl apply -f https://bret.run/pod.yml
```
> Look at it in the browser or curl
```shell
  curl -L https://bret.run/pod
 #  For Windows powershell run: 
      start https://bret.run/pod.yml 
```

## Dry Run with YAML

> dry-run a create (client side only)
```shell
kubectl apply -f app.yml --dry-run
```
> dry-run a create/update on server (resourse needs to be running)
 > (kubectl apply -f app.yml)
```shell
kubectl apply -f app.yml --dry-run='server'
```
> See a diff (changes) visually
```shell
kubectl diff -f app.yml
```
>