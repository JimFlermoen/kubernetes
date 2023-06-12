## Dry Run with YAML

> dry-run a create 
```shell
kubectl apply -f <filename> --dry-run=[client],[server],or[none]
```

> dry-run a create/update on server (resourse needs to be running for update)
 > (kubectl apply -f app.yml)
```shell
kubectl apply -f <filename> --dry-run='server'
```

# See a diff (changes) visually
 > to see changes before actual update
```shell
kubectl diff -f <filename>
```
