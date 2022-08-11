# Kubernetes Notes

Refer to a README file in each folder for further notes and example.


## Edit

``` bash
# edit a current running pod
kubectl edit pod pod-name
```

## Get output


``` bash
# use --dry-run=client to preview the object only(doesnt submit) then write info into yaml file
kubectl run redis --image=redis --dry-run=client -o yaml > pod-def.yml

# write a current running pod info into yaml file
kubectl get pod pod-name -o yaml > pod-def.yml
```



## Accessing a pod from another namespace
#### Example URL
```bash
db-service.dev.svc.cluster.local
```
`db-service` : service name(deployment, pod, etc)  
`dev` : namespace  
`svc` : service  
`cluster.local` : domain


