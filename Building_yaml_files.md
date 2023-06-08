## Building YAML Files

> kind: (List all Resources)
```shell
kubectl api-resources
```
> apiVersions: Versions
```shell
kubectl api-versions
```
> metadata:
  Only the Name is required
> spec: (the meat)
  Where all the action is at

## Building YAML specs  -- THE DETAILS  --

> Get all the keys each kind supports
```shell
kubectl explain services --recursive
```
> Explain only the SPECs  (Subkeys, description, and value supported: ie <boolean>, <map[string]string> etc)
```shell
kubectl explain services.spec
```
>  Walk through SPEC with (drills down spec with type):
```shell
kubectl explain services.spec.type
```
> SPEC: can have SUB-specs: of other Resources, (here is volumes).
```shell
kubectl explain deploy.spec.template.spec.volumes.nfs.server
```
> here is the API documentation page 
https://kubernetes.io/docs/reference/#api-refernce
  > Requires lots of digging
  