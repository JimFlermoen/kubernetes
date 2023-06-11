## Building YAML Files
  
  > PARTS IN THIS ORDER <
    # 1. apiVersions:
    # 2. kind:
    # 3. metadata:
    # 4. spec:

 
 1. FIRST STEP
# List all Resources/Types by printing out columns of:
 NAME| SHORTNAMES | APIGROUPS | NAMEDSPACE | KIND 
> get your Kind and Api Version from this list
```shell

kubectl api-resources


  ```
2. SECOND STEP
# Verify Api Version needed with this command
apiVersions: <version> 
```shell

kubectl api-versions


  ```
3. THIRD STEP
# Only the name is required in metadata
Labels go under metadata
> metadata:
>  name:
>    labels:
  

4. FOURTH STEP
# Building the specs (where all the action is at)
>>>  Follow down from the Kind: <service> <deployment> <configmap> etc  <<<
    > spec:
    >  [key]:
    >    [subkey]:


1. Get all the keys that each KIND (category) supports
  # Gives you names and spelling of keys for "service" resource
    > works for all resources IE. deployment, configmap, service, etc.
```shell

kubectl explain service --recursive     


  ```
2. Drill down more into spec
  # gives the subkeys of the spec and a small desription of them and the type of value they support.
    > works for <resource>.spec IE deployment.spec/ config.spec etc
```shell

kubectl explain service.spec    #(I only care about the specs)


  ```
2. Drill farther down into the key spec subkey 
  # Gives you the "value" of the key "type" you are looking for
    > works for all subkeys under that resource spec   
```shell

kubectl explain services.spec.type 


```
3. Keeps going into subkeys   <this example is on deployment resources going down into specs>
  # Spec: can have subspecs: of other resources
    > keeps going down by typing [spec.key.spec.subkey.spec.subkey] etc.
```shell

kubectl explain deployment.spec.template.spec.volumes.nfs.server


  ```
4. here is the API documentation page 
  # Requires lots of digging

  https://kubernetes.io/docs/reference/#api-refernce
    

